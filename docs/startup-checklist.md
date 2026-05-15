# Startup checklist

Seven questions to answer before the first commit on a new personal-AI project. Each one is here because skipping it has cost a real project months of cleanup.

---

## 1. Who is the user, in one sentence?

Not "anyone who wants X." A specific person with a specific job-to-be-done.

**Good:** "An indie iOS developer who wants a personal assistant that remembers their work context across days, runs entirely on-device for privacy, and never sends data to the cloud."

**Bad:** "Users who want AI-powered productivity."

If you cannot write the sentence, the product hasn't been framed yet. Frame first, then code.

---

## 2. What's the felt experience at second 5, day 1, day 7, day 30?

Before architecture. Before tools. Before the model. What should the user *feel*?

- **Second 5:** what creates the "oh, this is different" moment in the first five seconds?
- **Day 1:** at the end of the first session, what makes the user reopen the app tomorrow?
- **Day 7:** after a week of use, what does the user notice this product does that nothing else does?
- **Day 30:** at month one, what's the moment that makes them tell another person about it?

If you cannot describe these feelings, you will build engineering quality with no product moments. Users do not feel test coverage. They feel speed, visibility, trust, surprise.

---

## 3. What's the smallest testable hypothesis (v0.1)?

Strip features until removing one more would make the product non-functional. That's v0.1.

Every feature beyond v0.1 is a bet that should pay back its own complexity. Most bets do not.

Common failure mode: shipping v0.7 as v0.1 because each individual feature seemed cheap. The cost is not per-feature; it's the interaction surface between features, and that grows quadratically.

---

## 4. What user data, if any, is imported at launch?

If "their contacts / files / photos / history / messages" is on the list — **STOP**.

Test whether they want it imported first. Defaults are forever. Un-importing is a year of cleanup work.

If you decide to import something, the rule:

- **Opt-in per source.** Not "all your data" — per-source toggles.
- **Visible in the UI.** The user sees what was imported, source-tagged.
- **Removable individually.** Same screen where they see it, they can remove it.
- **Source-aware.** Imported data doesn't auto-merge with user-disclosed data.

Most defensible default: import nothing. The user adds what they want, when they want it.

---

## 5. What state does the product build that the user can't see, edit, or delete?

If anything — that's the bug.

Personal AI products that maintain memory, learn preferences, build recommendation models, or persist any state about the user must make that state visible and correctable. Otherwise the user cannot detect when the system is wrong, and once it's wrong, every subsequent correct claim is doubted.

This is the heart of `PRINCIPLES.md`. The full ordering: visible > correctable > complete > sophisticated.

If a sophistication layer is producing data the user cannot see or correct, the layer is wrong.

---

## 6. What's the regression catch?

A test cycle running in under 10 seconds, in place by day 7.

Without it, every bug becomes "deploy to device and hope," and the cycle time eats the project. With it, you can fix a class of bug in the same hour you find it.

For LLM-app code: a CLI assertion suite covering the *deterministic* paths (routing, classification, parsing, filtering). The non-deterministic parts (the LLM itself) need separate treatment (logging, journey traces, A/B comparison) but the deterministic shell can be tested mechanically.

See `docs/testing/cli-assertion-methodology.md` for the approach.

---

## 7. What's the one user-visible "wow" moment in the first session?

Not engineering wow ("look at the architecture"). User-visible wow ("the product knew this thing about me already" or "this was instant when I expected to wait" or "I asked one thing and it understood three").

If you cannot name the moment, the product is forgettable regardless of how well it's built.

Write the moment down before you write code that produces it. Test the moment in user research before scaling.

---

## How to use this checklist

- **Answer all seven before the first commit.** Bullet points are fine. Save them in your project's CLAUDE.md.
- **Reread them weekly.** Answers change as the product takes shape. The discipline of updating them keeps the project from drifting.
- **Share them with collaborators.** Disagreement on these answers is the most important kind of disagreement to surface early.
- **Hold them up against the product.** When something feels off, the answer is almost always that the product has drifted from one of the answers.

---

## The meta-lesson

These seven questions exist because, in a project where they were not asked or were asked late:

- The wrong default was shipped (contacts imported at launch).
- The product was built engineering-quality without product moments.
- The memory pipeline was sophisticated before it was visible.
- The test cycle stayed at "deploy to device and hope" for too long.
- The "wow" moment was assumed instead of designed.

Each of these cost weeks. Each was preventable by ten minutes of pre-commit discipline.

---

*Startup checklist · 2026 · For first commits.*
