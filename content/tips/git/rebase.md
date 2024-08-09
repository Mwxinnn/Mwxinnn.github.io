---
title: Rebase
layout: single
hideToc: false
date: 2024-08-08
---
## Rebase

**Merge** combines changes from one branch into another while *preserving the history of both branches*. It creates a new commit (a "merge commit") that combines the changes from the two branches. The merge commit has two parent commits (one from each branch being merged). It maintains a complete history of changes and the context in which they were made.

**Rebase** reapplies commits from one branch on top of another, creating *a linear history*. The commits from the source branch appear as if they were made directly on the target branch. It's useful for incorporating updates from a base branch without creating merge commits.

```bash
git rebase <branch_name>
```
If conflicts occur during a rebase, Git will pause and indicate the conflicted files. Resolve the conflicts as you would with a merge. If needed, one can abort the rebase.
```bash
git add <file_name>
git rebase --continue
git rebase --abort
```

After resolving conflicts and completing the merge or rebase, it's good practice to verify everything is as expected:
```bash
git status
git log
```