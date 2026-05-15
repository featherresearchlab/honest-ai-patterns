# P03 — Memory façade with multiple backings

**Tags:** `[LLM-app]`, `[universal]`

---

## Problem

Memory subsystems accumulate ad-hoc accessors. Different parts of the codebase call different stores directly: a vector DB here, a key-value store there, a relational schema elsewhere. Over time, business logic depends on implementation-specific behavior of each store. Swapping a store requires code-wide refactoring.

Worse: when multiple stores all hold "memory" but in different shapes, callers must know which store to query for which question, and any consolidation or guard logic must be replicated at every call site.

---

## Solution

One front door for every read and write of memory — typically named `Memory.shared` or `MemoryManager.instance`. Behind it, multiple specialized backing stores. Callers do not know which store served them.

Add a CI-enforced rule: the build fails if any production file outside an explicit whitelist imports a backing store directly.

```pseudocode
// public API: methods that express user intent, not storage operations
Memory.shared.remember(key, value)              // routes to FactStore
Memory.shared.searchEpisodes(query)             // routes to VectorIndex
Memory.shared.walk(fromEntity, maxDepth)        // routes to EntityGraph
Memory.shared.loadRecentNotes(days)             // routes to NoteStore
Memory.shared.recall(query, focalEntityID)      // composes across stores

// CI guard
test("noDirectBackingCalls"):
    for file in production_files:
        if file not in WHITELIST:
            assert "FactStore." not in file.source
            assert "VectorIndex." not in file.source
            // ...
```

---

## Shape

The façade exposes **intents**, not **operations**. Expose `recall(query, focalEntity)`, not `getEntityByID() + getMentions()`. Intent-level methods stay stable across implementation changes; operation-level methods leak the backing.

---

## When to use it

- Any product with persistent state in multiple stores
- Personal AI (typically four stores: facts, entities, episodes, notes)
- Knowledge management apps
- Anywhere migration between storage backings is foreseeable

---

## When it fails

### The façade mirrors backing-store methods 1:1

If `Memory.getEntity(id)` just calls `EntityGraph.getEntity(id)` with no abstraction, the façade is a pointless redirection. The discipline is: expose **intents** ("recall what's known about Juan"), not **operations** ("fetch entity row by ID, then fetch its mentions, then join").

If the façade has a method per backing operation, refactor it to combine related operations into intent-level methods.

### The whitelist grows uncontrolled

The CI guard whitelist exists for legitimate exceptions (the façade's own implementation file, the backing stores themselves, test fixtures). Over time, "just this once" exceptions accumulate. Review the whitelist quarterly; reject new additions unless they're inside the memory module.

---

## Evidence

From the originating project:

- Four backing stores (FactStore, EntityGraph, EpisodeIndex, DailyNotes) behind one `Memory.shared` façade
- Migrated EntityGraph from a hash-map prototype to SwiftData with CloudKit sync: zero call-sites changed
- The CI guard caught two direct-backing-call violations during the migration that would have leaked the old API into new code

---

## Related patterns

- **P04 — Source-aware merge:** Implemented inside the façade; callers don't see the merge logic.
- **P05 — First-person disclosure guard:** Enforced at the façade's write path; cannot be bypassed by callers.
- **P10 — CLI assertion suite:** The CI guard is itself a deterministic assertion.

---

## Anti-pattern to avoid

**Letting each call site decide which backing to query.** This is exactly the situation the façade prevents. If callers need to know "this question is a vector-index question, that one is an entity-graph question," the façade has not done its job. Either expose an intent-level method that abstracts over backings, or accept that the façade is leaky and document the leak.

---

*Pattern P03 · memory façade · 2026*
