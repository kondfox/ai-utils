# Seed prompt — bootstrap a project-specific LLM Wiki

> Paste everything below the line into Claude Code, running it from the root of the repository you
> want a wiki for. It is project-independent: the first thing it does is discover *your* project and
> adapt to it. It will pause for confirmation before scaffolding, then build out a populated wiki.

---

You are going to create and bootstrap a **persistent, LLM-maintained knowledge base** ("LLM Wiki")
for **this** repository, following Andrej Karpathy's LLM Wiki pattern
(https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

The wiki captures everything the code alone doesn't show — architecture, decisions and their
rationale, bug postmortems, known hacks and tech debt, integration contracts, runbooks, in-flight
projects, and domain vocabulary — and is kept current by every future coding session as a side
effect of normal work. The reasoning that today dies in chat history and PR threads gets written
down once and updated incrementally instead of re-derived every session.

This is a **project-independent** prompt. Do **not** assume any particular language, framework, build
tool, or editor. Discover this project and adapt every choice below to what you actually find.

## The pattern — three strictly separated layers

1. **Sources (read-only ground truth).** You read from these, never write to them during wiki
   maintenance: the codebase itself, version-control history (`git log` / `git show` / `git blame`),
   hosted PRs/issues (e.g. `gh` for GitHub, `glab` for GitLab — use whatever this repo uses), and an
   `assets/` folder inside the wiki for raw human-dropped material (notes, transcripts, screenshots,
   meeting summaries). Sources are immutable; the wiki is derived from them.
2. **The wiki (writable, compiled artifact).** A folder of cross-linked Markdown pages you create,
   edit, and delete. Cross-page links use Obsidian-style `[[wiki-link]]` syntax so the graph is
   navigable (in Obsidian or any compatible viewer — this works as plain Markdown regardless).
3. **The schema.** A `CLAUDE.md` at the wiki root that is the single source of truth for *when* to
   capture knowledge, *how* to write pages, what categories exist, and the format conventions. Every
   future session consults it.

## Phase 0 — Discover the project, then confirm parameters with me

Before creating anything, investigate the repo and report back a short proposal. Determine:

- **Repo shape.** Single project or monorepo? If a monorepo, which sub-project is actively developed
  and should get the wiki (or should it be repo-root)? List the candidate locations.
- **Stack & conventions.** Primary language(s), framework(s), build/package tooling, test
  framework, the architectural layering and main patterns (dependency injection, registries,
  service boundaries, etc.). Read the existing root agent-instruction file (`CLAUDE.md` /
  `AGENTS.md` / equivalent) if one exists.
- **Tooling that could fight the wiki.** Is there a code formatter (Prettier, gofmt, black, …) that
  re-wraps or rewrites Markdown? A linter pointed at Markdown? A TypeScript/other project config that
  might `include` the wiki folder? Note the exact ignore file(s) you'd need to touch
  (`.prettierignore`, `.gitattributes`, formatter config, …).
- **VCS host & CLI.** GitHub/GitLab/other, and whether the corresponding CLI (`gh`/`glab`) is
  available for reading PRs.
- **Transcript availability.** Whether prior Claude Code session transcripts exist for this project
  under `~/.claude/projects/` (they are a rich source for hacks, postmortems, and runbooks).

Then propose, and **ask me to confirm before proceeding**:

- the **wiki root path** (default: `wiki/` at repo root for a single project; `<active-project>/wiki/`
  for a monorepo),
- the **initial category set** (start from the default below; add/drop based on what this project
  actually has — e.g. a data-pipeline project may want a `data/` category, a library may not need
  `runbooks/`),
- whether to also set up the **optional weekly maintenance bot** (Phase 4),
- the **source-of-truth instruction file** to wire in Phase 3 (the root `CLAUDE.md`/`AGENTS.md`).

Use `AskUserQuestion` if a choice is genuinely mine to make; otherwise pick the sensible default and
state it. Do not start scaffolding until I confirm.

## Phase 1 — Scaffold the structure and write the schema

Create the wiki root and this folder shape (this is the **default starting taxonomy**, not a fixed
one — adapt it to what Phase 0 found):

```
<wiki-root>/
├── CLAUDE.md           # the schema — maintenance rules and page formats
├── index.md            # map of content — human-facing landing page
├── architecture.md     # high-level architecture + one worked end-to-end example
├── glossary.md         # domain vocabulary
├── faq.md              # questions worth remembering
├── _log.md             # chronological changelog of wiki updates (append-only)
├── apps/               # one page per top-level runtime surface / entry point
├── features/           # one page per non-trivial feature or flow
├── workflows/          # how things get done — dev, deploy, debug, ops
├── runbooks/           # what to do when something breaks (step-by-step, real commands)
├── integrations/       # external systems we depend on (contracts + quirks, not vendor docs)
├── scripts/            # notable scripts / CLIs
├── decisions/          # one page per decision: YYYY-MM-DD-<slug>.md
├── bugs/               # one postmortem per notable bug: YYYY-MM-DD-<slug>.md
├── tech-debt/          # one page per longer-lived owed work item
├── hacks/              # one page per short-lived workaround
└── assets/             # READ-ONLY raw human-dropped notes (sources, not wiki pages)
```

Give each folder an `_index.md` describing its purpose, the page format, and (for `decisions/`,
`bugs/`, `tech-debt/`, `hacks/`) an Active/Open vs Resolved/Removed split so items move rather than
get deleted when they're paid down.

Write `<wiki-root>/CLAUDE.md` as the schema. It must contain, **adapted to this project's actual
tooling and layout**:

- **Layers.** The three layers above, with the concrete list of *this* project's sources (which dirs
  are read-only ground truth, which VCS/PR CLI to use, where `assets/` lives).
