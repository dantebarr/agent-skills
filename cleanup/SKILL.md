---
name: cleanup
description: After a merge — switch to the default branch, pull, and delete the local branch you left if its remote is gone.
disable-model-invocation: true
---

# Cleanup

Run right after a merge, from the feature branch that just landed.

1. If the working tree is dirty, stop and say what is uncommitted.
2. Note the current branch, then `git checkout <default branch>` and `git pull --prune`.
3. If the branch you left no longer exists on the remote, delete it with `git branch -D <branch>` — squash merges make `-d` refuse. If its remote still exists, keep it.

Report what was deleted or kept, in one line.
