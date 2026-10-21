# 1. Ansible fundamentals

[← Home](../README.md) · [Next: Build the lab →](02-vmware-rhel10-lab.md)

## Learning outcomes

Explain how Ansible works, distinguish `ansible-core` from AAP, and recognize inventories, modules, plays, tasks, and playbooks.

## What is Ansible?

Ansible automates repetitive administration tasks through descriptions of a **desired state**. You ordinarily run Ansible from a **control node** and connect to **managed nodes** over SSH. Most Linux targets do **not** need an Ansible agent or the Ansible software installed locally; however, many modules require a usable Python interpreter on the target.

| Term | Beginner-friendly meaning | Lab example |
| --- | --- | --- |
| Control node | Machine where you run Ansible commands | `workstation.lab.example.com` |
| Managed node | Machine configured by Ansible | `servera.lab.example.com` |
| Inventory | File or plugin describing hosts and groups | `inventory.ini` |
| Module | Unit of work, such as creating a user | `ansible.builtin.user` |
| Task | A named invocation of a module | "Ensure user1 exists" |
| Play | Tasks assigned to a host pattern | `hosts: managed` |
| Playbook | YAML document containing one or more plays | `create-user.yml` |
| Collection | Package of related roles/modules/plugins/docs | `ansible.posix` |
| Role | Reusable structure of tasks and supporting files | `roles/webserver/` |
| Execution environment (EE) | Container image with Ansible and required runtime content | AAP / Navigator workflows |

## Product distinction: three common names

- **`ansible-core`**: Ansible's CLI engine and built-in content. RHEL 10 provides it for documented use cases, including RHEL system roles.
- **Community `ansible` package**: adds a distribution of community-maintained collections to `ansible-core`; content and compatibility can differ from Red Hat-supported products.
- **Red Hat Ansible Automation Platform (AAP)**: Red Hat's enterprise automation product, including supported automation tooling and platform services. A functioning `ansible` CLI on RHEL **does not** prove that AAP is installed or licensed.

## Ad hoc command vs. playbook

An **ad hoc command** runs an action without saving the procedure in a reusable YAML file:

```bash
ansible managed -m ansible.builtin.ping
```

Here `managed` matches a group from inventory. `ansible.builtin.ping` is an Ansible **Python/connectivity test**, **not ICMP ping**.

A **playbook** records the desired outcome:

```yaml
---
- name: Make sure an account exists
  hosts: managed
  become: true
  tasks:
    - name: Ensure user1 exists
      ansible.builtin.user:
        name: user1
        state: present
```

Save as `create-user.yml` and run it using `ansible-playbook create-user.yml --ask-become-pass` after configuring your lab.

## Idempotence: a core automation idea

An idempotent task only changes a machine when it is different from the requested state. For example, the `user` module normally reports `changed` when first creating an account, then `ok` on a second run. A raw `useradd` command is not a good substitute: it errors when the user already exists, and it does not express the desired state.

> [!TIP]
> Use module **fully qualified collection names (FQCNs)**, such as `ansible.builtin.user`, and read their built-in help with `ansible-doc ansible.builtin.user`.

## Architecture flow

```text
Playbook + configuration + inventory
                |
           Ansible CLI
                |
       SSH + privilege escalation
                |
       Managed Linux machines
                |
         Desired state reached
```

**Reading:** [Ansible architecture and getting started](https://docs.ansible.com/projects/ansible/latest/getting_started/index.html), [playbooks](https://docs.ansible.com/projects/ansible/latest/getting_started/get_started_playbook.html).
