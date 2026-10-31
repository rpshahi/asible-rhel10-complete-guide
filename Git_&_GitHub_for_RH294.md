# Git & GitHub for RHCE / RH294
## Practical Learning Guide: Beginner to Advanced

**Learning objective:** Learn the essential Git and GitHub commands required to manage Ansible automation projects effectively.

**Reference:** RHCE / RH294
**Environment:** Red Hat Enterprise Linux (RHEL), Linux terminal, Git, GitHub
**Approach:** Learn → Understand → Practice → Apply

# Table of Contents
1. Introduction to Git and GitHub
2. Git installation and configuration
3. Understanding Git fundamentals
4. Creating your first Git repository
5. Essential Git commands
6. Git file-tracking lifecycle
7. Working with commits
8. Branching and merging
9. Working with GitHub
10. Connecting local Git with GitHub
11. Push, pull, fetch and clone
12. Git collaboration workflow
13. Undoing mistakes and recovering files
14. Working with Ansible projects
15. Practical RHCE scenarios
16. Essential Git command cheat sheet
17. Hands-on exercises
18. Recommended learning sequence
# 1. Introduction to Git and GitHub
## **What is Git?**

Git is a distributed version control system used to track changes in files and directories.

It helps you:

- Track changes in Ansible playbooks.
- Maintain different versions of configuration files.
- Restore previous versions.
- Work with branches.
- Collaborate with other administrators.
### **What is GitHub?**

GitHub is an online platform that hosts Git repositories and provides collaboration features.

|**Git**	|**GitHub**|
|-------------|----------|
Version control tool	| Online repository hosting platform|
Works locally	| Works over a network
Tracks file changes | Hosts and shares repositories
Supports branches | Supports pull requests
Uses commits | Provides collaboration and review

**Remember:** Git is the version control tool. GitHub is a hosting and collaboration platform.

## 2. Git Installation and Configuration
**Step 1: Install Git on RHEL**
``` bash
sudo dnf install git -y
```
**Step 2: Verify installation**
```bash
git --version
```
**Step 3: Configure your username**
```bash
git config --global user.name "Your Name"
```
**Step 4: Configure your email**
```bash
git config --global user.email "your-email@example.com"
```

Use an email associated with your GitHub account if you want your commits associated with that account.

**Step 5: Verify configuration**
```bash
git config --global --list
```
**Important configuration commands**
```bash
git config --list
git config --global user.name
git config --global user.email
git config --global --edit
```
## 3. Understanding Git Fundamentals

# Understand these concepts before executing Git commands.

|Term	| Meaning|
|------|--------|
|Repository | Project tracked by Git|
|Working directory | Directory where you modify files|
|Staging area	| Area containing changes prepared for commit|
|Commit | Saved snapshot of project changes|
|Branch | Independent line of development|
|Merge	| Combines changes from branches|
|Remote | Another copy of a repository, often hosted online|
|Clone	| Creates a local copy of a repository|
|Push	| Sends local commits to a remote|
|Pull	| Fetches and integrates remote changes|
**Git workflow**

```text
Working Directory
       |
       | git add
       v
Staging Area
       |
       | git commit
       v
Local Repository
       |
       | git push
       v
Remote Repository (GitHub)
```
## 4. Creating Your First Git Repository
**Step 1: Create a project directory**
```bash
mkdir ansible-project
cd ansible-project
```
**Step 2: Initialize Git**
```bash
git init
```

This creates a local Git repository.

**Step 3: Verify repository status**
```bash
git status
```
**Step 4: Create a sample Ansible playbook**
```bash
vim install_apache.yml
```
Example content:

```yaml
---
- name: Install Apache web server
  hosts: webservers
  become: true

  tasks:
    - name: Install Apache
      ansible.builtin.dnf:
        name: httpd
        state: present
```
**Step 5: Check the repository***
```bash
git status
```

Git should show the new file as untracked.

## 5. Essential Git Commands

These are the commands you should learn first.

|Command	| Purpose|
|-------------|---------|
|git init	| Initialize a repository|
|git status	| Check repository status|
|git add	| Stage changes|
|git commit	| Save staged changes|
|git log	| View commit history|
|git diff	| View unstaged changes|
|git diff --staged	| View staged changes|
|git show	| Inspect a commit|
|git rm	| Remove a tracked file|
|git mv	| Rename or move a tracked file|
**Examples**

