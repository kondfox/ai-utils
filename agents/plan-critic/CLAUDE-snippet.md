# plan-critic — wiring snippet

> Paste the section below into your project's root `CLAUDE.md` (or `AGENTS.md`). It assumes
> `plan-critic.md` is installed in `.claude/agents/`. This is what actually invokes the agent —
> the agent file alone only defines it.

---

## Plan mode challenge protocol

When plan mode is active, after drafting your plan and before calling `ExitPlanMode`:

1. Dispatch the `plan-critic` subagent via the Task tool, passing the current plan as the prompt.
2. If the critic returns `"needs_revision"`, revise the plan to address every concern and
   re-dispatch. The critic — not the main agent — decides when the plan is "actually a solid and
   elegant solution"; do not self-approve.
3. Only call `ExitPlanMode` once the critic returns `"approved"`.
4. Cap at **5 rounds**. If the critic still isn't satisfied after 5 rounds, stop iterating, present
   the latest plan together with the critic's remaining concerns to the user, and let them decide.
