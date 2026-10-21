# 9. Practice labs, checkpoints and troubleshooting

[← Home](../README.md) · [← Exam roadmap](08-ex294-roadmap.md)

**Origin:** Additional classroom and self-study exercises based on the prepared three-VM lab.

## 9.1 Guided exercises

These can be performed sequentially. Use the virtual-machine console for any network operations that may interrupt SSH.

| Exercise | Task | Success criterion |
| --- | --- | --- |
| 1 — Network | Configure static `.9`, `.10`, `.11` without DHCP conflict | All three hosts have unique expected addresses; `ip route` is correct |
| 2 — Hostnames | Populate `/etc/hosts` or working DNS | `getent hosts servera.lab.example.com` resolves correctly |
| 3 — Software | Mount DVD and create two signed DNF repos | `dnf makecache` succeeds with `gpgcheck=1` |
| 4 — Remote access | Set up student SSH key + sudo | SSH key login and an Ansible `-b -K` test pass |
| 5 — Inventory | Add `control`, `webservers`, `dbservers`, `managed` | `ansible-inventory --graph` shows all nodes/groups |
| 6 — Ad hoc | Run `ansible.builtin.ping` and `ansible.builtin.user` | `user1` exists on both managed nodes |
| 7 — Playbook | Run `create-user.yml` twice | Second execution reports no unnecessary user change |
| 8 — Web automation | Apply `webserver.yml` on Server A | Apache is enabled/running and `/` serves the template |
| 9 — Reusability | Extract web-server tasks to a role | New role performs the same task from a clean snapshot |
| 10 — Security | Create an encrypted variable with Vault | No plaintext password committed; decrypt only when executing |

## 9.2 Troubleshooting quick reference

| Symptom | Likely cause | Checks and corrective action |
| --- | --- | --- |
| `UNREACHABLE` | SSH/network/hostname/firewall/host key problem | `ssh -vv student@servera...`, `ip route`, `getent hosts`, verify SSH fingerprint |
| `Missing sudo password` | Privilege escalation not configured/provided | Check `sudo -l`; use `-b -K` or `become: true` with `--ask-become-pass` |
| Ansible `ping` fails but ICMP succeeds | SSH key, Python or user permissions | Test manual SSH; confirm Python on managed host |
| Changed wrong IP or lost SSH | Incorrect NetworkManager profile/adapter | Recover via VM console; inspect `nmcli device status` and active profile |
| `Could not resolve host` | No DNS or `/etc/hosts` entry | `getent hosts`; add correct entries; verify inventory `ansible_host` |
| Cannot install package from DVD | No DVD, wrong ISO, repo path mismatch, key/repo problem | `lsblk`, `findmnt`, BaseOS/AppStream paths, `dnf makecache` |
| Navigator command not found | Different Python environment or not installed | Activate intended venv, `command -v`, `python -m pip show` |
| Navigator tries to download EE | EE enabled with missing image | Use supported registry/EE, or disable EE **for the basic local lab only** |
| `ERROR! ... YAML` | Bad indentation or malformed mapping | `ansible-playbook --syntax-check`, review spaces and colons |
| Module not found | External collection absent | `ansible-doc FQCN`; install correct compatible collection |
| Apache service works locally but not remotely | Firewall, route or service binding | Check `systemctl`, localhost `curl`, `firewall-cmd --list-services` |
| Second run changes every time | Non-idempotent command, unstable template or task definition | Prefer state modules; compare diffs and `changed_when` choices |

## 9.3 Validation checklist

- [ ] Three VMs boot successfully and use the expected private subnet.
- [ ] Static addresses are **outside** VMware's DHCP pool.
- [ ] Hostname resolution works or inventory IPs are correct.
- [ ] Managed systems accept SSH key login for a non-root user.
- [ ] `sudo` is functional without direct root SSH access.
- [ ] The full DVD ISO is attached and both repository directories exist.
- [ ] RPM GPG verification remains enabled.
- [ ] `ansible --version` and `ansible-inventory --graph` work.
- [ ] `ansible managed -m ansible.builtin.ping` succeeds.
- [ ] The training-account playbook completes and is safe to rerun.
- [ ] Secrets/private SSH keys are absent from Git commits.
- [ ] At least one complete playbook is tested after a VM reboot.

## 9.4 Self-test questions

1. Does `ansible --list-hosts` check whether SSH works? **No**, it enumerates selected inventory names.
2. Why use `ansible.builtin.user` instead of `useradd`? **Desired-state behavior and idempotence**.
3. What does `become: true` do? **Activates privilege escalation**, usually with sudo; it does not enable root SSH.
4. Why is `.5`–`.100` a bad DHCP pool for this lab? **It overlaps static `.9`–`.11`**.
5. Does mounting a RHEL DVD install Ansible Navigator? **No.** The DVD provides its included RPM content, not arbitrary AAP/PyPI tools.
6. Can RHEL DVD repos provide updates indefinitely? **No.** Their content is fixed to the ISO's build.
7. Can `ansible.builtin.command` interpret `|` as a shell pipe? **No.** Use an appropriate module or `ansible.builtin.shell` when necessary.
8. Is a passing first playbook run proof of idempotence? **No.** Verify rerun behavior and resulting system state.

**Next:** Repeat the exercises from a clean snapshot, keep a log of any changed values, and compare your results against the current official [EX294 objectives](https://www.redhat.com/en/services/training/ex294-red-hat-certified-engineer-rhce-exam-red-hat-enterprise-linux).
