# Engineering Notes from Building an On-Device Personal AI

A technical writeup, not a research paper. From one project, with substantial AI coding assistance, ended in May 2026.

---

## A note before anything else

This document is shared in case any of it is useful to someone building a similar system. It is not peer-reviewed. It has not been compared against the academic literature systematically. It claims no research novelty. The author is not well-known in the field; the work has to stand on whether it is useful, not on a name.

**What came from where.** The project's vision, strategic direction, and ethical frame predate the AI collaboration. Roey Tidhar built the conceptual foundation before any prompt was sent — the decision to ship on-device, the choice of Apple Intelligence as primary, the willingness to spend twelve weeks on a one-user product, and the design principle this document organizes around (visible > correctable > complete > sophisticated). The principle emerged from his judgment during the project — from living through a real trust break — not from an AI suggestion. An AI coding assistant (Anthropic Claude) joined as a collaborator on execution: writing substantial portions of the code, drafting substantial portions of this text, and helping extract patterns across a large codebase under his direction. Where this document says "we did X," it usually means Roey decided to do X, Claude implemented or drafted it, both reviewed. The collaboration was real and substantial on both sides. The thinking is Roey's; the speed is shared.

**What you can do with this.** Take what helps. Leave what doesn't. The patterns and reference notes in this repository are MIT-licensed. If something here is wrong, please tell us — the right move is to correct, not to defend.

---

## The project in brief

