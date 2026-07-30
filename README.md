# agent-skills

Source-controlled [Claude Code](https://claude.com/claude-code) skills.

| Skill | What it does |
|---|---|
| [`prd-writer`](prd-writer/SKILL.md) | Turns rough notes, transcripts, or a one-line idea into a structured PRD. |
| [`design-doc-writer`](design-doc-writer/SKILL.md) | Turns requirements into a technical design doc — architecture, data model, interfaces, alternatives, phased plan. |

The two are meant to run in sequence: `prd-writer` settles *what* to build, `design-doc-writer` settles *how*.

## Install

Claude Code discovers personal skills in `~/.claude/skills/`. Symlink them so edits stay under version control:

```sh
git clone git@gitlab.com:Infernite/agent-skills.git ~/workspace/agent-skills
ln -s ~/workspace/agent-skills/prd-writer        ~/.claude/skills/prd-writer
ln -s ~/workspace/agent-skills/design-doc-writer ~/.claude/skills/design-doc-writer
```

Run `/skills` in Claude Code to confirm both are picked up.

## Editing

Each skill is a single `SKILL.md`: YAML frontmatter (`name`, `description`) followed by the instructions. The `description` is what Claude matches against to decide whether to load the skill, so it carries the trigger phrasing — keep it specific when editing.
