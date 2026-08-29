# Development Log

Cross-concern executive summary — **one entry per working session**, newest on top. Assignment, reasoning, what went wrong, what was left undone. Implementation detail lives in the change logs, cross-referenced by date.

**Written every session without exception**, including sessions that produced no code: an investigation concluding "this is not the problem" is exactly the finding that otherwise gets repeated.

**Test for a good entry:** if it could be reconstructed from the commit log, it has failed. The value is entirely in what is not recoverable from the repository.

---

<!-- ============================================================
SAMPLE ENTRY — illustrates the expected shape. DELETE THIS BLOCK when writing the first real entry. ============================================================ -->

## Sample — Worker Wakeups Dropped Queued Events — YYYY-MM-DD

**Assignment.** Users reported commands "not registering" intermittently. No error surfaced anywhere. Branch `fix/worker-event-queue`.

**Outcome.** Root-caused and fixed. Events queued during an idle period went from ~30% loss to zero across 200 synthetic wakeups.

### What was actually wrong

The queue was held in memory by a process the runtime is free to terminate when idle. Nothing in the code says so; the runtime documentation does, in a sentence that reads as a performance note rather than a correctness constraint.

### The assumption that was wrong

**"Intermittent means a race condition."** Two sessions were spent looking for one. It was not intermittent at all — it was perfectly deterministic on a condition nobody was measuring (idle duration crossing the termination threshold). Recorded because the reasoning was sound and the conclusion was still wrong: *"no reproducible pattern"* meant *"not measuring the right variable"*, not *"non-deterministic."*

### Verification

200 synthetic wakeups across three idle durations, against a control build. Loss went 30% → 0% for durations past the threshold, and was unchanged (0%) below it — confirming the fix addressed the actual mechanism rather than masking it.

### Left undone

Persistence is now write-through on every enqueue, which is heavier than necessary. Batching was deferred — [TICKET-ID].

### Files changed
| File | Change |
|------|--------|
| `src/queue.ts` | Persist on enqueue; rehydrate on wake |
| `docs/documentation-dev/codebase-tricky-parts.md` | New section on the idle-termination trap |
