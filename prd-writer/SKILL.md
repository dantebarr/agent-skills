---
name: prd-writer
description: Turns rough input — notes, brain dumps, meeting transcripts, feature ideas, Slack threads — into a structured product requirements document (PRD) in Markdown. Use this skill whenever the user asks for a PRD, requirements doc, product spec, feature spec, or asks to "write up the requirements" for a feature or product, even if they only provide scattered notes or a one-line idea. Also use it when the user shares raw product thinking and wants it formalized into a document.
---

# PRD Writer

Produce a Markdown PRD from whatever input the user provides. The input may be rich (a full brain dump, meeting notes, an existing doc) or thin (a one-liner). Either way: draft the document now, don't interview the user first. Where information is missing, make the gap visible instead of inventing facts.

## Core principles

1. **Never fabricate.** A PRD gets read by engineers and stakeholders who will treat its contents as decisions. If the user didn't specify a target metric, platform, deadline, or user segment, don't make one up. Either mark it as an open question or, where a reasonable default exists, state it as an explicit assumption the user can veto.

2. **Requirements must be testable.** "The app should be fast" is an aspiration; "Search results return in under 500ms at p95" is a requirement. When the user's input is vague, translate it into the most concrete testable form the input supports, and if you had to guess at the threshold, flag it.

3. **Scale to the input.** A one-line idea deserves a lean one-to-two page PRD with many open questions — not five pages of padded boilerplate. Rich input deserves fuller treatment. Empty sections are worse than absent sections: if there's nothing real to say about, e.g., non-functional requirements, keep the section to a line or two or fold it into Open Questions.

4. **Separate what from how.** Requirements describe behavior and outcomes, not implementation. If the user's notes contain implementation ideas, capture them in a Technical Notes section rather than baking them into requirements.

5. **Write for skimming.** PRDs get skimmed far more often than read. Prefer one idea per line: short sentences, tables, and checklist items over dense paragraphs. If a section is growing a text block more than ~3 sentences long, break it into bullets or split it up. The Problem section is the one place a few sentences of connected prose belongs.

## Document structure

Save the PRD as a Markdown file named after the feature (e.g., `prd-dark-mode.md`). Use this skeleton, adapting as the input warrants:

```markdown
# PRD: [Feature/Product Name]

**Status:** Draft · **Author:** [user's name if known] · **Date:** [today]

## Problem
Why this exists. The user pain or business need, in a few sentences.

## Goals
What success looks like. Include measurable success metrics where possible.

## Non-goals
What this deliberately does not cover. Prevents scope creep — always include
this section, inferring sensible exclusions from context if needed.

## Users
Who this is for. Personas or segments, only as detailed as the input supports.

## Requirements
Numbered, testable, prioritized, each with acceptance criteria. These PRDs
are often handed to AI agents to implement, so every requirement needs an
explicit, checkable definition of done — an agent (or human reviewer) should
be able to go down the criteria and verify each one without judgment calls.

### R1. [Short title] (P0)
[Testable requirement statement.]

**Acceptance criteria:**
- [ ] [Verifiable condition — observable behavior, not implementation]
- [ ] [Another condition, including relevant edge cases]

P0 = launch blocker, P1 = important, P2 = nice-to-have.

Acceptance criteria must derive from the requirement and the input — don't
invent thresholds to make criteria look rigorous. If a criterion needs a
number nobody has given (latency budget, retention target), write it with a
bracketed placeholder like "[threshold — see Q2]" and raise it as an open
question or assumption.

## User flows (optional)
Only if the input describes interaction sequences worth capturing.

## Non-functional requirements (optional)
Performance, security, accessibility, compliance — only what the input
supports or what is clearly implied (e.g., handling payments implies
security requirements).

## Technical notes (optional)
Implementation ideas from the input, kept out of the requirements.

## Assumptions
Defaults you chose that the user should confirm. Each one is a decision
made on their behalf — make it easy to veto.

## Open questions
What the input didn't answer that materially affects scope or design.
Phrase each as a direct question someone can answer. Show at most 5 at a
time, ordered by impact — a wall of questions overwhelms rather than helps.
If more exist, note "N more deferred until these are resolved" and keep the
rest in a collapsed "Deferred questions" subsection at the bottom of the doc.
```

## Working from the input

- Extract every concrete fact from the user's material and place it in the right section. Meeting notes and threads often bury requirements in asides — read carefully.
- If the input contains contradictions (e.g., two different launch dates), surface the conflict in Open Questions rather than silently picking one.
- Preserve the user's terminology for product names, features, and teams.
- If the user provides an existing doc to revise, keep its structure where reasonable and improve content rather than forcing this template wholesale.

## After drafting

Present the file, then briefly (2-3 sentences max) point the user at the highest-impact open questions — the ones whose answers would most change the doc. Never surface more than 5 questions at once, in the doc or in conversation. Don't recap the document's contents; they can read it.

## Iterating on the doc

The PRD converges through rounds of the user answering open questions. When they come back with answers ("Q1: yes, Q3: iOS first"):

- **Edit in place** — update the existing file rather than regenerating it. Don't restructure sections the answers don't touch; the user may have already shared the doc with others.
- **Promote answers to their proper home.** An answered question becomes a requirement, an assumption confirmed, a non-goal, or a scope change — place it where it belongs and remove it from Open Questions.
- **Refill the question queue.** If deferred questions exist, promote the next most impactful ones into Open Questions, keeping the visible list at 5 or fewer. New questions raised by the answers themselves go into the mix too, ranked by impact.
- **Flag ripple effects.** If an answer changes priorities or invalidates an existing requirement, make the edit and mention it in one line ("R4 dropped to P2 since you deferred tablets").
- After each round, briefly note what changed and what's still open — again, no full recap.