Stage one file:

```bash
git add install_apache.yml
```

Stage all changes in the current directory:

```bash
git add .
```

Commit changes:

```bash
git commit -m "Add Apache installation playbook"
```

Check status:

```bash
git status
```

View commit history:

```bash
git log
```

View compact commit history:

```bash
git log --oneline
```

View file differences:

```bash
git diff
```

View staged differences:

```bash
git diff --staged
```
## 6. Git File-Tracking Lifecycle

Git tracks changes through different stages.

**Stage 1: Untracked**

A new file exists, but Git is not tracking it.

```bash
git status
```
**Stage 2: Staged**

Prepare a file for committing.

```bash
git add install_apache.yml
```
**Stage 3: Committed**

Save the staged changes.

```bash
git commit -m "Create Apache playbook"
```
**Stage 4: Modified**

Modify an existing tracked file.

```bash
vim install_apache.yml
```

Check what changed:

```bash
git diff
```

Stage and commit the modification:

```bash
git add install_apache.yml
git commit -m "Update Apache installation tasks"
```
## 7. Working with Commits

A commit represents a snapshot of your project.

**Essential commands**
```bash
git log
git log --oneline
git show HEAD
git show HEAD~1
```
**Understand HEAD**

HEAD represents the currently checked-out commit.

**View the latest commit**
```bash
git show HEAD
```
**View the previous commit**
```bash
git show HEAD~1
```
**Amend the latest commit**

Use this when you need to correct the most recent commit.

```bash
git commit --amend -m "Correct commit message"
```

> **Caution:** Avoid amending commits already shared with other users unless you understand the consequences of rewriting history.

## 8. Branching and Merging

Branches allow you to develop changes independently.

Example: You have a working Apache playbook and want to create a separate branch to add firewall configuration.

**List branches**
```bash
git branch
```
**Create a branch**
```bash
git branch firewall-config
```
**Switch to a branch**
```bash
git switch firewall-config
```
**Create and switch to a branch**
```bash
git switch -c firewall-config
```
**View current branch**
```bash
git branch --show-current
```
**Merge a branch**

First switch to the destination branch:

```bash
git switch main
```

Merge the changes:

```bash
git merge firewall-config
```
**Delete a merged branch**
```bash
git branch -d firewall-config
```
Important concepts
- main: Main development branch.
- Feature branch: Used for developing a particular feature.
- Merge: Combines branch changes.
- Merge conflict: Occurs when Git cannot automatically reconcile conflicting changes.
## 9. Working with GitHub

GitHub allows you to store and share your repositories.

**Step 1: Create a GitHub account**

Visit:

https://github.com

**Step 2: Create a repository**

Example repository name:

```text
rhce-ansible-project
```

For your first exercise, create an empty repository without automatically adding a README or .gitignore file.

**Step 3: Understand repository URLs**

HTTPS example:

```text
https://github.com/USERNAME/rhce-ansible-project.git
```

SSH example:

```text
git@github.com:USERNAME/rhce-ansible-project.git
```
**HTTPS authentication**

GitHub requires a supported authentication method. Account passwords are not accepted for Git operations over HTTPS.

You can use a personal access token or Git Credential Manager.

**SSH authentication**

SSH keys provide an alternative authentication method.

Generate an SSH key:

```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
```

Start the SSH agent:

```bash
eval "$(ssh-agent -s)"
```

Add your private key:

```bash
ssh-add ~/.ssh/id_ed25519
```

Display your public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the public key into:

GitHub → Settings → SSH and GPG keys → New SSH key.

Test the connection:

```bash
ssh -T git@github.com
```

**Security:** Never upload or share your private SSH key (id_ed25519). Only the public key (id_ed25519.pub) should be added to GitHub.

## 10. Connecting Local Git with GitHub

Assume you already created your local repository.

**Step 1: Check the current branch**
```bash
git branch
```

If necessary, rename the branch:

