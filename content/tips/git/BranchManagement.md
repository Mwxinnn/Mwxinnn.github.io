---
title: Branch Management
layout: single
hideToc: false
date: 2024-08-08
---
#
## Basic Understandings

A branch in Git is essentially a *pointer* to *a specific commit* in the repository's history. The primary branch, usually called `main`, represents the production-ready code. Additional branches are used to develop features, fix bugs, or experiment without affecting the main branch.

`main`: Default Local Branch created with `git init`. Prior to Git version 2.28 (released in 2020), the default branch was named `master`.\
`origin/main`: When you clone a repository using git clone <repository_url>, Git automatically sets up a remote named `origin`. This is the default remote name. The default remote branch is usually `origin/main`, which tracks the `main` branch of the remote repository.

## Local
### Create a New Branch
```bash
git checkout -b <branch_name> # create and switch to new branch
```
### Switch Between Branches
```bash
git checkout <branch_name>
```
### View All Branches
```bash
git branch
```
### Delete a Local Branch
```bash
git branch -d <branch_name>
git branch -D <branch_name> # Force Delete
```
`-D` is for a brach hasn't been merged yet and you want to delete it anyway.

## Work with Remote
### Push a Branch to a Remote
```bash
git push -u origin <loacl_branch_name> # push loacl_branch to origin and set it up to track origin/main 
```
### Track a Specific Remote Branch
If a branch exists on the remote but not locally, you can create a local branch that tracks the remote branch:
```bash
git checkout --track origin/<branch_name>
```
### Fetch All Remote Branches
```bash
git fetch # update local repository with the latest references to all branches from the remote
```
### Delete a Remote Branch
Once a branch is no longer needed, you can delete it from the remote:
```bash
git push origin --delete <branch_name>
```

## Merge and Rebase
### Merge
1. Switch to the branch you want to merge into
1. Perform the merge
1. (Optional) Resolve merge conflicts
```bash
git checkout main
git merge <branch_name>
#==============
git add <resolved_files>
git commit
```
### Rebase
For more details, please refer to [Details about Rebase]({{< relref "/tips/git/rebase.md">}})
```bash
git rebase <branch_name>
#e.g.
git rebase main # This rebases the current branch onto main.
git rebase -i <base_commit> # interactive rebasing to edit
```
