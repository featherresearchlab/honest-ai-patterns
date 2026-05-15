# Principles

A position on how persistent state should be designed in personal AI.

This document is short on purpose. Principles that need more than a page to explain are usually doctrine.

---

## The asymmetry

A personal AI that maintains state about its user faces a structural asymmetry:

- The system can produce plausible facts faster than the human can verify them.
- The system can persist those facts longer than the human can audit them.
- Once trust breaks, every subsequent correct recall is doubted.

This asymmetry is structural, not a property of any particular model. Larger models do not make it go away. Better prompts do not make it go away. The asymmetry can only be addressed through *design choices about what the system shows, what it lets the user edit, and what it persists.*

---

## The ordering

For any persistent state in personal AI, this priority ordering applies when tradeoffs arise:

1. **Visible.** The user can see what the system knows about them, in plain language, at all times. No "memory" that is invisible from the UI.
2. **Correctable.** Every stored fact is editable and deletable from the surface where it is shown. Edits take effect immediately.
3. **Complete.** Within the visible-and-correctable scope, the system captures as much true information as it can.
4. **Sophisticated.** Within the complete scope, the system uses advanced techniques (embedding, retrieval, fusion, consolidation).

All four matter. The ordering is for when they conflict. And they always conflict.

---

## Why this ordering

- **Visible before correctable.** The user must know what's there to know what to correct. Hidden state cannot be corrected.
- **Correctable before complete.** The cost of a wrong stored fact, uncorrected, is unbounded. It propagates through every subsequent retrieval. Incomplete-but-correct beats complete-but-wrong.
- **Complete before sophisticated.** Sophistication on incomplete data produces confident wrong answers. The model's confidence is calibrated to its training distribution, not to your user's actual data coverage.

---

## What this means in practice

Before the first commit on a new personal-AI product, decide what state you will persist about the user. For each piece of state:

1. **Design the visibility surface first.** Where does the user see this? Can they see it in plain language? Is it surfaced proactively or hidden behind navigation? If the answer to "where can the user see this?" is "they can't, but the system uses it internally," stop. That state is the bug.
2. **Design the correctability surface second.** From the same screen where the user sees the fact, can they edit it, delete it, mark it as incorrect? If editing requires three navigation hops, the user will not correct anything, and your "memory" is a one-way ratchet.
3. **Design the capture pipeline third.** What extracts facts? What writes them? Only after you know how the user will see and correct them.
4. **Design the sophistication layer fourth.** Embeddings, fusion, consolidation, recommendation. All wonderful. None of them allowed to produce data the user cannot see or correct.

If at any point a sophistication layer is producing data the user cannot see or correct, the layer is wrong. Not the user.

---

## Beyond memory

The same ordering applies to anything else the system persists about the user:

- **Preferences.** Visible, correctable, complete, sophisticated.
- **Learned patterns.** Visible, correctable, complete, sophisticated.
- **Recommendations.** The user must be able to see why a recommendation was made and reject the reason.
- **Connections (other people, organizations).** Source-aware merging. The user controls whether imported data joins user-disclosed data.

---

## What this is not

This is not a privacy argument, though it has privacy implications. Privacy is about *who else* sees the data. Honest-first is about *whether the user themselves* sees the data the system has about them. A perfectly private system that the user cannot audit is still a failure of honesty.

This is not an open-source argument, though it benefits from open infrastructure. A closed-source product can be honest-first. An open-source product can violate the ordering.

This is not a small-model argument, though it is compatible with on-device inference. A cloud-hosted personal AI can be honest-first. An on-device personal AI can violate the ordering.

The ordering is orthogonal to where the model runs, who can see the source, and how the data is encrypted. It is about what the user can see, edit, and trust.

---

## What goes wrong when the ordering is violated

The most common failure mode in personal AI today, in our observation:

> The team builds sophistication first (vector retrieval, memory fusion, multi-agent reasoning) before building the surfaces that let the user see and correct what the system stores. The product works for a few sessions. Then the system makes a confident, wrong claim. It misinterpreted a chat message, or pulled a stale fact from an unsynced backing, or hallucinated a relationship. The user has no way to correct it. The next correct claim is doubted. Trust collapses.

The fix is not in the recall system. The fix is building visibility first.

---

## A closing note

We are not the first to argue this. The HCI literature on human-AI trust has made related arguments for years. What this document adds is a specific, operationalizable ordering with consequences engineers can act on.

You may have stronger principles than these. We hope you do. Adopt them, write them down, and hold yourself to them.

If you have weaker principles than these, particularly if your personal AI keeps memory the user cannot see and cannot correct, please reconsider before shipping to other people.

The asymmetry is real. The user has no defense against an AI that fabricates plausible facts and persists them invisibly. We build the defense, or we don't ship.

---

*Principles · 2026 · For honest products.*