```bash
git branch -M main
```
**Step 2: Add GitHub as a remote**
```bash
git remote add origin git@github.com:USERNAME/rhce-ansible-project.git
```
**Step 3: Verify the remote**
```bash
git remote -v
```
**Step 4: Push your project**
```bash
git push -u origin main
```

The `-u` option establishes the upstream relationship, making subsequent pushes simpler.

After the first push:

```bash
git push
```
## 11. Clone, Push, Pull and Fetch

These commands are essential when working with remote repositories.

**Clone**

Download an existing repository.

```bash
git clone https://github.com/USERNAME/rhce-ansible-project.git
```
**Fetch**

Download remote changes without integrating them into your current branch.

```bash
git fetch origin
```
**Pull**

Fetch and integrate changes from the remote branch.

```bash
git pull origin main
```
**Push**

Upload local commits to GitHub.

```bash
git push origin main
```
Compare the commands
|Command	|Function|
|-------------|---------
|git clone	|Copies a remote repository locally
|git fetch	|Downloads remote updates
|git pull	|Downloads and integrates updates
|git push	|Uploads local commits

## 12. Git Collaboration Workflow

A typical team workflow:

1. Clone the repository.
2. Create a feature branch.
3. Modify the Ansible playbook.
4. Stage the changes.
5. Commit the changes.
6. Push the feature branch.
7. Create a pull request on GitHub.
8. Review and merge the pull request.
**Example**
```bash
git clone git@github.com:USERNAME/rhce-ansible-project.git

cd rhce-ansible-project

git switch -c add-firewall-playbook

vim firewall.yml

git status

git add firewall.yml

git commit -m "Add firewall configuration playbook"

git push -u origin add-firewall-playbook
```

Then open GitHub and create a pull request.

## 13. Undoing Mistakes and Recovering Files

These commands are important for safely managing accidental changes.

**Discard unstaged changes in a file**
```bash
git restore install_apache.yml
```

This discards uncommitted modifications in that file.

**Unstage a file**
```bash
git restore --staged install_apache.yml
```

This removes the file from the staging area but keeps your modifications.

**Revert a commit**
```bash
git revert COMMIT_ID
```

Creates a new commit that reverses the changes introduced by the specified commit.

**Reset the latest commit but keep changes staged**
```bash
git reset --soft HEAD~1
```
**Reset the latest commit and unstage changes**
```bash
git reset HEAD~1
```
**Important warning**
```bash
git reset --hard HEAD
```

This discards tracked working-directory and staging changes. Use it only when you are certain you no longer need those changes.

Prefer git revert for reversing commits already shared with a team.

## 14. Working with Ansible Projects

This section connects Git knowledge with RH294 practical work.

**Recommended project structure**
```text
rhce-ansible-project/
│
├── ansible.cfg
├── inventory/
│   └── hosts
│
├── playbooks/
│   ├── install_apache.yml
│   ├── users.yml
│   └── firewall.yml
│
├── roles/
│   └── webserver/
│
├── group_vars/
│   └── all.yml
│
├── host_vars/
│
├── README.md
└── .gitignore
```
Create the directory structure
```bash
mkdir -p rhce-ansible-project/{inventory,playbooks,roles,group_vars,host_vars}

cd rhce-ansible-project
```
**Initialize Git**
```bash
git init
```
**Create a `.gitignore` file**
```bash
vim .gitignore
```

Example:

```text
*.retry
*.log
*.swp
*.tmp
.vault_pass
__pycache__/
```

Do not commit passwords, private keys, vault password files, or other secrets.

**Track Ansible project files**
```bash
git add .
```
**Commit the initial project**
```bash
git commit -m "Create initial Ansible project structure"
```
**Practical workflow**

Whenever you modify your Ansible automation:

```bash
git status

git diff

git add .

git diff --staged

git commit -m "Update Ansible automation"

git push
```

This workflow helps you maintain a history of your Ansible automation changes.

