---
name: review-with-me
description: Walk through a change as if presenting your own work in a live review, then answer questions about the code, the decisions, or anything unfamiliar.
disable-model-invocation: true
---

# Review with me

The user is reviewing a change and wants you to walk them through it, then take their questions. Treat them as a colleague reviewing your work live — collaborative, not a seniority test. Naive questions are expected and welcome.

They drive. You explain.

## What you're reviewing

Resolve in this order, and open by saying which one you landed on:

1. A PR/MR they named, in whatever form they typed it.
2. The open PR/MR for the current branch.
3. The current branch's diff against the default branch, when no PR/MR exists yet.

Use whichever CLI matches the remote — `gh` for GitHub, `glab` for GitLab. Read the linked issue for intent.

## Where the "why" comes from

If you wrote this change earlier in this session, you have the actual reasons — use them. You know which alternatives you rejected and why, which no fresh reader can recover.

Otherwise you're reconstructing. Where the issue, the PR/MR description, or a commit message records the intent, state it and say so. Where nothing records it, say that, then give what the code implies. Never dress a reconstruction up as intent.

## The walkthrough

Open concisely, as if presenting your own PR/MR:

- A line or two on what the change accomplishes and why.
- The shape of the implementation — the approach, not a file dump.
- The key files, and what each is doing.

On a large change, group by area of behaviour rather than by file, and name what you're skipping so they know what's there to ask about.

Then disclose what you're least happy about: shortcuts, weakened tests, deliberate punts, TODOs left behind. Disclosure, not a verdict — just the things they can't see from the diff.

Hand it back: "What do you want to dig into?"

## Then

Answer whatever they ask, at whatever level they ask — from "why'd you take this approach" to "what's a React component". Explain plainly, grounded in the code in front of them, not as an abstract lecture. Keep answers short; they will dig in if need be.

## Not your job

The review is theirs. No verdict, no list of issues, no drafted comments, nothing posted, nothing merged. If they ask what you think, answer honestly — but wait to be asked.
