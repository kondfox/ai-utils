# The LLM Wiki pattern

A **persistent, LLM-maintained knowledge base** that captures everything the source material doesn't
show on its own — rationale, decisions, postmortems, vocabulary — and is kept current by every working
session as a side effect of normal work, rather than re-derived from scratch each time.

Origin: [Andrej Karpathy's LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).
This repo both **ships** the pattern as a seed ([[../seeds/llm-wiki]]) and **uses** it — the `wiki/`
folder you're reading is an instance.

## The three layers (the load-bearing idea)

The pattern works because three things are kept strictly separate:

1. **Sources** — read-only ground truth (the artifacts, git history, PRs, external writeups). Never
   edited during wiki maintenance.
2. **The wiki** — the writable, compiled artifact: cross-linked Markdown pages using Obsidian-style
   `[[wiki-link]]` syntax.
3. **The schema** — a `CLAUDE.md` at the wiki root that defines *when* to capture knowledge and *how*
   to write pages. Future sessions consult it instead of re-inventing conventions, which is what keeps
   the wiki coherent over time. See this wiki's own schema: [[../CLAUDE]].

## Why it stays current

Two disciplines do the work, both encoded in the schema rather than anyone's memory:

- The root instruction file points every session at the wiki — to read before work and to update
  after.
- **Wiki updates land in the same commit as the change that triggered them**, so `git log` ties the
  knowledge to the change and the wiki can't silently drift.

## Adapting the taxonomy

The generic pattern ships a codebase-oriented taxonomy (`apps/`, `features/`, `runbooks/`, `bugs/`,
`integrations/`, …). That is a *starting point*, not fixed. This repo is an artifact catalog, not a
codebase, so it dropped those folders for a catalog-shaped set (`seeds/`, `concepts/`, `decisions/`,
`glossary`) — see [[../decisions/2026-06-19-docs-to-wiki]]. The seed prompt explicitly instructs the
adopter to fit the taxonomy to their project.

## Known rough edge

The append-only `_log.md` is a concurrent-append conflict hotspot; the `union` merge driver fixes it
locally but not in the GitHub web UI. Documented in detail on [[../seeds/llm-wiki]].
