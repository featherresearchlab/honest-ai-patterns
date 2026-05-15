# Index — All files

All files in this repository, in reading-priority order. Use this as the navigation hub.

---

## Start here

1. [README.md](./README.md) — What this repo is and how to use it
2. [WRITEUP.md](./WRITEUP.md) — Engineering notes narrative (the human entry point)
3. [PRINCIPLES.md](./PRINCIPLES.md) — Honest-first design (the central principle)

## Operational artifacts

- [CLAUDE.md.template](./CLAUDE.md.template) — Starter CLAUDE.md for new AI-app projects
- [docs/startup-checklist.md](./docs/startup-checklist.md) — 7 questions before first commit
- [docs/testing/cli-assertion-methodology.md](./docs/testing/cli-assertion-methodology.md) — Deterministic testing for non-deterministic LLM pipelines

## Patterns

14 reusable engineering patterns. Tagged by scope: `[universal]`, `[LLM-app]`, `[mobile]`, `[iOS]`.

| ID | Title | Status |
|---|---|---|
| [P01](./docs/patterns/P01-pre-filter-before-llm.md) | Pre-filter before LLM | ✅ |
| [P02](./docs/patterns/P02-three-tier-extraction.md) | Three-tier extraction | ✅ |
| [P03](./docs/patterns/P03-memory-facade.md) | Memory façade with multiple backings | ✅ |
| P04 | Source-aware entity merge | _to be written_ |
| [P05](./docs/patterns/P05-first-person-disclosure-guard.md) | First-person disclosure overwrite guard | ✅ |
| [P06](./docs/patterns/P06-word-boundary-tokenization.md) | Word-boundary tokenization for safety filters | ✅ |
| P07 | Idempotent versioned migrations | _to be written_ |
| P08 | Journey log with correlation IDs | _to be written_ |
| P09 | `/loop` self-paced iteration | _to be written_ |
| P10 | CLI assertion suite | [see methodology](./docs/testing/cli-assertion-methodology.md) |
| P11 | `@MainActor` batching for serialized writes | _to be written_ |
| P12 | Defer-unload for in-flight resources | _to be written_ |
| P13 | Engine migration without dual-resident runtime | _to be written_ |
| P14 | Bench harness with sampler state restore | _to be written_ |

## Pitfalls

25 production failure modes with symptoms, fixes, and detection signals.

| ID | Title | Status |
|---|---|---|
| [PF01](./docs/pitfalls/PF01-substring-matching-short-tokens.md) | Substring matching against short tokens | ✅ |
| PF02 | Double-resume of CheckedContinuation | _to be written_ |
| PF03 | MainActor.run per loop iteration stalls UI | _to be written_ |
| PF04 | Await in parallel-fanout gate serializes fast path | _to be written_ |
| [PF05](./docs/pitfalls/PF05-llm-hallucinates-user-self-disclosures.md) | LLM hallucinates user-self disclosures | ✅ |
| PF06 | LLM emits object when prompt asked for array | _to be written_ |
| PF07 | Model echoes user's question as answer | _to be written_ |
| PF08 | Model self-identifies as base model | _to be written_ |
| PF09 | Extracted "interest" field accumulates pollution | _to be written_ |
| PF10 | Keyword router substring-matches inside names | _to be written_ |
| PF11 | Decorative LLM call warms cold model | _to be written_ |
| PF12 | Multiple sequential classifier calls per turn | _to be written_ |
| PF13 | "Tell me about" creates person row in knowledge graph | _to be written_ |
| PF14 | Self-test plants synthetic data on real entity | _to be written_ |
| PF15 | Imported data rendered alongside disclosed data | _to be written_ |
| PF16 | Disclosure prefilter and detector drift apart | _to be written_ |
| [PF17](./docs/pitfalls/PF17-dual-engine-memory-pressure.md) | Dual on-device LLM engines competing for Metal | ✅ |
| PF18 | Blue launch flash from default colorset | _to be written_ |
| PF19 | Button(role: .destructive) renders white-on-white | _to be written_ |
| PF20 | Reactions don't appear after long-press | _to be written_ |
| PF21 | Stale deploy from DerivedData glob | _to be written_ |
| PF22 | Reading freed C pointer across await suspension | _to be written_ |
| PF23 | Unrelated fix creeps into focused PR | _to be written_ |
| PF24 | In-session task list lost when session compacts | _to be written_ |
| PF25 | Author-graded retrospective misses substring leaks | _to be written_ |

## For AI agents

- [AGENTS.md](./AGENTS.md) — Direct instructions for AI coding assistants
- [llms.txt](./llms.txt) — Quick summary for LLM ingestion

## Meta

- [LICENSE](./LICENSE) — MIT

---

## Progress as of the first publication

- 5 of 14 patterns written
- 2 of 25 pitfalls written

The remaining files are placeholders. Pull requests welcome to fill them in. Each new file should match the existing structure (problem, solution, when it fails, evidence, related, anti-pattern) and stay under 600 words to keep the catalog browsable.

---

*Last updated 2026-05-15.*
