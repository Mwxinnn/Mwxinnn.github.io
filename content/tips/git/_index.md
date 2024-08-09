---
title: 🗒️ Git/GitHub Tips
layout: single
date: 2024-08-08
description: This list contains fundamental Git commands commonly used in everyday programming, covering tasks such as change tracking, branch operations, and remote management.
ignore_sub_pages: true
priority: 5
---
# 
Git is a distributed version control system widely used for tracking changes in source code during software development. Below are the basic commands and concepts to get started with Git:
## Configuration
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

## Basic Commands
After entering the working directory:
### Initialize a Repository
```bash
git init # This creates a new Git repository in the current directory.
```
### Clone a Repository
```bash
git clone <repository_url> # This copies an existing Git repository (e.g. GitHub repo) to your local machine.
```
### Check Status
```bash
git status # This shows the current status of your working directory and staging area.
# Including: which Branch we are at, Changes not staged for commit, and Untracked files.
```
### Add Changes
```bash
git add <file_name> # This adds changes to the staging area. 
git add . # We can use . to add all changes.
```
### Commit Changes
```bash
git commit -m "Commit message" # It records changes in the repository with a descriptive message.
```
#### View Commit History
```bash
git log # It displays a log of all commits in the current branch.
```

## Branching and Merging
### Create a New Branch
```bash
git branch <branch_name> # It creates a new branch for development.
```
### Switch to a Branch
```bash
git checkout <branch_name> # It switches to the specified branch.
```
### Merge a Branch
```bash
git merge <branch_name> # It merges changes from the specified branch into the current branch.
# Typical usage:
git checkout <target_branch>
git merge <source_branch>
```
### (Optional) Rebase
[Details about Rebase]({{< relref "/tips/git/rebase.md">}})

### Conflit Handling 
[Handling Conflict]({{< relref "/tips/git/ConflictHandling.md">}})

## Remote Repositories
Before starting operating the remote repository, one should notice that the remote repository and local repository are two different repositories. They may have different commits and branches. One should setup which remote branch to track (which is set to `origin/main` by default), and track that branch manually by commands such as `push` and `pull`.

### Add a Remote
A remote server should be specified before the first push.  
```bash
git remote add <name> <repository_url> # It links your local repository to a remote repository.
```
It adds a **reference** to a remote repository named `name`. This is typically used to push changes to or fetch updates from that remote repository. The URL specified in `repository_url` should point to an existing remote repository. 

### Push Changes
```bash
git push <remote_name> <branch_name> # It uploads your local commits to the remote repository.
```
`remote_name`: The name of the remote repository to which you want to push your changes (default: `origin`).\
`branch_name`: The name of the local branch you want to push to the remote repository.

For the first push:
```bash
git push -u/--set-upstream origin main
```
This not only pushes your changes but also sets origin/main as the default upstream branch for future push and pull operations.

Use the --all option to push all local branches to the remote repository:
```bash
git push origin --all
```

Use the `--force` (or `-f`) option to forcefully push changes. This can overwrite commits on the remote branch and should be used with caution.
```bash
git push --force origin main
```

Push a local branch to *a different branch on the remote*.
```bash
git push origin local-branch:remote-branch
```


### Pull Changes
```bash
git pull origin <branch_name>
```
`<branch_name>`: The name of the branch on the remote repository you want to pull changes from.

This command performs two actions: 
1) **Fetch**: Retrieves changes from the remote repository.
2) **Merge**: Merges the fetched changes into your current local branch.

If you only need specific files from the remote branch *without switching branches or merging*, one might use a combination of fetch and checkout commands:
```bash
git fetch origin
git checkout origin/branch_name -- <file_path>
```

There might be conflicts during `pull`. These conflicts should be resolved and then `merge` or `rebase`.
To apply changes from the remote branch onto your current branch with a linear history, you can use `--rebase` with git pull:
```bash
git pull --rebase origin <branch_name>
```
This rebases your local commits on top of the changes from the remote branch, avoiding a merge commit.

## Branch Management
[Local and Remote Branch Management]({{< relref "/tips/git/BranchManagement.md">}})