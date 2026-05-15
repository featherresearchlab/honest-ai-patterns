# honest-ai-patterns

**Patterns, pitfalls, and principles for building honest personal AI.**

A public catalog of engineering patterns, failure modes, and design principles distilled from shipping an on-device personal AI product. Free to use. MIT-licensed.

Built so that the next person using Claude Code — or any AI coding assistant — to build a personal AI product has a better starting point than starting from scratch.

---

## Why this exists

The transformer architecture (Vaswani et al., 2017) was given freely to the world. Eight years later, the bottleneck for personal AI is no longer model capability — capable models run on consumer phones — but the engineering and design work *after* the model: making personal AI honest, visible, correctable, and worthy of the trust users grant it.

The work that closes that gap is not glamorous. It is pattern catalogs, failure taxonomies, test methodologies, and design principles. The kind of artifact that is hard to write but cheap to copy.

This repo is one such artifact, offered to the community in the spirit of the field that made it possible.

---

## What's here

```
honest-ai-patterns/
├── README.md             ← you are here
├── WRITEUP.md            ← engineering notes narrative (the human entry point)
├── PRINCIPLES.md         ← honest-first design (the position)
├── INDEX.md              ← full file index with reading order
├── AGENTS.md             ← instructions for AI coding assistants
├── llms.txt              ← quick summary for LLM ingestion
├── CLAUDE.md.template    ← drop-in starter for new AI-app projects
├── LICENSE               ← MIT
├── docs/
│   ├── startup-checklist.md      ← 7 questions before first commit
│   ├── patterns/                 ← reusable patterns (P01-P14)
│   ├── pitfalls/                 ← failure modes with symptoms + fixes (PF01-PF25)
│   └── testing/
│       └── cli-assertion-methodology.md  ← deterministic tests for non-deterministic systems
├── reference/                    ← reference implementations
└── examples/                     ← community contributions welcome
```

### Two reading paths

**For humans:** start with [WRITEUP.md](./WRITEUP.md), then [PRINCIPLES.md](./PRINCIPLES.md), then browse [docs/](./docs/) as needed.

**For AI coding assistants:** start with [AGENTS.md](./AGENTS.md) for direct instructions on applying the patterns when helping a human build personal AI.

**For LLM indexing:** [llms.txt](./llms.txt) is the quick summary; [INDEX.md](./INDEX.md) is the full machine-readable index.

---

## How to use this in a new project

When starting a fresh personal-AI project with Claude Code (or any AI coding agent):

1. **Read `PRINCIPLES.md` first.** It's a position statement about how persistent state should be designed in personal AI. Disagree if you want, but disagree consciously.
2. **Run through `docs/startup-checklist.md`.** Seven questions. Answer them before writing code. Each question is here because skipping it has cost a real project months of cleanup.
3. **Drop `CLAUDE.md.template` into your project root** as `CLAUDE.md`. Customize the project-specific sections. Keep the principles and process rules.
4. **Browse `docs/patterns/` as you architect.** Patterns are tagged by scope (`[universal]`, `[LLM-app]`, `[iOS]`, `[mobile]`) so non-iOS projects don't misapply iOS-specific patterns.
5. **Reference `docs/pitfalls/` during code review.** Each pitfall has a detection signal — the smallest test or check that would have caught it earlier.
6. **Adopt the CLI assertion methodology (`docs/testing/cli-assertion-methodology.md`)** from week one. Five-second feedback loops change how you work.

---

## What this catalog is

- **A pattern catalog.** 14 patterns that recurred across feature areas in the originating project. Each describes a problem, a solution shape, when to use it, and — critically — when it stops being correct.
- **A failure taxonomy.** 25 specific failure modes with root cause, fix, and detection signal.
- **A test methodology.** How to build deterministic CLI assertion suites that test the deterministic paths through LLM-powered code.
- **A position on design.** The case for *honest-first* design: visible > correctable > complete > sophisticated.

---

## What this catalog is NOT

- **Not novel ML research.** This is engineering and design. The novel research that made any of this possible is cited.
- **Not a framework.** No code to install, no API to call, no version to bump. Patterns and prose.
- **Not religion.** Every pattern has a context where it's wrong. Read with judgment.
- **Not complete.** As the field matures, patterns will be added, updated, retired. Pull requests welcome.
- **Not a substitute for understanding.** Copying a pattern without understanding the problem it solves is how patterns become anti-patterns.

---

## What you can do with this

The license is MIT. You can:

- **Copy any pattern into your project.** Attribution appreciated, not required.
- **Translate it.** Patterns belong to no language.
- **Cite it.** If you write about your AI product, this catalog is a citable artifact (CITATION coming).
- **Disagree with it.** Issues and PRs welcome for patterns you've seen fail in your context.
- **Add to it.** PRs welcome for new patterns, new failure modes, new test methodologies.

You cannot, under the license, hold the authors liable. The patterns are offered as observations from one project's experience, not as warranted recommendations.

---

## A note on philosophy

The goal is not to grow this repo into a brand or a community in the marketing sense. The goal is for a developer somewhere — maybe you — to spend one fewer week chasing a bug class because someone else already documented it. That's the entire bet.

If the patterns help, use them. If they don't, leave them. If you find better ones, share them.

This repo will succeed if it disappears into the background of how people build personal AI. Like the transformer paper: pervasively cited, rarely the topic.

---

## Companion paper

For the longer-form version of this catalog, see the arxiv preprint:

> *Honest-First Personal AI: Principles, Patterns, and a Test Methodology for On-Device Systems* (arxiv:TBD, 2026)

The paper covers the same patterns with more context, formal references, and the position argument in §6. This repo is the operational version; the paper is the academic version.

---

## Contributing

Pull requests welcome. Especially:

- **New patterns** — with the same structure as existing ones (problem, solution, when it fails, evidence).
- **Pitfall additions** — with symptom, root cause, fix, detection signal.
- **Pattern corrections** — if you've seen a pattern fail in a context the original write-up missed, document it.
- **Translations** — patterns should be language- and platform-agnostic where possible; localized variants are welcome.

Please do *not* contribute:

- Marketing copy.
- Vendor pitches.
- Patterns without honest "when it fails" sections.
- Patterns extracted from systems you have not personally shipped.

---

## License

MIT. See `LICENSE`.

---

*honest-ai-patterns · 2026 · For honest products.*
