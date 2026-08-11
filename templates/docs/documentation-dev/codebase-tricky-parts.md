# Codebase Tricky Parts — Don't Forget

**Current-state truth, rewritten in place.**

**Test for inclusion:** would a competent engineer, reading the code, reach the *wrong* conclusion? If yes, it belongs here.

**Lead every entry with the symptom**, not the cause — the symptom is what a future reader is searching for.

---

<!-- SAMPLE — delete when adding the first real entry. -->

## Sample — State held by an idle-terminated process disappears without an error

**Symptom.** Work queued during a quiet period is silently missing later. No exception, no log line, no failed request. Reproduces "intermittently" — meaning reliably, on a variable nobody is measuring.

**Cause.** The runtime is free to terminate the host process when idle. In-memory state does not survive it. This is documented as a performance characteristic, which reads as a note about speed rather than a constraint on correctness.

**Rule.** Anything that must survive an idle period is persisted at write time, not at shutdown — there is no shutdown hook you can rely on.

**How to notice you have this bug:** the failure rate correlates with *idle duration*, not with load or concurrency. If a "race condition" will not reproduce under stress but appears when the system is quiet, look here first.
