# Glossary

Domain vocabulary used across this repo. Terms link to the pages that explain them in depth.

- **Artifact** — any reusable thing this repo publishes from real agentic-engineering workflows: a
  prompt, tool, technique, script, or agent. Currently the only category is seeds.

- **Seed** — a project-independent prompt you paste into Claude Code at the root of your own repo; it
  discovers your project and bootstraps a project-specific utility. See [[seeds/_index]] and
  [[concepts/seed-design-principles]]. (The seed is the *prompt*, not the thing it builds.)

- **LLM Wiki** — a persistent, LLM-maintained knowledge base kept current as a side effect of normal
  work. See [[concepts/llm-wiki-pattern]]; the `wiki/` folder is itself one.

- **The three layers** — the LLM Wiki's separation of *sources* (read-only ground truth), *the wiki*
  (writable Markdown), and *the schema* ([[CLAUDE]]). See [[concepts/llm-wiki-pattern]].

- **Schema** — the `CLAUDE.md` at a wiki's (or seed system's) root that is the single source of truth
  for conventions, read at runtime rather than restated. See [[concepts/seed-design-principles]].

- **`union` merge driver** — a built-in Git merge strategy that keeps both sides of a conflict instead
  of failing. Used for `_log.md`; works locally but unsupported in the GitHub web UI. See
  [[seeds/llm-wiki]].

- **`feedback-distiller`** — the read-only subagent in the [[seeds/lesson-command]] system that
  classifies feedback and proposes edits but never writes them.

- **polarity / generality / seenBefore** — the two axes plus derived flag that classify a feedback
  lesson and drive the dispatch table. See [[concepts/human-feedback-loop]].

- **Dispatch table** — the decision matrix mapping a lesson's polarity/generality/seenBefore to an
  action (discard, new rule, harden, log-only, codify pattern). See [[concepts/human-feedback-loop]].

- **Anti-survivor-bias** — capturing confirmed *wins*, not just corrections, so the system remembers
  what to keep doing — but only from unambiguous signals, never from silence. See
  [[concepts/human-feedback-loop]].

- **gitmoji** — the [commit-message convention](https://gitmoji.dev/) this repo uses
  (`:emoji_code: Capitalized imperative summary`). Defined in the root `CLAUDE.md`.

- **Plan mode** — Claude Code's mode where it researches and proposes a plan without making changes
  until the human approves. Seeds are meant to be run in it first; see
  [[concepts/seed-design-principles]].
