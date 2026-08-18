---
name: plan-prs
description:
  Split a specification into reasonable, PR-sized chunks — an ordered chain where each chunk is one
  reviewable pull request's worth of work.
disable-model-invocation: true
---

# Plan PRs

Split a specification into **PR-sized chunks**: an ordered chain in which each chunk is one pull
request's worth of work, and each builds on the one before it.

This skill's only job is the **chunking** — deciding where the seams go, in what order, and what
each chunk defers to the next. It produces the breakdown and hands it back. What happens to the
chunks afterwards is the caller's business: a human may work them by hand, or a workflow may turn
them into tracker issues and branches. This skill creates nothing, and needs no issue tracker, no
branch, and no runner to be useful.

## Process

### 1. Read the specification

The argument is a specification: an issue reference, a URL, a file path, or just the conversation so
far. Fetch it if it is a reference and read it whole — the acceptance criteria and any explicit
out-of-scope statements decide where the seams can legitimately fall.

If a previous breakdown of the same spec already exists (the caller will say so, or it is in the
conversation), read it and produce only what is missing rather than re-cutting chunks that are
already settled.

### 2. Explore the codebase (optional)

If you have not already explored the codebase, do so. Titles and descriptions use the project's
domain glossary vocabulary and respect ADRs in the area you're touching. Look for prefactoring that
would make later chunks easier — "make the change easy, then make the easy change" — and give it its
own early chunk when it's substantial.

### 3. Cut the chunks

Break the spec into an ordered chain, bottom first.

<chunk-rules>

- **A chunk is PR-sized**: a coherent change one reviewer can hold in their head in one sitting. If
  you cannot describe what it does in a sentence without "and also", split it.
- **A chunk is independently correct.** It builds, its tests pass, and it does not depend on a chunk
  above it. A reviewer approving only this chunk must not be approving something broken.
- **A chunk is a vertical slice, not a horizontal one** — it cuts through every integration layer it
  touches (schema, service, delivery, UI, tests) rather than being "all the types" or "all the
  endpoints".
- **The order is a straight line, not a tree.** Each chunk builds on exactly one predecessor. If two
  chunks are genuinely independent, they still need an order — pick one.
- **State what each chunk defers.** Every chunk names what belongs to a later one. This is the main
  defence against a chunk swallowing its successors.
- **Prefer fewer, larger chunks over many tiny ones.** Each chunk costs a review pass, and rework
  whenever something below it changes. Three to six is typical; more than eight is a sign the spec
  should have been two specs.

</chunk-rules>

### 4. Quiz the user

Present the chain as a numbered list, bottom chunk first. For each chunk show:

- **Title**: in the repo's subject-line schema, `[project] Application: <gitmoji> Imperative
  description` (see [`CLAUDE.md`](../../../CLAUDE.md#commit-messages--pr-titles)) — it is the natural
  title for the pull request this chunk becomes
- **Builds on**: the chunk below it, or "nothing — this is the first"
- **What it delivers**: one sentence
- **Deferred to later chunks**: one sentence

Ask the user:

- Is each chunk really PR-sized, and independently correct on its own?
- Is the order right — does anything depend on something above it?
- Should any chunks be merged or split?

Iterate until the user approves the chain.

### 5. Hand back the breakdown

Output the approved chain, in order, with each chunk in the shape below. That is the deliverable —
do not create issues, branches, or pull requests, and do not run `gh`.

A caller that wants these on a tracker (for example the `/temper` workflow) takes it from here and
decides how to record them; a human caller can work straight from the list.

<chunk-template>

## Title

The chunk's title, in the repo's subject-line schema.

## Builds on

The title (or number) of the chunk this one builds on, or `nothing` for the first in the chain.

This is the **only** ordering relationship between chunks — the chain is a straight line, so a chunk
never lists more than one predecessor.

## What to build

What this chunk delivers, end to end, in behavioural terms. One coherent change — if this section
needs "and also", it should have been two chunks.

Avoid specific file paths or code snippets; they go stale fast. Exception: a snippet that encodes a
decision more precisely than prose can (state machine, schema, type shape), trimmed to the
decision-rich part.

## Architectural constraints

The architectural direction this chunk must follow: which layer the new code belongs in, what gets
injected rather than imported, which existing seam to extend, and any known anomaly in the blast
radius to flag rather than fix.

**Each constraint must be self-contained** — concrete enough to follow as-is. A bare reference like
"follow §4.2" is not a constraint; "the Twilio client is wrapped as a skid external service and
injected — service code never imports it directly" is. Keep any source citation in parentheses as
provenance for a reviewer, not as the instruction itself.

Unlike `## What to build`, this section **may** name files and layers: it is implementation
direction, and it is acted on within hours.

Omit for a chunk with no architectural dimension (docs, config, a copy change).

## Out of scope

What this chunk deliberately defers, and which later chunk picks it up.

Not optional except for the last chunk in the chain.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2

**Behavioural and demoable**, at the altitude of the whole chunk: what a reviewer can observe about
this chunk's diff, not how it was built. These are what say the chunk works end to end.

**Make "apply to all X" criteria executable.** When a chunk must touch _every_ instance of
something, back it with a test or lint that fails when one is missed. Green tests cannot detect a
_partial_ rollout: doing 3 of 10 sites passes.

</chunk-template>
