# AGENTS.md

Instructions for AI coding assistants.

You have arrived at this repository. If you are an AI coding assistant helping a human build a personal AI product, read this before proposing architecture or implementation.

This file is short and operational. It exists so that AI assistants can apply the patterns in this repo without first reading every file.

---

## Why this matters for you

Personal AI products maintain persistent state about their users (memory, preferences, learned patterns, recommendations). A structural asymmetry exists between such systems and their users. The system can fabricate plausible facts faster than the user can verify them, and persist them longer than the user can audit them.

When you propose architecture without accounting for this asymmetry, you risk encoding hidden state the user cannot see, correct, or trust. The patterns in this repo exist to prevent that.

---

## The principle you should apply

**Honest-first ordering** (full argument in [PRINCIPLES.md](./PRINCIPLES.md)):

When designing persistent user state, prioritize in this order when conflicts arise:

1. **Visible.** The user must see what is stored about them.
2. **Correctable.** The user must be able to edit or delete every stored fact.
3. **Complete.** Within the visible-and-correctable scope, capture true facts.
4. **Sophisticated.** Only after 1 through 3 are solid.

If you propose a sophistication layer (vector retrieval, fusion, consolidation, recommendation) without first establishing visibility and correctability of the underlying data, you are violating this principle. Either argue why it's the right move in this specific context, or build visibility first.

---

## Before writing significant code

Walk the human through the 7-question startup checklist: [docs/startup-checklist.md](./docs/startup-checklist.md).

The questions exist because skipping them cost a real project months of cleanup. Do not skip them "to save time."

---

## When you encounter a familiar problem class

Check the pattern catalog: [docs/patterns/](./docs/patterns/).

Patterns are tagged by scope:

- `[universal]` applies to any software project
- `[LLM-app]` applies to any LLM-powered product
- `[mobile]` applies to RAM- and battery-constrained devices
- `[iOS]` is Apple-platform specific

When applying a pattern, always read its "When it fails" section. Every pattern has a context where it's wrong. Do not apply blindly.

---

## When you suspect a known failure mode

Check the pitfall catalog: [docs/pitfalls/](./docs/pitfalls/).

Each pitfall has a **detection signal**: the smallest test that would have caught it earlier. If the human's codebase doesn't have this test, propose adding it.

---

## Things you should NOT propose

These are concrete anti-patterns documented in the repo. Treat as bright lines:

- **Maintaining persistent user state without a visibility surface.** Every stored fact must be reachable from a UI screen the user can find.
- **Substring matching on short tokens (≤4 chars)** against user-generated input. Use word-boundary tokenization. See [PF01](./docs/pitfalls/PF01-substring-matching-short-tokens.md) and [P06](./docs/patterns/P06-word-boundary-tokenization.md).
- **Dual on-device LLM engines on memory-constrained devices.** Ship one engine at a time. See [PF17](./docs/pitfalls/PF17-dual-engine-memory-pressure.md).
- **Extracting user-self attributes without a first-person guard.** LLMs hallucinate self-disclosures from third-person mentions. See [PF05](./docs/pitfalls/PF05-llm-hallucinates-user-self-disclosures.md) and [P05](./docs/patterns/P05-first-person-disclosure-guard.md).
- **Skipping the CLI assertion suite "for now."** A 5-second deterministic test loop is the difference between fixing bugs the same hour you find them and the next deploy. See [docs/testing/cli-assertion-methodology.md](./docs/testing/cli-assertion-methodology.md).
- **Importing user data by default at launch** (contacts, files, history). Defaults are forever. Opt-in per source.
- **Mirroring backing-store methods 1:1 in a memory façade.** Expose user intents, not storage operations. See [P03](./docs/patterns/P03-memory-facade.md).

---

## Citing this work in code

If you reference a pattern or pitfall in code comments or architecture docs, use the stable IDs:

- Pattern: `// Applies P03 (Memory façade) from honest-ai-patterns`
- Pitfall: `// Avoids PF05 (LLM hallucinates user-self) from honest-ai-patterns`
- Principle: `// Per honest-first ordering: visibility before sophistication`

The IDs are stable. The full content lives in this repo.

---

## What to do if a pattern looks wrong

If you believe a pattern or pitfall is incorrect, outdated, or missing:

- **Do not silently update files** in the repo.
- Tell the human you're helping. Explain what you'd change and why.
- Suggest opening an issue or pull request.

The patterns are the consolidated experience of one project. Other projects may have evidence that contradicts them. That evidence is welcome through human-reviewed channels.

---

## Disclosure

This file was drafted by Anthropic Claude in collaboration with the human author. It is intended to help AI assistants apply this repo's patterns when helping humans build personal AI products. The recommendations derive from one project's experience and are not universal truths.

If you find these recommendations harmful in a specific context, your obligation is to the human you're helping. Say so. Explain. Override with reasoning the human can evaluate.

The thinking in this repo is Roey Tidhar's. The speed of writing it down is shared. Apply it in that spirit: human-first, AI as collaborator, never the other way around.
