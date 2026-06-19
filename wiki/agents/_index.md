# agents/

One page per published **agent** artifact. An agent is a ready-to-use Claude Code subagent you drop
into your project's `.claude/agents/` and wire up by pasting a snippet into your root `CLAUDE.md`
(or `AGENTS.md`). Unlike a [[../seeds/_index|seed]] (a prompt that *generates* something), an agent is
the finished thing — installed, not bootstrapped.

Each agent is a **bundle folder** under `agents/<name>/`: the agent definition (`<name>.md`) plus its
own `CLAUDE-snippet.md` (the wiring text). Each integrates independently — adopt one without the other.

**Page format** (see [[../CLAUDE]] for the full skeleton): What it is / How to use it (install) / What
it produces / Caveats / Links to the agent file + its snippet.

## Pages

- [[plan-critic]] — challenges a plan in plan mode before `ExitPlanMode`.
- [[solution-critic]] — challenges a finished implementation before it's announced done.
