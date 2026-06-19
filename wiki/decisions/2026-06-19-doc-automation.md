# 2026-06-19 — Automate the documentation contract with a command + guard hook

## Context

[[../CLAUDE|The wiki schema]] requires that when an artifact is added or changed, the README table and
the wiki are updated **in the same commit**. That contract was enforced only by memory — easy to forget,
especially for commits made outside a Claude Code session. We wanted automation that keeps docs in
lockstep without silently committing machine-written prose the author never saw.

## Decision

Two pieces:

- **`/document`** (`.claude/commands/document.md`) — holds all the documenting logic, with two modes:
  `--apply` (default) generates/updates the README + wiki for staged artifact changes but **never
  commits**; `--check` is read-only and prints a `DOC_STATUS: DOCUMENTED` / `UNDOCUMENTED` verdict.
- **A guard pre-commit hook** (`.githooks/pre-commit`) — on commits that stage files under the artifact
  source dirs (`seeds/`, and future `tools/`, `agents/`, `scripts/`), it runs `claude -p "/document
  --check"` and **blocks** the commit on an `UNDOCUMENTED` verdict, pointing the author at `/document`.
  It is shared via `git config core.hooksPath .githooks` (opt-in per clone).

## Why

- **Guard-only, not auto-write.** The hook never writes docs. The author runs `/document`, **reviews
  the generated docs locally**, then stages and commits. We explicitly did not want unreviewed,
  machine-generated documentation landing in a commit. The reviewable command is the source of truth;
  the hook is a cheap gate.
- **One command, two modes.** Keeping generate and check in the same file means the "what counts as
  documented" rules live in exactly one place (which itself defers to [[../CLAUDE|the schema]]).
- **Fail-open on tooling errors.** This repo is public; people clone it. If `claude` is missing, auth
  fails, or the check errors, the hook warns and allows the commit — a contributor without Claude is
  never wedged. It blocks only on a definite `UNDOCUMENTED` verdict, and `git commit --no-verify`
  always bypasses it.
- **No `--bare`.** That flag would ignore OAuth/keychain auth (honoring only `ANTHROPIC_API_KEY`), so
  the hook runs `claude` normally to inherit whatever auth the developer already uses.
- **A cheap gate first.** The hook only invokes `claude` when staged paths fall under an artifact dir,
  so ordinary commits (wiki-only, config, root files) stay instant and free.

## Consequences

- Enabling the hook is a one-time opt-in (`git config core.hooksPath .githooks`); documented in the
  README's Contributing section.
- New artifact categories are added to the `ARTIFACT_DIRS` list at the top of `.githooks/pre-commit`
  (and to [[../CLAUDE|the schema]]).
- The hook and command are **tooling, not catalog artifacts**, so they are recorded here as a decision
  rather than on a `wiki/seeds/` page.
