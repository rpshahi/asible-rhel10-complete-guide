# 7. Write, run and verify Ansible playbooks

[← Home](../README.md) · [← Ad hoc commands](06-inventory-ssh-ad-hoc.md) · [Next: Exam roadmap →](08-ex294-roadmap.md)

**Origin:** Added material beyond the Day 1 PDF. The handout does not teach complete YAML playbooks, yet playbook authoring is central to EX294.

## 7.1 YAML essentials

YAML uses **spaces for indentation**, `-` for list items and `key: value` for mappings. Do not indent with tabs. Quote values when YAML might interpret them unexpectedly. A playbook is generally a **list of plays**, even when there is only one play.

```yaml
---
- name: Create a training account
  hosts: managed
  become: true
  tasks:
    - name: Ensure user1 is present
      ansible.builtin.user:
        name: user1
        state: present
        create_home: true
        shell: /bin/bash
```

| Key | Meaning |
| --- | --- |
| `name` | Human-readable play/task explanation |
| `hosts` | Inventory group or host pattern |
| `become` | Privilege escalation (`sudo` on our managed hosts) |
| `tasks` | Ordered list of module actions |
| `ansible.builtin.user` | Built-in user management module (FQCN) |

The executable version is [`examples/create-user.yml`](../examples/create-user.yml).

## 7.2 Validate and execute safely

```bash
ansible-playbook --syntax-check create-user.yml
ansible-playbook --list-hosts create-user.yml
ansible-playbook --list-tasks create-user.yml
ansible-playbook --check --diff --ask-become-pass create-user.yml
ansible-playbook --ask-become-pass create-user.yml
ansible-playbook --ask-become-pass create-user.yml
```

After the second real run, expect most well-written state-management tasks to be `ok` rather than `changed`. **Check mode is predictive, not a full simulation**; not all tasks support it, and external dependencies may affect results.

## 7.3 Practical example: a simple web server

The downloadable [`webserver.yml`](../examples/webserver.yml) configures only the `webservers` group (Server A):

```yaml
---
- name: Configure a basic HTTP server
  hosts: webservers
  become: true
  tasks:
    - name: Ensure Apache is installed
      ansible.builtin.dnf:
        name: httpd
        state: present

    - name: Deploy a generated home page
      ansible.builtin.template:
        src: templates/index.html.j2
        dest: /var/www/html/index.html
        owner: root
        group: root
        mode: '0644'

    - name: Ensure Apache is started at boot
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: true
```

Example template (`examples/templates/index.html.j2`):

```html
<!doctype html>
<html lang="en">
  <head><meta charset="utf-8"><title>Ansible Lab</title></head>
  <body><h1>Managed by Ansible</h1>
  <p>Host: {{ inventory_hostname }}</p></body>
</html>
```

Run only after confirming Server A has usable package repositories:

```bash
ansible-playbook --syntax-check webserver.yml
ansible-playbook --ask-become-pass webserver.yml
ansible webservers -m ansible.builtin.command -a 'systemctl is-active httpd'
ansible webservers -m ansible.builtin.command -a 'curl --fail --silent http://127.0.0.1/'
```

If `curl` is missing on Server A, install it or use another approved HTTP client. Remote access to TCP/80 may require opening the correct firewalld service; see [EX294 roadmap](08-ex294-roadmap.md) for its collection requirements. Package installation from `httpd` requires enabled, functional DNF repositories **on Server A**, not just on the control node.

## 7.4 Facts, variables and conditions

A `when:` expression selects whether a task runs. Facts are collected by default unless `gather_facts: false` is set.

```yaml
---
- name: Demonstrate facts and variables
  hosts: managed
  gather_facts: true
  vars:
    training_packages:
      - vim-enhanced
      - tar
  become: true
  tasks:
    - name: Install utilities on RHEL systems
      ansible.builtin.dnf:
        name: "{{ training_packages }}"
        state: present
      when: ansible_facts['os_family'] == 'RedHat'
```

Use Jinja2 delimiters `{{ ... }}` when a module option expects a rendered value; a direct `when:` condition is an expression and does **not** need `{{ }}`.

## 7.5 Loops, handlers and registered output

```yaml
---
- name: Demonstrate repeatable tasks
  hosts: managed
  become: true
  tasks:
    - name: Create training users
      ansible.builtin.user:
        name: "{{ item }}"
        state: present
      loop:
        - learner1
        - learner2

    - name: Read the configured hostname
      ansible.builtin.command: hostname -f
      register: hostname_result
      changed_when: false

    - name: Show command output
      ansible.builtin.debug:
        msg: "Host says {{ hostname_result.stdout }}"
```

Handlers run when a notifying task reports a change, typically to reload a service after modifying its **configuration**. Example:

```yaml
- name: Install an httpd configuration snippet
  ansible.builtin.template:
    src: templates/security.conf.j2
    dest: /etc/httpd/conf.d/security.conf
    mode: '0644'
  notify: Reload httpd

# At play level (same indentation level as tasks):
handlers:
  - name: Reload httpd
    ansible.builtin.service:
      name: httpd
      state: reloaded
```

This is a **snippet for an existing play**, not a standalone YAML playbook. Create the named template before running it; handlers are unnecessary for changes to a static HTML homepage.

## 7.6 Project-level repeatability

Keep playbooks, inventories, roles, templates and configuration in a Git repository. Test `--syntax-check`, `--list-hosts`, and appropriate `--check` runs; use `ansible-lint` when the tool is available. Do not commit plaintext passwords, SSH private keys or Vault passwords.

**References:** [Ansible playbook getting started](https://docs.ansible.com/projects/ansible/latest/getting_started/get_started_playbook.html), [privilege escalation](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_privilege_escalation.html).
