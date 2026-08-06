# agent-skills

Source-controlled [Claude Code](https://claude.com/claude-code) skills.

| Skill | What it does |
|---|---|
| [`prd-writer`](prd-writer/SKILL.md) | Turns rough notes, transcripts, or a one-line idea into a structured PRD. |
| [`design-doc-writer`](design-doc-writer/SKILL.md) | Turns requirements into a technical design doc — architecture, data model, interfaces, alternatives, phased plan. |
| [`grill-with-docs`](grill-with-docs/SKILL.md) | Relentless interview to sharpen a plan or design, writing ADRs and a glossary as it goes. |
| [`grilling`](grilling/SKILL.md) | Interviews you in rounds over a design tree, one frontier of questions at a time, until nothing is silently assumed. |
| [`domain-modeling`](domain-modeling/SKILL.md) | Challenges fuzzy terms and captures the results as a `CONTEXT.md` glossary and ADRs under `docs/adr/`. |

`prd-writer` and `design-doc-writer` are meant to run in sequence: `prd-writer` settles *what* to build, `design-doc-writer` settles *how*.

`grill-with-docs` is a thin composite over the other two: it runs a `/grilling` session with `/domain-modeling` active, so the interrogation and the write-up happen together. Either half is also useful alone — `/grilling` to stress-test thinking without producing docs, `/domain-modeling` to pin down vocabulary outside an interview.

### Upstream

Those three are vendored from the [`mattpocock-skills`](https://github.com/mattpocock/skills) plugin at v1.2.2 (`skills/engineering/{grill-with-docs,domain-modeling}`, `skills/productivity/grilling`), copied verbatim. The set is self-contained — no plugin install required. To refresh against a newer upstream, re-copy the three directories and diff.

Note that if you *also* have the plugin installed, each of these names exists twice: the plugin's copies are namespaced (`mattpocock-skills:grilling`) while these are bare (`grilling`). That is not an error, but `/grilling` becomes ambiguous, and the two can drift apart. Pick one source.

## Install

Claude Code discovers personal skills in `~/.claude/skills/`. Symlink them so edits stay under version control:

```sh
git clone git@gitlab.com:Infernite/agent-skills.git ~/workspace/agent-skills
ln -s ~/workspace/agent-skills/prd-writer        ~/.claude/skills/prd-writer
ln -s ~/workspace/agent-skills/design-doc-writer ~/.claude/skills/design-doc-writer
ln -s ~/workspace/agent-skills/grill-with-docs   ~/.claude/skills/grill-with-docs
ln -s ~/workspace/agent-skills/grilling          ~/.claude/skills/grilling
ln -s ~/workspace/agent-skills/domain-modeling   ~/.claude/skills/domain-modeling
```

`grill-with-docs` needs the last two symlinked as well — it does nothing on its own.

Run `/skills` in Claude Code to confirm they are all picked up.

## Editing

Each skill is built around a `SKILL.md`: YAML frontmatter (`name`, `description`) followed by the instructions. The `description` is what Claude matches against to decide whether to load the skill, so it carries the trigger phrasing — keep it specific when editing.

Skills may carry supporting files next to `SKILL.md`, referenced by relative link and read on demand — `domain-modeling` keeps its `CONTEXT.md` and ADR templates in `CONTEXT-FORMAT.md` and `ADR-FORMAT.md` that way.

The three vendored skills also carry an `agents/openai.yaml` from upstream. Claude Code ignores it; it is kept so the copies stay a faithful match against the source.
