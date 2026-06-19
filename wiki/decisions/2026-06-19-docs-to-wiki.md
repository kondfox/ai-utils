# 2026-06-19 — Replace `docs/` with an LLM Wiki

## Context

The repo had just gained a flat `docs/seeds/` folder with one description page per seed. That captured
*per-artifact* documentation but had no home for the **cross-cutting knowledge** the repo accumulates —
the ideas the artifacts share, the rationale behind repo-level choices, and domain vocabulary. As the
repo grows (more seeds, then tools/agents/scripts), that knowledge would keep dying in chat history and
PR threads.

## Decision

Replace `docs/` with a `wiki/` knowledge base built on the [[../concepts/llm-wiki-pattern|LLM Wiki
pattern]] — the same pattern the [[../seeds/llm-wiki]] seed ships. Use a **catalog-oriented taxonomy**
(`seeds/`, `concepts/`, `decisions/`, `glossary.md`) rather than the generic codebase taxonomy
(`apps/`, `features/`, `runbooks/`, `bugs/`, `integrations/`, `tech-debt/`, `hacks/`). The two existing
`docs/seeds/*.md` pages moved into `wiki/seeds/` and gained `[[wiki-link]]` cross-links. The root
`CLAUDE.md` and `README.md` were wired to point at the wiki.

## Why

- This repo is an **artifact catalog, not a codebase**, so the generic codebase folders don't fit —
  there are no apps to run, services to integrate, or incidents to write runbooks for. Keeping empty
  codebase folders would be noise.
- The valuable, easily-lost knowledge here is *about the artifacts and the ideas behind them*, which
  the `concepts/` + `decisions/` + `glossary` set captures directly.
- Dogfooding: the repo publishes the LLM Wiki seed, so using the pattern itself keeps us honest and
  gives readers a worked example.

## Consequences

- Future sessions read `wiki/CLAUDE.md` before non-trivial work and update the wiki in the same commit
  as the change (wired in the root `CLAUDE.md`).
- New artifact categories (e.g. `tools/`) extend the taxonomy under "Evolving the schema" in
  [[../CLAUDE]] rather than reusing codebase folders.
- `_log.md` inherits the `union` merge-driver rough edge documented on [[../seeds/llm-wiki]].
