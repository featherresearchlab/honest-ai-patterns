# PF17 — Dual on-device LLM engines competing for Metal context

**Severity:** Critical. Reproducible production crashes.
**Category:** Mobile / iOS platform.
**Detection:** Device-class memory budget calculation before architecture commitment.

---

## Symptom

Reproducible SIGABRT crashes on memory-constrained devices (8 GB iPhones) during sustained chat use. User-visible symptoms:

- App crashes mid-response
- iOS jetsam (low-memory killer) evicts SpringBoard alongside the app, producing a "all my home-screen apps are reloading" effect
- Crash logs show a stack like:
  ```
  ggml_uncaught_exception
  → mlx::core::gpu::check_error
  → IOGPU
  → SIGABRT
  ```

The crashes occur preferentially during turns where both engines are warm (parallel-fanout patterns racing two engines for the same reply).

---

## Root cause

Two on-device LLM inference engines were resident in memory simultaneously. Memory budget on an 8 GB iPhone (specific to the originating project):

- iOS reservation: ~1.5 GB
- Engine A (MLX-Swift): ~4.5 GB resident at runtime including KV cache
- Engine B (llama.cpp): ~2.7 GB resident
- App + UI overhead: ~0.8 GB
- **Total: ~9.5 GB on a device with ~8 GB total RAM**

The system survived light use because not all memory was committed simultaneously. Under sustained chat load (memory pressure → KV cache grows → GPU command queue contention), Engine A's GPU error check threw a C++ exception. This C++ exception routed through Engine B's globally-registered uncaught-exception handler (which ggml's dylib installs at process startup), producing SIGABRT.

The cause is not either engine in isolation. Both are well-engineered. The cause is **the dual-resident architecture on a device class that cannot host both.**

The handler interaction is especially insidious: even if Engine A is set as primary and Engine B's inference functions are never called, *linking Engine B's framework* installs the global C++ terminate handler that swallows Engine A's errors.

---

## Fix

**Hard-cut to a single resident engine on memory-constrained devices.** Do not ship both engines simultaneously.

If migrating between engines:

1. Make the new engine primary behind a feature flag.
2. Verify stability for one week of telemetry.
3. **Remove the old engine's framework dependency entirely** — not just stub the call sites. Linking the framework is enough to install global handlers.
4. Ship the framework removal as a hard cut.

Verification after removal:

```bash
nm App.debug.dylib | grep ggml_uncaught_exception   # must return empty
```

If the handler is still in the binary, the framework is still linked. Hunt the residual dependency.

---

## How to detect it earlier

### Memory budget calculation before architecture commitment

For each target device class:

```
device_RAM
  - iOS_reservation
  - peak_RAM_engine_A
  - peak_RAM_engine_B  // if running concurrently
  - app_overhead
  = remaining_headroom
```

If `remaining_headroom < 0` on your minimum supported device, you have this bug class waiting.

### Telemetry during early development

If you are considering dual-engine, log peak memory pressure events. If you see `.warning` or `.critical` jetsam events during sustained inference, the architecture is already too tight; jetsam will eventually evict the app.

### Crash log signature

The specific stack `ggml_uncaught_exception → mlx::core::gpu::check_error` is diagnostic. If you see it more than once across users on the same device class, the architecture is the cause, not a transient bug.

---

## Generalization

This is a special case of **resource competition on a constrained platform with global framework handlers**. The same class includes:

- Two Metal frameworks competing for GPU command queues
- Two CUDA contexts on a single GPU (server-side analog)
- Multiple inference runtimes in a single Python process where one installs a signal handler
- Generally: any platform where "framework X is linked" implies "X's global state is live"

The general lesson: on resource-constrained platforms, "one resource consumer at a time" must be enforced at the **link layer**, not just the call-site layer.

---

## Discovery story

In the originating project, this bug class was discovered through four reproducible crashes on a single day on an iPhone 16 Pro. Crash logs all showed the same stack. The initial fix attempt was call-site stubbing of the old engine; this did not resolve the issue because the framework was still linked. The hard-cut framework removal one day later closed the crash class.

Discovery cost: one week of investigation and multiple failed call-site-only fixes. Prevention cost if planned from the start: zero — just do not ship both engines.

---

## Severity rationale

**Critical** because:

- **Production crashes** on the minimum supported device class
- **Cascading impact** — iOS jetsam evicts other apps too, harming trust in the device, not just the app
- **Hard to diagnose** without the specific crash-log stack
- **No graceful degradation** — when the crash fires, the user loses the in-progress chat

---

*Pitfall PF17 · dual on-device LLM engines · 2026*
