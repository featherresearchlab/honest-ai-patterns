# P1 — Pre-filter before LLM

**Tags:** `[LLM-app]`

---

## Problem

LLM calls cost latency, battery, and quota. Many inputs do not require model judgment. Calling the model on every input wastes resources and slows the user-facing experience.

This is especially expensive when:
- The LLM is on-device (drains battery, generates heat)
- The LLM is cloud-hosted (paid per token, rate-limited)
- The call is on a hot path (every user action) rather than a deliberate user request

---

## Solution

A cheap, deterministic check *before* the LLM call. Inputs that fail the check skip the model entirely.

The check can be:

- **A regex.** "Does this message contain at least one disclosure verb?"
- **A keyword scan.** "Does this message contain any of these 50 trigger phrases?"
- **A length gate.** "Is this message at least 30 characters?"
- **A language detector.** "Is this in a language the model handles well?"
- **A heuristic combination.** Multiple cheap signals composed.

The fail-closed default matters: when the pre-filter is uncertain, it should *let the input through* to the LLM rather than reject it. Better to over-call the model than to silently drop legitimate inputs.

---

## Shape

```pseudocode
function processInput(input):
    if not preFilter(input):
        return  // skip LLM entirely

    result = llm.call(input)
    return postProcess(result)

function preFilter(input):
    // cheap deterministic check
    if length(input) < MIN_LENGTH:
        return false
    if not containsAnyTrigger(input, TRIGGER_PHRASES):
        return false
    return true
```

---

## When to use it

- **Memory extraction pipelines.** Only ~20% of conversational turns contain extractable facts; the rest can be rejected cheaply.
- **Intent classification on hot paths.** A keyword router can handle 70-90% of routing decisions in milliseconds; the LLM classifier only sees the ambiguous cases.
- **Content moderation.** Cheap signals (URL detection, length, all-caps) before expensive model calls.
- **Smart-reply suggestions.** Many turns don't warrant suggestions; cheap heuristics skip them.

---

## When it fails

### Multilingual products

Pre-filters in multilingual products become maintenance debt. Each new language requires keyword translation. The filter risks rejecting legitimate inputs whose vocabulary the maintainer did not anticipate.

**Mitigation:**
- Maintain trigger lists per language.
- Add a "uncertain" path that defaults to the LLM rather than rejection.
- Test the filter with a multilingual assertion suite (see Pattern P10).

### Adversarial inputs

A motivated attacker can craft inputs that pass the pre-filter but waste LLM resources, or fail the pre-filter to exfiltrate around it.

**Mitigation:**
- Don't use pre-filtering for security boundaries. Use it for resource optimization only.
- Defense-in-depth: pre-filter → LLM → post-filter, each catching different classes.

### Drift

The pre-filter and the LLM's actual behavior drift apart over time. The filter starts rejecting things the LLM would have handled fine, or accepting things the LLM consistently fails on.

**Mitigation:**
- Periodic audit: sample inputs that the filter rejected, run them through the LLM, check whether the rejection was correct.
- Log filter decisions to a structured journey log (see Pattern P8).

---

## Evidence

From the originating project (Feather):

- Memory extraction pre-filter: rejected approximately 60-90% of conversational turns. Of the rejected turns, manual sampling found a false-rejection rate of approximately 2-5% (turns the LLM would have extracted something useful from). The cost of those false rejections was lower than the cost of paying for LLM calls on every turn.
- 16-language trigger lists, ranging from a few dozen phrases per language to over 100 for English.
- Detection signal for filter bugs: the CLI assertion suite (Pattern P10) included multilingual disclosure inputs; any regression in the filter showed up as a failed assertion in under 10 seconds.

---

## Related patterns

- **P2 — Three-tier extraction:** Pre-filter is conceptually the zeroth tier; the LLM is tier 1; the post-filter is tier 2/3.
- **P10 — CLI assertion suite:** Essential for maintaining a pre-filter without regression.
- **P8 — Journey log:** Log every pre-filter decision so drift is detectable.

---

## Anti-pattern to avoid

**Hard-coding pre-filter logic in the LLM call site.** Keep the pre-filter as a separate function with its own tests. Otherwise refactoring the LLM call also affects routing, and the assertion suite cannot test the filter in isolation.

---

*Pattern P1 · pre-filter before LLM · 2026*
