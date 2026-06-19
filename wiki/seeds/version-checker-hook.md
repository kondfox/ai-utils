# Version-checker hook seed

**Source:** [`seeds/version-checker-hook-seed.md`](../../seeds/version-checker-hook-seed.md)
**Background:** [Version-checker hook — Claude Code](https://app.notion.com/p/bobcats-coding/Version-checker-hook-Claude-Code-31b1c06aab6e809496c2de877f9b77ab)
**Reference implementation:** [kondfox/isuperhero-claude](https://github.com/kondfox/isuperhero-claude) (`scripts/check-versions.ts`, TypeScript/Bun)
**Principles:** [[../concepts/seed-design-principles]]

## What it is

A seed that bootstraps a **dependency version-checker feedback loop** in the target repo: a checker
script plus two hook integrations that keep AI-added dependencies sane. The problem it solves — when
an agent adds a dependency it often picks a version several majors behind latest, introduces a
version mismatch across workspaces, or causes a transitive conflict; without end-to-end coverage these
surface as silent, hard-to-debug runtime misbehavior. The loop turns that into automatic feedback:
after any manifest edit it reports problems back into the session, and before any commit it auto-aligns
versions and runs the full test gate.

Like every seed it is **stack-independent** ([[../concepts/seed-design-principles]]): it detects the
project's package manager and ecosystem first, then generates a checker + hooks that fit it, pausing
for confirmation before writing files.

## How to use it

Paste [the seed prompt](../../seeds/version-checker-hook-seed.md) into Claude Code at the root of your
repository. It proceeds in phases:

1. **Detect the ecosystem** — manifest, package-manager CLI verbs (outdated / lockfile / override /
   install), workspace shape, existing pre-commit mechanism, and test/lint/type commands — then pauses
   for your confirmation.
2. **Checker script** (`scripts/check-versions.<ext>`) — flag-driven: version-mismatch, outdated
   (informational only), and transitive-conflict detection, plus `--fix` to auto-align.
3. **PostToolUse hook** — runs the fast `--mismatch --json` check after manifest edits and injects the
   result into the session.
4. **Pre-commit gate** — runs `--fix` plus the project's lint/type/test/E2E commands so only
   version-aligned, green commits land.

For a JS/TS project, reuse and adapt the TypeScript/Bun reference implementation linked above;
otherwise build the equivalent for the project's ecosystem.

## What it produces

A `scripts/check-versions.<ext>` checker (wired into the task runner), a `.claude/hooks/check-versions.sh`
PostToolUse hook registered in `.claude/settings.json`, and an addition to the project's pre-commit
mechanism. All paths are resolved portably (`git rev-parse --show-toplevel`, runtime from `PATH`) so
the result works across machines and checkouts.

## Caveats

- **`--fix` aligns to the highest pinned/resolved version.** On a **legacy project whose deps are far
  behind**, a forced alignment can cascade — run report-only first there and upgrade in a dedicated PR.
- A `--fix` that bumps to a not-yet-compatible major is meant to be **caught by the test gate**; the
  loop then iterates until green. That iteration is the point — don't suppress it.
- Keep dependency bumps in a **separate commit/PR** from feature work so the diff stays reviewable.
- **Outdated is informational only** — it never fails the run; only unresolved mismatches do.
