# 3. Linux preparation, networking, SSH and sudo

[← Home](../README.md) · [← Lab setup](02-vmware-rhel10-lab.md) · [Next: Offline DNF →](04-offline-dnf-repositories.md)

**Source coverage:** PDF pages 3–4, with additional prerequisites for remote automation.

## 3.1 Set and inspect the hostname

On the workstation:

```bash
sudo hostnamectl set-hostname workstation.lab.example.com
hostnamectl status
```

Use `servera.lab.example.com` and `serverb.lab.example.com` on their respective VMs. You do **not** need to log in as root once to “create the root profile”; use `sudo -i` only for system administration when required.

## 3.2 Find the real network device and profile

```bash
nmcli -f NAME,UUID,TYPE,DEVICE connection show
nmcli device status
ip -brief link
```

The PDF first shows `ens18`, then configures **`ens160`**. Interface names are *not interchangeable*: use the actual **device** and **connection profile** shown by **your** VM. Avoid accidentally adding a second active profile to the same NIC.

### Preferred: modify the existing connection profile

Substitute the actual profile name for `"System ens160"`; the following commands apply to **Workstation** only:

```bash
PROFILE='System ens160'   # replace with your actual nmcli connection name
sudo nmcli connection modify "$PROFILE" \
  ipv4.method manual \
  ipv4.addresses 172.25.250.9/24 \
  ipv4.gateway 172.25.250.2 \
  connection.autoconnect yes
sudo nmcli connection up "$PROFILE"
```

`172.25.250.2` above is an **example**; replace it with the gateway reported by VMware's **NAT Settings**. If DNS is available, configure **its actual address** with `ipv4.dns`. A gateway is not necessarily a DNS server. Running `nmcli connection up` over SSH can interrupt the session; use the VM console for network changes.

### Alternative: create a new connection if needed

When no suitable profile exists and the actual interface name is `ens160`:

```bash
sudo nmcli connection add type ethernet \
  con-name ansible-lab ifname ens160 \
  ipv4.method manual ipv4.addresses 172.25.250.9/24 \
  ipv4.gateway 172.25.250.2 connection.autoconnect yes
sudo nmcli connection up ansible-lab
```

Deactivate/remove unintended competing profiles only after confirming the active configuration. For Server A and B use `.10` and `.11` respectively, and their actual profile/device names.

## 3.3 Verify network state

```bash
ip -4 address show
ip -4 route
nmcli -f GENERAL.DEVICES,IP4.ADDRESS,IP4.GATEWAY connection show --active
ping -c 2 172.25.250.10    # from workstation
ping -c 2 172.25.250.11
```

`ping` tests ICMP reachability and may be blocked by a firewall. It is different from the Ansible `ansible.builtin.ping` module, which tests the remote Ansible connection and Python.

## 3.4 Resolve names without depending on external DNS

On **all three VMs**, add entries to `/etc/hosts` (preserve existing localhost entries):

```text
172.25.250.9   workstation.lab.example.com workstation
172.25.250.10  servera.lab.example.com servera
172.25.250.11  serverb.lab.example.com serverb
```

Check:

```bash
getent hosts servera.lab.example.com
getent hosts serverb.lab.example.com
```

Ansible may also reach hosts by inventory `ansible_host=` IP without relying on DNS, but SSH and manual troubleshooting benefit from consistent name resolution.

## 3.5 Create a safe automation login

On each **managed host** (console or an existing administrative session):

```bash
sudo id student                  # verify the account exists
sudo usermod -aG wheel student   # only if your authorized lab policy allows sudo
sudo systemctl enable --now sshd
sudo -l -U student               # inspect configured sudo permissions
```

The `wheel` group is commonly granted sudo on RHEL, but you must **verify local sudoers policy** instead of assuming membership is sufficient. If `student` does not exist, create it explicitly with `sudo useradd -m student` followed by `sudo passwd student` (use a unique lab password).

On **Workstation**, as `student`:

```bash
ssh-keygen -t ed25519 -C 'ansible-lab-student'
ssh-copy-id student@servera.lab.example.com
ssh-copy-id student@serverb.lab.example.com
ssh student@servera.lab.example.com 'hostname -f'
ssh student@serverb.lab.example.com 'hostname -f'
```

Accept host keys only after verifying the fingerprint via a trusted channel. Keep private keys private. Use `sudo` on managed nodes rather than SSH as root.

### Privilege escalation with Ansible

```bash
ansible managed -m ansible.builtin.command -a 'id' --become --ask-become-pass
```

`--become` (`-b`) activates escalation; `--ask-become-pass` (`-K`) requests the sudo password when needed. Ansible can authenticate over SSH as `student` and **separately** run privileged tasks as root.

> [!TIP]
> For an isolated exam lab, instructors sometimes configure passwordless `sudo` to simplify practice. Do so **only** with a conscious policy decision and correct sudoers validation. This guide defaults to prompting instead of embedding any passwords.

**References:** [Red Hat: prepare RHEL 10 control and managed nodes](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/automating_system_administration_by_using_rhel_system_roles/preparing-a-control-node-and-managed-nodes-to-use-rhel-system-roles), [Ansible: become](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_privilege_escalation.html).
