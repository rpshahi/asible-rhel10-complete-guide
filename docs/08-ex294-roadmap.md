# 8. RHCE (EX294) learning roadmap: beyond Day 1

[← Home](../README.md) · [← Playbooks](07-playbooks.md) · [Next: Practice and troubleshooting →](09-exercises-and-troubleshooting.md)

> [!IMPORTANT]
> **This chapter is additional instructional content, not content from the 10-page Day 1 PDF.** It maps the main skills advertised on the [current official EX294 objectives page](https://www.redhat.com/en/services/training/ex294-red-hat-certified-engineer-rhce-exam-red-hat-enterprise-linux). Exact product versions, available documentation, commands and exam objectives can depend on the **exam variant you booked**. Treat the official version-specific exam page as authoritative.

## 8.1 Understand the objective areas

| EX294 skill area | What to practice | Suggested checkpoints |
| --- | --- | --- |
| RHCSA administration | Packages, users, groups, services, network, storage, SELinux, filesystems, firewalls | Can perform tasks manually before automating |
| Ansible components | Inventories, config, modules, playbooks, variables, facts, loops, conditions, roles, failure handling | Can explain and locate module docs |
| Ansible configuration | `ansible.cfg`, static inventory, host groups, `ansible-navigator.yml` | `ansible-config dump --only-changed`, `ansible-inventory --graph` |
| Node preparation | SSH keys, sudo, required Python and services | Ansible `ping` and `-b -K` both work |
| Playbooks and ad hoc commands | FQCN modules, idempotent tasks, remote host patterns | Same playbook runs twice safely |
| Roles and collections | Install, inspect, create, reuse | Refactor a playbook into a role |
| Content management | Jinja2 templates, file operations, Vault | Produce host-specific files without exposed secrets |
| Version control and tools | Git, VS Code, Navigator, development containers/EE | Edit, run, validate, commit and push project |
| Failure handling | `block`/`rescue`/`always`, controlled failed/changed tests | Recover from expected errors |

Red Hat's current EX294 page explicitly mentions **Git**, **VS Code**, **Ansible Navigator** and development containers, in addition to traditional playbook authoring.

## 8.2 Variables, precedence and facts

Ansible variables can be defined in inventories, `group_vars/`, `host_vars/`, playbooks, roles and at invocation time. Their precedence varies by source; learn the [official variable precedence rules](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_variables.html) rather than guessing.

Example `group_vars/webservers.yml`:

```yaml
---
web_service: httpd
web_port: 80
```

Using those variables:

```yaml
---
- name: Start the chosen web service
  hosts: webservers
  become: true
  tasks:
    - name: Enable and start the service
      ansible.builtin.service:
        name: "{{ web_service }}"
        state: started
        enabled: true
```

The service must already be installed. Place the group-vars file under `group_vars/` in an ordinary Ansible project structure when practicing.

Facts supply managed-node attributes; for example:

```yaml
- name: Show OS family
  ansible.builtin.debug:
    msg: "OS family is {{ ansible_facts['os_family'] }}"
```

Facts require prior fact gathering (`gather_facts: true` or `ansible.builtin.setup`). Avoid hard-coded facts that change with OS version.

## 8.3 Conditionals, loops, and reusable state

```yaml
---
- name: Configure tools on Red Hat-family machines
  hosts: managed
  become: true
  tasks:
    - name: Install one utility at a time
      ansible.builtin.dnf:
        name: "{{ item }}"
        state: present
      loop:
        - tar
        - vim-enhanced
      when: ansible_facts['os_family'] == 'RedHat'
```

In production, passing a list of packages directly to `dnf` is often more efficient than looping. The loop is shown because it is an explicit exam skill. `when:` is a Boolean expression, not quoted Jinja2 output.

## 8.4 Error handling and command results

```yaml
---
- name: Inspect a target safely
  hosts: managed
  tasks:
    - name: Inspect an optional configuration file
      block:
        - name: Read its existence and metadata
          ansible.builtin.stat:
            path: /etc/training-app.conf
          register: config_file

        - name: Report if the file is available
          ansible.builtin.debug:
            msg: "Config exists: {{ config_file.stat.exists }}"
      rescue:
        - name: Report an unexpected inspection error
          ansible.builtin.debug:
            msg: Inspection failed
      always:
        - name: Finish the inspection
          ansible.builtin.debug:
            msg: Inspection completed
```

`rescue` handles task **failures**, not false conditions; missing paths commonly produce a successful `stat` with `exists: false`, so that example ordinarily executes no rescue step. Learn `failed_when`, `changed_when`, `ignore_errors` and handlers, but do not hide genuine errors indiscriminately.

For command output:

```yaml
- name: Read the system uptime
  ansible.builtin.command: uptime
  register: uptime_result
  changed_when: false

- name: Print uptime
  ansible.builtin.debug:
    var: uptime_result.stdout
```

These are snippets for a play's `tasks` section, not separate complete playbooks.

## 8.5 Roles and Ansible Content Collections

From a project directory:

```bash
mkdir -p roles
ansible-galaxy role init roles/webserver
ansible-galaxy role list
ansible-galaxy collection list
ansible-doc ansible.builtin.template
```

A role normally has paths such as `tasks/main.yml`, `handlers/main.yml`, `defaults/main.yml`, `templates/`, and `files/`. Move related tasks into `roles/webserver/tasks/main.yml`, then call it from a play:

```yaml
---
- name: Apply the webserver role
  hosts: webservers
  become: true
  roles:
    - webserver
```

Collections contain fully qualified modules such as `ansible.posix.firewalld`. **That example module is not part of `ansible-core`**; check `ansible-doc ansible.posix.firewalld` and install the appropriate collection from an approved repository, if absent. In offline environments, prepare collection artifacts in advance and confirm compatibility with your core version.

Example `requirements.yml` for a system that **already has approved collection access**:

```yaml
---
collections:
  - name: ansible.posix
```

```bash
ansible-galaxy collection install -r requirements.yml
```

For a firewall example **after installing the collection and ensuring `firewalld` is installed/running on the target**:

```yaml
- name: Allow HTTP permanently
  ansible.posix.firewalld:
    service: http
    permanent: true
    immediate: true
    state: enabled
```

Check the module documentation and supported versions for your exam environment; do not assume community content is available simply because `ansible-core` is installed.

## 8.6 Automate RHEL system administration

Use purpose-built modules where available, and understand prerequisites:

| Administrative goal | Example content | Additional preparation |
| --- | --- | --- |
| Software packages | `ansible.builtin.dnf` | Working repositories on each target |
| Service management | `ansible.builtin.service` / `ansible.builtin.systemd_service` | Package installed, unit name correct |
| Users/groups | `ansible.builtin.user`, `ansible.builtin.group` | `become: true` |
| Files/permissions | `ansible.builtin.copy`, `.file`, `.lineinfile`, `.template` | Correct ownership, modes, SELinux context |
| Scheduling | `ansible.builtin.cron` | Compatible cron service/setup |
| Archives | `community.general.archive` or documented platform alternatives | Verify collection and runtime dependencies |
| Filesystems and storage | `community.general.parted`, `community.general.lvol`, or Red Hat storage system role | Verify collections, test disks, preserve data |
| Firewall rules | `ansible.posix.firewalld` | Collection installed; firewalld installed and active |
| SELinux | `ansible.posix.selinux` or RHEL SELinux system role | Check content availability and desired policy |

> [!WARNING]
> **Storage operations can destroy data.** Use dedicated empty virtual disks and test with snapshots. Do not blindly apply partitioning/formatting examples against the OS drive. Module names and roles outside `ansible.builtin` can require separate collections.

RHEL ships **system roles**, including the `redhat.rhel_system_roles` collection, for supported administration patterns. See [Red Hat's RHEL 10 system-role documentation](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/automating_system_administration_by_using_rhel_system_roles/introduction-to-rhel-system-roles) to learn the installed role list and supported parameters.

## 8.7 Templates and secrets with Vault

Create a template such as `templates/motd.j2`:

```jinja2
Welcome to {{ inventory_hostname }}
Managed by Ansible
```

Deploy it using:

```yaml
- name: Render the MOTD
  ansible.builtin.template:
    src: templates/motd.j2
    dest: /etc/motd
    owner: root
    group: root
    mode: '0644'
```

For real credentials, use **Ansible Vault** (or an approved secrets manager), never plaintext repository variables:

```bash
ansible-vault create group_vars/managed/vault.yml
ansible-vault edit group_vars/managed/vault.yml
ansible-playbook --ask-vault-pass --ask-become-pass secure-playbook.yml
```

The last command assumes you have actually created `secure-playbook.yml`; it is a **usage pattern**, not a file included in the example project. Do not commit vault password files or output decrypted secrets. Vault protects **secrets at rest**, not every way a running task can accidentally disclose them; use `no_log: true` for tasks handling sensitive data.

## 8.8 Git, editor and validation workflow

Inside a working project:

```bash
git init
git status
git add README.md ansible.cfg inventory.ini create-user.yml
git commit -m 'Create Ansible lab baseline'
```

To obtain an existing project, use `git clone <YOUR-REPOSITORY-URL>` with a real URL you are authorized to access. To publish the local project to a remote repository:

```bash
git remote add origin <YOUR-REPOSITORY-URL>
git branch -M main
git push -u origin main
```

The angle-bracketed URL is a **placeholder**, not a command to paste unmodified. Configure Git identity and remote authentication before committing/pushing. Use the VS Code Ansible extension, linting and a matching **development container/EE** where your AAP version supports them.

```bash
ansible-playbook --syntax-check create-user.yml
ansible-playbook --list-hosts create-user.yml
ansible-navigator run create-user.yml -i inventory.ini -m stdout
```

The Navigator command assumes the tool and its appropriate EE/local-mode settings are ready. Red Hat's current EX294 objectives list operation from VS Code and with a development container. Check exam-specific availability of plugins and offline collections.

## 8.9 Progressive practice plan

| Stage | Deliverable | Proof to collect |
| --- | --- | --- |
| Beginner | Inventory + SSH keys + localhost/user ad hoc run | Inventory graph and successful module output |
| Junior admin | Idempotent user, package and service playbooks | First run `changed`, second run generally `ok` |
| Intermediate | Fact-driven conditions, variables, loops, templates, handlers | Per-host generated config; repeatable behavior |
| Advanced | Role + collection + Vault + error-handling exercises | Local syntax checks, tests, encrypted material |
| Exam rehearsal | Fresh rebuild from a clean VM snapshot without internet | Correct final state after reboot and playbook rerun |

**Official exam reminder:** Red Hat states EX294 is performance-based and the configured end state must survive reboot. Do not rely on this guide as a replacement for the exam-specific objectives and any materials permitted during the actual exam.

**References:** [Official EX294 skills](https://www.redhat.com/en/services/training/ex294-red-hat-certified-engineer-rhce-exam-red-hat-enterprise-linux), [Ansible documentation](https://docs.ansible.com/projects/ansible/latest/), [Ansible Vault](https://docs.ansible.com/projects/ansible/latest/vault_guide/vault.html), [Red Hat AAP 2.6 development tools workflow](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/develop-ref_devtools_components).
