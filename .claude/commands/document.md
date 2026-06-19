---
description: Document staged artifact changes in the README + wiki (or check that they're documented)
argument-hint: "[--check]"
allowed-tools: Read, Edit, Write, Glob, Grep, Bash(git diff:*), Bash(git show:*), Bash(git status:*), Bash(git log:*), Bash(git ls-files:*)
---

You maintain this repo's documentation contract. The single source of truth for *when* and *how* to
document is `wiki/CLAUDE.md` (the wiki schema). **Read it first** — its "When to update" list and page
skeletons are authoritative; this command only orchestrates them.

Arguments: `$ARGUMENTS`

- If `$ARGUMENTS` contains `--check` → run **Check mode** (read-only verdict; make NO edits).
- Otherwise → run **Apply mode** (generate/update docs; never commit).

## Scope: what counts as an artifact

Artifacts are the publishable things this repo ships, under these **artifact source directories**:
`seeds/`, `tools/`, `agents/`, `scripts/`. A change to a file under one of these is what must be
documented. Changes anywhere else (the wiki itself, `.claude/`, `.githooks/`, root files) are **not**
artifacts and need no catalog documentation here.

An artifact may be a **single file** (`seeds/<name>.md`) or a **bundle folder** (`agents/<name>/`,
holding e.g. an agent definition + its `CLAUDE-snippet.md`). For a bundle, treat the whole folder as
one artifact mapping to one wiki page (`agents/<name>/` → `wiki/agents/<name>.md`); staging any file
inside it counts as touching that artifact.

## Determine the change set (both modes)

Evaluate the **staged snapshot** — what is about to be committed, not the working tree:

```
git diff --cached --name-only --diff-filter=ACMR
```

Keep only paths under the artifact source directories. For the staged *content* of any file, read the
staged blob with `git show :<path>` (not the working-tree file, which may differ). Read the staged
versions of `README.md` and the `wiki/` pages the same way, so your verdict reflects what will land.

If no staged artifact files exist, there is nothing to do: Apply mode reports "no artifact changes
staged"; Check mode prints `DOC_STATUS: DOCUMENTED`.

## What "documented" means

For each added or modified artifact, per `wiki/CLAUDE.md`, all of the following must hold **in the
staged snapshot**:

1. A wiki artifact page exists under the matching category folder (e.g. `wiki/seeds/<name>.md`) and
   accurately reflects the artifact (follows the artifact-page skeleton: What it is / How to use it /
   What it produces / Caveats / Links to the source file + any external writeup).
2. The README's category table (e.g. the Seeds table) has a row linking to that page.
3. `wiki/index.md` links the page.
4. `wiki/<category>/_index.md` lists the page.
5. `wiki/_log.md` has an entry covering this change.
6. Schema extras **only where genuinely warranted**: a `glossary.md` term for a new domain word, a
   `concepts/` page for a genuinely new cross-cutting idea, a `decisions/` page for a repo-level
   decision. Do not demand these for a routine new artifact whose ideas are already covered.

The artifact-name → page-name mapping is by meaning, not filename (e.g. `llm-wiki-seed-prompt.md` →
`wiki/seeds/llm-wiki.md`, `lesson-command-seed.md` → `wiki/seeds/lesson-command.md`). Match
semantically; a page that clearly covers the artifact counts even if the basename differs.

## Check mode (`--check`) — read-only

Do not edit anything. Decide whether every staged artifact change is documented per the rules above.
Output a short checklist of what is present vs missing, then **exactly one** sentinel line as the
final line:

- `DOC_STATUS: DOCUMENTED` — every staged artifact change is fully documented (or nothing to document).
- `DOC_STATUS: UNDOCUMENTED` — at least one staged artifact change is missing required documentation.

When in doubt, prefer `UNDOCUMENTED` and say precisely what's missing — the human will fix it and
re-run. The pre-commit hook greps for this sentinel, so emit it verbatim and last.

## Apply mode (default) — generate/update, never commit

For each staged artifact change that is not yet fully documented, create or update the required pages
following `wiki/CLAUDE.md`'s skeletons and the existing pages' voice (`wiki/seeds/llm-wiki.md` and
`wiki/seeds/lesson-command.md` are the reference examples). Cross-link with `[[wiki-link]]` syntax,
add provenance (link the source artifact file and any external writeup), keep dates absolute
`YYYY-MM-DD`, and append a one-line entry to `wiki/_log.md`.

Then:
- **Do not run `git add` or `git commit`.** Leave staging to the human so they can review.
- Print a summary: which files you created/edited and a one-line "review these, then stage and commit".
- If everything was already documented, change nothing and report "already documented — no changes".

Be conservative (the repo's house style): a smaller true wiki beats a padded one. Prefer linking to
source over copying, and don't invent decisions, concepts, or glossary terms that aren't real.
