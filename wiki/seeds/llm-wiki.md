# LLM Wiki seed

**Source:** [`seeds/llm-wiki-seed-prompt.md`](../../seeds/llm-wiki-seed-prompt.md)
**Background:** [Karpathy's LLM Wiki in production — self-writing RFCs, postmortems, and runbooks](https://app.notion.com/p/bobcats-coding/Karpathy-s-LLM-Wiki-in-production-self-writing-RFCs-postmortems-and-runbooks-35d1c06aab6e800584cecd08ce158024)
**Concept:** [[../concepts/llm-wiki-pattern]] · **Principles:** [[../concepts/seed-design-principles]]

## What it is

An implementation of [Andrej Karpathy's LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
**directly in your codebase**: a persistent, LLM-maintained knowledge base that captures everything
the code alone doesn't show — architecture and its rationale, decisions, bug postmortems, known hacks
and tech debt, integration contracts, runbooks, and domain vocabulary.

The reasoning that today dies in chat history and PR threads gets written down once and updated
incrementally, instead of being re-derived from scratch every session. The wiki is kept current by
every future coding session as a side effect of normal work — not as a separate documentation chore.

> This very repo's `wiki/` is an instance of this pattern (with a catalog-shaped taxonomy — see
> [[../decisions/2026-06-19-docs-to-wiki]]).

## How to use it

Paste [the seed prompt](../../seeds/llm-wiki-seed-prompt.md) into Claude Code at the root of your
repository and run it in **plan mode** first. It will:

1. **Discover your project** — repo shape (single vs monorepo), stack, conventions, tooling that
   could fight a Markdown wiki (formatters, linters), and your VCS host.
2. **Propose parameters and pause for your confirmation** — the wiki root path, the initial category
   set, and which instruction file to wire in.
3. **Scaffold and bootstrap** — create the structure, write the schema, and populate real,
   provenance-backed pages by mining your code, git history, PRs, and prior Claude Code transcripts.

## What it produces

A wiki built on the [[../concepts/llm-wiki-pattern|three-layer pattern]]: **sources** (read-only
ground truth), **the wiki** (cross-linked Markdown with `[[wiki-link]]` syntax, organized into
categories like `architecture`, `decisions/`, `bugs/`, `runbooks/`, `integrations/`, `glossary`), and
**the schema** (a `CLAUDE.md` at the wiki root). The project's root instruction file is wired to point
every session at the wiki.

The load-bearing rule: **wiki updates land in the same commit (or PR) as the code change that
triggered them**, so `git log` ties them together and the wiki stays honest.

## Caveats — `_log.md` merge conflicts

The wiki keeps an append-only `_log.md` changelog. Because every session appends to it, it's a
natural **concurrent-append hotspot**: two branches that both add a log line conflict on the same
spot.

The seed's current solution is a **`union` merge driver** (declared via `.gitattributes`), which
makes Git keep both sides automatically. This **works perfectly locally** — conflicts on `_log.md`
resolve themselves on merge.

The caveat to be aware of: **GitHub does not support the `union` merge driver.** So when a `_log.md`
conflict surfaces in a pull request, you **cannot resolve it with one click in the GitHub web UI** —
it still requires a local merge to apply the union behavior. This is an open rough edge of the
pattern, not a solved problem; if you have a cleaner approach, feedback is very welcome.
