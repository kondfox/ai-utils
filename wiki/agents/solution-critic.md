# solution-critic

**Agent:** [`agents/solution-critic/solution-critic.md`](../../agents/solution-critic/solution-critic.md)<br>
**Wiring:** [`agents/solution-critic/CLAUDE-snippet.md`](../../agents/solution-critic/CLAUDE-snippet.md)<br>
**Background:** [I Taught Claude to Criticize Itself So I Don't Have To](https://app.notion.com/p/bobcats-coding/I-Taught-Claude-to-Criticize-Itself-So-I-Don-t-Have-To-36c1c06aab6e805e81def38da0dc7af3)<br>
**Related:** [plan-critic](plan-critic.md)<br>
**Principles:** [Seed design principles](../concepts/seed-design-principles.md)

## What it is

A skeptical-senior-engineer subagent that **challenges a completed implementation before the main
agent announces the task done**. It first sees what *actually* changed — running `git diff` against the
merge-base with the default branch, plus `git status` for uncommitted work — rather than trusting the
engineer's summary, then challenges the change on: solves-the-right-problem, simplicity, elegance, edge
cases, pattern fit, **test quality** (tautological mocks, tests one rename away from going green on
broken code), scope drift, loose ends, reuse-vs-reinvented, and code-quality smells.

It is the implementation-time half of a two-critic loop; its plan-time counterpart is [[plan-critic]].

## How to use it (install)

1. Drop [`solution-critic.md`](../../agents/solution-critic/solution-critic.md) into your project's
   `.claude/agents/` directory.
2. Paste the **Implementation challenge protocol** section from
   [`CLAUDE-snippet.md`](../../agents/solution-critic/CLAUDE-snippet.md) into your root `CLAUDE.md` (or
   `AGENTS.md`).
3. After code is complete and lint/type-check/tests pass (one-shot, not watch mode), dispatch the
   critic with the task description, a short summary, and the touched files; revise until it returns
   `approved`, then announce done (capped at 5 rounds, after which the human decides).

It uses `model: sonnet` and tools `Read, Grep, Glob, Bash` (Bash so it can inspect the real diff).

## What it produces

A short narrative for the human, followed by a single parseable JSON object on the last line:

```
{"verdict": "approved" | "needs_revision", "concerns": ["..."], "questions": ["..."]}
```

Concerns must point at specific files/lines/behaviors; it defaults to `needs_revision` on material doubt.

## Caveats

- **The critic decides, not the main agent** — no self-approval; iterate to approval or the 5-round cap.
- **Substitute your default branch** if it isn't `main`; the diff commands assume the branch's
  merge-base with `main`.
- **Project-independent** ([[../concepts/seed-design-principles]]): it holds new code to whatever the
  repo's own lint rules and instruction files already establish.