- **When to update the wiki** (the maintenance contract — the heart of the schema). A trigger list
  that future sessions treat as a checklist:
  - new app / top-level surface → `apps/` (rare),
  - non-trivial feature implemented or modified → `features/` (+ entry-point `file:line` links),
  - architectural/design decision made → `decisions/YYYY-MM-DD-<slug>.md` (capture the *why*; skip
    if the why is obvious from the diff),
  - workaround/hack added → `hacks/`; longer-lived accepted debt → `tech-debt/` (+ `_index` entry),
  - non-trivial bug fixed → `bugs/YYYY-MM-DD-<slug>.md` (only when the root cause was surprising or
    the class is likely to recur; skip trivial fixes the diff explains),
  - external integration touched/changed → `integrations/`,
  - script/command/workflow added/renamed/removed → `scripts/` or `workflows/`,
  - non-obvious question came up → `faq.md`; domain term came up → `glossary.md`,
  - incident handled / recurring failure found → `runbooks/`,
  - files dropped in `assets/` → compile them into the right pages.
  - Always append a one-line entry to `_log.md`.
- **When NOT to update.** Trivial changes (renames, formatting, lint, dep bumps); anything already in
  the root instruction file (link, don't duplicate); anything trivially derivable from code;
  speculative future plans (the wiki describes what *is*); areas the current task didn't touch. When
  in doubt, prefer linking to source (`file:line`, commit SHA, PR number) over copying.
- **How to write pages.** One concept per page (split past ~200 lines); lead with the answer then
  provenance; cross-reference liberally with `[[page-name]]`; provenance mandatory for non-obvious
  claims (file paths with line numbers when stable, commit SHAs, PR numbers); flag contradictions
  rather than silently overwriting (keep the truer version, note it in `_log.md`); dates are absolute
  `YYYY-MM-DD`, never "recently".
- **Page skeletons** for `decisions/` (Context / Options considered / Decision / **Why** /
  Consequences — the Why is load-bearing) and `bugs/` (Symptom / **Root cause** / Fix / Lessons),
  plus the `_log.md` format (append-only, newest first, **one self-contained line per entry**).
- **Evolving the schema.** The taxonomy is a starting point. When content genuinely fits no existing
  category, the schema may grow: create the folder + `_index.md`, add it to the tree in this file,
  add a "When to update" trigger, link it from `index.md`, log it. Default to *not* adding a category
  if an existing one absorbs the content. Evolution should be explicit and rare.
- **Tooling boundaries** (fill in from Phase 0): the wiki is excluded from the code formatter and
  from any TypeScript/build project; it is committed to git; wiki updates land in the **same commit
  or PR** as the code change that triggered them so `git log` ties them together. If `_log.md` is a
  concurrent-append hotspot, consider giving it a `union` merge driver via `.gitattributes`.

Write `index.md` as the human-facing map of content, linking the major sections, and seed `_log.md`
with the bootstrap entries.

## Phase 2 — Bootstrap real content (the experiment)

Don't leave empty stubs and don't write the pages from imagination. Treat the bootstrap as a set of
**explicit research tasks** — each one a job worth doing on its own — and run them in parallel
(dispatch subagents). Each task reads sources and writes real, provenance-backed pages:

1. **Architecture.** Walk every workspace/module. Identify the layering and dependency-injection /
   composition patterns. Write a tight `architecture.md` including **one worked end-to-end example**
   tracing a single feature through every layer — the template future features follow.
2. **Glossary.** Sweep service/module folders and type definitions; build ~20–40 domain-term entries
   with code references.
3. **Features.** Read the end-to-end / integration tests together with the UI or public surface;
   compile `features/` pages for what users (or callers) actually experience.
4. **Decisions from history.** Read the last ~100 commits and fetch the corresponding PRs from the
   VCS host. Decide which are wiki-worthy and write `decisions/` pages for the non-obvious ones —
   pulling the *why* from PR descriptions, not just diffs.
5. **Mine prior transcripts.** If Claude Code session transcripts exist under `~/.claude/projects/`
   for this repo, read them and propose entries for everything that looks like a hack, a postmortem
   (`bugs/`), a runbook, a tech-debt item, or a decision — the understanding built during past
   "fix-it-and-ship" sessions that would otherwise be compressed and forgotten.
6. **Integrations & runbooks.** From config, infra, env vars, and the transcripts above, write
   `integrations/` pages (contract + quirks) and `runbooks/` for recurring operational failures
   (step-by-step, with the actual commands).

Keep a **conservative bias**: when unsure whether something is wiki-worthy, prefer a smaller, true
wiki over a padded one. Every non-obvious claim cites its source.

## Phase 3 — Wire the project's instruction file so sessions self-maintain it

This is what makes it self-sustaining. Add a section to the **top** of the project's source-of-truth
instruction file (the root `CLAUDE.md`/`AGENTS.md` confirmed in Phase 0) that points every session at
the wiki — both as a thing to read and a thing to update. Adapt paths and the formatter name:

```markdown
## Project knowledge base — `<wiki-root>/`

`<wiki-root>/` is an LLM-maintained knowledge base for this project. It captures everything the code
alone doesn't show: architectural patterns, decisions and their rationale, bug postmortems, known
hacks and tech debt, integration contracts, runbooks, in-flight projects, and the domain glossary.

**On every non-trivial task in this project, before starting work:**

- Read `<wiki-root>/CLAUDE.md` — the schema. It defines when to update the wiki and how to write
  pages. Treat the schema's "When to update" list as a checklist for the current task.
- Skim `<wiki-root>/index.md` — the map of content. Find the pages relevant to the area you're
  touching and read them. The wiki is the fastest way to absorb context prior sessions spent hours
  learning.

**While working:**

- If a decision is made, a non-obvious bug is fixed, a hack is added, an integration changes, a
  domain term comes up, or an incident is handled — create or update the relevant wiki page per the
  schema, and append a one-line entry to `<wiki-root>/_log.md`. Wiki updates land in the same commit
  (or PR) as the code change that triggered them, so `git log` ties them together.
- The wiki is excluded from the formatter/build tooling (see the schema's tooling boundaries).

**For new feature work:**

`<wiki-root>/architecture.md` contains a worked end-to-end example showing the layered pattern every
new feature follows. Use it as the template.
```

The load-bearing instruction is *"wiki updates land in the same commit as the code change that
triggered them."* The discipline lives in the instructions, not in anyone's memory.

## Phase 4 — (Optional, if I confirmed it) the weekly maintenance bot

Commits from teammates using other editors/assistants never pass through the project's instruction
file, so their knowledge would never reach the wiki. A scheduled job closes that gap.

Add a CI workflow (e.g. `.github/workflows/wiki-weekly.yaml`) that runs Claude Code headless
(`claude -p --dangerously-skip-permissions`) on a weekly cron, in two passes producing **one PR for
human review**:

1. **Update pass** — reads `git log --since="8.days.ago"`, walks the schema's triggers per commit
   (skipping prior bot/wiki commits), and proposes page updates, pulling the *why* from PRs.
2. **Refactor pass** — scans the wiki itself for dangling `[[links]]`, stale claims contradicted by
   current code, duplication, contradictions, and page bloat; makes conservative fixes.

Store the two prompts under a `wiki-prompts/` (or `.github/wiki-prompts/`) folder; the schema
`CLAUDE.md` remains the source of truth both prompts consult. Bake into both: **"when in doubt,
don't write — a noisy weekly PR is worse than a quiet one."** Add a safeguard that **fails the run if
any edit lands outside the wiki directory.** Run it on a dedicated workspace with a small spending
cap. (Reference runs of this pattern cost roughly \$0.65–\$2/week.)

## Phase 5 — Tooling boundaries and verification

- Add the wiki to the formatter's ignore file (`.prettierignore` or equivalent) so `format`/
  `format-check` never rewraps Markdown prose. Ensure no TypeScript/build config `include`s it.
- If `_log.md` will see concurrent appends, declare a `union` merge driver for it in `.gitattributes`
  and keep every log entry a single self-contained line.
- Verify: every `[[link]]` resolves to a file you created; `index.md` links are all live; the
  formatter/lint/build still pass and do not touch the wiki; `git status` shows the wiki and the
  instruction-file edit staged together.
- Report a summary: wiki root path, categories created, page counts per category, what each
  bootstrap research task produced, and the instruction-file + tooling changes made.

Proceed through the phases in order. Pause for my confirmation at the end of Phase 0. Keep a
conservative bias throughout: a smaller true wiki beats a padded one.
