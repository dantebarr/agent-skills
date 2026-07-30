---
name: design-doc-writer
description: Turns requirements — a PRD, a ticket, a feature description, or a list of "it must do X" statements — into a technical design document (TDD) in Markdown covering architecture, data model, interfaces, alternatives, risks, and a phased implementation plan. Use this skill whenever the user asks for a design doc, technical design, architecture doc, RFC, implementation plan, or asks "how should we build this". Also use it as the follow-on step after a PRD, when the user is ready to move from what to build to how to build it — and whenever they want to revise, update, or fold review feedback into a design doc that already exists.
---

# Design Doc Writer

Produce a Markdown technical design document from the requirements the user provides. A design doc answers **how** something gets built; the requirements define **what** it must do. Ground every decision in the actual codebase, not in a generic best-practice architecture.

## The one thing to check first: are there requirements?

This skill needs requirements, but **not a formal PRD**. Any of these is enough to start:

- A PRD or spec document (ideal — read it fully)
- A ticket, issue, or bug report with acceptance criteria
- A prose description of desired behavior with enough specificity to design against
- A list of "the system must..." statements
- A prior conversation in this session where requirements were established

If the user gives you only a topic — "design doc for the billing system", "write up the auth redesign" — you don't have requirements, you have a title. **Ask before drafting.** Keep it to one short round, no more than 4 questions, aimed at the minimum needed to design: what behavior must exist, who/what consumes it, what constraints are non-negotiable, and what's explicitly out of scope. Don't turn it into a full requirements interview — that's `prd-writer`'s job, and if the answers reveal genuinely unformed product thinking, say so and offer to write a PRD first.

Once requirements exist in any form, **draft the document — don't keep interviewing.** Missing detail becomes an open question or a stated assumption, not a blocking question. This applies to *information you'd be asking the user to supply* — it does not override **Checkpoints**, which present decisions you've already picked a side on.

## Before writing: read the code

A design doc that doesn't match the codebase is worse than no design doc. Before proposing anything:

- Find the modules, services, and boundaries the change touches. Read them.
- Note the existing patterns — how this codebase does persistence, background jobs, auth, error handling, config, testing. Your design should extend those patterns, not introduce a parallel universe. If you deliberately break a pattern, say so and justify it in Alternatives.
- Identify what already exists that can be reused, and what has to be built.
- Check for prior art: existing migrations, feature flags, adjacent features solving a similar problem.

**Scope this deliberately — it's the easiest place to burn a lot of time for little return.** Dispatch a search agent for the locating phase if one is available; it parallelizes well and keeps the main context clear. Read in full only the files the change will actually modify, plus one representative example of each pattern you intend to follow. **You're done exploring when you can name the specific files the change will touch and describe how this codebase already solves each problem you're about to solve.** Further reading past that point mostly buys confidence, not design quality.

## When there's no codebase

Greenfield work, or designing outside a repo you can see, changes the job — say so plainly in Current State rather than implying an existing architecture. In that mode:

- Design against the stack the user names. If they haven't named one, take it to Checkpoint A with a recommendation — neither a silent choice on your part nor an open question parked at the bottom of a doc that already assumed an answer.
- "Extend existing patterns" has nothing to attach to. Instead, make the foundational choices explicit and note which ones are expensive to reverse later.
- Current State describes the constraints you're designing into — team, timeline, systems this must integrate with — not existing code.
- Weight Alternatives more heavily. Early structural decisions get revisited far more often than incremental ones, and the reasoning is what survives.

## Checkpoints

Draft in two passes with a checkpoint before each, rather than handing over a finished doc. A ten-page doc is hard to review as one artifact, and a wrong load-bearing choice invalidates every section built on top of it. Two passes cost one extra round trip and save a rewrite.

Use `AskUserQuestion` for both, recommended option first and marked `(Recommended)`. Agreeing must be one keystroke.

**Checkpoint A — after reading the code, before proposing anything.** Two parts:

1. The problem as you understand it — the requirements, plus the constraint, coupling, or assumption that makes this non-trivial. Five lines, not a draft of Current state.
2. The load-bearing choices you're about to make. At most 3, each with a recommendation and one line of reasoning.

Then draft Summary, Requirements, Current state, and Proposed design — and stop.

**Checkpoint B — after Proposed design, before the rest.** Present those sections and confirm the architecture and data model hold. Failure modes, migration, testing, implementation plan, and risks all derive from the decisions above them; generating them against bones the user would have changed is the easiest way to waste a long draft.

Then write the remaining sections and run the trace check.

### What qualifies as a checkpoint choice

Only decisions that are **expensive to reverse** — ones that would invalidate the data model, the interfaces, or the phasing if changed later. Everything else stays an open question or a stated assumption.

Not checkpoint choices:

- **Anything discoverable in the repo.** If the answer is on disk, go read it — a question here just makes the user do your reading.
- **Details that are cheap to change** once the doc is circulating: log field names, error copy, whether a helper is one function or two.

