# Seed design principles

The shared philosophy behind every [[../seeds/_index|seed]] in this repo. A seed is a prompt pasted
into Claude Code at the root of *someone else's* repo to bootstrap a project-specific utility, so all
seeds hold to the same principles. Apply these when authoring or editing one. (Provenance: distilled
from `seeds/llm-wiki-seed-prompt.md` and `seeds/lesson-command-seed.md`, and the root `CLAUDE.md`
"Conventions for editing seeds".)

## Project-independent

A seed never assumes the target repo's shape, stack, or commands. Its first job is to **discover the
target repo and adapt** — taxonomy, paths, build commands, conventions are all learned, never
hardcoded. Each seed must make sense pasted cold into a fresh session with no other context.

## Human-in-the-loop, plan-mode-first

Seeds run in plan mode first: discover, present findings, and **pause for explicit confirmation before
writing files**. Edits apply only when the human accepts the live permission prompt — that approval
*is* the safeguard. Seeds deliberately avoid automation (hooks, cron, CI pipelines) unless the human
opts in; judgment has to earn trust by being hand-run before it's trusted to run unattended.

## Conservative bias

"A smaller true wiki beats a padded one"; "when in doubt, leave it out." Seeds favor not-writing over
speculative or low-value output, and prefer linking to source over copying it.

## Schema-as-source-of-truth

Each seed centers on a `CLAUDE.md` schema file in the target repo that the seed's other artifacts
(subagents, commands) **read at runtime** rather than restate. One authoritative definition, consulted
live, avoids the drift you get when the same rules are copied into several places. This wiki follows
the same principle — see its schema [[../CLAUDE]].

## Where these show up

- [[../seeds/llm-wiki]] — discovers the target project before scaffolding; schema governs the wiki.
- [[../seeds/lesson-command]] — distiller and `/lesson` both read one feedback schema; human gates
  every edit; see [[human-feedback-loop]].
