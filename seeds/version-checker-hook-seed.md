# Seed prompt — set up a dependency version-checker hook

> Paste everything below the line into Claude Code, running it from the root of the repository you
> want it in. It is stack-independent: it detects your package manager and generates a checker +
> hooks that fit it. A reference TypeScript/Bun implementation
> (https://github.com/kondfox/isuperhero-claude — `scripts/check-versions.ts`) exists; reuse and
> adapt it if this is a JS/TS project, otherwise build the equivalent for this project's ecosystem.

---

You are going to add a **dependency version-checker feedback loop** to **this** repository: a script
plus two hook integrations that keep AI-added dependencies sane. The problem it solves: when an agent
adds a dependency it often picks a version several majors behind latest, or introduces a version
mismatch across workspaces, or a transitive conflict — and without E2E coverage these surface as
silent, hard-to-debug runtime misbehavior. This hook turns that into an automatic, tight feedback
loop: after any manifest edit it reports the problems back into the session, and before any commit it
auto-aligns versions and runs the full test gate.

This is a **stack-independent** prompt. Do **not** assume JavaScript/Node/bun. Detect this project's
ecosystem first and build everything to fit it.

## Phase 0 — Detect the ecosystem and confirm

Inspect the repo and determine:

- **Manifest & ecosystem.** `package.json` (JS/TS), `pyproject.toml`/`requirements.txt` (Python),
  `go.mod` (Go), `Cargo.toml` (Rust), `pom.xml`/`build.gradle` (JVM), `Gemfile`, etc.
- **Package manager & its CLI verbs** — the equivalents of the four capabilities the script needs:
  - list outdated deps (e.g. `bun outdated`, `npm outdated --json`, `pnpm outdated`,
    `pip list --outdated --format=json`, `uv pip list --outdated`, `go list -m -u all`,
    `cargo outdated`),
  - the lockfile to parse for transitive versions (`bun.lock`, `package-lock.json`, `pnpm-lock.yaml`,
    `Cargo.lock`, `go.sum`, `uv.lock`),
  - the override/pin mechanism (`overrides`/`resolutions` in `package.json`, `[tool.uv] override`,
    Cargo `[patch]`, Go `replace`),
  - install/sync to apply changes.
- **Workspace/monorepo shape** — is there a workspace list to walk (npm `workspaces`, pnpm
  `packages:`, Cargo workspace `members`, Go modules), or a single manifest?
- **Existing pre-commit mechanism** — Husky, `pre-commit` (the Python framework), `lefthook`, a raw
  `.git/hooks/pre-commit`, or none. Reuse what's there rather than introducing a second system.
- **Test/lint/type commands** — the project's own scripts for lint-fix, type-check, unit tests, and
  E2E, to wire into the commit gate.

Report what you found and the script language you'll use (the checker only needs to *read* manifests
and *call* the package-manager CLI, so it can be written in whatever is most natural — a shell script,
the project's own language, or a small Bun/Node/Python script if that runtime is available). Then
**ask me to confirm** before writing files. Use the JS/TS reference implementation as the starting
point if this is a JS/TS project.

## Phase 1 — The checker script

Create a single script (default location `scripts/check-versions.<ext>`) that implements these
capabilities, driven by flags:

- **Version-mismatch detection** — the same external dependency pinned to *different* ranges across
  workspaces. (Skip workspace-internal deps like `workspace:*`.) For a single-manifest project this is
  a no-op; keep the check anyway so it scales if workspaces are added later.
- **Outdated detection** — wraps the package manager's "outdated" command across the root and every
  workspace, dedupes, and classifies each as patch / minor / major by semver delta. **Informational
  only — never fails the run on outdated alone.**
- **Transitive-conflict detection** — parses the lockfile, groups resolved versions by package name,
  flags any package resolved to more than one version.
- **`--fix`** — auto-resolves: rewrite mismatched ranges to the highest pinned range across
  workspaces; add an override/pin (root manifest) for each transitive conflict, choosing the highest
  resolved version; then run install/sync to apply.
- **Modes / flags:**
  - `--mismatch` — mismatch check only (fast, no network) — for the PostToolUse hook,
  - `--json` — machine-readable output `{ "versionCheck": { "status": "pass"|"fail", ... } }` — for
    the hook to inject into the session,
  - `--fix` — the auto-resolve path above — for pre-commit,
  - no flag — all checks, human-readable colored output.
- **Exit codes:** `0` when clean (or `--fix` resolved everything); non-zero (e.g. `2`) when
  *mismatches* remain and `--fix` was not passed. Outdated-only must stay exit `0`.

Add a convenience entry to the project's task runner (e.g. a `check-versions` script in
`package.json`, a `Makefile` target, or a `justfile` recipe) so it's runnable by name.

