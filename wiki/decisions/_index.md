# decisions/

One page per repo-level decision, named `YYYY-MM-DD-<slug>.md`. Records the **why** behind a choice
that shaped the repo, so it isn't re-litigated later. Skip decisions whose rationale is obvious from
the diff.

**Page format** (see [[../CLAUDE]]): Context / Decision / **Why** (load-bearing) / Consequences.

When a decision is reversed or replaced, don't delete its page — move it to the Superseded list below
and note what replaced it, so the history stays legible.

## Active

- [[2026-06-19-docs-to-wiki]] — replace the flat `docs/` folder with an LLM Wiki using a
  catalog-oriented taxonomy.
- [[2026-06-19-doc-automation]] — automate the documentation contract with a `/document` command and a
  guard pre-commit hook.

## Superseded

_(none yet)_
