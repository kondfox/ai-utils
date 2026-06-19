# ai-utils wiki

The knowledge base for this repo — what the artifacts are, the ideas behind them, the decisions that
shaped the repo, and its vocabulary. Maintained as a side effect of normal work; see
[[CLAUDE]] for the maintenance schema.

> New here? The repo's front door is the root [`README.md`](../README.md). This wiki is the deeper
> layer the README links into.

## Artifacts

### Seeds
Project-independent prompts you paste into Claude Code to bootstrap a project-specific utility.

- [[seeds/llm-wiki]] — an in-codebase implementation of Karpathy's LLM Wiki.
- [[seeds/lesson-command]] — a human-feedback → organizational-memory loop.
- [[seeds/version-checker-hook]] — a dependency version-checker feedback loop.
- [[seeds/quick-win-routine]] — a daily unattended routine that finds one tiny fix and opens a PR.

### Agents
Ready-to-use Claude Code subagents you drop into `.claude/agents/` and wire up with a snippet.

- [[agents/plan-critic]] — challenges a plan in plan mode before `ExitPlanMode`.
- [[agents/solution-critic]] — challenges a finished implementation before it's announced done.

_Future artifact categories (tools, scripts…) will appear here as they're published._

## Concepts

Cross-cutting ideas the artifacts embody or teach.

- [[concepts/llm-wiki-pattern]] — the three-layer self-maintaining knowledge-base pattern.
- [[concepts/human-feedback-loop]] — turning corrections and confirmed wins into organizational memory.
- [[concepts/seed-design-principles]] — the shared philosophy behind every seed.

## Decisions

Repo-level choices and their rationale.

- [[decisions/2026-06-19-docs-to-wiki]] — why `docs/` became this wiki, with a custom taxonomy.
- [[decisions/2026-06-19-doc-automation]] — the `/document` command + guard pre-commit hook that keep docs in lockstep.

## Reference

- [[glossary]] — domain vocabulary.
- [[_log]] — chronological changelog of wiki updates.
