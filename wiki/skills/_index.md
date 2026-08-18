# skills/

One page per published **skill** artifact. A skill is a ready-to-use Claude Code skill you drop into
your project's `.claude/skills/` (or `~/.claude/skills/` for personal use) and invoke by name. Unlike
a [[../seeds/_index|seed]] (a prompt that *generates* something), a skill — like an
[[../agents/_index|agent]] — is the finished thing: installed, not bootstrapped. Where an agent runs
as a separate subagent with its own context, a skill loads instructions into the session you're
already in.

Each skill is a **bundle folder** under `skills/<name>/` holding at least a `SKILL.md` (the skill
definition: YAML frontmatter + instructions), plus any reference files it ships.

**Page format** (see [[../CLAUDE]] for the full skeleton): What it is / How to use it (install) / What
it produces / Caveats / Links to the `SKILL.md` and any external writeup.

## Pages

- [[plan-prs]] — splits a specification into an ordered chain of PR-sized chunks.
