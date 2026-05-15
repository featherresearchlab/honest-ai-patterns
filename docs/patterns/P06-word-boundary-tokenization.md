# P06 — Word-boundary tokenization for safety filters

**Tags:** `[universal]`

---

## Problem

Substring matching against short tokens produces silent corruption. Code that looks correct silently misclassifies, mis-routes, or deletes things it should not.

Concrete examples:

- A filter blocking the word `"war"` silently matches **Edward**, **Howard**, **Stewart**
- A safety filter on `"ass"` silently matches **associate**, **assistant**, **passion**
- A routing keyword `"ok"` matches **looking**, **Tokyo**, **Oklahoma**, **Bangkok**

The shorter the token, the higher the false-positive rate. A 2-character token will appear inside almost every English sentence.

---

## Solution

Tokenize on whitespace and punctuation. Require exact-token match for short tokens (≤4 characters); longer tokens may substring.

```pseudocode
function containsToken(text, target):
    if length(target) <= 4:
        // exact-token match required
        tokens = tokenize(text, on=[whitespace, punctuation, apostrophe, hyphen])
        return target in tokens
    else:
        // longer tokens: word-boundary match is sufficient
        return regex.match("\\b" + escape(target) + "\\b", text) is not null
```

Or equivalently, always use word-boundary regex regardless of length:

```pseudocode
function containsToken(text, target):
    return regex.match("\\b" + escape(target) + "\\b", text) is not null
```

The length-gate version is slightly more permissive on longer tokens (allows compound matches if your language uses compounds heavily); the always-regex version is stricter. Choose based on your input language.

---

## When to use it

Always, when:

- Filter, router, or migration logic uses string comparison
- Input is user-generated natural language
- Token length is short (≤4 chars) or could be a substring of common words
- Wrong matches have consequences (data deletion, mis-routing, content filtering)

---

## When it fails

It does not fail much in practice. Word-boundary matching is rarely *over*-restrictive on natural language. The performance cost is negligible (regex with word boundaries is microseconds on typical filter inputs).

The only situation where substring matching is correct: when the input is *known to be tokenized already* (a list of single words from a database, not user-generated prose). Even then, the safer default is word-boundary matching.

---

## Evidence

From the originating project:

- A migration designed to scrub topic-pollution entities (`"war"`, `"king"`, `"best"`, etc.) used substring matching in its first version
- The migration would have hard-deleted real user entities — anyone named **Edward**, **Howard**, **Stewart** (all contain "war"), **Pete Best** (contains "best"), **Kingsley** (contains "king")
- Caught by external review, not by the author's own testing
- The v2 migration used word-boundary tokenization and ran safely

---

## Related patterns

- **P10 — CLI assertion suite:** Catch substring-leak bugs by including test inputs that contain target tokens inside legitimate longer words.
- **P07 — Idempotent versioned migrations:** If your v1 has a substring leak, the v2 namespacing pattern lets you retry safely.

---

## Anti-pattern to avoid

**`text.contains(token)` against any user-generated string,** even for tokens longer than 4 characters. The few microseconds saved are not worth the latent risk of silent corruption.

Even more dangerous: **chained `contains()` checks with OR.** If any single token in the list is short, the entire filter has a substring leak.

---

*Pattern P06 · word-boundary tokenization · 2026*
