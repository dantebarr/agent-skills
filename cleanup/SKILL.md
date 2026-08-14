---
name: cleanup
description: Clean up the repository — prune branches that are merged or whose remote is gone, clear leftover working files, then bring stale documentation up to date.
disable-model-invocation: true
---

# Cleanup

Run from `main`, usually right after something merged.

## Branches and files

Sync first: `git checkout main`, `git pull`, `git fetch --prune`.

Then delete local branches that are safe to lose — merged into `main`, or tracking a remote that is already gone. Use `git branch -d` and let it refuse; never `-D`. A branch holding commits that aren't on `main` stays, and you name it in your report.

Clear out what the work left behind: scratch files, stray build output, one-off scripts. If you can't tell whether a file is deliberate, leave it and ask.

## Documentation

Then read the repo's docs against what the code now does — README, `AGENTS.md` / `CLAUDE.md`, `docs/`, and any comments describing structure that has since moved. Fix what has drifted.

Make the edits, then stop. Report what you changed and why, and ask whether to commit and push. Never commit documentation changes unasked, `main` included.
