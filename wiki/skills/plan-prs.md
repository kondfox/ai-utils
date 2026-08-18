# plan-prs

**Skill:** [`skills/plan-prs/SKILL.md`](../../skills/plan-prs/SKILL.md)<br>
**Related:** [plan-critic](../agents/plan-critic.md)

## What it is

A skill that splits a specification into **PR-sized chunks**: an ordered chain in which each chunk is
one pull request's worth of work and each builds on exactly one predecessor. It answers the question
that sits between "we agreed what to build" and "someone starts building" — *where do the seams go, in
what order, and what does each chunk defer to the next?*

Its scope is deliberately narrow: chunking, and nothing else. It creates no issues, no branches, no
pull requests, and never runs `gh` — it hands the breakdown back and lets the caller decide what to do
with it. A human can work straight from the list; a workflow can turn each chunk into a tracker issue
and a branch. That restraint is what makes it usable without an issue tracker, a runner, or any
particular team process.

The chunking rules it enforces are the load-bearing part:

- **PR-sized** — one coherent change a reviewer holds in their head in one sitting; if you need "and
  also" to describe it, split it.
- **Independently correct** — it builds, its tests pass, and approving only this chunk never means
  approving something broken.
- **A vertical slice, not a horizontal one** — through every layer it touches (schema, service,
  delivery, UI, tests), not "all the types" then "all the endpoints".
- **A straight line, not a tree** — each chunk has exactly one predecessor; genuinely independent
  chunks still get an order.
- **Explicit deferrals** — every chunk names what belongs to a later one, which is the main defence
  against a chunk swallowing its successors.
- **Fewer, larger chunks** — three to six is typical; past eight, the spec should have been two specs.

## How to use it (install)

1. Copy the `skills/plan-prs/` folder into your project's `.claude/skills/` (or `~/.claude/skills/`
   for personal use across repos). Claude Code discovers it automatically.
2. Invoke it explicitly with `/plan-prs <spec>` — the argument is an issue reference, a URL, a file
   path, or just the conversation so far. It ships `disable-model-invocation: true`, so Claude never
   triggers it on its own; you decide when a spec gets chunked.
3. It presents the chain as a numbered list, bottom chunk first, and **quizzes you** — is each chunk
   really PR-sized and independently correct, is the order right, should anything be merged or split?
   Iterate until you approve, then it outputs the final breakdown.

Point it at an existing breakdown of the same spec and it fills in only what's missing rather than
re-cutting settled chunks.

## What it produces

An approved, ordered chain of chunks. Each one is written to a fixed template:

| Section | Holds |
| --- | --- |
| **Title** | The chunk's pull-request title, in the repo's subject-line schema |
| **Builds on** | Its single predecessor, or `nothing` for the first in the chain |
| **What to build** | What it delivers end to end, in behavioural terms — no file paths, which go stale |
| **Architectural constraints** | Self-contained implementation direction (layer, injection, seam to extend); omitted when there is no architectural dimension |
| **Out of scope** | What it defers, and which later chunk picks it up |
| **Acceptance criteria** | Behavioural, demoable checkboxes at the altitude of the whole chunk |

Two rules in the template do real work: architectural constraints must be **self-contained** (a bare
"follow §4.2" is not a constraint), and "apply to all X" acceptance criteria must be **executable** —
backed by a test or lint that fails when one site is missed, since green tests can't detect a partial
rollout.

## Caveats

- **It assumes a host repo with conventions.** Chunk titles use "the repo's subject-line schema" and
  the skill links to the host project's `CLAUDE.md` for it; it also expects domain-glossary vocabulary
  and ADRs to exist for step 2. In a repo without those, expect to state the title format yourself.
  The relative link inside `SKILL.md` is written for a skill living at `.claude/skills/plan-prs/`, so
  it resolves once installed, not from this catalog.
- **Human-in-the-loop by design** ([[../concepts/seed-design-principles]]). Step 4 is a quiz, not a
  formality — the skill is not meant to run unattended.
- **The chain is strictly linear.** Work that genuinely wants to fan out into parallel branches gets
  flattened into an arbitrary order; that's a deliberate simplification, not an oversight.
- **Chunking only.** If you want issues, branches, or PRs created from the result, that's the caller's
  job — the skill will refuse to do it.
