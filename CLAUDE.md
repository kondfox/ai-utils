# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project knowledge base — `wiki/`

`wiki/` is an LLM-maintained knowledge base for this repo (an instance of the LLM Wiki pattern one of
the seeds ships). It holds what the artifacts alone don't show: the ideas behind them, repo-level
decisions and their rationale, and a glossary.

**Before any non-trivial task:** read `wiki/CLAUDE.md` (the schema — it defines when and how to update
the wiki; treat its "When to update" list as a checklist) and skim `wiki/index.md` (the map of
content) for pages relevant to what you're touching.

**While working:** when you publish or change an artifact, make a repo-level decision, clarify a
cross-cutting idea, or introduce a domain term, update the relevant wiki page per the schema and append
a line to `wiki/_log.md`. **Wiki updates land in the same commit as the change that triggered them.**

**Tooling that enforces this:** the `/document` command (`.claude/commands/document.md`) generates or
updates the docs for staged artifact changes — run it, review, then stage. A guard pre-commit hook
(`.githooks/pre-commit`, enabled via `git config core.hooksPath .githooks`) blocks a commit when
staged artifacts aren't documented and points at `/document`; it only checks, never writes. See
`wiki/decisions/2026-06-19-doc-automation.md`.

## What this repo is

A continuously-growing, public collection of artifacts that proved useful in real agentic engineering
workflows — prompts, tools, techniques, scripts, and agents — packaged for others to reuse. It is
shared for the common good; feedback is welcome.

`seeds/` is the first and currently only category. The other categories (tools, agents, scripts,
techniques) don't exist yet — they'll be added as they're built. There is no build, no test suite,
no lint, no package manifest. Work in this repo is authoring and editing Markdown.

### Seeds

A **seed** is a self-contained, project-independent meta-prompt the user pastes into Claude Code at
the root of *some other* repository to bootstrap a reusable agentic system there. Each file under
`seeds/` is a deliverable artifact (a prompt), not source code that runs here.

- `seeds/lesson-command-seed.md` — bootstraps a human-feedback → organizational-memory loop
  (`.claude/feedback/` schema, a `feedback-distiller` subagent, a `/lesson` command, per-author
  journals) so a target repo's agent-instruction files self-improve from corrections and confirmed wins.
- `seeds/llm-wiki-seed-prompt.md` — bootstraps a persistent, LLM-maintained knowledge base (an
  "LLM Wiki", per Karpathy's pattern) in a target repo, kept current as a side effect of normal work.

## Repo layout — where things go

- **`seeds/<name>.md`** — the seed prompts themselves. Keep these **pure and copy-paste-clean**: a
  reader pastes the whole file into Claude Code, so no metadata, badges, or repo-internal cross-links
  belong here. Both seeds use a `> Paste everything below the line…` marker followed by `---`.
- **`wiki/seeds/<name>.md`** — the longer human-facing description of each seed (what it is, how to
  use it, what it produces, caveats), plus `[[wiki-link]]` cross-links. These link back to the prompt
  file and out to any external writeup. This is where prose about a seed lives, not in the prompt.
- **`wiki/`** — the knowledge base (artifact pages, `concepts/`, `decisions/`, `glossary.md`). See
  `wiki/CLAUDE.md` for its schema and taxonomy. Prose about *ideas* (not a single artifact) goes here.
- **`README.md`** — the top-level index. The Seeds section is a table linking each seed to its
  `wiki/seeds/` page. New categories follow the same pattern: a category gets a live README section
  only once it actually has content (no empty placeholders).

## Conventions for editing seeds

The two existing seeds share a design philosophy — preserve it when editing them or writing new ones:

- **Project-independent.** A seed never assumes the target repo's shape, stack, or commands. It
  instructs the agent to *discover the target repo first*, then adapt. Don't hardcode paths,
  build commands, or framework assumptions into a seed.
- **Human-in-the-loop, plan-mode-first.** Seeds tell the agent to run in plan mode, present findings,
  and pause for explicit confirmation before writing files. They deliberately avoid automation
  (hooks, cron, CI pipelines) unless the user opts in — the human approving each step *is* the safeguard.
- **Conservative bias.** "A smaller true wiki beats a padded one"; "when in doubt, leave it out."
  Seeds favor not-writing over speculative content.
- **Schema-as-source-of-truth.** Each seed centers on a `CLAUDE.md` schema file in the target repo
  that other artifacts read at runtime rather than restate, to avoid drift.

When proposing changes, treat each seed as a standalone document: it must still make sense pasted
cold into a fresh Claude Code session with no other context from this repo.

## Git & commit messages

Default branch is `main`.

Commit messages follow [gitmoji](https://gitmoji.dev/):

```
:emoji_code: Capitalized imperative summary
```

- Lead with the **text shortcode** form of the gitmoji (`:seedling:`), not the raw Unicode emoji.
  Pick the code whose meaning matches the change — see https://gitmoji.dev/ for the full list.
- Follow it with a single space, then a **capitalized, imperative** summary with **no trailing
  period** (e.g. `Add LLM wiki seed`, not `Added the LLM wiki seed.`).
- A body is optional — omit it for self-explanatory changes; add one (after a blank line) when the
  *why* isn't obvious from the diff.

Existing history for reference:

```
:seedling: Add /lesson command seed
:seedling: Add LLM wiki seed
```

`:seedling:` is used for adding new seeds. Common others: `:memo:` (docs), `:bug:` (fix),
`:recycle:` (refactor), `:wrench:` (config/tooling).
