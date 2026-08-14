# agent-skills

Source-controlled [Claude Code](https://claude.com/claude-code) skills.

| Skill | What it does |
|---|---|
| [`review-with-me`](review-with-me/SKILL.md) | Walks you through a change as if presenting it in a live review, then takes your questions. |
| [`cleanup`](cleanup/SKILL.md) | Post-merge tidy-up: prunes dead branches and leftover files, then refreshes documentation that has drifted. |

Both set `disable-model-invocation: true`, so Claude never reaches for either on its own — they run only when you type `/review-with-me` or `/cleanup`.

`review-with-me` resolves what to review in order: an MR you named, the open MR for the current branch, then the branch diff against `main`. It assumes GitLab and `glab`.

`cleanup` runs at the other end of that workflow, once the MR is merged in the browser. Branch deletion is deliberately conservative, and documentation edits always stop for approval before being committed.

## Install

Claude Code discovers personal skills in `~/.claude/skills/`. Symlink them so edits stay under version control:

```sh
git clone git@gitlab.com:Infernite/agent-skills.git ~/workspace/agent-skills
ln -s ~/workspace/agent-skills/review-with-me ~/.claude/skills/review-with-me
ln -s ~/workspace/agent-skills/cleanup        ~/.claude/skills/cleanup
```

Run `/skills` in Claude Code to confirm they are picked up.

## Editing

Each skill is built around a `SKILL.md`: YAML frontmatter (`name`, `description`) followed by the instructions. The `description` is what Claude matches against to decide whether to load the skill, so it carries the trigger phrasing — keep it specific when editing. A skill meant to be run only by hand also sets `disable-model-invocation: true`.

Skills may carry supporting files next to `SKILL.md` — references read on demand, `examples/`, `scripts/`, `assets/` — linked by relative path from `SKILL.md`.
