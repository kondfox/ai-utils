# seeds/

One page per published **seed** artifact. A seed is a project-independent prompt you paste into Claude
Code at the root of your own repo; it discovers your project and bootstraps a project-specific utility.
See [[../concepts/seed-design-principles]] for the philosophy all seeds share.

These pages are the long-form companions to the README's Seeds table — the README links here.

**Page format** (see [[../CLAUDE]] for the full skeleton): What it is / How to use it / What it
produces / Caveats / Links to the prompt file and any external writeup.

## Pages

- [[llm-wiki]] — an in-codebase implementation of Karpathy's LLM Wiki (the [[../concepts/llm-wiki-pattern]]).
- [[lesson-command]] — a [[../concepts/human-feedback-loop]] that self-improves the repo's agent instructions.
- [[version-checker-hook]] — a dependency version-checker feedback loop (checker script + PostToolUse hook + pre-commit gate).