A checkpoint presents decisions; it does not gather requirements. If you're asking the user to supply information rather than to confirm a pick, you've drifted into interviewing — draft instead, and raise it as an open question.

Each option still carries a recommendation. A neutral menu hands the decision back to the reader, which is the surveying failure in **Core principles** wearing a different hat.

### Greenfield: the stack is a checkpoint choice

With no codebase and no stack named, that pick belongs in Checkpoint A. Data model, interfaces, and control flow are all stack-shaped, so drafting them against an unnamed stack produces sections that read as settled while resting on nothing. Same for the other foundational choices in **When there's no codebase** — name them at the checkpoint and flag which are expensive to reverse.

### Example: Checkpoint A

> **Problem:** Refunds are issued synchronously in `PaymentsController#refund`, so a gateway
> timeout surfaces as a 500 with no row written. R1–R3 require refunds to survive a gateway
> outage and to be safe to retry. The hard part: `payments` has no uniqueness constraint that
> would make a retry idempotent.
>
> **1. Async mechanism**
> - Reuse the existing Sidekiq queue (`app/jobs/`) **(Recommended)** — ~200 refunds/day doesn't justify new infra, and its retry semantics already match
> - Dedicated queue — isolates refund load, adds a component to operate
>
> **2. Idempotency key**
> - `payment_id` + `amount_cents` **(Recommended)** — matches the gateway's own 24h dedup window, no caller changes
> - Client-supplied token — more precise, requires every caller to change

## Core principles

1. **Decide, don't survey.** A design doc's value is a chosen approach with reasoning. Pick one, commit to it, and put the rejected options in Alternatives with a sentence on why each lost. A doc that presents three options and picks none has pushed the work back onto the reader.

   > **Surveying (weak):** "There are a few options for running this asynchronously. We could reuse the existing Sidekiq setup, stand up a dedicated queue, or process inline. Each has tradeoffs around latency and operational complexity."
   >
   > **Deciding (strong):** "Reuse the existing Sidekiq queue (`app/jobs/`) rather than adding dedicated infrastructure — at ~200 events/day the volume doesn't justify a new component, and Sidekiq's retry semantics already match what we need here. Dedicated queue and inline processing are in Alternatives."

2. **Never fabricate.** Don't invent table names, service names, latency budgets, or team ownership. Every concrete detail comes from the requirements, the codebase, or the user. If you need a number nobody gave you, use a bracketed placeholder and raise it as an open question.

3. **Trace back to requirements.** Every major design element should map to a requirement, and every requirement should be visibly handled. If a requirement can't be met by the design, say that plainly — that's the single most valuable thing a design doc can surface.

4. **Design at the level of decisions, not code.** Interfaces, data shapes, sequencing, and failure behavior belong here. Full function bodies don't. Use short signatures, schemas, and diagrams. If you're writing more than ~15 lines of code in a block, you're implementing, not designing.

   > **Implementing (weak):** a 40-line `def process_refund` with the validation branches, the DB transaction, the API call, and the error handling written out.
   >
   > **Designing (strong):**
   > ```
   > process_refund(payment_id: UUID, amount_cents: int, reason: RefundReason)
   >   -> Result[Refund, RefundError]
   > ```
   > "Idempotent on `payment_id` + `amount_cents`; a duplicate call inside the 24h window returns the original `Refund` rather than issuing a second one. Fails closed if the gateway times out — the refund row is written as `pending` and reconciled by the existing sweeper job."

5. **Failure modes are part of the design.** What happens on partial write, timeout, duplicate delivery, concurrent edit, bad input? A design that only describes the happy path isn't finished.

6. **Scale to the change.** A one-endpoint addition gets two pages. A new subsystem gets more. Empty sections are worse than absent ones — drop any section with nothing real in it rather than padding it.

7. **Write for skimming.** Reviewers skim. One idea per line, tables over paragraphs, headers that state conclusions. Keep prose blocks under ~3 sentences.

## Where to save the doc

Work through these in order rather than guessing — a doc filed somewhere unexpected doesn't get read:

1. **Where design docs already live in this repo.** Look for `docs/design/`, `docs/rfcs/`, `docs/adr/`, or existing `design-*.md` / `rfc-*.md` files. Match the established naming convention exactly, including any numbering scheme.
2. **Next to the PRD or ticket doc** this design follows from, matching its naming.
3. **`docs/design/`**, if a `docs/` tree exists but has no design docs yet.
4. **Ask.** If none of the above apply, ask where it belongs rather than dropping it in the repo root.

Absent an existing convention, name it after the change: `design-dark-mode.md`.

## Document structure

Adapt this skeleton to the change. Sections down through **Proposed design** are the first pass; everything after it is written once Checkpoint B clears. For the date, run `date +%Y-%m-%d` rather than guessing — you don't reliably know today's date.

