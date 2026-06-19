# Seed prompt — build the human→AI self-learning loop in this repo

> Paste everything below the line into Claude Code (Opus or Sonnet) **at the root of your repo**.
> Run it in **plan mode** first so you approve the design before any file is written.

---

You are setting up a **human-feedback → organizational-memory loop** in this repository, ported
from another team's working implementation. The goal, stated up front: capture every human-AI
correction and confirmed win during agentic work, and feed it back into the repo's own
agent-instruction files, so the harness gets a little sharper every day **without anyone sitting
down to improve it by hand** — and without asking the human to do the distilling. A model does the
distilling; the human only judges and accepts.

This is a port, not a copy. Do **not** assume my repo looks like the source repo (it does not have
its `bars/`, `skid/`, `wiki/`, or its exact build commands). Your first job is to learn how *this*
repo is shaped, then build the loop to fit it.

## The system you are building (four artifacts)

All four live under `.claude/` and are committed, so every developer gets the whole thing with zero
setup.

1. **`.claude/feedback/CLAUDE.md` — the schema file. This is ~90% of the value; spend your effort
   here.** It is the single source of truth that the distiller agent and the `/lesson` command both
   *read at runtime* rather than restate. It defines:
   - **What counts as feedback** and the two axes plus one derived flag that drive every decision:
     - **`polarity`** — `correction` (the agent did something the human had to fix) or
       `reinforcement` (the agent did something the human confirmed was right).
     - **`generality`** — `one-off` (specific to this file/task/value) or `generalizable` (a rule
       that will recur).
     - **`seenBefore`** (derived, boolean) — a *semantic* check against the journal: does this lesson
       restate one already recorded (by any developer)? Not a literal grep — paraphrases must match.
   - **The dispatch table** (the authoritative decision model; the `/lesson` command mirrors it):

     | polarity | generality | seenBefore | Action |
     | --- | --- | --- | --- |
     | any | `one-off` | — | **Discard.** Write nothing. Report it in session output so the human sees it was considered, but do not journal it. |
     | `correction` | `generalizable` | `false` | **Propose a new-rule edit** per the routing table (apply on the human's accept). |
     | `correction` | `generalizable` | `true` | **Propose a *hardening* edit** — the rule exists but didn't stick; make it firmer / more prominent. |
     | `reinforcement` | `generalizable` | `false` | **Log only — do not edit.** One "ship it" is not a rule; record it as a first sighting so a second can confirm it. |
     | `reinforcement` | `generalizable` | `true` | **Propose a "Preferred patterns" edit** — confirmed twice, now worth codifying. |

     A **contradiction** with an existing rule overrides any "propose an edit" row: surface the old
     line and the new lesson side by side, never auto-apply, let the human resolve.
   - **The routing table** — where a generalizable lesson belongs. Adapt the targets to *this* repo
     (see "Discover this repo" below). The canonical mapping is:
     - workflow / behavioral / factual rule → the nearest `CLAUDE.md` (root, or a package-scoped one)
     - recurring concern about **plans** (scope, missing edge cases, over-engineering at plan time)
       → a **plan-critic** agent
     - recurring concern about **finished work** (test quality, loose ends, pattern drift) → a
       **solution-critic** agent
     - mistake while **executing a skill / slash command** → that command's file
     - durable domain knowledge / architectural decision / integration quirk → the repo's docs or
       wiki surface
     - a repeated **win** → a "Preferred patterns" example in whichever artifact above governs it
     - **Pick the narrowest target that still captures the lesson.**
   - **Contradiction handling** — `Read` the target before editing; if the lesson conflicts with an
     existing line, flag both sides, don't overwrite.
   - **The anti-survivor-bias rule** (the part the source team cares most about): human feedback is
     overwhelmingly negative — if you only learn from corrections you build a wall of guardrails and
     forget what to *keep* doing. So positive signal is a first-class capture target, **but only from
     unambiguous signals**: explicit human praise captured via `/lesson`; a plan/solution critic
     verdict that was `approved` on its **first** try with no preceding `needs_revision`; or a PR
     **merged with zero review comments**. **Never infer a win from silence** — a quiet session looks
     identical to one that went well.
   - **The distiller's output shape** — one JSON object per candidate, one per line:
     ```json
     {"polarity":"correction|reinforcement","generality":"one-off|generalizable",
      "seenBefore":false,"lesson":"<one generalized sentence>",
      "target":"<artifact path from the routing table, or null>",
      "proposedEdit":"<concrete text to add, or a diff, or null>",
      "conflict":"<null | the exact existing line it contradicts>","provenance":"<session id / PR #>"}
     ```
     No `confidence` field — the human reviews and accepts every edit live, so the human *is* the gate.
   - **Where to read feedback from** — current session (work from the conversation already in
     context; do **not** read transcript files from disk, they may not be flushed); a past session
     (transcripts at `~/.claude/projects/<slug>/`, where `<slug>` is the absolute repo path with
     every `/` replaced by `-` — derive it, never hardcode); or a PR's review comments
     (`gh pr view <n> --comments` plus `gh api` for inline threads). **Verify the transcript slug and
     JSONL shape for this machine before documenting it** (human turns are `type:"user"` with a
     *string* `message.content`; tool results are `type:"user"` with an *array* `content` — skip
     those).
   - **The journal spec** (see artifact 4) and **what's worth logging**: journal an entry only if
     *seeing it again should change a decision* — i.e. a codified correction (`applied`/`hardened`), a
     `conflict` awaiting resolution, or a non-obvious reinforcement (first sighting `logged`). Do
     **not** journal one-offs, table-stakes behavior, or the human's own slips. When in doubt, leave
     it out.

2. **`.claude/agents/feedback-distiller.md` — the judgment engine.** A subagent (model: a fast tier
   like Sonnet/Haiku is fine) with tools **`Read, Grep, Glob, Bash` only — it must NOT have Edit or
   Write.** It reads the schema file first (Step 0), then scans the source for candidates, classifies
   each on both axes, does the semantic journal check to set `seenBefore`, routes generalizable items
   to the narrowest target, checks the target file for contradictions, and drafts a concrete
   `proposedEdit`. It returns judgment (a short narrative + the JSON objects), **never edits
   anything**. Tell it to be selective (most turns are not feedback), to default `correction` and
   lean `one-off` when unsure, and to never invent lessons to look productive.

3. **`.claude/commands/lesson.md` — the `/lesson` slash command (the human's entry point).** It
   reads the schema, determines the source from its argument (`empty` → current session; `--pr <n>`;
   `--session <id>`; anything else → a freeform note the human wrote), dispatches the distiller, then
   walks the candidates through the dispatch table — applying edits with `Edit` **only when the human
   accepts the permission prompt** (that prompt *is* the accept/reject gate; never bypass it). Then it
   appends the decision-relevant candidates to the journal. **Check that `/lesson` is not a reserved
   Claude Code built-in name in your version** (run `/help` or check the docs); if it clashes, pick a
   distinct name and wire references to it consistently.

4. **`.claude/feedback/journal/<slug>.md` — per-author append-only ledger.** One file per developer,
   `<slug>` = the local-part of `git config user.email` (before the `@`) — ascii and stable, and
   per-author specifically to avoid the merge-conflict hotspot a single shared file would create.
   **Reads are unified**: the distiller reads *every* `journal/*.md` for the seen-twice check, so a
   lesson hit by a *different* developer still counts. One line per entry:
   ```
   - <YYYY-MM-DD> — [<polarity> · <generality>] <lesson> → <target> (<status>)
   ```
   `<target>` is `—` when nothing was written; `<status>` ∈ `applied | hardened | logged | conflict`.
   Create the developer's own file lazily on first write. (Cross-author dedup is
   eventually-consistent: a teammate's entries become visible only once their branch merges — say so
   in the schema.)

## Deliberate non-goals (do not build these)

- **No automation.** No hook, no scheduled job, no PR pipeline, no queue. A human runs `/lesson` by
  hand and accepts each edit live. The human in the loop *is* the safeguard. The routing judgment has
  to earn trust over weeks of hand-running before any automation would be anything but confident
  noise. Keep it lean.
- **No personal/home-dir memory writes.** The loop writes only to repo-shared, committed surfaces so
  a lesson one developer captures benefits the whole team.
- **No `confidence` scoring**, no auto-applied edits, no silent reconciliation of contradictions.

## Discover this repo before you design (do this first, in plan mode)

Inspect and report back what you find — do not assume:

1. **Build/test/lint commands** — read `package.json` / `Makefile` / `pyproject.toml` / CI config.
   These become the *factual* routing examples in the schema (the source team's canonical example was
   "the bare test command runs in watch mode and never exits — use the single-pass one"; find this
   repo's equivalent gotchas).
2. **Existing instruction surfaces** — every `CLAUDE.md` (root and nested), `.claude/agents/`,
   `.claude/commands/`, and any docs/wiki directory. List them; these are the routing *targets*.
3. **Do plan-critic and solution-critic agents already exist?** The article found the critics are
   the natural home for most judgment-shaped lessons. If they exist, route to them. **If they don't,
   flag it** — the loop still works routing only to `CLAUDE.md` + docs, but offer to scaffold minimal
   `plan-critic` / `solution-critic` agents (verdict shape `{"verdict":"approved|needs_revision",
   "concerns":[...],"questions":[...]}`) so the routing table has those homes. Ask me which I want.
4. **Git identity & conventions** — `git config user.email` (for the journal slug), commit-message
   style, default branch, whether a `Co-Authored-By` trailer is used.
5. **The transcript path** for this machine — derive and verify the `~/.claude/projects/<slug>/`
   directory actually exists and confirm the JSONL line shapes before writing them into the schema.

## Process

1. Run the discovery above and **present a short findings summary + a plan** (which targets the
   routing table will use, whether you're creating critics, the journal slug, any repo-specific
   factual lessons to seed). Use plan mode; get my approval before writing files.
2. On approval, create the four artifacts, adapted to this repo's real paths and commands. Match each
   target file's existing voice and formatting. Keep the schema file tight and authoritative — it is
   the deliverable.
3. Do a dry run: invoke `/lesson` (or the distiller directly) against this very session or a recent
   PR, and show me the proposed candidates *without* applying, so we can sanity-check the judgment
   before trusting it.
4. Don't commit unless I ask. When you do, follow this repo's commit conventions.

Begin by exploring the repo and reporting what you find.
