# 6. Inventory, SSH and ad hoc automation

[← Home](../README.md) · [← Installation](05-ansible-installation-and-navigator.md) · [Next: Playbooks →](07-playbooks.md)

**Source coverage:** PDF pages 9–10. The source starts directly with `localhost` and a user-creation command; this chapter adds the inventory/SSH prerequisites needed to manage the two servers.

## 6.1 Project layout

The downloadable [`examples`](../examples/) folder contains:

```text
examples/
├── ansible.cfg
├── ansible-navigator.yml
├── inventory.ini
├── create-user.yml
├── webserver.yml
└── templates/
    └── index.html.j2
```

Work from this directory so Ansible can find its local `ansible.cfg` and inventory. Files should be owned by your unprivileged control-node user, **not placed in a world-writable project directory**.

## 6.2 Create a static inventory

[`examples/inventory.ini`](../examples/inventory.ini):

```ini
[control]
workstation.lab.example.com ansible_host=172.25.250.9 ansible_connection=local

[webservers]
servera.lab.example.com ansible_host=172.25.250.10

[dbservers]
serverb.lab.example.com ansible_host=172.25.250.11

[managed:children]
webservers
dbservers

[all:vars]
ansible_user=student
```

`ansible_host` specifies the network destination; the inventory name remains the human-friendly hostname. `ansible_connection=local` runs control-targeted modules locally. `managed` groups both managed-server groups via `:children`.

Validate before making changes:

```bash
ansible-inventory -i inventory.ini --graph
ansible-inventory -i inventory.ini --list
ansible managed --list-hosts
```

> [!NOTE]
> `ansible localhost --list-hosts` from the PDF lists a matching target and may use Ansible's **implicit localhost**. It does **not** test network reachability, SSH authentication, Python, or sudo. An explicit inventory entry makes classroom examples easier to follow.

## 6.3 Choose an `ansible.cfg`

[`examples/ansible.cfg`](../examples/ansible.cfg):

```ini
[defaults]
inventory = ./inventory.ini
remote_user = student
host_key_checking = True
```

**Security:** Do not disable SSH host-key checking to silence trust errors. Verify and register host fingerprints correctly. View the active settings:

```bash
ansible-config view
ansible-config dump --only-changed
```

## 6.4 Test SSH, Python, sudo and targeting

First ensure SSH keys work as described in [Linux preparation](03-linux-and-networking.md).

```bash
ssh student@servera.lab.example.com 'hostname -f'
ansible managed -m ansible.builtin.ping
ansible managed -m ansible.builtin.setup -a 'filter=ansible_distribution*'
ansible webservers -m ansible.builtin.command -a 'hostname -f'
ansible managed -m ansible.builtin.command -a 'id' --become --ask-become-pass
```

| Command | What it actually verifies |
| --- | --- |
| `ansible ... --list-hosts` | Inventory host-pattern selection |
| `ssh ...` | SSH/network authentication |
| `ansible ... -m ansible.builtin.ping` | Ansible remote connection plus Python module execution |
| `... -m ansible.builtin.command -a 'hostname -f'` | Target command execution |
| `... -b -K` | Sudo/privilege-escalation path |

## 6.5 Correct user creation with a purpose-built module

The PDF uses:

```bash
ansible localhost -m command -a "sudo useradd user1"
```

This is **not recommended**: it invokes `sudo` inside a `command`-module process rather than using Ansible privilege escalation, and `useradd` fails on a repeat run. `ansible.builtin.command` does **not** provide a shell by default, nor is it a replacement for account-management modules.

For the **local workstation**, use:

```bash
ansible localhost -c local \
  -m ansible.builtin.user -a 'name=user1 state=present' \
  --become --ask-become-pass
```

For **both managed servers**, use:

```bash
ansible managed -m ansible.builtin.user \
  -a 'name=user1 state=present create_home=true shell=/bin/bash' \
  --become --ask-become-pass
```

Run it twice. The first run should report a change if the account was missing; the second normally reports `SUCCESS` without a change. Idempotence is a goal of the module, but real change status can vary if external configuration changes the account between runs.

## 6.6 Verify user creation properly

```bash
ansible managed -m ansible.builtin.command -a 'id user1'
ansible managed -m ansible.builtin.command -a 'getent passwd user1'
```

`id` confirms account/group identity; `getent` checks the account database (which need not be backed solely by `/etc/passwd`). Use `ansible.builtin.command` to invoke them. The PDF's `sudo grep '^user1:' /etc/passwd` is unnecessary for a public account database and does not check non-file NSS sources.

## 6.7 `command` versus `shell`

```bash
ansible managed -m ansible.builtin.command -a 'uptime'
```

The `command` module runs a program directly; operators such as `|`, `>`, `&&` are **not interpreted** as shell syntax. For a genuinely necessary pipeline, use `ansible.builtin.shell`, with carefully controlled input:

```yaml
- name: Example where a shell pipeline is actually needed
  ansible.builtin.shell: 'journalctl -b --no-pager | tail -n 5'
  changed_when: false
```

Prefer structured modules where available, and use `changed_when: false` for genuinely read-only command checks.

**References:** [Ansible inventory guide](https://docs.ansible.com/projects/ansible/latest/getting_started/get_started_inventory.html), [implicit localhost](https://docs.ansible.com/projects/ansible/latest/inventory/implicit_localhost.html), [user module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/user_module.html), [command module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/command_module.html).
