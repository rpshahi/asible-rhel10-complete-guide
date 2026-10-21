# 2. Build the VMware/RHEL 10 lab

[← Home](../README.md) · [← Fundamentals](01-ansible-fundamentals.md) · [Next: Linux preparation →](03-linux-and-networking.md)

**Source coverage:** PDF pages 1–3 (Workstation installation, Virtual Network Editor screenshots, VM definitions and RHEL installer steps).

## 2.1 Obtain software safely

1. Install a supported **VMware Workstation Pro** release from [Broadcom's official download instructions](https://knowledge.broadcom.com/external/article/368667/download-and-license-free-vmware-desktop.html). The PDF names **26H1u1** and provides a third-party TechSpot mirror; we intentionally do **not** reproduce or endorse that mirror. Check platform compatibility and checksum information from the vendor.
2. Obtain an appropriate **RHEL 10 binary DVD ISO** from [Red Hat Developer](https://developers.redhat.com/products/rhel/download) or your organization's authorized Red Hat download channel. Confirm entitlement, architecture (`x86_64` for the examples), and checksum. A boot ISO is *not* interchangeable with a full DVD when creating an offline BaseOS/AppStream repository.
3. Have adequate CPU/RAM/storage available on the host. Virtualization support must be enabled where applicable.

## 2.2 Create the virtual network

In VMware **Edit → Virtual Network Editor**, open administrative settings if required. Configure **VMnet8** in **NAT** mode (the equivalent dialog may vary by release).

| Setting | Proposed lab value | Important distinction |
| --- | --- | --- |
| Network type | NAT | Can provide outbound access; **not an air gap** |
| Subnet | `172.25.250.0` | Private example network |
| Mask | `255.255.255.0` (`/24`) | Network `172.25.250.0/24` |
| Static guest IPs | `.9`, `.10`, `.11` | Assigned inside RHEL |
| Suggested DHCP pool | `.100`–`.199` | Does **not** overlap static addresses |
| NAT gateway | Check **NAT Settings** | Often `.2`, but environment-specific |
| DNS | Use an actual working resolver | Do not assume the NAT gateway is a DNS resolver |

> [!WARNING]
> The PDF's DHCP range **`172.25.250.5`–`172.25.250.100` overlaps all three proposed static IPs**. Change the range or make explicit DHCP exclusions/reservations before starting guests. Likewise, changing VMnet8 may affect other existing VMs that use it.

> [!NOTE]
> **NAT** permits some traffic between the lab and other networks and is not equivalent to a disconnected, isolated lab. If you need an air-gapped exercise, use a suitable isolated virtual network and arrange offline installation media; that configuration is different from the PDF.

## 2.3 Create three VMs

Choose **File → New Virtual Machine**, select the RHEL 10 64-bit guest template if available, allocate resources and connect each guest's network adapter to **VMnet8 (NAT)**. Attach the RHEL DVD ISO through the virtual CD/DVD device.

| VM label | Required hostname | Static IPv4 | vCPU | RAM | Disk |
| --- | --- | --- | --- | --- | --- |
| Workstation | `workstation.lab.example.com` | `172.25.250.9/24` | 4 | 4–8 GiB | 64 GiB |
| Server01 | `servera.lab.example.com` | `172.25.250.10/24` | 2 | 2–4 GiB | 24 GiB |
| Server02 | `serverb.lab.example.com` | `172.25.250.11/24` | 2 | 2–4 GiB | 24 GiB |

**Handout discrepancies:** The initial table says Workstation **64 GB**, whereas a later build step says **32 GB**. Server B is initially listed as **34 GB**, but subsequent instructions say **24 GB for both managed hosts**. The values here are suggested, internally consistent lab sizes—not official Red Hat system requirements. Increase disk/RAM if your host resources permit.

**VM folders:** Creating an optional VMware library folder such as `RHCE-RH294-Lab` makes the machines easier to identify. USB/audio hardware may be removed if unused; these changes are optional and version-dependent.

## 2.4 Install RHEL on all three machines

1. Boot the appropriate RHEL 10 DVD ISO and choose **Install Red Hat Enterprise Linux** (label varies).
2. Choose language, time zone (the PDF uses `Asia/Kolkata`), installation disk and software profile. Workstation can use a GUI; managed hosts can use a minimal/server installation with `sshd` and Python available.
3. Set the respective hostnames and static IPs. Set the gateway to the **verified** VMware NAT gateway; configure DNS only if you know its functioning address.
4. Create the `student` user with an administrator/sudo-capable role. Use **unique strong passwords**, preferably SSH keys for regular automation.
5. **Do not enable direct root SSH login** just to make Ansible work. Remote `student` + `sudo` is safer. Finish installation and reboot.

> [!WARNING]
> The PDF demonstrates root password `redhat`, student password `student`, and enabling root SSH. Those values are weak and unsafe outside a disposable demonstration. This guide deliberately replaces them with unique credentials and privilege escalation.

## 2.5 Confirm each VM before moving on

```bash
hostnamectl status
ip -brief address
ip route
nmcli device status
systemctl is-active sshd
```

At the end, you should have three bootable RHEL systems on one virtual network, with unique static addresses and no DHCP collisions. The next chapter shows how to correct Linux networking after installation.

**Official references:** [Broadcom official VMware download guide](https://knowledge.broadcom.com/external/article/368667/download-and-license-free-vmware-desktop.html), [RHEL 10 documentation](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10).
