# Change Log — product

Append-only history of changes to the product itself — features, fixes, behaviour, architecture. **Newest entry on top.** Implementation detail behind the `dev-log.md` executive summary for sessions that touched this concern; cross-referenced by date. Pending work lives in `todo-product.md`.

**Append-only.** Entries are never rewritten. A correction is a new dated entry that supersedes — editing a past entry destroys the record of what was believed at the time, which is frequently what someone is looking for.

**Every entry carries:** the symptom or goal in plain terms as a leading bolded sentence; the mechanism and why this fix over the alternatives; **assumptions that turned out wrong**; and how it was verified, with named checks and a named control.

---

<!-- SAMPLE ENTRY — delete when writing the first real entry. -->

## Sample — [X.Y.Z] - YYYY-MM-DD

### Fixed

- **Events queued while the worker was idle were silently dropped on wake.** The queue lived in memory in a process the runtime terminates when idle; nothing in the code indicated this, and the runtime documents it as a performance note rather than a correctness constraint. Now persisted on enqueue and rehydrated on wake (`src/queue.ts`). **Two assumptions were wrong and are worth recording:** that "intermittent" implied a race — it was fully deterministic on idle duration, a variable nobody was measuring; and that the runtime would surface termination to the application — it does not, which is why no error appeared anywhere. Verified with 200 synthetic wakeups across three idle durations against a control build: loss went 30% → 0% past the termination threshold and was unchanged below it, confirming the mechanism rather than masking it. Closes [TICKET-ID].
