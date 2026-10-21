# Publish this project on GitHub (@rpshahi)

The proposed dedicated repository is `rpshahi/rhce-ansible-rhel10-lab-guide`.

## Option A — easiest: GitHub web upload

1. Sign in at <https://github.com/> and open <https://github.com/new>.
2. Choose owner **rpshahi**, repository name **rhce-ansible-rhel10-lab-guide**, and visibility **Public**.
3. Enter the project description from the response or the README. **Do not initialize with another README, .gitignore, or license**, because those files are already in the upload folder (license intentionally omitted until rights are confirmed).
4. Click **Create repository**.
5. Extract the publishing ZIP. Open the folder **rhce-ansible-rhel10-lab-guide** on your PC.
6. In the new repository, choose **Add file → Upload files**. Drag the **contents inside** the extracted project folder into the upload area, including `README.md`, `.gitignore`, `.github`, `docs`, `examples`, `assets`, and remaining `.md` files. **Do not upload only the ZIP** or nest the entire project folder one level too deep.
7. Use commit message `docs: publish RHCE Ansible RHEL 10 study guide` and commit the files. Depending on GitHub's UI, you may be asked to propose changes via a pull request.
8. The README should automatically display at the repository root. Test links to the chapter docs and sample files.

If your browser does not handle multiple folders cleanly, use Option B (Git) below.

## Option B — Git CLI on Windows (Git Bash or PowerShell)

First create an **empty public** repository as in steps 1–4 above, then install Git if needed. Open a terminal **inside the extracted project folder**, not in its parent directory.

```bash
git init -b main
git add .
git commit -m "docs: publish RHCE Ansible RHEL 10 study guide"
git remote add origin https://github.com/rpshahi/rhce-ansible-rhel10-lab-guide.git
git push -u origin main
```

Git will require authentication; use a browser sign-in via Git Credential Manager or another supported GitHub authentication method. Do **not** type a GitHub account password into a Git terminal prompt for HTTPS.

For subsequent edits:

```bash
git add .
git commit -m "docs: improve networking chapter"
git push
```

## Option C — store as a chapter within existing `ansible_learn`

Existing repo: <https://github.com/rpshahi/ansible_learn>. It already contains an `AGPL-3.0` license, `README.md`, `ansible.cfg`, and `inventory`.

1. **Do not replace** its existing root `README.md`, `ansible.cfg`, inventory, or LICENSE.
2. Clone it locally: `git clone https://github.com/rpshahi/ansible_learn.git`.
3. Copy the entire publishing folder as a new subdirectory named `rhce-day1-guide` inside the clone.
4. Add to its existing root `README.md`:

   ```markdown
   ## RHCE study guide
   [Explore RHEL 10 / Ansible Day 1 labs and EX294 study notes](rhce-day1-guide/README.md)
   ```

5. Run `git add .`, `git commit -m "docs: add RHCE Day 1 lab guide"`, and `git push origin main` from the clone.
6. Check the pre-existing AGPL-3.0 terms and your rights to share the imported classroom-derived content.

## Make the repository discoverable

On your repository homepage, click the gear icon near **About**, and set:

- **Description:** `RHCE EX294 study notes and hands-on Ansible labs for RHEL 10, covering VMware setup, Linux networking, inventories, playbooks, automation, and troubleshooting.`
- **Website:** optional (do not invent a documentation website).
- **Topics:** `ansible`, `rhce`, `ex294`, `rhel`, `rhel10`, `linux-automation`, `ansible-playbook`, `devops`, `red-hat`, `homelab`.

Under **Settings → Social preview**, upload `assets/social-preview.png` (it should appear when the repository URL is shared on supported platforms).

Visit <https://github.com/rpshahi> and use **Customize your pins** to select this new repository. Optionally add a link to it in a profile README by creating a separate public repository called `rpshahi` and adding a root `README.md`.

## Before sharing widely

- Check the README and documentation links work.
- Ensure none of the inventory values refer to production servers or contain secrets.
- Confirm publishing rights before adding a license or any vendor/course screenshots.
- Avoid claiming official Red Hat endorsement, RHCE certification, or exam guarantees.