Wire it to the project root **portably** — resolve the repo root with `git rev-parse --show-toplevel`
or the script-relative path, **not** a hardcoded absolute path, and invoke the runtime from `PATH`,
**not** a hardcoded `~/.../bin/...`.

## Phase 2 — PostToolUse hook (inject results into the session)

Create `.claude/hooks/check-versions.sh` that:

1. reads the hook JSON payload from stdin,
2. extracts `.tool_input.file_path`,
3. **exits 0 immediately unless the edited file is a manifest** (`package.json`, `pyproject.toml`,
   `go.mod`, `Cargo.toml`, … — match the ecosystem from Phase 0),
4. otherwise runs the checker in fast mode (`--mismatch --json`) from the repo root and echoes the
   result so it lands in Claude's context.

Resolve the repo root via `git rev-parse --show-toplevel` (or `$CLAUDE_PROJECT_DIR`) — **no hardcoded
absolute paths**, so the file is portable across machines and checkouts. `chmod +x` it.

Register it in `.claude/settings.json` under `hooks.PostToolUse` with matcher `Edit|Write`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/check-versions.sh" }
        ]
      }
    ]
  }
}
```

(If a `PostToolUse` block already exists, add this entry to it rather than overwriting.)

## Phase 3 — Pre-commit gate (enforce before commit)

Add the checker's `--fix` to the project's existing pre-commit mechanism (Phase 0), followed by the
project's lint-fix, type-check, unit-test, and E2E commands — so only version-aligned, green commits
enter the repo. Generic shape (adapt commands and the absolute path of the runtime — prefer `PATH`):

```sh
#!/usr/bin/env sh
cd "$(git rev-parse --show-toplevel)"
<run> scripts/check-versions.<ext> --fix
<lint-fix command>
<type-check command>
<unit-test command>
<e2e command>
```

If the repo has no pre-commit framework yet, prefer the lightest one idiomatic to the stack (Husky for
JS/TS, the `pre-commit` framework for Python, lefthook otherwise) rather than a raw `.git/hooks` file.

## Phase 4 — Verify and report

- Trigger the PostToolUse path: make a trivial manifest edit and confirm the hook fires only for
  manifests and injects JSON; confirm a non-manifest edit produces no output.
- Run the checker directly in all modes (`--mismatch`, `--json`, plain, `--fix` on a scratch branch)
  and confirm exit codes (clean → 0, unresolved mismatch → non-zero, outdated-only → 0).
- Run the pre-commit gate once end to end.
- Report: ecosystem detected, package-manager verbs used, files created, the two integration points
  wired, and any deviations from the reference behavior.

### Notes & caveats (from production use)

- When `--fix` bumps to a major that isn't yet cross-compatible, expect the *test gate* to catch it;
  the loop then iterates on minor versions (and may apply a small temporary patch) until green. That
  iteration is the point — don't suppress it.
- Be cautious enabling auto-`--fix` on a **legacy project whose deps are far behind** — a forced
  highest-version alignment can cascade. Consider running report-only first there, and upgrading in a
  dedicated PR.
- Keep dependency bumps in a **separate commit/PR** from feature work so the diff stays reviewable.

Proceed in order; pause for my confirmation at the end of Phase 0.
