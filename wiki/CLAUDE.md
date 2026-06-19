# Wiki schema

This file is the **single source of truth** for how the `wiki/` knowledge base is maintained: when to
capture knowledge, where it goes, and how to write a page. Every session that touches this wiki reads
this file first. (This wiki is itself an instance of the pattern it documents — see
[[concepts/llm-wiki-pattern]].)

This repo is an **artifact catalog**, not a codebase, so this wiki deliberately drops the
codebase-oriented folders the generic pattern uses (`apps/`, `features/`, `runbooks/`, `bugs/`,
`integrations/`, `tech-debt/`, `hacks/`). Our taxonomy fits a catalog of reusable artifacts and the
ideas behind them.

## The three layers

1. **Sources — read-only ground truth.** Read from these; never edit them as part of wiki
   maintenance:
   - the artifact files themselves (`seeds/*.md`, and future `tools/`, `agents/`, `scripts/`),
   - the root `README.md` and root `CLAUDE.md`,
   - git history (`git log` / `git show`) and GitHub PRs (`gh`),
   - external writeups (the Notion articles linked from artifact pages).
2. **The wiki — the writable, compiled artifact.** The cross-linked Markdown pages under `wiki/`.
   Cross-page links use Obsidian-style `[[page-name]]` syntax (works as plain Markdown anywhere).
3. **The schema — this file.** Governs the wiki; consulted at runtime rather than restated elsewhere.

## Folder map

| Folder / file | Holds |
| --- | --- |
| `index.md` | Human-facing map of content (the landing page). |
| `_log.md` | Append-only changelog, newest first, one self-contained line per entry. |
| `glossary.md` | Domain vocabulary. |
| `seeds/` | One page per published **seed** artifact. Mirrors the README's Seeds category. |
| `agents/` | One page per published **agent** artifact. Mirrors the README's Agents category. |
| `concepts/` | Cross-cutting ideas/techniques the artifacts embody or teach. |
| `decisions/` | One page per repo-level decision: `YYYY-MM-DD-<slug>.md`. |

Each category folder has an `_index.md` describing its purpose and page format.

An artifact may be a **single file** (e.g. a seed: `seeds/<name>.md`) or a **bundle folder** (e.g. an
agent: `agents/<name>/` holding the agent definition + its `CLAUDE-snippet.md`). Either way it maps to
one wiki page named for the artifact (`agents/<name>/` → `wiki/agents/<name>.md`).

## When to update the wiki (the maintenance contract)

Treat this as a checklist for the task at hand:

- **New artifact published** (a new seed, or later a tool/agent/script) → add a page in the matching
  category folder, **add it to the README table**, and link it from `index.md`.
- **Existing artifact changed or extended** → update its page.
- **A cross-cutting idea emerges or gets clarified** (a recurring technique, principle, or pattern) →
  add/update a `concepts/` page.
- **A repo-level decision is made** → `decisions/YYYY-MM-DD-<slug>.md`, capturing the **why**.
- **A domain term comes up** → add it to `glossary.md`.
- **A new artifact *category* appears** → see "Evolving the schema" below.
- **Always** append a one-line entry to `_log.md`.

Wiki updates land in the **same commit** as the change that triggered them, so `git log` ties them
together.

## When NOT to update

- Trivial changes (typos, formatting, renames).
- Anything already stated in the root `CLAUDE.md` — **link to it, don't duplicate**.
- Speculative future plans — the wiki describes what *is*, not what might be.
- Anything trivially derivable from the source. When in doubt, prefer a `[[link]]` or a source
  reference over copying. A smaller true wiki beats a padded one.

## How to write a page

- **One concept per page.** Split a page that grows past ~200 lines.
- **Lead with the answer, then the provenance.** Don't bury the point.
- **Cross-link liberally** with `[[page-name]]` so the graph stays navigable. A `[[link]]` to a page
  that doesn't exist yet is a fine TODO marker.
- **Provenance is mandatory for non-obvious claims**: a file path, a commit SHA, a PR number, or the
  Notion URL it came from.
- **Flag contradictions, don't silently overwrite.** If new info conflicts with an existing page, keep
  the truer version and note the change in `_log.md`.
- **Dates are absolute** `YYYY-MM-DD`, never "recently".

### Page skeletons

**Artifact page** (`seeds/`, future `tools/` …):
```
# <Name>
**Source:** <link to the artifact file><br>
**Background:** <external writeup, if any>
## What it is        (and the problem it solves)
## How to use it
## What it produces
## Caveats / known limitations
```
One metadata field per line, each ending with a trailing `<br>` (except the last) so the block renders
as separate lines on GitHub — GitHub-flavored Markdown collapses a bare newline into a space, where
Obsidian wouldn't. In the header, use **standard Markdown links** (`[text](../concepts/foo.md)`), not
`[[wiki-links]]`: GitHub renders Obsidian `[[ ]]` as literal text, and Markdown links to `.md` files
resolve in both GitHub and Obsidian. (`[[wiki-links]]` are still fine in body prose.)

**Decision page** (`decisions/`):
```
# <YYYY-MM-DD> — <Title>
## Context      ## Decision      ## Why  (load-bearing)      ## Consequences
```

**`_log.md` line**: `- YYYY-MM-DD — <one self-contained sentence> (<commit/PR if relevant>)`

## Evolving the schema

The taxonomy is a starting point, not a cage. When content genuinely fits no existing category (e.g.
the first `tools/` artifact ships), create the folder + its `_index.md`, add it to the folder map
above, add a "When to update" trigger, and link it from `index.md`. Default to reusing an existing
category; schema growth should be explicit and rare.

## Tooling boundaries

- There is no code formatter or build in this repo, so nothing needs to ignore the wiki.
- `_log.md` is a concurrent-append hotspot. It uses a `union` merge driver (declared in the repo-root
  `.gitattributes`), which auto-resolves concurrent appends **locally**. Note the GitHub-web-UI
  limitation documented in [[seeds/llm-wiki]] — GitHub doesn't support `union`, so a `_log.md`
  conflict in a PR still needs a local merge. Keep every `_log.md` entry a single self-contained line.
- The wiki is committed to git; updates land in the same commit as the change that triggered them.
