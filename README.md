# honest-ai-patterns

**Patterns that keep AI honest with the people who use it.**

**Save weeks. Ship AI users trust.** A catalog of patterns, failure modes, and a design principle for any product that maintains state about its users. Distilled from twelve weeks of shipping personal AI on-device.

→ **5 patterns** that prevented real production failures
→ **3 bugs** with symptom, fix, and the test that catches them next time
→ **1 design principle**: visible > correctable > complete > sophisticated (we call it honest-first)
→ **1 test methodology** that runs in 5 seconds and catches what device testing misses
→ **1 starter template**: drop [CLAUDE.md.template](./CLAUDE.md.template) into your next project on day one

Free. MIT-licensed. Ready to copy. Full argument for the principle lives in [PRINCIPLES.md](./PRINCIPLES.md).

---

## What this saves you (the numbers we tracked)

Specific measurements from one project. Not guarantees. Your savings depend on which patterns you adopt and what you're building.

| Pattern | What it saved us |
|---|---|
| **CLI assertion suite** | 5 seconds per bug iteration vs 2-15 minutes for device redeploy. For an LLM-app team running 20+ iterations a week, that's hours per week of pure cycle time. |
| **Defer-unload for in-flight resources** | 6 SIGSEGV crashes prevented over 12 weeks. Each crash in production is ~½ day of debugging, plus trust damage. |
| **First-person disclosure guard** | Zero user-self attribute overwrites across 3,200 logged extractions after we shipped it. The bug class had shipped undetected for weeks before external review caught it. |
| **Word-boundary tokenization** | Caught a migration that would have hard-deleted real users named Edward, Howard, Stewart (all contain "war"). Recovery from accidental user-data deletion: incalculable. |
| **Memory façade** | Migrated the entity store from hash-map prototype to SwiftData with CloudKit sync. Zero call sites changed. Without the façade pattern, the same migration touches most of the codebase. |
| **v1/v2 versioned migrations** | 3 destructive migrations retried safely. Without the pattern, retries compound bugs and often require manual data recovery. |

Convert to dollars using your team's hourly rate. We won't quote a single number because the savings depend on your context: how often the bug class would have occurred, how much your engineers cost, how much user trust is worth to your product.

The honest upper bound: **the twelve weeks we spent learning these the hard way.** If everything here applies to your project, that's the amount of time you don't have to spend.

---

## Why this exists

We spent twelve weeks building Feather, a personal AI for iOS. Fully on-device. Memory, multilingual extraction, five product surfaces.

Mid-project, something happened we did not expect. The system silently overwrote a stored fact about its user, based on a third-person chat message. Other facts went wrong too. The recall pipeline was sophisticated. The visibility was nothing. The user could not see what was stored, could not correct it. Trust eroded.

That bug taught us the principle this repo is built around. Sophistication on data the user cannot see produces confident wrong answers, every time. The fix was never in the recall system. The fix was building visibility first, correctability second, then re-grounding the recall on a substrate the user could trust.

We wrote it down so the next person does not have to learn it the same way.

---

## How to use it

1. Read [WRITEUP.md](./WRITEUP.md). That's the story, the patterns, and what we still do not know.
2. Read [PRINCIPLES.md](./PRINCIPLES.md). That's the ordering, with an argument for why it works.
3. Browse [docs/patterns/](./docs/patterns/) and [docs/pitfalls/](./docs/pitfalls/) for operational reference.
4. Drop [CLAUDE.md.template](./CLAUDE.md.template) into your next project on day one if you're using AI coding assistance.

If you ARE an AI coding assistant helping someone build personal AI, start at [AGENTS.md](./AGENTS.md). It tells you which patterns to apply and which mistakes to flag.

---

## On the collaboration

This is human plus AI, in the open. The team behind it:

- **Roey Tidhar** (human author, Feather Research Lab)
- **Anthropic Claude** (AI coding and drafting collaborator)

The thinking came from the human side of the team. Twelve weeks of decisions, framing, the lived experience of using a working product daily. The design principle this repo organizes around (honest-first) emerged from a real trust break with a working system, not from an AI suggestion. Claude did not propose the principle. The human learned it.

The code in the patterns, the structure of this repo, and the words on this page came faster because the team worked together throughout. The thinking is the human's. The speed is shared.

This is what independent research with AI tools can look like now. One human, one AI, twelve weeks, producing something that would have taken much longer alone. We are not hiding it. Other builders should know it is possible.

---

## What's here, and what's coming

5 patterns written. 9 more slotted in [INDEX.md](./INDEX.md). 3 pitfalls written. 22 ready to be filled in by readers who have seen them.

Pull requests welcome, especially from people who have shipped real AI products and want to share what bit them.

Honest caveats: one project, one platform (iOS), self-reported failure data, no peer review. The patterns are observations from one team's experience, not warranted recommendations. Several novelty claims in here are unverified by a literature search. If you have seen the same patterns under different names, tell us. We'll update the catalog.

The [What we do not know](./WRITEUP.md) section in WRITEUP.md is the most honest part of this work. Read it before generalizing.

---

## License

MIT. Copy whatever helps. Attribution appreciated, never required.

If something here saves you a debugging session, that is the entire bet. If it doesn't, leave it. If you find better patterns, share them.

---

## A note about the name

Feather Research Lab is named after a feather. The reason is personal: our founder's mother liked feathers. She would have liked this project getting shared with the world. That's why the repo lives under that org instead of a personal account.

That's all. Welcome.

---

*honest-ai-patterns · Feather Research Lab · Roey Tidhar with Anthropic Claude · 2026 · For honest products.*
