# PF05 — LLM hallucinates user-self disclosures from third-person mentions

**Severity:** High. Causes silent data corruption of user profile.
**Category:** LLM extraction.
**Detection:** Ground-truth corpus of third-person disclosures with assertions that no user-self attribute is updated.

---

## Symptom

A user-self attribute (name, location, age, work, role, birthplace) is silently overwritten with a value extracted from a message that was actually about a third party.

Concrete examples from production:

- User wrote *"My sister Dana just moved to Madrid and she loves jazz."* Stored: `user_location = "Madrid"`, `user_interests = "jazz"`. Correct would have been: `user_sister = "Dana"`, `dana.location = "Madrid"`, `dana.interests = "jazz"`.
- User wrote *"I had coffee with my friend Liat today. She's an architect."* Stored: `user_friend = "Liat"` (correct), `user_role = "architect"` (incorrect — that's Liat's role, not the user's).
- User wrote *"My brother Avi died last year. He was 41."* Stored: `user_age = "41"` (incorrect, and emotionally awful).

The bug is **silent.** No error logged, no warning displayed. The user only finds out when the AI later says something confidently wrong: *"You're 41 years old."*

---

## Root cause

LLM extraction pipelines run pattern matching on the input. They identify strong attribute signals (`"lives in"`, `"is 41"`, `"loves jazz"`) and emit them as facts. When the message is about a third party, the LLM sometimes fails to correctly attribute the fact to its true subject.

Smaller models hallucinate this attribution more often than larger ones. We observed it with Apple Foundation Models, Gemma 4 E2B, and others. It is not a single-model bug; it is a broad extraction failure mode.

The bug is amplified when the extraction prompt says *"extract facts about the user"* — the LLM reads "the user" as a fixed referent and biases toward attributing extracted facts to it, regardless of the actual subject in the message.

---

## Fix

Implement the **first-person disclosure overwrite guard** (Pattern P05): before overwriting an existing user-self attribute, require a first-person verb in the originating message.

```pseudocode
function applyExtraction(extraction, originatingMessage):
    for fact in extraction:
        if isUserSelfAttribute(fact.key) and existingState.has(fact.key):
            // would overwrite
            if not containsFirstPersonVerb(originatingMessage):
                log.warning("blocked overwrite without first-person disclosure")
                continue
        existingState.set(fact.key, fact.value)
```

First-person verb detection requires per-language verb lists. Critically: use **word-boundary matching** (Pattern P06) for the verb detection — `text.contains("I am")` matches "Liam" and "Miami" without word boundaries.

For NEW user-self attributes (first time the slot is being filled), the guard is more permissive — accept the extraction unless there's a clear third-person marker. For **overwrites** of existing data, the bar is high.

---

## How to detect it earlier

### Test inputs to include in your assertion suite

```pseudocode
state = StateWithUserLocation("Madrid")

assert applyExtraction("My sister Dana lives in Berlin", state).user_location == "Madrid"
assert applyExtraction("I just moved to Berlin", state).user_location == "Berlin"
assert applyExtraction("My friend works as a doctor", state).user_role == state.user_role
```

Include at least one message per user-self attribute, with both the third-person case (assert NOT changed) and the first-person case (assert changed).

### Code review heuristic

Any code path that writes to a user-self attribute should be gated by either:

- A first-person verb check on the originating message, OR
- An explicit "this is a first-time fill, not an overwrite" path that is only taken when the slot is currently empty

Search the codebase for direct writes to `user_location`, `user_age`, etc. Verify each is gated correctly.

---

## Generalization

This is a special case of **attribution-failure-during-extraction**. The same class includes:

- Extracting third-party preferences as the user's preferences (*"My boss hates Mondays"* → user hates Mondays)
- Extracting third-party traits as user traits (*"My ex was a liar"* → user is a liar — particularly damaging in personality-aware AI)
- Misattributing relationship contexts (*"My partner's mom"* → user's mom)

The general fix: explicit subject-of-attribution tracking during extraction, not implicit "facts about the user" semantics.

---

## Discovery story

In the originating project, this bug was found by external review of stored memory state, not by the author's own testing. The author was the only user and was not surprised by the AI's claims — until an obviously wrong attribute (an age that was not the user's age) surfaced. By then, several attributes had been silently overwritten.

The fix landed late in the project. Discovery cost: weeks. Prevention cost if implemented from the start: hours.

---

## Severity rationale

**High** because:

- **Silent.** No error. The user only finds out from confidently wrong AI claims.
- **Emotionally damaging.** Wrong personal facts (age, location, relationships, history) feel like the system "doesn't know me" or worse.
- **Compounding.** Every wrong overwrite increases distrust. Once trust is gone, every correct claim is doubted.

---

*Pitfall PF05 · LLM hallucinates user-self disclosures · 2026*
