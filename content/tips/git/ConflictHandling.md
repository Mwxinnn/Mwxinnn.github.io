---
title: Handling Conflict
layout: single
hideToc: false
date: 2024-08-08
---
# 
When attempting to merge a branch that results in conflicts, Git will output which files are in conflict.
Checking the status of the repository, conflicted files will be marked with "both modified".
One need to manually edit the file to resolve conflicts by choosing which changes to keep or combining them.

To see conflict markers in files:
```bash
git diff
```

After resolving the conflicts in the file, add the resolved file to the staging area and complete the merge by committing the changes.
If the conflicts are too complex or the merge should be abandoned, you can abort the merge process:
```bash
git merge --abort
```
