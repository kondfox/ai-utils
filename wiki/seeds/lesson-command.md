# /lesson command seed

**Source:** [`seeds/lesson-command-seed.md`](../../seeds/lesson-command-seed.md)
**Background:** [Turning Human Corrections Into Organizational Memory](https://app.notion.com/p/bobcats-coding/Turning-Human-Corrections-Into-Organizational-Memory-3741c06aab6e80f6a79ee47c42b31a89)
**Concept:** [[../concepts/human-feedback-loop]] · **Principles:** [[../concepts/seed-design-principles]]

## What it is

A **human-feedback → organizational-memory loop** for your repository. During agentic work you
constantly correct the agent ("no, do it this way") and occasionally confirm a win ("yes, ship that").
Today that signal dies in the chat. This loop captures it and feeds it back into the repo's own
agent-instruction files, so the harness gets a little sharper every day — **without anyone sitting
down to improve it by hand.** A model does the distilling; the human only judges and accepts. See
[[../concepts/human-feedback-loop]] for the underlying idea.

## How to use it

Paste [the seed prompt](../../seeds/lesson-command-seed.md) into Claude Code at the root of your
repository and run it in **plan mode** first. It discovers how your repo is shaped, then builds four
committed artifacts under `.claude/` so every developer gets the whole system with zero setup:

1. **`.claude/feedback/CLAUDE.md`** — the schema. The single source of truth for what counts as
   feedback, the dispatch table that decides what to do with each lesson, and the routing table that
   sends generalizable lessons to the narrowest target (a `CLAUDE.md`, a plan/solution critic, a
   command file, or the docs).
2. **`.claude/agents/feedback-distiller.md`** — a read-only subagent that classifies feedback and
   drafts concrete proposed edits, but never edits anything itself.
3. **`.claude/commands/lesson.md`** — the `/lesson` slash command, the human's entry point. It runs
   the distiller and walks each candidate through the dispatch table.
4. **`.claude/feedback/journal/<author>.md`** — a per-author, append-only ledger of what was learned.

## What it produces

The four artifacts above, wired together so corrections and confirmed wins become durable edits to the
repo's instruction surfaces — judged on two axes (polarity, generality) plus a derived `seenBefore`
flag. See [[../concepts/human-feedback-loop]] for how the dispatch table uses them.

## Caveats / design points

- **Human-in-the-loop, by design.** Edits are only applied when the human accepts the live permission
  prompt — that prompt *is* the accept/reject gate. There is **no automation** (no hook, no cron, no
  pipeline): the routing judgment has to earn trust by being hand-run over weeks first. This is the
  [[../concepts/seed-design-principles|shared seed philosophy]].
- **Anti-survivor-bias.** Human feedback skews negative, so learning only from corrections builds a
  wall of guardrails and forgets what to *keep* doing. Confirmed wins are a first-class capture
  target — but only from unambiguous signals, never inferred from silence.
- **Repo-shared, not personal.** The loop writes only to committed surfaces, so a lesson one developer
  captures benefits the whole team.
