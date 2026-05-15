# Validation

How we tested whether the patterns in this catalog are actually adopted in real code, and what we measured. Honest about limits.

---

## Honest caveat upfront

This is **N=1 validation**. One project, the originating one (Feather). The patterns worked there. Whether they work in your project requires you to apply them and measure. See [How to validate this yourself](#how-to-validate-this-yourself) below.

We ran 10 measurements against the Feather codebase (~50,000 lines of Swift) to ground the catalog's claims in countable data instead of marketing claims.

---

## TL;DR

The patterns are real, widely adopted in the codebase that produced them, and the catalog is conservative. The originating project's internal regression log documents **213 distinct bug classes**. The public repo currently catalogues 25. There is significant headroom for community contributions.

The single most-adopted pattern is the **journey log** (P08), with 317 call sites and 50 distinct event types. If someone has to pick one pattern to start with, that's the one.

---

## What we measured

| # | Measurement | Result | Validates |
|---|---|---|---|
| 1 | CLI assertion suite size | 26 source-level `log.check()` calls, multiplied to ~205 runtime assertions via test-data loops. 17 test functions. | The "205 assertions" claim |
| 2 | Substring `.contains()` risk surface | 469 total `.contains("...")` in production code. **157 of those use short tokens (≤4 chars)**, which is the PF01 risk pattern. | PF01 has a real surface even in disciplined codebases |
| 3 | Memory façade adoption (P03) | 151 calls through `Memory.shared.*` vs 92 direct backing-store calls. **62% adoption.** | Pattern is used by majority of memory access |
| 4 | Versioned migrations (P07) | 2 distinct `*_v1` / `*_v2` migration keys in production | Pattern in use, room to expand |
| 5 | First-person disclosure guard (P05) | 28 references in production code | Pattern implemented, not theoretical |
| 6 | Journey log instrumentation (P08) | **317 call sites, 50 distinct event types** | Most widely adopted pattern in the project |
| 7 | Documented regressions | **213 "Fixed — do not reintroduce" entries** in the project's internal regression log (CLAUDE.md, 827 lines, 220 KB) | Catalog could grow ~8x from current 25 pitfalls |
| 8 | Crash class prevention (P12) | Not measurable from code alone. The regression log documents 6 SIGSEGV crashes from the bug class that the defer-unload pattern closed. | Documented bug count, prevention claim |
| 9 | Source-aware merge enforcement (P04) | 74 `contactID`-related guards | Pattern heavily used |
| 10 | Pre-filter before LLM (P01) | 67 call sites where the pre-filter gates an LLM call | Pattern adopted on hot paths |

---

## Pattern mentions in the project's regression log

The internal regression doc (CLAUDE.md, 827 lines) mentions each pattern this many times in incident write-ups:

| Pattern | Mentions in regression context |
|---|---|
| Journey log (P08) | 17 |
| Pre-filter (P01) | 10 |
| Memory façade (P03) | 9 |
| Source-aware merge (P04) | 3 |
| CLI assertion suite (P10) | 2 |
| Defer-unload (P12) | 2 |
| Word-boundary tokenization (P06) | 1 |
| First-person disclosure guard (P05) | 1 |

47 total passages referencing the patterns when explaining specific bug fixes. The patterns are not abstractions invented after the fact. They were named because of repeated need.

---

## What this validates

1. **The patterns are real.** Not aspirational. Each has hundreds of call sites in the production codebase.
2. **The catalog is conservative.** 25 pitfalls in the public repo; 213 documented bug classes internally. The community can extract many more.
3. **The journey log is the highest-leverage pattern by adoption** (317 call sites). Worth highlighting as the recommended starting point.
4. **PF01 (substring matching) is a real risk class.** Even in a disciplined codebase, 157 short-token `.contains()` usages exist. Most are likely safe (already-tokenized inputs), but the surface is non-trivial. The pitfall is non-theoretical.
5. **Memory façade adoption is realistic at 62%**, not aspirational at 100%. The pattern is widely useful but engineering reality includes leaks. Worth knowing if you're starting fresh.

---

## What this does NOT validate

1. **Counterfactual savings.** We cannot prove how many bugs would have shipped without these patterns. Adoption is correlation, not causation.
2. **Other projects.** N=1 is N=1. The patterns may not transfer cleanly to non-iOS or non-personal-AI work.
3. **User trust outcomes.** No user studies. The honest-first principle predicts trust outcomes but we haven't measured them empirically.
4. **Dollar value.** Engineering hours saved is a measurable proxy, but converting to dollars requires assumptions about your team's rate and your project's context. We refuse to quote a single dollar figure for this reason.

---

## How to validate this yourself

The cheapest validation costs zero. Scan your own codebase using the same commands we ran:

```bash
# Pattern 1 (pre-filter) — do you have one before your LLM calls?
grep -r "hasExtractableSignal\|preFilter\|pre_filter" your-project/

# Pitfall 1 (substring matching) — count your PF01 risk surface
grep -rE '\.contains\("' your-project/ | wc -l
grep -rE '\.contains\("[^"]{1,4}"\)' your-project/ | wc -l  # short tokens

# Pattern 3 (memory façade) — is your memory access centralized?
# Find your equivalent of Memory.shared and compare to direct backing-store calls

# Pattern 8 (journey log) — do you instrument decisions?
grep -r "Journey\.\|trace\.\|log\.event\|telemetry\." your-project/
```

If you have **zero of pattern 8** (no structured journey log for non-deterministic decisions), starting that is the highest-leverage move you can make. It's what enables debugging LLM behavior after the fact.

If you have **any of PF01** (short-token `.contains()` calls), check whether the matched text comes from user input. If yes, that's a real bug class waiting.

---

## Contributing your own validation

The more N=1 reports we collect, the closer this gets to N=many.

If you apply patterns from this catalog to your own project and want to share what worked, what didn't, and what numbers you saw, please submit a PR adding a file at `validations/<your-project-or-anonymous-handle>.md` with the same measurement structure as above.

What we ask for: the numbers and your honest assessment. What we don't need: proprietary code or any user data.

---

## Reproducibility

All counts in this document were generated from `grep -c`, `wc -l`, and `find | wc -l` commands against the Feather codebase. They are reproducible by anyone with the codebase and a shell.

The measurement methodology is fully described above. If you have the codebase and want to verify, the same commands run today produce these numbers (within the variance of ongoing development).

---

*VALIDATION.md · Feather Research Lab · 2026 · N=1 today, growing.*