## 15. Practical RHCE Scenarios
**Scenario 1: Create an Ansible project and track it**
```bash
mkdir apache-automation
cd apache-automation

git init

vim install_apache.yml

git add install_apache.yml

git commit -m "Add Apache installation playbook"
```
**Scenario 2: Update an existing playbook**
```bash
vim install_apache.yml

git diff

git add install_apache.yml

git commit -m "Update Apache configuration"
```
**Scenario 3: Create a separate branch for firewall automation**
```bash
git switch -c firewall-automation

vim firewall.yml

git add firewall.yml

git commit -m "Add firewall automation"

git switch main

git merge firewall-automation
```
**Scenario 4: Upload your project to GitHub**
```bash
git remote add origin git@github.com:USERNAME/rhce-ansible-project.git

git push -u origin main
```
**Scenario 5: Download an existing Ansible project**
```bash
git clone git@github.com:USERNAME/rhce-ansible-project.git

cd rhce-ansible-project
```
## 16. Essential Git Command Cheat Sheet
|Command	|Purpose
|-------------|-------
|git --version	|Check Git version
|git config --global --list	|View configuration
|git init	|Initialize repository
|git status	|Check status
|git add FILE	|Stage a file
|git add .	|Stage current-directory changes
|git commit -m "MESSAGE"	|Create commit
|git log --oneline	|View commit history
|git show	|Inspect commit
|git diff	|View unstaged changes
|git diff --staged	|View staged changes
|git branch	|List branches
|git switch BRANCH	|Switch branches
|git switch -c BRANCH	|Create and switch branch
|git merge BRANCH	|Merge branch
|git branch -d BRANCH	|Delete merged branch
|git remote -v	|View remotes
|git remote add origin |URL	Add remote
|git clone URL	|Clone repository
|git fetch origin	|Fetch remote updates
|git pull	|Pull remote changes
|git push	|Push local commits
|git restore FILE	|Discard unstaged changes
|git restore --staged FILE	|Unstage a file
|git revert COMMIT_ID	|Reverse a commit
## 17. Hands-on Exercises

Complete these exercises in order.

**Exercise 1: Git fundamentals**
- [ ] Install Git on RHEL.
- [ ] Configure username and email.
- [ ] Create an Ansible project directory.
- [ ] Initialize a Git repository.
- [ ] Create a sample playbook.
- [ ] Stage and commit the playbook.
- [ ] Check commit history.

**Exercise 2: File tracking**
- [ ] Modify an existing playbook.
- [ ] Use git diff.
- [ ] Stage the changes.
- [ ] Use git diff --staged.
- [ ] Commit the changes.

**Exercise 3: Branch management**
- [ ] Create a feature branch.
- [ ] Switch between branches.
- [ ] Add a new Ansible playbook.
- [ ] Commit your changes.
- [ ] Merge the feature branch into main.

**Exercise 4: GitHub integration**
- [ ] Create a GitHub account.
- [ ] Create a repository.
- [ ] Configure SSH authentication.
- [ ] Connect your local repository.
- [ ] Push your project to GitHub.
- [ ] Clone the repository into another directory.

**Exercise 5: Real-world Ansible workflow**
- [ ] Create an Ansible project.
- [ ] Track inventory and playbook files.
- [ ] Create a feature branch.
- [ ] Modify an Ansible role.
- [ ] Commit the changes.
- [ ] Push the branch to GitHub.
- [ ] Create a pull request.
- [ ] Merge the changes.
## 18. Recommended Learning Sequence
|Phase	|Topics	|Expected result
|------|-------------|---------------
1	|Installation and configuration	|Git ready on RHEL
2	|Repository fundamentals	|Understand Git workflow
3	|Add, status and commit	|Track Ansible files
4	|Log and diff	|Inspect project history
5	|Branch and merge	|Manage independent changes
6	|GitHub and SSH	|Connect remote repository
7	|Clone, push and pull	|Manage remote projects
8	|Restore and revert	|Recover from mistakes
9	|Ansible integration	|Manage automation projects
10	|Practical exercises	|Apply skills independently
## Final Learning Advice

Do not memorize commands without understanding their purpose.

For every command:

Understand what it does.
Learn its syntax.
Execute it on your RHEL machine.
Observe the output.
Practice it with an Ansible project.

**Our learning priority:** Become comfortable managing Ansible automation files with Git, collaborating through GitHub, and maintaining a reliable version history.

**Important RHCE note:** Prioritize RH294 Ansible objectives and hands-on automation practice. GitHub collaboration is a useful supporting skill, but it should not take time away from core Ansible preparation.