```markdown
# Design: [Change Name]

**Status:** Draft · **Author:** [user's name if known] · **Date:** [YYYY-MM-DD]
**Requirements:** [link/path to PRD or ticket, or "see Requirements below"]

## Summary
The design in 3-5 lines. What we're building, the core approach, and the
main tradeoff accepted. A reader who stops here should know the shape of it.

## Requirements
What this design must satisfy, as a short table traced to the source. If a
PRD exists, reference its IDs rather than restating it.

| ID | Requirement | Source | Addressed by |
|----|-------------|--------|--------------|
| R1 | ...         | PRD R1 | §Data model  |

## Current state
How the relevant part of the system works today, with real file/module
names. Include what makes the change non-trivial — the constraint, coupling,
or assumption that has to be worked around.

## Proposed design
The core of the doc. Cover, as the change warrants:

- **Architecture** — components, responsibilities, and how they interact.
  Use a mermaid diagram when the interactions are non-obvious.
- **Data model** — new/changed tables, fields, types, indexes, constraints.
- **Interfaces** — API endpoints, function signatures, events, message
  schemas. Signatures and payload shapes, not implementations.
- **Control flow** — the sequence for the main paths, including who calls
  what and in what order.
- **State and lifecycle** — what's persisted, what's derived, what's cached,
  and what invalidates it.

## Alternatives considered
Each with one line on the approach and one on why it lost. Include the
"do nothing" or "simplest possible" option when it's a real contender.

## Failure modes and edge cases
Table or list: what can go wrong, what the system does, what the user sees.
Cover retries, idempotency, partial failure, and concurrency where relevant.

## Migration and rollout (optional)
Data backfill, schema migration order, feature flags, backwards
compatibility, and how to roll back. Required whenever existing data or
live traffic is affected.

## Security and privacy (optional)
Authn/authz changes, new trust boundaries, PII handling, secrets. Include
whenever the change touches user data, auth, or an external surface.

## Performance and scale (optional)
Expected load, hot paths, query patterns, and where this breaks down.
Include when the change is on a hot path or handles unbounded input.

## Observability (optional)
What gets logged, measured, or alerted so we know this works in production.

## Testing strategy
What gets tested at which level, and what specifically is hard to test.
Name the cases that would catch a broken implementation.

## Implementation plan
Ordered, independently shippable phases. Each phase: what it delivers, what
it depends on, and roughly how big it is. This is the section that gets
handed to an implementer (often an AI agent), so each phase needs a clear
definition of done.

### Phase 1: [Title]
- [ ] [Concrete, verifiable unit of work]
- [ ] [...]

## Risks
What could make this design wrong, and the early signal for each.

## Open questions
Questions that materially affect the design, as direct answerable questions.
Show at most 5, ordered by impact. If more exist, note "N more deferred"
and keep them in a collapsed "Deferred questions" section at the bottom.
```

## Diagrams

Use mermaid when a picture beats prose — component relationships, request sequences, state machines. Keep them small; a diagram with 15 boxes communicates less than two diagrams with 7. Skip diagrams for linear flows that a numbered list handles fine.

## Before presenting: check the traces

The requirements table is the doc's integrity check, and it silently rots unless you verify it. Before handing the doc over, confirm:

- **Every requirement row has a real "Addressed by" entry** pointing at a section that actually contains the relevant design. An empty or hand-wavy cell means either the design has a hole or the requirement is out of scope — resolve which, don't leave it blank.
- **Any requirement the design cannot meet is stated in the Summary**, not buried in Open Questions. This is the finding most likely to change what the team does next, so it shouldn't depend on the reader reaching the bottom.
- **Every bracketed placeholder has a matching open question.** A placeholder with no question attached will get read as a real value.

## After drafting

Present the file, then in 2-3 sentences point at the decisions most worth challenging — the load-bearing choice and the highest-impact open question. Don't recap the doc; they can read it. If the design revealed that a requirement is unmet or self-contradictory, lead with that.

## Iterating on the doc

Design docs converge through review. When the user comes back with answers or pushback:

- **No checkpoints on revision rounds.** Checkpoints A and B belong to the initial draft. Once the doc exists, the user's feedback *is* the steering mechanism — don't make them re-approve decisions they already signed off on.
- **Edit in place** — the doc may already be circulating. Don't restructure sections the feedback doesn't touch.
- **Promote answers to their proper home.** An answered question becomes a design decision, a constraint, or a scope change — put it where it belongs and remove it from Open Questions.
- **Move rejected approaches to Alternatives** rather than deleting them. The reasoning is the point, and reviewers will re-propose anything that isn't visibly ruled out.
- **Propagate the change.** A change to the data model usually touches interfaces, migration, and the implementation plan. Follow it through and mention the ripples in one line.
- **Re-run the trace check** above — edits are exactly when the requirements table drifts out of sync with the design.
- **Refill the question queue** from deferred questions, keeping the visible list at 5 or fewer.
- After each round, note what changed and what's still open — no full recap.