Feather was a personal AI assistant for iOS, built over twelve weeks by one independent developer with AI coding assistance, shipped to one device (the author's own), and declared complete at the end of that window.

It ran two inference engines on-device: Apple Intelligence (Apple Foundation Models) for classification and multilingual chat fallback, and MLX-Swift (Gemma 4 E2B text-only at 4-bit quantization) for primary chat. It maintained four kinds of memory behind a single façade — small key-value facts, a typed entity graph, a vector episode index, and long-form daily notes — and ran a periodic consolidation pass over them.

The project's strongest achievement was that it ran end-to-end: a fully on-device personal AI with memory consolidation, multilingual extraction, and five product surfaces, built by one person in twelve weeks. The number of products in this exact shape that have actually shipped is small.

The project's clearest failure was the gap between engineering quality and felt product. Rigor was built; wonder was not. The lessons below come from both — what worked structurally, and what didn't translate into a product the user would have loved.

---

## The principle we operated by

The central design principle, learned the hard way over the first half of the project:

> For any product that builds persistent state about its user, that state should be **visible**, **correctable**, **complete**, and **sophisticated** — in that order of priority when these requirements conflict.

We call this honest-first design. The full argument lives in [PRINCIPLES.md](./PRINCIPLES.md); the operational summary:

- **Visible before correctable** — the user must see what is stored to know what to correct.
- **Correctable before complete** — incomplete-but-correct beats complete-but-wrong.
- **Complete before sophisticated** — sophistication on incomplete data produces confident wrong answers.

We arrived at this ordering because violations produced predictable failures. We built a sophisticated vector-graph memory system before we built the UI that let the user see what was stored. The system silently overwrote facts based on misinterpreted context (a chat about the user's sister produced a hallucinated update of the user's own location). The user could not see this had happened. Once trust eroded, every subsequent correct recall was doubted.

The fix was not in the recall system. The fix was building visibility, then correctability, then re-grounding the recall against a now-correctable substrate. Sophistication is a luxury good purchased with the trust accrued from visibility and correctability. There is no shortcut.

### What we do not know about this principle

We do not know whether this exact priority ordering has been formulated elsewhere. We have not done a systematic literature search. The HCI field has discussed transparency, agency, and user control for decades; some adjacent version of this argument almost certainly exists. We offer this formulation because it was operationally useful for us — when we had to choose between two design directions under deadline pressure, the ordering gave us a clear answer.

If this exact ordering already exists under another name, we'd like to know. The principle matters more than its branding.

---

## Engineering patterns we used

The patterns below worked for us in the context of this project. Each one has detailed notes in [docs/patterns/](./docs/patterns/). We do not claim any of these are new. We claim they were useful.

### Pre-filter before LLM

Every LLM call costs latency, battery, and quota. A cheap deterministic check (regex, keyword scan, length gate) before the model call lets the model skip inputs it doesn't need to see. In our memory extraction pipeline, this rejected 60-90% of conversational turns at the pre-filter, calling the LLM only on turns containing extractable signals.

Maintenance cost: the trigger lists drift across languages. Be prepared to maintain them.

### Three-tier extraction

For structured extraction from natural language, three layers in sequence: a prompt with explicit format and few-shot examples; a structured-output constraint (JSON schema, Apple `@Generable`, equivalents); a post-filter that rejects out-of-vocabulary or out-of-shape outputs. Each tier catches what the previous missed.

Maintenance cost: the post-filter accumulates rules. Budget for periodic consolidation passes.

### Memory façade with multiple backings

One front door (`Memory.shared`) for every read and write of memory. Behind it, multiple specialized stores. Callers don't know which store served them. A CI test fails the build if any non-whitelisted file imports a backing store directly.

When we migrated the entity graph from a hash-map prototype to SwiftData with CloudKit sync, zero call sites needed to change. The CI test caught two violations during the migration that would have leaked the old API into new code. We recommend the pattern; we do not claim it's new.

### Source-aware entity merge

Entities from different sources (chat-disclosed, contacts-imported, web-scraped) track their source. Same name + different source = different rows. The user's Memory view filtered to chat-sourced entities only; imported data was available to skills but invisible by default.

This pattern saved us from a class of bugs where an imported contact named "Juan" was contaminated by chat content about a different Juan. It also gave us the clean filter the user actually wanted ("only show people I've talked about in chat") without losing the contacts data for skills that needed it.

### First-person disclosure overwrite guard

Before overwriting an existing user-self attribute (name, location, work, role, birthplace), require a first-person verb in the originating message. Without this guard, our LLM extraction pipeline produced confident wrong updates from third-person mentions: "My sister lives in Madrid" → user location overwrote to Madrid.

**What we do not know.** We have not done a literature search for this guard. We claim it was useful for us; we do not claim it is novel. If you have seen this pattern in industry documentation, internal architecture documents, or published work — please tell us. We will update the writeup with the prior reference.

### Word-boundary tokenization in safety filters

Never use `text.contains("ok")` against unstructured input — it matches "looking", "Tokyo", "Oklahoma". Tokenize on whitespace and punctuation; require exact-token match for short tokens (≤4 characters); longer tokens may substring.

This is a standard engineering pattern. We mention it here because we missed it in production: a migration that scrubbed topic-pollution entities matched substring "war" inside legitimate names ("Edward", "Howard", "Stewart") and would have deleted real user data if it had run without external review.

### Idempotent versioned migrations

Destructive migrations should be retryable. We use UserDefaults keys of the form `migration_X_done_v1`. When the v1 logic has bugs and the v2 fixes them, use a *new* key (`v2`); do not reuse the v1 key. Always log what was deleted or changed so recovery is possible.

We had to retry three migrations during the project. The v1/v2 namespacing meant none of them double-applied or got stuck.

### Defer-unload for in-flight resources

A memory-pressure handler that wants to free an inference model needs to check whether inference is in flight. If yes, set a deferred-unload flag instead of freeing. The decrement-on-exit logic completes the unload when the in-flight count reaches zero.

Without this, we had six recorded SIGSEGV crashes from freeing a model mid-inference. After: zero.

### Journey log with correlation IDs

Every meaningful decision (intent classification, tool dispatch, model call, fallback) writes a structured event (JSON line) with timestamp, event type, payload, and a correlation ID for the user turn. The log is queryable offline.

Most production bugs we debugged late in the project were diagnosed from journey logs alone. Without this, several bug classes would have remained "the app sometimes feels wrong" instead of "here is exactly what happened on turn N."

---

## Failure modes we encountered

Catalogued in [docs/pitfalls/](./docs/pitfalls/). Some representative classes, with short descriptions; the docs folder has detailed notes for each.

- **Substring matching against short tokens.** Filters using `contains("ok")` matched "Tokyo", "Oklahoma", "Bangkok". Names that contained dictionary tokens nearly got deleted by a migration. Tokenization is the fix; word-boundary regex is the fix.
- **LLM hallucinated user-self disclosures from third-person mentions.** Solved by the first-person guard above.
- **LLM emitted JSON object when prompt asked for array, in the single-fact case.** Tolerant parser: try array first, fall back to object, wrap.
- **Model echoed the user's question back as the answer when uncertain.** Echo guard (token-overlap + prefix match) at multiple pipeline layers.
- **Model self-identified as the base model ("I'm Gemma, trained by Google") regardless of persona prompt.** Identity-leak regex post-filter with persona-fallback substitution.
- **Imported data (contacts) rendered alongside user-disclosed data with no source distinction.** Source-aware merge and source-aware UI filtering.
- **Dual on-device LLM engines competing for GPU/Metal context on 8 GB iPhones, producing reproducible crashes.** Hard cut to a single engine; ship one engine at a time.
- **Reactions did not appear after long-press because in-place array mutation did not trigger SwiftUI's snapshot rebuild.** Explicit rebuild after in-place mutation.
- **Stale deploy — code shipped to git but device ran old build.** Pinned `-derivedDataPath` to a stable path in both build and install steps; without the pin, `xcodebuild` picks DerivedData by canonical project disk path, and a build from one path with an install from another deploys stale binaries.

The full list is 25 specific failure modes. Several are iOS-specific; many are not.

---

## How we tested

We adopted a deterministic CLI assertion suite — a Mac command-line executable that runs the deterministic code paths in the system (regex filters, keyword routers, JSON parsers, post-filters, tokenizers) with hundreds of input-output assertions. No LLM in the loop. No device. No emulator. Just the host machine, in seconds, in CI.

Our suite reached 205 assertions running in five seconds. The full methodology is in [docs/testing/cli-assertion-methodology.md](./docs/testing/cli-assertion-methodology.md).

The suite did not replace device testing. It tested the deterministic shells around non-deterministic LLM cores. Device testing remained necessary for hardware-specific behavior, integration issues, and real-model output. The two layers complement each other.

The suite was useful enough that we recommend the approach. We do not claim it is novel. Other teams almost certainly do equivalent things; we have not surveyed industry practice. If you have not built one for your LLM-app yet, it may be worth the afternoon to start one — and to keep it under ten seconds as it grows.

---

## What we do not know

This section deserves its own space because it is the most honest part of this writeup.

- We do not know whether the honest-first priority ordering is genuinely new or a restatement of existing HCI principles. We have not done a systematic literature search.
- We do not know whether the first-person disclosure overwrite guard exists in other codebases under different names. Industry teams may have similar guards in internal documentation we have not seen.
- We do not know whether the CLI assertion methodology produces the same cycle-time advantage in other contexts. Our number (a 30× speedup over device E2E for the specific bug classes the CLI suite covers) is a project-specific measurement, not a generalization.
- We do not know whether the patterns transfer to non-iOS, non-personal-AI systems. The patterns tagged `[iOS]` definitely don't; the others we believe transfer, but we have not verified.
- We have no user studies. The project served one user — the author. What we report worked for that user; we do not claim it works for all users.
- We do not know what we did not learn. There were almost certainly bugs we never noticed, patterns we never recognized, alternatives we never considered.

If you find that any of these claims is wrong — please tell us. We will update the writeup. The version of this document in the public repository will always be the latest.

---

## What this is not, restated

- **Not a paper.** No peer review. No academic submission. No claim of research novelty.
- **Not a comprehensive survey.** One project, one team (mostly one person plus AI assistance), one platform, one timeline.
- **Not a finished thought.** This is the state of our learning at project end. Better engineers will find better patterns. Send pull requests.
- **Not a substitute for understanding.** Copying a pattern without understanding the problem it solves is how patterns become anti-patterns. The "when it fails" section in each pattern file matters.

---

## Why we published this anyway

We could have kept these notes private. We did not, for two reasons.

**First:** the transformer architecture, on which every personal AI product today depends, was given freely to the world. The patterns and lessons that emerge from building on top of it should be given freely too, in the same spirit. We are not contributing comparable scientific weight; we are contributing what one project's experience can contribute. That is a small thing, but it is the right thing.

**Second:** somewhere, another developer is about to make a personal AI product. They will face the same asymmetry between an AI that can fabricate plausible facts and a human who must trust them. They will probably make at least some of the same mistakes we made. If even one of the patterns or pitfalls here saves them a debugging session, the time spent writing this down was repaid.

The work is offered to anyone who finds it useful. No restrictions beyond the MIT license. No requirement to credit; credit is appreciated but never required.

---

## Acknowledgments

To Anthropic for Claude, which was a real collaborator on both the code and this writeup. The collaboration model is part of the methodology we disclosed.

To Apple for Foundation Models and MLX, the on-device infrastructure that made the project a personal-AI product rather than a cloud-API wrapper.

To the maintainers of `mlx-swift`, `mlx-swift-lm`, `llama.cpp`, `swift-transformers`, and the wider open-source ML community on whose work the system depends.

To the field that built the transformer architecture and gave it freely. We hope this small artifact returns a fraction of the gift.

---

## Closing

The work that follows the model is not, primarily, the work of larger models or longer contexts. It is the work of making personal AI worthy of the trust users place in it.

We tried for twelve weeks. We learned some things. We share them here in case any of them help the next person.

If they do, that is enough.

---

*Written 2026-05-15. Roey Tidhar with substantial AI coding and drafting assistance from Anthropic Claude. No peer review. No novelty claims. MIT license on all included code and reference implementations. The current version of this document lives at `WRITEUP.md` in the [honest-ai-patterns](https://github.com/) repository (URL to be added when the repo is published).*
