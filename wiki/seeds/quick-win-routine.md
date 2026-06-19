# Quick-win routine seed

**Source:** [`seeds/quick-win-routine-seed.md`](../../seeds/quick-win-routine-seed.md)<br>
**Background:** [Tip of the Day But It's a PR: Our Daily Claude Routine for Tiny Fixes](https://app.notion.com/p/bobcats-coding/Tip-of-the-Day-But-It-s-a-PR-Our-Daily-Claude-Routine-for-Tiny-Fixes-3781c06aab6e8191abf8fe5e2153d4df)<br>
**Principles:** [Seed design principles](../concepts/seed-design-principles.md)

## What it is

A seed that sets up a **daily "quick win" routine** in the target repo: every morning an unattended
Claude Code Routine (a scheduled cloud agent) finds exactly one tiny, safe, fully-verified improvement
and opens a pull request for it. Quick wins are the things no one tickets — an `any` that should be
`unknown`, a stale TODO with a one-line fix, a doc page contradicting the code, a patch-level
Dependabot bump. Individually none is worth a context switch; left alone they accumulate. The routine
changes the economics, not the priorities.

Like every seed it is **stack-independent** ([[../concepts/seed-design-principles]]): it detects the
project's toolchain and tailors the routine to it, pausing for confirmation before generating files.

## How to use it

Paste [the seed prompt](../../seeds/quick-win-routine-seed.md) into Claude Code at the root of your
repository. It works in phases:

1. **Inspect the project** — toolchain, verification commands, candidate knowledge sources (wiki,
   `src` paths, Dependabot), whether a [[../agents/solution-critic]]-style loop exists, and the
   toolchain's bootstrap behavior — then pauses for confirmation.
2. **Routine prompt** (`quick-win-routine.prompt.md`) — the unattended-agent instructions: preflight,
   dedupe against open PRs, gather candidates in priority order, pick the smallest/safest, implement &
   verify, open one PR, or do nothing if nothing qualifies.
3. **Setup script** (`quick-win-setup.sh`) — the routine's pre-launch environment script, following
   the sandbox rules (always `exit 0`, in-band health sentinel, `printf` over heredocs, vendored-binary
   shims).
4. **Configuration checklist** — printed steps to wire it up as a Routine in the Claude Code web UI.

## What it produces

The routine prompt, the environment setup script, and a web-UI configuration checklist — not committed
repo files but the three things you paste into a Claude Code Routine (prompt + environment + trigger).
On its schedule, Anthropic spins up a fresh cloud session, clones the repo, runs the prompt to
completion, and shuts down — pushing only to `claude/`-prefixed branches.

It pairs with the [[llm-wiki]] seed: if the project has a wiki, the routine uses it as a candidate
source for outdated pages and broken links; without one, it still works from Dependabot + the codebase.

## Caveats

- **No human in the loop.** A routine runs unattended, so everything you'd say in the moment must be
  written into the prompt in advance — the seed keeps the discipline rules verbatim for that reason.
- **A skipped day is a success.** The hardest rule to make an agent believe: if nothing qualifies or
  the build environment is broken, exit without a PR — no empty or speculative PRs, no lowered bar.
- **Stacking autonomous layers needs explicit precedence.** When a routine inherits a critic loop,
  their conflicts are no longer arbitrated by a human, so the order is written down: the **size budget
  (~50 lines, ≤3 files) outranks the [[../agents/solution-critic]]**. Never grow the PR to satisfy the
  critic; abandon the candidate instead.
