# Seed prompt — set up a daily "quick win" Claude routine

> Paste everything below the line into Claude Code, running it from the root of the repository you
> want the routine for. It inspects the project and produces three things: (1) the routine prompt,
> tailored to this repo; (2) the environment setup script, tailored to this toolchain; (3) a
> configuration checklist for wiring it up as a Claude Code Routine in the web UI. It is
> stack-independent — it detects the toolchain rather than assuming one.
>
> Background: a Claude Code **Routine** is a saved cloud configuration (prompt + repo(s) +
> environment + connectors + trigger). On its schedule, Anthropic spins up a fresh cloud session,
> clones the repo, runs the prompt unattended to completion, and shuts down. By default a routine can
> only push to `claude/`-prefixed branches and there is no human in the loop — so everything you'd
> say in the moment has to be written down in advance.

---

You are going to set up a **daily "quick win" routine** for **this** repository: every morning an
unattended cloud agent finds one tiny, safe, fully-verified improvement and opens a pull request for
it. Quick wins are the things no one ever tickets — an `any` that should be `unknown`, a stale TODO
with a one-line fix, a doc page contradicting the code, a patch-level Dependabot bump. Individually
none is worth a context switch; left alone they accumulate. The routine changes the economics, not
the priorities.

This is a **stack-independent** prompt. Do **not** assume any particular language or package manager.
Detect this project's toolchain first and tailor everything to it. Two things are non-negotiable
regardless of stack: the unattended-agent discipline rules (below) are kept verbatim, and the setup
script must follow the sandbox gotchas (below) exactly.

## Phase 0 — Inspect the project, then confirm

Determine and report:

- **Repo identity is derived at runtime, never hardcoded.** The routine runs inside the cloned repo
  with `gh` authenticated, so: use `gh pr list …` with no `--repo` (it infers from the clone), and
  `gh api repos/{owner}/{repo}/…` (gh expands the literal `{owner}`/`{repo}` tokens). Do not write a
  literal `org/repo` anywhere in the generated prompt.
- **Stack & toolchain.** Language(s), package manager and version, monorepo/workspace shape, Node/
  runtime version constraints.
- **Verification commands.** The project's own type-check, lint, unit-test (and how to scope it to an
  area), and format-check / E2E commands — these go into Step 4 of the routine.
- **Knowledge sources for candidates.** Is there a project wiki (e.g. from the LLM-Wiki seed)? Where
  do sources live (the `src`/app paths)? Is Dependabot enabled (does `gh api
  repos/{owner}/{repo}/dependabot/alerts` return data)?
- **Critic loop.** Does the repo's `CLAUDE.md`/`AGENTS.md` define a `solution-critic` (or any
  before-done review loop)? If so, the "size budget outranks the critic" precedence rule must be the
  first hard rule. If not, omit that clause.
- **Toolchain bootstrap behavior (drives the setup script).** Does the package manager download its
  own binary at runtime (e.g. Corepack fetching Yarn)? Does the repo **vendor/pin** that binary
  (e.g. `.yarn/releases/*.cjs`)? Is there internal tooling that must be built before the checks run?
  A unique file that identifies the repo root (for the find-repo loop)?
- **Default branch** and the routine's push-branch prefix (`claude/`).

Then **ask me to confirm** the detected toolchain and the candidate sources before generating files.

## Phase 1 — Generate the routine prompt

Write the routine prompt to `quick-win-routine.prompt.md` (this is the text pasted into the routine's
prompt field, not a repo file you must commit). Keep the **universal core verbatim** and fill only the
bracketed project slots from Phase 0.

Open with: "You are running as an unattended daily routine in **this** repository (`<stack summary>`).
Your job: find exactly ONE 'quick win', implement it, and open a pull request. If no qualifying quick
win exists, or the build environment is broken, exit without opening a PR." If a critic loop exists,
add: "This routine inherits the repo's `CLAUDE.md`, including the <critic> loop. Before anything else,
read the 'Scope discipline' hard rule: for quick wins, the size budget outranks the critic."

Then these steps (keep the structure and the discipline; substitute commands/paths):

- **Step 0 — Preflight.** Read `/tmp/build-tools-status` (written by the setup script). `ok` →
  proceed. Anything else / missing → the build is broken and every test result would be
  untrustworthy: print the last ~40 lines of `/tmp/setup.log` into the run output, then exit without
  a PR. A skipped day on a broken build is a successful run.
- **Step 1 — Dedupe.** `gh pr list --state open --json number,title,headRefName,labels`. Treat as
  off-limits any PR on a `claude/` branch, any title containing `[quick win]`, any `quick-win` label.
  Never pick something overlapping an open PR; never retry a previously closed-unmerged quick win.
- **Step 2 — Gather candidates (priority order).** (1) **Dependabot**:
  `gh api repos/{owner}/{repo}/dependabot/alerts --jq '[.[] | select(.state == "open")]'` — only
  patch/minor-bump-fixable alerts; on 403 skip silently. (2) **The wiki** (if present): outdated
  pages, broken internal links, TODO markers, docs contradicting code. (3) **The codebase** (the
  detected src paths): TODO/FIXME with obvious resolutions, unused imports/exports, dead code,
  narrowable `any`/loose types, typos. Stop at 3–5 viable candidates.
- **Step 3 — Pick one.** Highest-priority that is safest and smallest: security patch > code cleanup >
  docs fix, but drop down if the higher one is risky/large. When in doubt, pick smaller.
