# Wiki changelog

Append-only. Newest first. One self-contained line per entry.

- 2026-06-19 — Added the [[seeds/version-checker-hook]] seed (dependency version-checker feedback loop: checker script + PostToolUse hook + pre-commit gate); documented it in the README Seeds table, [[index]], and [[seeds/_index]], with a background article link.
- 2026-06-19 — Linked the background article ("I Taught Claude to Criticize Itself So I Don't Have To") on [[agents/plan-critic]] and [[agents/solution-critic]].
- 2026-06-19 — Added the `agents/` artifact category (first entries: [[agents/plan-critic]], [[agents/solution-critic]]) — ready-to-use subagents shipped as bundle folders (agent def + per-agent `CLAUDE-snippet.md`). Extended the schema folder map and the README/index accordingly.
- 2026-06-19 — Added documentation automation: a `/document` command (`.claude/commands/document.md`) and a guard pre-commit hook (`.githooks/pre-commit`) that blocks commits with undocumented staged artifacts. Rationale in [[decisions/2026-06-19-doc-automation]].
- 2026-06-19 — Bootstrapped the wiki from the former `docs/` folder: added the schema ([[CLAUDE]]), [[index]], glossary, the two seed artifact pages, three concept pages, and the [[decisions/2026-06-19-docs-to-wiki]] decision. Adopted a catalog-oriented taxonomy instead of the generic codebase one.
