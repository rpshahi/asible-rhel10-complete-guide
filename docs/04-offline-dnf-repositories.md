# 4. Create a local RHEL DVD repository

[← Home](../README.md) · [← Networking](03-linux-and-networking.md) · [Next: Control-node tools →](05-ansible-installation-and-navigator.md)

**Source coverage:** PDF pages 5–7. This lab section applies when you have a **full RHEL 10 binary DVD ISO** attached to your VM. It is not a general replacement for subscribed, regularly updated Red Hat repositories.

## 4.1 Verify the DVD device and contents

In VMware, ensure the guest's **CD/DVD device is connected** and points to the intended ISO. On Workstation:

```bash
lsblk -f
findmnt /mnt/rhel10dvd || true
sudo mkdir -p /mnt/rhel10dvd
sudo mount -o ro /dev/sr0 /mnt/rhel10dvd
ls -ld /mnt/rhel10dvd/BaseOS /mnt/rhel10dvd/AppStream
findmnt /mnt/rhel10dvd
```

`/dev/sr0` is typical, not guaranteed; check `lsblk` first. If already mounted at `/run/media/...`, you may use that location or unmount/re-mount with care. If the directories are absent, verify you attached the **binary DVD** rather than a boot/minimal ISO.

> [!NOTE]
> `iso9660` is a read-only optical filesystem; that is expected. `df -hT /mnt/rhel10dvd` can show the mounted filesystem, but `findmnt` gives a clearer mount-specific answer.

## 4.2 Create a repository file with signature verification

First verify the official Red Hat RPM key exists:

```bash
ls -l /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```

Create `/etc/yum.repos.d/rhel10dvd.repo` as root (for instance using `sudoedit`) with:

```ini
[rhel10-dvd-baseos]
name=RHEL 10 DVD - BaseOS
baseurl=file:///mnt/rhel10dvd/BaseOS
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release

[rhel10-dvd-appstream]
name=RHEL 10 DVD - AppStream
baseurl=file:///mnt/rhel10dvd/AppStream
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```

**Correction:** The PDF uses `gpgcheck=0`. Keep `gpgcheck=1` for signed Red Hat packages and use the trusted installed key. Never globally disable signature checks to work around an installation error.

## 4.3 Validate repository usability, not just visibility

```bash
sudo dnf clean all
sudo dnf --disablerepo='*' \
  --enablerepo='rhel10-dvd-*' makecache
sudo dnf repolist --enabled
sudo dnf --disablerepo='*' \
  --enablerepo='rhel10-dvd-*' info ansible-core
```

`dnf repolist` shows configured repositories; **it does not by itself prove the DVD is mounted or metadata is readable**. `makecache` and `info` verify the chosen repo more directly.

## 4.4 Install `ansible-core` from available media

If `dnf info ansible-core` confirms it is provided by the configured media:

```bash
sudo dnf --disablerepo='*' \
  --enablerepo='rhel10-dvd-*' install -y ansible-core
ansible --version
```

If no package is available, confirm your RHEL ISO version/content and repository metadata; use an authorized Red Hat package source when necessary. **Do not assume the DVD includes `ansible-navigator` or AAP development tools.**

## 4.5 Mount persistence after reboot (optional)

A VM's virtual CD/DVD must stay **connected**. For a deliberately persistent, attached lab DVD, you may add the following line to `/etc/fstab` after confirming the device path:

```fstab
/dev/sr0  /mnt/rhel10dvd  iso9660  ro,nofail  0  0
```

Then validate with:

```bash
sudo findmnt --verify
sudo mount -a
findmnt /mnt/rhel10dvd
```

This line is **optional** and may lead to unavailable repository media when the virtual DVD is ejected. If you do not keep the ISO connected, prefer mounting it manually when needed. For production automation use a managed, supported repository/mirror rather than relying on a VM optical device.

## 4.6 Useful diagnostics

```bash
rpm -q ansible-core
sudo dnf repolist all
sudo dnf --disablerepo='*' --enablerepo='rhel10-dvd-*' list available ansible-core
ls /mnt/rhel10dvd/{BaseOS,AppStream}/repodata
```

**References:** [Red Hat: RHEL 10 package repositories](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/package_manifest/repositories), [Red Hat: offline installation example with GPG checks](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/install-assembly_disconnected_installation).
