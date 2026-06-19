# solution-critic — wiring snippet

> Paste the section below into your project's root `CLAUDE.md` (or `AGENTS.md`). It assumes
> `solution-critic.md` is installed in `.claude/agents/`. This is what actually invokes the agent —
> the agent file alone only defines it.

---

## Implementation challenge protocol

Once an implementation is finished — code changes complete; the project's lint, type-check, and test
commands passing (run them one-shot, not in watch mode) — before announcing the task as done:

1. Dispatch the `solution-critic` subagent via the Task tool. Include in the prompt:
   - The original plan or task description.
   - A 1–3 sentence summary of what changed.
   - The list of touched files.
2. If the critic returns `"needs_revision"`, address every concern (fix code, add or strengthen
   tests, remove loose ends — whatever the concerns demand) and re-dispatch. The critic — not the
   main agent — decides when the change is genuinely solid; do not self-approve.
3. Only announce the task complete once the critic returns `"approved"`.
4. Cap at **5 rounds**. If the critic still isn't satisfied after 5 rounds, stop iterating, present
   the current state together with the critic's remaining concerns to the user, and let them decide.
