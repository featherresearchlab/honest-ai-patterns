# CLI assertion methodology

**How to test the deterministic paths in non-deterministic LLM systems with a 5-second feedback loop.**

---

## The problem this solves

LLM-app code contains two kinds of decision-making:

1. **Deterministic** — regex, keyword filters, JSON parsers, routing tables, post-filters, language detectors, tokenizers. Given the same input, the code always produces the same output.

2. **Non-deterministic** — the LLM call itself. Given the same input, the model may produce different outputs across runs.

The non-deterministic part is hard to test in CI (requires LLM mocking, statistical-significance machinery, expensive infrastructure). Many teams default to end-to-end testing on real devices because the deterministic and non-deterministic code is intermingled.

But the *deterministic* part is easy to test mechanically. The trick is to separate it and test it on its own.

---

## The shape

A CLI executable that:

1. Runs the deterministic code paths directly (no LLM in the loop).
2. Asserts on input-output pairs deterministically.
3. Runs in under 10 seconds.
4. Runs in CI on every commit.
5. Runs on the host machine — no device, no emulator, no LLM API key required.

Concrete example (in Swift, for an iOS project):

```swift
// Sources/AssertionSuite/main.swift
import YourAppCore  // import the deterministic modules

print("=== Filter assertions ===")
assertEqual(YourFilter.shouldProcess("hello"), true, "basic")
assertEqual(YourFilter.shouldProcess(""), false, "empty")
assertEqual(YourFilter.shouldProcess("Edward came over"), true, "name with substring")
// ... 200 more assertions

print("=== Tokenizer assertions ===")
assertEqual(YourTokenizer.tokens("hello world"), ["hello", "world"], "basic split")
// ... etc

print("PASSED \(passed) / \(passed + failed)")
exit(failed == 0 ? 0 : 1)
```

Run via `swift run AssertionSuite`. Total wall clock: 5 seconds.

The same shape works in any language. Python: a script with `assert` statements. JavaScript: a Node script. Go: a `_test.go` package. The key is that it runs *fast* and *deterministically*.

---

## What it catches

These are bug classes the methodology has caught in practice:

- **Substring-matching pitfalls** (PF1) — filters using `text.contains("ok")` matching "Tokyo".
- **Pattern-set drift** (PF16) — two filter lists that should be identical drifting apart.
- **Tokenization bugs** — splitting on the wrong delimiter, dropping punctuation.
- **Canonical-key collisions** — fact extraction normalizing different inputs to the same key.
- **Language-detection misclassification** — where the detector is rule-based.
- **Echo guard regressions** — output overlap-detection failing on edge cases.
- **JSON parser shape assertions** — model emits an object when the schema expects an array (PF6).
- **Post-filter blocklist coverage** — checking that the filter rejects all known bad outputs.

---

## What it does NOT catch

Be honest about the boundary:

- **LLM hallucinations.** The model itself isn't in the loop.
- **Latency regressions.** The CLI is not the production runtime.
- **Memory pressure issues.** No GPU, no NPU, no production memory budget.
- **Concurrency bugs.** Single-threaded test execution.
- **UI rendering.** No view layer.
- **Anything that requires the actual model running.**

For those, you need different layers (journey logging in production, profiling on device, manual QA, etc.). The CLI suite is one layer in a multi-layer strategy, not a substitute for the others.

---

## Cycle time

The argument for the methodology is *speed of feedback*:

| Approach | Wall time | Iterations per hour |
|---|---|---|
| Device deploy + manual reproduction | 2-15 min | 4-30 |
| Simulator launch + manual reproduction | 30-60 s | 60-120 |
| Simulator launch + automated E2E | 20-30 s | 120-180 |
| **CLI assertion suite** | **5 s** | **~720** |

The 30× speedup is the difference between "I'll get to it after the next deploy" and "I'll fix it right now."

This is not theoretical. In the originating project, the CLI suite grew from zero assertions to 205 over twelve weeks. Each assertion was added the moment a bug was found, *before* the fix landed. The discipline meant the same bug class could not regress without being caught in 5 seconds.

---

## The discipline

The methodology is not the binary file. The methodology is the *discipline*:

1. **Every bug fix adds an assertion first.** Before changing the production code, write the assertion that would catch the bug. The assertion should fail on the current code. Run the suite, see the red, then fix the bug, run the suite, see the green.
2. **Every new filter, router, parser, or tokenizer gets a baseline assertion set.** Not "I'll add tests later." Day one.
3. **The suite runs in CI on every commit.** Not "every PR." Every commit. CI catches regressions before they leave the developer's machine.
4. **The suite stays fast.** If it grows beyond 10 seconds, split it. The 5-second feedback loop is the load-bearing property.
5. **Assertions are written for *coverage*, not *celebration*.** Don't write 50 assertions for the same edge. Write one assertion for each *class* of input the code handles, plus the regression assertions from real bugs.

---

## Composition with other testing layers

The CLI suite is the innermost test layer. Build outward:

```
┌─────────────────────────────────────────────────┐
│ Production journey logs                          │ ← what actually happens
├─────────────────────────────────────────────────┤
│ Manual QA on device                              │ ← end-to-end correctness
├─────────────────────────────────────────────────┤
│ Simulator E2E tests (optional)                   │ ← UI integration
├─────────────────────────────────────────────────┤
│ CLI assertion suite                              │ ← deterministic logic (fast)
└─────────────────────────────────────────────────┘
```

Each layer catches what the inner layers cannot. The CLI suite catches the most bugs per second of test execution; the journey log catches the most bugs per second of user activity. Both are necessary.

---

## Common objections

### "We don't have time to write a CLI suite."

You don't have time *not* to. Every bug found by manual device testing instead of by the CLI suite costs minutes-to-hours of cycle time. The break-even is usually within 2-3 debugging sessions.

### "Our routing logic isn't deterministic — it depends on the LLM."

The *routing decision* may depend on the LLM, but the *routing inputs and outputs* are usually deterministic. Test the pre-filter that decides whether to call the LLM. Test the parser that extracts structured output. Test the post-filter that validates the result. The LLM call itself is one node in a graph of deterministic nodes.

### "We have unit tests."

Great. The CLI suite is unit tests, just organized as a single fast-running executable that prints pass/fail counts. The format matters less than the discipline.

### "We use AI to write the tests."

Also great. The methodology doesn't care who writes the assertions, only that they exist, run fast, and catch real bug classes.

---

## A minimal starting suite

If you're starting from zero, these assertion categories cover the most ground:

1. **Input validation** — empty string, whitespace-only, very long input, Unicode edge cases, RTL text, mixed-language text.
2. **Boundary conditions** — exactly at minimum length, exactly at maximum length, one over each.
3. **Known false positives from production** — every bug you've ever fixed.
4. **Multilingual inputs** — at least one per supported language.
5. **Adversarial inputs** — names that contain dictionary words, common typos, prompt-injection attempts.

20 assertions per category × 5 categories = 100 assertions. Achievable in one afternoon. Catches 80% of the routine bug classes.

---

## A closing note

This methodology is not glamorous. There's no framework to evangelize, no conference talk in it, no growth chart. Just a CLI binary that runs in 5 seconds and tells you whether the deterministic code paths still behave.

That's the entire point. The methodology disappears into the background of how you build, leaving you with more time and attention for the work that actually requires human judgment.

If the suite runs every commit and stays under 10 seconds, you've got it.

---

*CLI assertion methodology · 2026 · For fast feedback.*
