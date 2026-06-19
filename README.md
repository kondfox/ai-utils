# ai-utils

A continuously-growing collection of artifacts that proved useful in my real agentic engineering
workflows — prompts, tools, techniques, scripts, and agents — packaged so others can reuse them.

This repo is shared for the common good. Feedback is more than welcome: we're all experimenting
together in this new agentic-engineering era, and the best patterns will surface by trying things,
comparing notes, and improving each other's work.

## Contents

- [Seeds](#seeds) — copy-paste bootstrap prompts
- [Agents](#agents) — ready-to-use Claude Code subagents + wiring

_More categories (tools, scripts, techniques…) will be added as they're built._

The [`wiki/`](wiki/index.md) is this repo's knowledge base — the ideas behind the artifacts,
the decisions that shaped the repo, and its glossary.

## Seeds

Seeds are **project-independent** prompts. You paste a seed into Claude Code at the root of your own
repository (run it in **plan mode** first), and it discovers your project, proposes a design, pauses
for your confirmation, and then bootstraps a project-specific utility tailored to your codebase. The
seed is not the thing it builds — it's the prompt that builds it.

| Seed | What it bootstraps | Details |
| --- | --- | --- |
| LLM Wiki | An in-codebase implementation of Karpathy's LLM Wiki — a self-maintaining, LLM-written knowledge base | [details](wiki/seeds/llm-wiki.md) |
| /lesson command | A human-feedback → organizational-memory loop that self-improves the repo's agent instructions | [details](wiki/seeds/lesson-command.md) |
| Version-checker hook | A dependency version-checker feedback loop (checker + PostToolUse hook + pre-commit gate) that keeps AI-added deps aligned and current | [details](wiki/seeds/version-checker-hook.md) |

## Agents

Ready-to-use Claude Code subagents. Each lives in its own folder under `agents/<name>/` as a bundle:
the agent definition (drop it into your project's `.claude/agents/`) plus a `CLAUDE-snippet.md` whose
section you paste into your root `CLAUDE.md` / `AGENTS.md` to actually invoke it. Each integrates
independently.

| Agent | What it does | Details |
| --- | --- | --- |
| plan-critic | Challenges a proposed plan in plan mode before `ExitPlanMode`, until it's genuinely solid | [details](wiki/agents/plan-critic.md) |
| solution-critic | Challenges a finished implementation (diff, tests, scope, loose ends) before it's announced done | [details](wiki/agents/solution-critic.md) |

## Contributing

Contributions go through a **pull request** — `main` is protected, so open a PR from a branch or fork
for any new artifact or change, and a maintainer will review and merge it.

Docs (the README table + `wiki/`) are kept in lockstep with the artifacts. Two pieces help:

- **`/document`** — a Claude Code command that generates or updates the docs for staged artifact
  changes. Run it, review what it wrote, then stage and commit.
- **A guard pre-commit hook** — blocks a commit when staged artifacts under `seeds/` (or future
  `tools/`, `agents/`, `scripts/`) aren't documented yet, pointing you at `/document`. It only
  *checks* — it never writes docs — so you always review them first.

Enable the hook once after cloning:

```bash
git config core.hooksPath .githooks
```

It fails open: with no `claude` CLI available it simply skips the check, and `git commit --no-verify`
bypasses it anytime. See [the decision record](wiki/decisions/2026-06-19-doc-automation.md) for the rationale.
