# P02 — Three-tier extraction

**Tags:** `[LLM-app]`

---

## Problem

Extracting structured data from natural language with an LLM produces outputs that:

- Drift from the requested format (whitespace, extra fields, prose around the JSON)
- Contain hallucinations (made-up fields, fabricated values)
- Include out-of-vocabulary content (categories or keys not in your schema)

A single layer of defense — just the prompt, or just structured output, or just a post-filter — leaves gaps. The model is non-deterministic; one layer is not enough.

---

## Solution

Three tiers running in sequence:

1. **Tier 1 — Prompt with explicit format and few-shot examples.** Teach the model the shape you want.
2. **Tier 2 — Structured-output constraint.** JSON schema (Apple `@Generable`, OpenAI structured output, Outlines, jsonformer). Force the shape at decoding time.
3. **Tier 3 — Post-filter.** Reject outputs that violate semantic rules the schema cannot express (vocabulary blocklists, length constraints, value-key consistency).

Each tier catches what the previous missed:

- Tier 1 reduces drift but does not enforce it
- Tier 2 enforces shape but cannot enforce semantics
- Tier 3 enforces semantics but is only reachable if Tier 2 passed

---

## Shape

```pseudocode
function extract(input):
    // Tier 1: prompt
    prompt = buildPrompt(input, fewShotExamples)

    // Tier 2: structured output
    rawOutput = llm.generate(prompt, schema=ExtractionSchema)

    // Tier 3: post-filter
    cleaned = applyPostFilters(
        rawOutput,
        vocabulary=ALLOWED_KEYS,
        blocklist=POLLUTION_TOKENS
    )
    return cleaned

function applyPostFilters(facts, vocabulary, blocklist):
    output = []
    for fact in facts:
        if fact.key not in vocabulary: continue
        if any(token in blocklist for token in tokenize(fact.value)): continue
        if not passesValueShapeCheck(fact): continue
        output.append(fact)
    return output
```

---

## When to use it

- Any structured extraction from natural language with an LLM
- Fact extraction from chat for memory storage
- Entity extraction for knowledge graphs
- Intent + slot extraction for tool calling
- Anywhere a single-tier extraction misses bugs

---

## When it fails

### The post-filter becomes maintenance debt

Every new failure mode adds a rule to Tier 3. Rules accumulate. Eventually the filter is harder to reason about than the model.

Mitigation:
- Periodic consolidation: review the post-filter quarterly; merge similar rules; remove rules that have not fired in months
- Document each rule's origin (the bug that motivated it)
- Test the post-filter independently with a CLI assertion suite (Pattern P10)

### Tier 2 is not available

Some inference engines do not support structured output natively. Without Tier 2, Tier 3 has to do twice the work (enforcing both shape and semantics).

Mitigation:
- Use a parsing library tolerant of common LLM output drift (extract JSON from prose, handle object-vs-array, etc.)
- Or: explicitly post-validate the schema in Tier 3 before semantic checks

---

## Evidence

From the originating project:

- Memory extraction pipeline used all three tiers
- Tier 1 alone (prompt only): roughly 30% of extractions had at least one format issue
- Tier 1 + Tier 2 (prompt + Apple `@Generable`): roughly 5% had remaining semantic issues (e.g., pollution in "interest" fields)
- Tier 1 + Tier 2 + Tier 3 (full): essentially zero schema or semantic violations across the project's logged extractions

(Numbers are project-specific. Your mileage will vary by domain, model, and schema.)

---

## Related patterns

- **P01 — Pre-filter before LLM:** Tier 0, applied before any of these tiers, to skip the LLM entirely on inputs that don't need extraction.
- **P05 — First-person disclosure guard:** A Tier 4 if you count it. Applies after extraction passes all three tiers, to enforce additional cross-fact constraints.
- **P10 — CLI assertion suite:** Essential for testing the post-filter against drift.

---

## Anti-pattern to avoid

**Relying on Tier 1 alone "because the model is good enough."** Even frontier models drift on edge cases. The structured-output constraint (Tier 2) is cheap to add and catches a different bug class. Skipping it is false economy.

**Combining all three tiers into one giant prompt with rules.** The model treats prompt rules as suggestions. Schema constraints are enforced; post-filter checks run regardless of the model's compliance. Do not conflate them.

---

*Pattern P02 · three-tier extraction · 2026*
