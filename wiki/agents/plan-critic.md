# plan-critic

**Agent:** [`agents/plan-critic/plan-critic.md`](../../agents/plan-critic/plan-critic.md)
**Wiring:** [`agents/plan-critic/CLAUDE-snippet.md`](../../agents/plan-critic/CLAUDE-snippet.md)
**Background:** [I Taught Claude to Criticize Itself So I Don't Have To](https://app.notion.com/p/bobcats-coding/I-Taught-Claude-to-Criticize-Itself-So-I-Don-t-Have-To-36c1c06aab6e805e81def38da0dc7af3)
**Related:** [[solution-critic]] · **Principles:** [[../concepts/seed-design-principles]]

## What it is

A skeptical-senior-engineer subagent that **challenges a proposed implementation plan before you call
`ExitPlanMode`**. It reads the plan, verifies its claims against the codebase (Read/Grep/Glob — it does
not take the plan at face value), and pushes hardest on **simplicity and scope**: what's over-built,
what existing utility makes new code unnecessary, whether this is one PR pretending to be three. It
also checks unchecked assumptions, edge cases, pattern fit, and testability.

It is the plan-time half of a two-critic loop; its counterpart for finished work is [[solution-critic]].

## How to use it (install)

1. Drop [`plan-critic.md`](../../agents/plan-critic/plan-critic.md) into your project's
   `.claude/agents/` directory. Claude Code discovers it automatically.
2. Paste the **Plan mode challenge protocol** section from
   [`CLAUDE-snippet.md`](../../agents/plan-critic/CLAUDE-snippet.md) into your root `CLAUDE.md` (or
   `AGENTS.md`). That section is what actually invokes the agent — the agent file alone only defines it.
3. From then on, in plan mode you dispatch the critic, revise until it returns `approved`, and only
   then call `ExitPlanMode` (capped at 5 rounds, after which the human decides).

It uses `model: sonnet` and read-only tools (`Read, Grep, Glob`).

## What it produces

A short narrative for the human, followed by a single parseable JSON object on the last line:

```
{"verdict": "approved" | "needs_revision", "concerns": ["..."], "questions": ["..."]}
```

It defaults to `needs_revision` on material doubt — "approved" is meant to feel earned.

## Caveats

- **The critic decides, not the main agent.** The protocol forbids self-approval; iterate until the
  critic approves or the 5-round cap is hit.
- **Project-independent**, like every artifact here ([[../concepts/seed-design-principles]]). It reads
  the repo's own instruction files and `wiki/index.md` if present, and adapts to conventions it finds.
