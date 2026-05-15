# PF1 — Substring matching against short tokens

**Severity:** High. Causes silent data corruption.
**Category:** Filter / routing logic.
**Detection:** CLI assertion suite covering names that contain dictionary tokens.

---

## Symptom

A filter, router, or migration scrubs data that should not be scrubbed. The bug is silent — no error, no log, just unexpectedly missing data.

Concrete examples from production:

- A migration intended to delete topic-pollution entities named "war", "king", "best" silently deleted users named **Edward** (contains "war"), **Howard** (contains "war"), **Stewart** (contains "war"), **Kingsley** (contains "king"), **Pete Best** (contains "best").
- A casual-disclosure detector intended to catch emotion words like "ok" routed messages containing "looking", "Tokyo", "Oklahoma", "Bangkok" to the wrong pipeline.
- A profanity filter blocking "ass" silently flagged "associate", "assistant", "passion".

---

## Root cause

Code like:

```pseudocode
if text.contains(badToken):
    rejectOrDelete()
```

When `badToken` is short (≤4 characters), it is statistically certain to appear inside legitimate longer words. `contains()` does not respect word boundaries.

The shorter the token, the higher the false-positive rate. A 2-character token will appear inside almost every English sentence.

---

## Fix

Tokenize on whitespace and punctuation. Require exact-token match for short tokens.

```pseudocode
function containsToken(text, target):
    tokens = tokenize(text, on: whitespace + punctuation + apostrophe + hyphen)
    return target in tokens
```

Or, equivalently, use word-boundary regex:

```pseudocode
function containsToken(text, target):
    return regex.match("\\b" + escape(target) + "\\b", text) is not null
```

For more nuance, use length-gated rules:

- Tokens ≤4 characters: exact-token match required.
- Tokens 5-8 characters: word-boundary match (allows compound words but not internal substring).
- Tokens >8 characters: substring match acceptable (false positive rate is negligible).

---

## How to detect it earlier

### Test inputs to include in your assertion suite

For every short-token filter, include test cases:

- Each forbidden token followed by a legitimate longer word that contains it
- Common names that happen to contain dictionary tokens (Edward, Howard, Stewart, Bangkok, Tokyo)
- Edge cases: token at start of word, end of word, middle of word, alone, with adjacent punctuation

Example assertions (in your CLI suite):

```pseudocode
assert not containsToken("Edward came over", "war")
assert not containsToken("flying to Tokyo", "ok")
assert containsToken("the war ended", "war")
assert containsToken("you ok?", "ok")
```

### Code review heuristic

When reviewing filter / routing / migration code, scan for:

- `.contains(token)` calls where `token` is a string literal
- String literals shorter than 5 characters in any filter context
- Any forbidden-words list with short entries

Flag every one for review.

---

## Generalization

This pitfall is a special case of *implicit assumption about input shape*. The author wrote `.contains()` assuming the input would be a single word or a clean token list. The input was actually arbitrary natural language.

The same class includes:

- **Email validation by `.contains("@")`** — matches "user@example" without TLD, fails on `user+tag@example.com` corner cases.
- **URL detection by `.contains("http")`** — matches "httpfoo".
- **Code-block detection by `.contains("\`\`\`")`** — fragile against escaped backticks.

The general fix: **never trust string `contains()` against unstructured input for any check that has consequences.** Parse, tokenize, or use a structured matcher.

---

## Discovery story

In the originating project, this bug class was discovered by *ultrareview* — an external review process that audited migration outputs against expected user data. The internal CLI assertion suite covered many cases but had not anticipated names that contained migration target tokens. After the bug was found, the assertion suite grew to specifically cover the class; subsequent regressions were caught in under 10 seconds.

The meta-lesson: author-graded testing misses this class. Outside review catches it. Plan for outside review on any code that touches user data destructively. See Pitfall PF25 for the process pattern.

---

## Severity rationale

**High** because:

- **Silent.** No error is raised. The data is simply gone.
- **Irreversible.** Once user data is deleted, recovery requires backups.
- **Compounds.** Every migration that uses substring matching adds latent risk.

---

*Pitfall PF1 · substring matching against short tokens · 2026*
