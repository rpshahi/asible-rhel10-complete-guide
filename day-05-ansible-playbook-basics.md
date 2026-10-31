# Day 5: Writing and Running Ansible Playbooks (AU294 / RHCE Study Guide)

## Overview

This guide covers the first hands-on playbook work in the RHCE preparation course based on AU294 (*Red Hat Enterprise Linux Automation with Ansible*). By the end you should be able to:

- Set up a project directory with an inventory, `ansible.cfg` and `ansible-navigator.yml`.
- Verify connectivity to managed nodes with ad-hoc commands.
- Write single-play and multi-play playbooks using the `user`, `dnf` and `service` modules.
- Validate a playbook with `--syntax-check`, preview it with `--check`, run it, and prove idempotency.
- Revert every change made during the exercises.

## Table of Contents

1. [Lab environment and assumptions](#lab-environment-and-assumptions)
2. [Prerequisites](#prerequisites)
3. [Project setup](#project-setup)
4. [Playbook anatomy](#playbook-anatomy)
5. [The standard run workflow](#the-standard-run-workflow)
6. [Example 1: create a user](#example-1-create-a-user)
7. [Example 2: install and start httpd and postfix](#example-2-install-and-start-httpd-and-postfix)
8. [Example 3: multi-play playbook](#example-3-multi-play-playbook)
9. [Idempotency and output verbosity](#idempotency-and-output-verbosity)
10. [Key points](#key-points)
11. [Common mistakes and exam tips](#common-mistakes-and-exam-tips)
12. [Cleanup](#cleanup)
13. [Quick reference](#quick-reference)

## Lab environment and assumptions

| Node | FQDN | Short name | IP address | Role |
|------|------|------------|------------|------|
| Control | `control.lab.example.com` | `control` | `172.25.250.9` | Control node (runs Ansible) |
| Managed 1 | `servera.lab.example.com` | `servera` | `172.25.250.10` | Managed node |
| Managed 2 | `serverb.lab.example.com` | `serverb` | `172.25.250.11` | Managed node |

Assumptions used throughout this guide:

1. The managed hosts run RHEL 10. The AU294 course book targets a specific RHEL release, so package names and output may differ slightly from the book.
2. Every node has a `student` user and a `devops` user.
3. `student` has sudo access on every node. `devops` has sudo with `NOPASSWD`.
4. You work on the control node as `student`. Ansible connects to the managed nodes as `devops`.
5. SSH key-based, passwordless login from the control node to `devops@servera` and `devops@serverb` is configured.
6. `/etc/hosts` on all three nodes resolves the FQDNs and the short names (`servera`, `serverb`, `control`).
7. `ansible-navigator.yml` exists in the project directory and uses the `ee-supported-rhel8:latest` execution environment (EE). The EE runs on the control node, so it does not need to match the RHEL release of the managed nodes.
8. Ad-hoc commands use `ansible`, installed on the control node. Playbooks use `ansible-navigator`, which runs them inside the EE.
9. Playbooks for this chapter are stored in `playbooks/ch01/`.
10. The managed nodes receive no firewall, SELinux, mount or cron changes.

## Prerequisites

### Name resolution

`/etc/hosts` on all three nodes:

```text
172.25.250.9   control.lab.example.com   control
172.25.250.10  servera.lab.example.com   servera
172.25.250.11  serverb.lab.example.com   serverb
```

### SSH key-based access (control node, as `student`)

Skip generating the key if one already exists. `ssh-copy-id` needs an existing key pair.

```bash
ssh-keygen -t ed25519
ssh-copy-id devops@servera
ssh-copy-id devops@serverb
```

Each `ssh-copy-id` asks for the `devops` password of that host once. Verify key login and passwordless sudo:

```bash
ssh devops@servera sudo -n true && echo "servera OK"
ssh devops@serverb sudo -n true && echo "serverb OK"
```

### Baseline check (before running any playbook)

Record what is already installed so that cleanup does not remove packages that existed before Day 5. You can run this from the project directory after the project setup below.

```bash
ansible production -m ansible.builtin.command -a "rpm -q httpd postfix mariadb-server"
ansible servera -m ansible.builtin.command -a "id alex"
```

`package ... is not installed` and `no such user` results (`FAILED | rc=1`) are expected on a clean host.

### Vim settings for YAML

Add this line to `~/.vimrc`:

```vim
autocmd FileType yaml setlocal ai ts=2 sw=2 et
```

| Setting | Meaning |
|---------|---------|
| `ai` | Auto-indent new lines |
| `ts=2` | A tab character displays as 2 columns |
| `sw=2` | Indentation uses 2 columns |
| `et` | Insert spaces instead of tabs |

## Project setup

Create the project directory and the playbook directory:

```bash
mkdir -p ~/ansible-craw/playbooks/ch01
cd ~/ansible-craw
```

### Inventory

Create `inventory`:

```ini
[production]
servera
serverb

[webservers]
servera

[dbservers]
serverb
```

`control` is not listed because no tasks run against it. The short names must resolve, which `/etc/hosts` provides.

Verify the inventory:

```bash
ansible-inventory --graph
```

Expected output (similar):

```text
@all:
  |--@ungrouped:
  |--@dbservers:
  |  |--serverb
  |--@production:
  |  |--servera
  |  |--serverb
  |--@webservers:
  |  |--servera
```

### ansible.cfg

Create `ansible.cfg` in the project directory:

```ini
[defaults]
inventory = ./inventory
remote_user = devops
ask_pass = false

[privilege_escalation]
become = true
become_user = root
become_method = sudo
ask_become_pass = false
```

Because `become = true` is set here, tasks escalate to root by default. Adding `become: true` to a play is redundant, but harmless and explicit.

### ansible-navigator.yml

Example (your file may contain more settings, such as a registry path):

```yaml
---
ansible-navigator:
  execution-environment:
    image: ee-supported-rhel8:latest
    pull:
      policy: missing
```

Check that the EE image is available:

```bash
ansible-navigator images
```

### Test connectivity with an ad-hoc command

```bash
ansible all -m ansible.builtin.ping
```

Expected output for each host (the interpreter path may differ):

```text
servera | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.12"
    },
    "changed": false,
    "ping": "pong"
}
```

Test a single host:

```bash
ansible servera -m ansible.builtin.ping
```

## Playbook anatomy

A playbook is a YAML file that contains a **list of plays**. Each play maps a group of hosts to a list of tasks.

```yaml
---
- name: First play                    # a play is one item of the playbook list (starts with "-")
  hosts: servera                      # required: which hosts to target
  tasks:                              # list of tasks for this play
    - name: First task                # a task is one item of the tasks list
      ansible.builtin.ping:           # module name, with its arguments below it
    - name: Second task
      ansible.builtin.debug:
        msg: Hello

- name: Second play
  hosts: serverb
  tasks:
    - name: Another task
      ansible.builtin.ping:
```

| Element | Notes |
|---------|-------|
| `---` | Optional YAML document start marker. Commonly used. |
| `...` | Optional YAML document end marker. Rarely used and not required. |
| `- name:` (play) | Optional but recommended. It describes the play's purpose and helps in output with multiple plays. |
| `hosts:` | Required. A host, a group, `all` or a host pattern from the inventory. |
| `tasks:` | A list. Each task calls one module. |
| Module arguments | A YAML dictionary indented under the module name. |

Notes:

- Use spaces for indentation, never tabs.
- Key order within a play is not significant, but `name`, `hosts`, then `tasks` is the conventional order.
- Use fully qualified collection names (FQCN) such as `ansible.builtin.user`.

## The standard run workflow

Run these steps for every playbook, from the project directory (`~/ansible-craw`):

| Step | Purpose | Command |
|------|---------|---------|
| 1. Syntax check | Validates YAML and playbook structure without running tasks | `ansible-navigator run -m stdout <playbook> --syntax-check` |
| 2. Dry run | Shows what would change, without changing anything | `ansible-navigator run -m stdout <playbook> --check` |
| 3. Run | Applies the changes | `ansible-navigator run -m stdout <playbook>` |
| 4. Verify | Confirms the result on the managed host | An ad-hoc command, as shown in each example |
| 5. Re-run | Confirms idempotency (`changed=0`) | The same command as step 3 |

`-m stdout` (`--mode stdout`) prints results to the terminal instead of opening the interactive text interface.

A successful syntax check prints only the playbook path:

```text
playbook: /home/student/ansible-craw/playbooks/ch01/create_alex_user.yml
```

## Example 1: create a user

Create `playbooks/ch01/create_alex_user.yml`:

```bash
vim playbooks/ch01/create_alex_user.yml
```

```yaml
---
- name: Configure important user consistently
  hosts: servera
  tasks:
    - name: Alex exists with UID 4000
      ansible.builtin.user:
        name: alex
        uid: 4000
        state: present
```

Run the workflow:

```bash
ansible-navigator run -m stdout playbooks/ch01/create_alex_user.yml --syntax-check
ansible-navigator run -m stdout playbooks/ch01/create_alex_user.yml --check
ansible-navigator run -m stdout playbooks/ch01/create_alex_user.yml
```

Verify on `servera`:

```bash
ansible servera -m ansible.builtin.command -a "id alex"
ansible servera -m ansible.builtin.shell -a "grep alex /etc/passwd"
```

Expected output is similar to the following. No `comment` was set, so the GECOS field is empty, and the GID may differ.

```text
servera | CHANGED | rc=0 >>
alex:x:4000:4000::/home/alex:/bin/bash
```

## Example 2: install and start httpd and postfix

Create `playbooks/ch01/deploy_httpd_postfix.yml`:

```bash
vim playbooks/ch01/deploy_httpd_postfix.yml
```

```yaml
---
- name: Configure HTTPD and Postfix services
  hosts: servera
  tasks:
    - name: Install httpd package
      ansible.builtin.dnf:
        name: httpd
        state: present

    - name: Start and enable httpd service
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: true

    - name: Install postfix package
      ansible.builtin.dnf:
        name: postfix
        state: present

    - name: Start and enable postfix service
      ansible.builtin.service:
        name: postfix
        state: started
        enabled: true
```

Validate, preview and run:

```bash
# Check the playbook for syntax errors
ansible-navigator run -m stdout playbooks/ch01/deploy_httpd_postfix.yml --syntax-check

# Perform a dry run without making changes to managed hosts
ansible-navigator run -m stdout playbooks/ch01/deploy_httpd_postfix.yml --check

# Execute the playbook
ansible-navigator run -m stdout playbooks/ch01/deploy_httpd_postfix.yml
```

Verify:

```bash
ansible servera -m ansible.builtin.command -a "systemctl is-active httpd postfix"
ansible servera -m ansible.builtin.command -a "systemctl is-enabled httpd postfix"
```

Both commands should print `active` / `enabled` once per service.

Notes:

1. `--syntax-check` validates the syntax without executing tasks.
2. `--check` previews changes without applying them.
3. Without either option, the playbook executes and applies the changes.
4. A second run reports no changes because tasks are **idempotent**: if a host is already in the desired state, nothing is changed.

## Example 3: multi-play playbook

A multi-play playbook contains two or more plays. Each play can target different hosts and do different work. Scenario: a two-tier application.

| Tier | Inventory group | Host | Software |
|------|-----------------|------|----------|
| Database | `dbservers` | `serverb` | MariaDB |
| Web | `webservers` | `servera` | Apache HTTP Server |

Create `playbooks/ch01/webserver_setup.yml`:

```bash
vim playbooks/ch01/webserver_setup.yml
```

```yaml
---
# PLAY 1: Configure Database Server
- name: Configure Database Server
  hosts: dbservers
  become: true

  tasks:
    - name: Install MariaDB
      ansible.builtin.dnf:
        name: mariadb-server
        state: present

    - name: Start MariaDB service
      ansible.builtin.service:
        name: mariadb
        state: started
        enabled: true

# PLAY 2: Configure Web Server
- name: Configure Web Server
  hosts: webservers
  become: true

  tasks:
    - name: Install Apache
      ansible.builtin.dnf:
        name: httpd
        state: present

    - name: Start Apache service
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: true
```

Validate, preview and run:

```bash
# Check the playbook for syntax errors
ansible-navigator run -m stdout playbooks/ch01/webserver_setup.yml --syntax-check

# Perform a dry run without making changes to managed hosts
ansible-navigator run -m stdout playbooks/ch01/webserver_setup.yml --check

# Execute the playbook
ansible-navigator run -m stdout playbooks/ch01/webserver_setup.yml
```

Verify:

```bash
ansible dbservers -m ansible.builtin.command -a "systemctl is-active mariadb"
ansible webservers -m ansible.builtin.command -a "systemctl is-active httpd"
```

## Idempotency and output verbosity

### Idempotency

Run the playbook a second time. Every task should report `ok` and the play recap should show `changed=0`.

```bash
ansible-navigator run -m stdout playbooks/ch01/webserver_setup.yml
```

Typical recap on the second run:

```text
PLAY RECAP *********************************************************
servera   : ok=3  changed=0  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0
serverb   : ok=3  changed=0  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0
```

### Verbosity

Add `-v` for more detail. Each extra `v` increases the detail:

| Option | Adds |
|--------|------|
| `-v` | Task results (module return data) |
| `-vv` | Task input details, such as the playbook file and line of each task |
| `-vvv` | Connection details (SSH commands and users) |
| `-vvvv` | Extra connection debugging (plugins and scripts) |

Options that `ansible-navigator` does not recognise are passed to `ansible-playbook`, so you can add the verbosity flag to the same command:

```bash
ansible-navigator run -m stdout playbooks/ch01/deploy_httpd_postfix.yml -vv
```

## Key points

- A playbook is a YAML list of plays, and each play contains a list of tasks.
- `hosts` is required. `name` is optional but strongly recommended.
- Use `ansible.builtin.dnf`, `ansible.builtin.service` and `ansible.builtin.user` for packages, services and users.
- Modules are declarative. You describe the desired **state** (`present`, `started`, `absent`), not the commands.
- Idempotency means a second run makes no changes.
- Always run `--syntax-check`, then `--check`, then the real run.
- `ansible.cfg` in the current directory supplies the inventory, remote user and privilege escalation settings.
- `ansible` (ad-hoc) uses the local installation. `ansible-navigator run` uses the EE. Both read the same `ansible.cfg`.

## Common mistakes and exam tips

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Missing colon after a module name (for example `ansible.builtin.dnf` without `:`) | Syntax error | Write `ansible.builtin.dnf:` and re-run `--syntax-check` |
| Tabs or wrong indentation | YAML parse error, or arguments applied to the wrong level | Use spaces only. Set the `.vimrc` options above. |
| `hosts:` does not match an inventory name (for example an FQDN when the inventory uses short names) | `Could not match supplied host pattern` and no hosts matched | Use names exactly as listed in the inventory, or a group name. Confirm with `ansible-inventory --graph`. |
| Running from the wrong directory | `ansible.cfg` is ignored and the inventory is not found | `cd ~/ansible-craw`. `ansible --version` shows the config file in use. |
| Starting a service in `--check` mode before its package is installed | Check mode may fail because the service unit does not exist yet | Expect this on a first run. A real run, then a re-run, is the proof. |
| Expecting `changed=0` on the first run | Confusion about idempotency | Changes on the first run are normal. `changed=0` is expected on the second. |
| Verifying on a host name that is not in the inventory | `Could not match supplied host pattern` | Verify on `servera` or `serverb`. |
| Mixed-up file names (for example `create_newbie_user.yml` vs `create_alex_user.yml`) | Wrong file executed or file not found | Keep one consistent name per playbook. |
| Assuming a package name exists on a newer RHEL release | Package not found | Check with `dnf info <package>` on the managed host. |

Exam tips:

- Use `ansible-navigator doc ansible.builtin.user` (or `ansible-doc ansible.builtin.user`) to find module arguments. Examples are included at the bottom of the output.
- Run `--syntax-check` before every real run.
- Re-run the playbook at the end. A clean second run shows `changed=0`, which also shows that it is idempotent.
- Confirm the final state yourself with an ad-hoc command rather than trusting the recap alone.
- Check the exam environment for the exact commands it provides (`ansible-navigator` and/or `ansible-playbook`).

## Cleanup

This reverts everything created during Day 5. The managed hosts receive no firewall, SELinux, mount or cron changes, so no steps are needed for those.

Items that are kept on purpose because they are lab prerequisites:

- The `devops` SSH key on the managed hosts
- The `/etc/hosts` entries
- The `~/ansible-craw` project folder

> **Before you run cleanup:** compare with your baseline check. If `httpd`, `postfix` or `mariadb-server` was already installed before Day 5, remove it from the cleanup playbook.

### What gets reverted

| Host | Change made by Day 5 | Cleanup |
|------|----------------------|---------|
| `servera` | User `alex` (UID 4000) and its home directory | Remove the user and its home directory |
| `servera` | `httpd` package, enabled and started | Stop and disable the service, remove the package |
| `servera` | `postfix` package, enabled and started | Stop and disable the service, remove the package |
| `serverb` | `mariadb-server` package, enabled and started | Stop and disable the service, remove the package, delete the data directory |

### Cleanup playbook

Create `playbooks/ch01/cleanup_day05.yml`:

```bash
vim playbooks/ch01/cleanup_day05.yml
```

```yaml
---
- name: Revert Day 5 changes on servera
  hosts: servera
  tasks:
    - name: Gather service facts
      ansible.builtin.service_facts:

    - name: Stop and disable httpd and postfix if present
      ansible.builtin.service:
        name: "{{ item }}"
        state: stopped
        enabled: false
      loop:
        - httpd
        - postfix
      when: (item ~ '.service') in ansible_facts.services

    - name: Remove httpd and postfix packages
      ansible.builtin.dnf:
        name:
          - httpd
          - postfix
        state: absent

    - name: Remove user alex and the home directory
      ansible.builtin.user:
        name: alex
        state: absent
        remove: true

- name: Revert Day 5 changes on serverb
  hosts: serverb
  tasks:
    - name: Gather service facts
      ansible.builtin.service_facts:

    - name: Stop and disable mariadb if present
      ansible.builtin.service:
        name: mariadb
        state: stopped
        enabled: false
      when: "'mariadb.service' in ansible_facts.services"

    - name: Remove mariadb-server package
      ansible.builtin.dnf:
        name: mariadb-server
        state: absent

    - name: Delete the MariaDB data directory created by the first start
      ansible.builtin.file:
        path: /var/lib/mysql
        state: absent
```

Run it:

```bash
ansible-navigator run -m stdout playbooks/ch01/cleanup_day05.yml --syntax-check
ansible-navigator run -m stdout playbooks/ch01/cleanup_day05.yml --check
ansible-navigator run -m stdout playbooks/ch01/cleanup_day05.yml
```

Notes:

- `dnf` removes dependencies that were installed only as dependencies by default, so `autoremove` is not needed.
- Removing a package does not delete every file it created. `/etc/httpd`, `/var/www` and the postfix spool directories may remain. Remove them only if you need a completely clean host.
- Deleting `/var/lib/mysql` destroys all MariaDB data. That is acceptable only in a throwaway lab.

### Equivalent ad-hoc commands

These achieve the same result without the playbook. Run each one only if the corresponding item was created.

```bash
# servera: stop services, remove packages, remove user
ansible servera -m ansible.builtin.service -a "name=httpd state=stopped enabled=false"
ansible servera -m ansible.builtin.service -a "name=postfix state=stopped enabled=false"
ansible servera -m ansible.builtin.dnf -a '{"name": ["httpd", "postfix"], "state": "absent"}'
ansible servera -m ansible.builtin.user -a "name=alex state=absent remove=true"

# serverb: stop service, remove package, delete data
ansible serverb -m ansible.builtin.service -a "name=mariadb state=stopped enabled=false"
ansible serverb -m ansible.builtin.dnf -a "name=mariadb-server state=absent"
ansible serverb -m ansible.builtin.file -a "path=/var/lib/mysql state=absent"
```

The `service` commands fail if the package is not installed. Run them before the package removal.

### Verify the cleanup

```bash
ansible production -m ansible.builtin.command -a "rpm -q httpd postfix mariadb-server"
ansible servera -m ansible.builtin.command -a "id alex"
```

Expected: `package ... is not installed` and `id: 'alex': no such user` (`FAILED | rc=1`). Failed results here confirm a clean state.

### Optional: remove temporary files on the control node

`ansible-navigator` writes playbook artifacts (`*-artifact-*.json`) and a log file. Removing them does not touch the project itself:

```bash
find ~/ansible-craw -name '*-artifact-*.json' -delete
rm -f ~/ansible-craw/ansible-navigator.log
```

## Quick reference

| Task | Command |
|------|---------|
| Show the inventory | `ansible-inventory --graph` |
| Ping all managed nodes | `ansible all -m ansible.builtin.ping` |
| Copy the SSH key | `ssh-copy-id devops@servera` |
| Show the config in use | `ansible --version` |
| List available EEs | `ansible-navigator images` |
| Syntax check | `ansible-navigator run -m stdout <playbook> --syntax-check` |
| Dry run | `ansible-navigator run -m stdout <playbook> --check` |
| Run | `ansible-navigator run -m stdout <playbook>` |
| More detail | add `-v`, `-vv`, `-vvv` or `-vvvv` |
| Module documentation | `ansible-navigator doc ansible.builtin.user` |

| File | Location |
|------|----------|
| Project directory | `~/ansible-craw/` |
| Inventory | `~/ansible-craw/inventory` |
| Configuration | `~/ansible-craw/ansible.cfg` |
| Navigator settings | `~/ansible-craw/ansible-navigator.yml` |
| Playbooks | `~/ansible-craw/playbooks/ch01/` |
