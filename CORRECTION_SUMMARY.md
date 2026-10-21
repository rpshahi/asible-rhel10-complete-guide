# Major Corrections — Quick Summary

[← Home](README.md) · [Full page-by-page audit](CHANGELOG.md)

**Reviewed:** 21 October 2026 against the 10-page *Day1_10-Oct-2026-AU294* PDF and current official Red Hat/Ansible documentation.

1. **Fix IP conflicts:** the original VMware DHCP pool `.5`–`.100` conflicts with the proposed static guest addresses `.9`–`.11`. Use a nonoverlapping range such as `.100`–`.199`.
2. **Use official downloads:** replace the third-party VMware installer link with Broadcom's official download guidance; use a verified full RHEL binary DVD ISO from Red Hat.
3. **Protect accounts and access:** do not reuse `redhat`/`student` demonstration passwords or enable direct root SSH; use a non-root SSH user with key authentication and authorized sudo.
4. **Correct network commands:** the PDF mixes `ens18` and `ens160`; inspect the actual VMware guest NIC and NetworkManager profile. Confirm the gateway/DNS instead of assuming `.2` serves both roles.
5. **Verify RPM signatures:** change DVD `gpgcheck=0` to `gpgcheck=1` with the Red Hat release RPM key. Validate metadata with `dnf makecache`.
6. **Separate products:** RHEL `ansible-core`, separately installed `ansible-navigator`, and Red Hat Ansible Automation Platform are not the same install. A RHEL DVD does not automatically include Navigator or an execution-environment image.
7. **Avoid system-wide pip:** for community learning, install Navigator in a non-root virtual environment and make its online/offline package requirements explicit.
8. **Use idempotent modules:** replace `ansible -m command -a "sudo useradd user1"` with `ansible.builtin.user`, plus Ansible `--become`/`--ask-become-pass`.
9. **Clarify validation:** `ansible localhost --list-hosts` checks inventory matching, not host connectivity; use Ansible `ping`, SSH checks and second-run validation.
10. **Expand missing exam skills:** add complete inventories, YAML playbooks, templates, facts, roles, collections, Vault, Git, Navigator, execution environments, validation and RHCE practice tasks.

**Not verified in a live VM:** runtime behavior of commands in the user's actual VMware/RHEL environment. The repository's standalone example YAML was parsed locally and documentation links were checked for local path consistency; exam version and product dependencies still require real-world verification.
