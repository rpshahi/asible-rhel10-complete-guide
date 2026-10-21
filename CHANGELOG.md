# Technical audit, major corrections and improvements

[← Home](README.md)

**Primary source:** `Day1_10-Oct-2026-AU294.pdf`, 10 pages.  
**Review date:** 21 October 2026.  
**Method:** Page-by-page extraction and visual review of the embedded VMware/RHEL screenshots; cross-check of relevant commands, supported products, security choices and EX294 learning objectives against the official references below.

## Important technical corrections

| PDF page(s) | Original note or gap | Correction in this edition | Why it matters |
| --- | --- | --- | --- |
| 1 | Directs students to a third-party VMware Workstation download link; names `26H1u1` as if that particular build were required | Use [Broadcom's official download portal/instructions](https://knowledge.broadcom.com/external/article/368667/download-and-license-free-vmware-desktop.html) and an appropriate supported build | Avoid unverified installers and release-specific assumptions |
| 1–2 | VMnet8 DHCP pool `.5`–`.100`, but static VMs are `.9`, `.10`, `.11` | Set a nonoverlapping pool, e.g. `.100`–`.199`, or explicitly reserve/exclude static addresses | Prevent duplicate-IP leases and intermittent lab failures |
| 1–3 | Claims VMware NAT changes make lab "closed to Red Hat" | Clarify that NAT is **not** an air gap and can support outbound access | Correct network security concept |
| 2–3 | Workstation disk is 64 GB in VM table and 32 GB later; Server B disk 34 GB in table and 24 GB later | Provide internally consistent **suggested** disk sizes, identify both source discrepancies | Eliminate ambiguous VM-sizing instructions |
| 3 | Root password `redhat`, student password `student`, direct root SSH allowed | Replace with unique passwords, non-root SSH, SSH keys and sudo | Reduce credential and remote-login risk |
| 3 | "Log in once as root to create the root user's home profile" | Explain that this is unnecessary; use `sudo -i` when justified | Avoid unsupported rationale for root login |
| 3–4 | Shows a connection/device `ens18` and then configures `ens160` | Inspect actual interface and connection profile before using `nmcli`; prefer modifying the correct existing profile | Prevent configuring a nonexistent NIC or competing profiles |
| 3–4 | Assumes `172.25.250.2` is both gateway and DNS server | Require confirmation of gateway and **working** DNS; `/etc/hosts` works for offline lab names | NAT gateway/DNS details depend on virtual networking |
| 5–6 | Uses `lsblk`, `df` and `/dev/sr0` without clearly distinguishing device detection, mounting and DVD variant | Check `lsblk`, `findmnt`, and BaseOS/AppStream on the **full binary DVD** | Avoid misleading verification of a missing mount or boot ISO |
| 6 | Local repository uses `gpgcheck=0` for BaseOS and AppStream | Use `gpgcheck=1` and the installed official Red Hat RPM GPG key | Keep RPM package integrity verification |
| 6–7 | `dnf repolist` is treated as sufficient repository validation | Check `makecache` and availability of a real package with only DVD repos enabled | An enabled repo entry need not be usable |
| 6 | Unconditionally appends DVD mount to `/etc/fstab` as a routine step | Make optional; verify optical device, preserve file integrity and run `findmnt --verify` | Missing/ejected media can disrupt expected availability |
| 7–8 | Suggests `dnf install ansible-navigator` using only RHEL DVD as a possibility | Separate the RHEL-provided `ansible-core` path from version/entitlement-specific AAP Navigator packaging | RHEL DVD ≠ AAP repository |
| 7–8 | Shows system-level `pip install` as root, then proposes a root-owned virtual environment | Use a **non-root user-owned venv**, and acknowledge online/local Python package source requirements | Avoid modifying the system-managed Python environment |
| 8 | Navigator may pull an EE, but run mode isn't distinguished | Explain EE/container-image prerequisites and provide explicitly labeled **local learning-only** config | Avoid confusing native Ansible with a containerized AAP runtime |
| 9 | Uses `pip list` without matching the Python environment | Use `python -m pip` inside the intended virtual environment | Correctly attribute the package to its interpreter |
| 9 | `ansible localhost --list-hosts` | Explain that it displays matches, **not connectivity**; add explicit inventory and SSH tests | Prevent false-positive connectivity assumptions |
| 10 | `ansible localhost -m command -a "sudo useradd user1"` | Use `ansible.builtin.user` with Ansible's `--become` | Idempotent state management and proper privilege escalation |
| 10 | Verifies using a `sudo grep` invocation within the `command` module | Use `ansible.builtin.command` with `id`/`getent` (or fact-oriented modules) | Cleaner verification without unnecessary sudo/limited NSS checks |
| 8–9 | Screen sharing Full Screen and Spotlight steps embedded among Ansible topics | Retain in a clearly identified **classroom visibility** subsection | Preserve original information without confusing it with Ansible |

## Added instructional content (absent from the PDF)

1. Ansible architecture, core vocabulary and idempotence.
2. Consistent inventory groups, `ansible.cfg`, SSH key distribution, sudo and host-key safety.
3. Functional module-based ad hoc commands and a first complete playbook.
4. Syntax checking, check/diff mode, rerun verification, facts, handlers, templates and service automation.
5. An EX294 study roadmap covering variables, loops, conditions, registered results, error handling, roles, collections, Vault, system administration modules, Git, VS Code and execution environments.
6. Troubleshooting reference, beginner exercises, validation checklist, and directly reusable example files.

## Source-to-chapter coverage map

| Source PDF pages | Material retained in | Notes |
| --- | --- | --- |
| 1–2 | [Chapter 2](docs/02-vmware-rhel10-lab.md) | Download, Virtual Network Editor, DHCP screenshot, VM organization |
| 2–3 | [Chapter 2](docs/02-vmware-rhel10-lab.md) | VM specs, creating VMs, RHEL 10 installation and user setup |
| 3–4 | [Chapter 3](docs/03-linux-and-networking.md) | Hostname, `nmcli` and static networking |
| 5–7 | [Chapter 4](docs/04-offline-dnf-repositories.md) | Optical media, local repo, `dnf`, `ansible-core` |
| 7–9 | [Chapter 5](docs/05-ansible-installation-and-navigator.md) | Navigator installation, Python/pip checks, EE awareness, screen viewing tips |
| 9–10 | [Chapter 6](docs/06-inventory-ssh-ad-hoc.md) | localhost matching, user creation and verification |
| Not present in PDF | [Chapters 1, 7–9](README.md#contents) | Fundamentals, playbook examples, official exam skills, exercises |

## Version and validation caveats

- The source specifies **RHEL 10.0**. Installation screens and VMware UI may change across host and guest versions.
- Red Hat documents RHEL 10's supplied `ansible-core` as 2.16 in its system-roles context, but other installed AAP/Community versions can differ.
- Red Hat's AAP **2.6** documentation demonstrates tools and EE workflows on supported RHEL versions; an unspecified RHEL 10 AAP Navigator RPM path was **not invented**.
- Example YAML syntax is checked locally. The repository was **not validated by running against the user's three VMs**; network, ISO, subscriptions, packages and SSH permissions must be verified in the actual lab.
- The roadmap summarizes publicly listed objectives; a version-specific EX294 offering can differ. Exam eligibility and exam conditions are set by Red Hat.

## Official sources used in technical review

- [EX294 objectives and format](https://www.redhat.com/en/services/training/ex294-red-hat-certified-engineer-rhce-exam-red-hat-enterprise-linux)
- [RHEL 10 Ansible system-roles architecture](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/automating_system_administration_by_using_rhel_system_roles/introduction-to-rhel-system-roles)
- [RHEL 10 preparing control/managed nodes](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/automating_system_administration_by_using_rhel_system_roles/preparing-a-control-node-and-managed-nodes-to-use-rhel-system-roles)
- [AAP 2.6 Navigator concepts](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/develop-con_about_ansible_navigator)
- [AAP 2.6 development tool workflow](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/develop-ref_devtools_components)
- [Navigator configuration reference](https://docs.ansible.com/projects/navigator/settings/)
- [Ansible Inventory](https://docs.ansible.com/projects/ansible/latest/getting_started/get_started_inventory.html)
- [Ansible user module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/user_module.html)
- [Ansible command module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/command_module.html)
- [Ansible Vault](https://docs.ansible.com/projects/ansible/latest/vault_guide/vault.html)
- [Broadcom official VMware desktop hypervisor download](https://knowledge.broadcom.com/external/article/368667/download-and-license-free-vmware-desktop.html)
