# P5 — First-person disclosure overwrite guard

**Tags:** `[LLM-app]`

---

## Problem

LLM extraction will sometimes confidently assert that a user-self attribute should be updated based on a third-person mention.

Example: a user writes "My sister Dana lives in Madrid. She loves jazz." An LLM extraction pipeline might extract:

```json
[
  {"key": "user_sister", "value": "Dana"},
  {"key": "user_location", "value": "Madrid"},
  {"key": "user_interests", "value": "jazz"}
]
```

The first extraction is correct. The second is a *hallucinated overwrite* of the user's own location, based on a fact about their sister. The third is a contamination of the user's interest list with their sister's interest.

This is not rare. We observed it across multiple LLM backends (Apple Foundation Models, Gemma 4 E2B, others). It tends to fire on messages where the LLM identifies a strong attribute signal ("lives in Madrid") and incorrectly attributes the subject.

---

## Solution

Before overwriting an existing user-self attribute, require a *first-person verb* in the message that originated the extraction.

User-self attributes are those that describe the user themselves: name, location, birthplace, age, work, role, interests, birthday. These have a single canonical owner — the user.

Third-party attributes are those that describe other people the user has disclosed: sister, brother, friend, partner. These have multiple owners.

The guard rejects extractions that would *overwrite* a user-self attribute when the originating message lacks a first-person verb.

---

## Shape

```pseudocode
function applyExtraction(extraction, originatingMessage, existingState):
    for fact in extraction:
        if isUserSelfAttribute(fact.key):
            if existingState.has(fact.key):
                // would overwrite
                if not containsFirstPersonVerb(originatingMessage):
                    log.warning("blocked overwrite of user-self attribute without first-person disclosure")
                    continue
            existingState.set(fact.key, fact.value, source: "first-person")
        else:
            // third-party attribute, apply normally
            existingState.set(fact.key, fact.value, source: "disclosed")
```

First-person verb detection requires per-language verb lists:

```pseudocode
function containsFirstPersonVerb(text, language):
    verbs = FIRST_PERSON_VERBS[language]  // "I am", "I live", "I work", etc.
    return any(verb appears as a word boundary match in text)
```

Critically, the match must be word-boundary aware (Pattern P6) — `text.contains("I am")` matches "Liam" and "Miami" without boundaries.

---

## When to use it

- **Any personal AI** that maintains user-self attributes derived from natural-language extraction.
- **Especially** when the LLM is not state-of-the-art (smaller on-device models hallucinate attribution more often).
- **Always for irreversible writes.** If the system overwrites instead of appending, this guard is mandatory.

---

## When it fails

### Languages without explicit subject pronouns

Some languages (Spanish, Italian, Hebrew, Japanese) routinely drop the subject pronoun. "Vivo en Madrid" (I live in Madrid) has no separate "I". The verb conjugation carries the first-person marker.

**Mitigation:** Verb lists must include conjugated first-person forms. Maintain per-language detection logic.

### Fused first-person markers

Some languages (Hebrew, Arabic) fuse first-person possessive markers as affixes. "אחותי" (my-sister) is one word combining "sister" and "my."

**Mitigation:** Language-specific tokenization. Possessive-aware regex per language.

### Indirect disclosures

"I'm flying to Madrid tomorrow" doesn't say the user *lives* in Madrid, but a careless LLM might extract `user_location = "Madrid"`. The guard passes (first-person verb present), but the extraction is still wrong.

**Mitigation:** This is a separate problem — the guard prevents third-person contamination; correctness within first-person disclosures is the LLM's job. Use structured extraction (Pattern P2) with explicit semantic distinctions.

---

## Evidence

From the originating project:

- Before this guard: user's stored location was overwritten from Madrid to Tel Aviv based on a chat about their sister. User's friend slot was overwritten from Dana to Liat (Liat was someone else's friend mentioned in the conversation). User's age was deleted entirely.
- After this guard: zero such cross-contaminations across 3,200 logged extractions.
- The bug class was caught by ultrareview (external review of memory state), not by the author. See Pitfall PF25 for the meta-lesson.

---

## Related patterns

- **P4 — Source-aware entity merge:** Companion pattern. The guard prevents user-self contamination; source-aware merge prevents contact-vs-chat contamination.
- **P6 — Word-boundary tokenization:** The first-person verb detection must be word-boundary aware.
- **P10 — CLI assertion suite:** Test inputs should include third-person disclosure messages with assertions that the guard fires.

---

## Anti-pattern to avoid

**Allowing extraction to overwrite user-self attributes "to keep memory up to date."** Users rarely change their own name or birthplace. Frequent overwrites of these attributes are a strong signal of LLM misattribution, not of true user updates.

If a user genuinely changes their location, they will say so in the first person ("I just moved to Berlin"). That message passes the guard.

---

*Pattern P5 · first-person disclosure guard · 2026*