- **Step 4 — Implement & verify.** Change on a new branch (auto `claude/`-prefixed). Run the detected
  checks, scoped as tightly as possible (type-check, lint, area-scoped tests, format-check). **A
  failing test is not "pre-existing" by assertion** — preflight confirmed health, so prove it
  (`git stash`, re-run on clean default branch, compare) or show the files are unrelated to the diff;
  a failure in a file you touched is yours. If a fix isn't obvious within budget, abandon and try the
  next candidate; **after two abandoned candidates, exit without a PR.** **Pre-PR size gate:**
  `git diff --stat origin/<default-branch>...HEAD`; if additions exceed ~50 lines (excluding
  lockfiles/generated), trim to the core change or abandon — do not open the PR.
- **Step 5 — Open the PR.** Title `chore: <short description> [quick win]`. Body: **What & why**
  (1–3 sentences), **Source** (which signal surfaced it — Dependabot #, wiki page, file path),
  **Verification** (exact commands + results), **Risk** (one sentence on blast radius), and a
  transcript link: `https://claude.ai/code/${CLAUDE_CODE_REMOTE_SESSION_ID/#cse_/session_}`. Add the
  `quick-win` label only if it already exists. Exactly one PR per run.
- **Step 6 — Nothing qualifies?** Do nothing. No empty/speculative PR, no lowered bar. A skipped day
  is a successful run. (This is the hardest rule to make an agent believe — state it plainly.)

**What counts as a quick win** (keep verbatim): ≤~50 changed lines (excl. lockfiles/generated); one
concern per PR; no public-API/schema changes, no refactor touching >3 files; must be verifiable by
existing checks or be trivially safe. Examples: Dependabot patch, dead-code/unused-export removal,
typo, type tightening, obvious TODO resolution, lint-warning fix, stale comment/wiki page. **NOT**
quick wins: major bumps, breaking upgrades, architectural/perf changes needing benchmarks, anything
in sensitive logic (auth, payments), CI/CD changes.

**Hard rules** (keep verbatim; include the first only if a critic loop exists):
- **Scope discipline (this rule outranks the solution-critic).** The size budget (~50 lines, ≤3
  files) is binding. If the critic — or your own review — demands changes that push past it (e.g.
  backfilling tests for code this PR doesn't modify), abandon the candidate; do not grow the PR. Only
  add/adjust tests covering the lines you changed. Never backfill pre-existing coverage gaps.
- Never push to the default branch or any non-`claude/` branch.
- Never modify lockfiles except as a direct consequence of the chosen dependency patch.
- Never touch files outside the project and its wiki.
- Never commit secrets, tokens, or `.env` content.
- Never disable, skip, or weaken a test to make the build pass.

## Phase 2 — Generate the environment setup script

Write `quick-win-setup.sh` (pasted into the routine's environment setup field). It runs **before**
Claude launches, in the **home directory, not the repo**, and its results are snapshot-cached. It must
obey these hard-won sandbox rules (they generalize across stacks):

- **Always `exit 0`.** A non-zero exit aborts the whole session before the routine's preflight can
  run. Signal failure **in-band** instead.
- **Write a health sentinel.** Default `failed` into `/tmp/build-tools-status` first; flip to `ok`
  only after the toolchain builds successfully. Tee all output to `/tmp/setup.log` (`exec > >(tee
  /tmp/setup.log) 2>&1`). The routine's Step 0 reads these.
- **Find the repo first.** Loop over `"$PWD"` and `"$PWD"/*` and detect the repo by the unique
  root-marker file from Phase 0, then `cd` into it. If not found, log and `exit 0` (leaving
  `failed`).
- **Build files with `printf`, not here-documents.** The web UI textarea indents pasted scripts,
  which silently breaks heredoc terminators.
- **Handle self-downloading tooling.** If the package manager fetches its binary at runtime and the
  sandbox blocks that host, and the repo vendors the binary, shadow the shim with a wrapper in
  `~/.local/bin` that calls the vendored binary directly, and prepend it to `PATH`. (If the toolchain
  installs cleanly offline, skip this.)
- Then run the offline install (e.g. `--immutable`/`--frozen-lockfile` equivalent) and build any
  internal tooling the checks depend on, in dependency order. On success write `ok`.

Use the article's reference script (Yarn-4-vendored example) as the shape, adapted to this project's
package manager, root-marker, and internal build steps.

## Phase 3 — Routine configuration checklist (print, don't execute)

Output the steps to create the Routine in the Claude Code web UI, since these are console settings,
not repo files:

- **Prompt:** paste `quick-win-routine.prompt.md`.
- **Repository:** this repo. **Trigger:** daily schedule (e.g. 07:00 local).
- **Environment:** custom; **setup script** = `quick-win-setup.sh`. **Network access:** "Trusted"
  covers GitHub + common registries — but verify the toolchain doesn't fetch binaries from a blocked
  host (prefer the vendored binary, per Phase 2). **Env vars:** stored in **plaintext**, visible to
  any environment editor — treat as config, not secrets; built-in GitHub access usually covers
  everything (incl. the Dependabot API), so aim for **none**.
- **Guardrails/toggles:** leave **"unrestricted git push" OFF** (keeps the `claude/`-branch guard).
  Turn **"Auto-fix pull requests" ON** so the routine watches CI/review comments on its own PRs and
  pushes follow-ups to the same branch.
- Note the setup-script cache rebuilds on edit or network-setting change and expires after ~a week.

## Notes

- This pairs with the LLM-Wiki seed: source #2 (the wiki) only fires if the project has one. If it
  doesn't, the routine still works on Dependabot + codebase sources.
- The deepest lesson, worth stating in the prompt: when you stack autonomous layers (routine →
  critic), their conflicts are no longer arbitrated by a human in the moment, so the precedence has
  to be written down. Here: the size budget outranks the critic.

Proceed in order; pause for my confirmation at the end of Phase 0.
