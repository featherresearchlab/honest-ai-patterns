# honest-ai-patterns

**Patterns that keep AI honest with the people who use it.**

If you're building AI that remembers things about your user (memory, preferences, learned context), this catalog gives you the engineering patterns that prevent the worst failure modes. Free. MIT-licensed. Ready to copy.

---

## What you get

| | |
|---|---|
| **5 patterns** | recurred across feature areas in production |
| **3 pitfalls** | with symptom, fix, and the test that catches the class next time |
| **1 testing methodology** | runs in 5 seconds, catches bugs most teams find by deploying |
| **1 design principle** | for any product that maintains state about its users |
| **1 starter template** | drop [CLAUDE.md.template](./CLAUDE.md.template) into your next project on day one |

The principle in one line: **visible > correctable > complete > sophisticated**. We call it honest-first design. The full argument lives in [PRINCIPLES.md](./PRINCIPLES.md).

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
