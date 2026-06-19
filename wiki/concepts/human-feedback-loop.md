# The human-feedback → organizational-memory loop

A loop that **captures the corrections and confirmed wins a human gives an AI agent during normal
work, and feeds them back into the repo's own agent-instruction files** — so the harness gets sharper
over time without anyone sitting down to improve it by hand. A model does the distilling; the human
only judges and accepts. Shipped as a seed: [[../seeds/lesson-command]].

## The core insight: a model distills, a human gates

The expensive part of learning from feedback is *distilling* a one-off correction into a general rule
and *routing* it to the right instruction file. A fast model does that and proposes a concrete edit.
The human never writes the rule — they only accept or reject the proposed edit at a live permission
prompt. That prompt **is** the gate, which is why the loop needs no confidence scoring and no
automation.

## Two axes drive every decision

Every candidate lesson is classified on:

- **polarity** — `correction` (the agent did something the human had to fix) vs `reinforcement` (the
  human confirmed something was right).
- **generality** — `one-off` (specific to this file/value) vs `generalizable` (a rule that recurs).

A derived **seenBefore** flag (a *semantic* check against the journal, not a grep) decides whether a
generalizable lesson is new, needs hardening, or is now confirmed twice. These feed a dispatch table
that decides: discard, propose a new rule, harden an existing rule, log-only, or codify a preferred
pattern.

## Anti-survivor-bias (the part that's easy to get wrong)

Human feedback skews overwhelmingly negative. If you only learn from corrections you build a wall of
guardrails and forget what to *keep* doing. So **confirmed wins are a first-class capture target** —
but only from unambiguous signals (explicit praise, a first-try critic approval, a PR merged with zero
comments). The loop **never infers a win from silence**, because a quiet session looks identical to
one that went well.

## Relationship to the other artifacts

Shares the [[seed-design-principles]] every seed follows — notably human-in-the-loop and
schema-as-source-of-truth (the dispatch/routing tables live in a schema file both the distiller
subagent and the `/lesson` command read at runtime). Background: the Notion writeup linked from
[[../seeds/lesson-command]].
