# 5. Install Ansible and automation content navigator

[← Home](../README.md) · [← Offline DNF](04-offline-dnf-repositories.md) · [Next: Inventory and ad hoc →](06-inventory-ssh-ad-hoc.md)

**Source coverage:** PDF pages 7–9, including installing with `dnf` or `pip`, checking Python packages and starting Navigator.

## 5.1 Install `ansible-core` on the control node

After the authorized DNF repositories work:

```bash
sudo dnf install -y ansible-core
ansible --version
ansible-playbook --version
command -v ansible
ansible-doc ansible.builtin.user
```

This installs the Ansible engine and built-in modules. **Do not install Ansible on every managed host** just to run SSH-based Linux automation; install necessary managed-host prerequisites such as `sshd` and Python instead.

> [!IMPORTANT]
> Red Hat's RHEL 10 system-roles documentation describes the RHEL-supplied **`ansible-core` 2.16** and its intended managed-node scope as RHEL 9 and RHEL 10. The version that actually installs depends on your enabled repositories. Always record the output of `ansible --version` and use documentation suitable for the installed version.

## 5.2 Understand Ansible Navigator

**`ansible-navigator`**, called *automation content navigator* in Red Hat documentation, is a content-development command-line/TUI tool that can run playbooks, browse inventories, inspect collections, read module documentation and inspect **execution environments**.

An **execution environment (EE)** is a container image holding a compatible Ansible runtime, collections and Python/system dependencies. To use an EE, you need a supported container runtime such as Podman, an accessible or locally present EE image, and compatible host/container configuration.

**Navigator is not automatically bundled with RHEL 10 `ansible-core`.** Nor does `dnf install ansible-navigator` necessarily succeed using only the RHEL 10 DVD.

## 5.3 Choose the correct installation path

| Situation | Recommended approach | Limit |
| --- | --- | --- |
| RHEL system-role exercises with `ansible-core` | Install `ansible-core` from approved RHEL repositories | No AAP installation implied |
| Supported AAP developer workstation | Use the documented `ansible-dev-tools`/Navigator RPMs for your **specific AAP and RHEL version** | Requires correct Red Hat entitlement/repos |
| Unsubscribed personal learning workstation with package-index access | Install **community** Navigator in a user-owned Python virtual environment | Not a substitute for supported AAP packaging |
| Offline lab with only a RHEL DVD | Use `ansible-playbook` and other installed `ansible-core` commands first | DVD does not provide arbitrary PyPI packages or EE images |

Red Hat's [AAP 2.6 installation documentation](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6) gives supported development-tool procedures and example repos for **RHEL 8/9**. Do **not** enable an AAP-for-RHEL-9 RPM repository on RHEL 10 without explicit vendor compatibility documentation.

## 5.4 Community Python virtual environment (optional)

This route is useful for **learning**, assumes functioning network access to PyPI (or a curated package mirror), and should not be confused with a supported Red Hat AAP package installation:

```bash
sudo dnf install -y python3-pip
python3 -m venv "$HOME/.venvs/ansible-navigator"
"$HOME/.venvs/ansible-navigator/bin/python" -m pip install --upgrade pip
"$HOME/.venvs/ansible-navigator/bin/python" -m pip install ansible-navigator
source "$HOME/.venvs/ansible-navigator/bin/activate"
ansible-navigator --version
python -m pip show ansible-navigator
python -m pip list
```

If `python3 -m venv` is unavailable, install the appropriate Python virtual-environment support for your RHEL release from an approved source before proceeding.

The PDF proposes `python3 -m pip install ansible-navigator` **as root** and a root-owned `/opt` virtual environment. Prefer an unprivileged **per-user venv**. An externally managed system Python can reject global pip installs; do not override that restriction with `--break-system-packages` or other force options.

## 5.5 Run Navigator with or without an EE

For an **offline learning exercise using the host-installed Ansible runtime**, create a project-level `ansible-navigator.yml`:

```yaml
---
ansible-navigator:
  execution-environment:
    enabled: false
  mode: stdout
```

Then run:

```bash
ansible-navigator --help
ansible-navigator inventory -i inventory.ini --mode stdout
ansible-navigator run create-user.yml -i inventory.ini --mode stdout
```

This only works once Navigator is installed and the current project is configured. **`enabled: false` means no EE**; it is a fallback for first exercises, **not a substitute for practice with execution environments**.

For a supported AAP environment, follow the Red Hat documentation to authenticate to the container registry and select a compatible EE image. Do not assume an image will download in an offline lab.

## 5.6 Diagnose a missing executable or package

```bash
command -v ansible
command -v ansible-navigator
ansible --version
ansible-navigator --version
python3 -m pip list
python3 -m pip show ansible-navigator
```

`python3 -m pip show` searches **that Python interpreter's environment**. If Navigator was installed in a venv, activate the same environment first. `which ansible`/`which ansible-navigator` also work when installed, but `command -v` is a portable shell built-in.

### Classroom screen visibility (from PDF p. 8–9)

The original handout includes instructions for making an instructor's meeting screen easier to see: use the conferencing application's **Full Screen** and **Spotlight/Adjust View** functions if available. These are **presentation controls**, not Ansible features; exact menus depend on the video-conferencing application.

**References:** [Red Hat AAP 2.6 Navigator concepts](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/develop-con_about_ansible_navigator), [Ansible Navigator configuration](https://docs.ansible.com/projects/navigator/settings/), [Ansible installation methods](https://docs.ansible.com/projects/ansible/latest/installation_guide/).
