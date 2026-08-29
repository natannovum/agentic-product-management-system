# Retired Implementation Specs

**Superseded sections of `latest-implementation-specs.md`, kept rather than deleted.**

**Why the ceremony is worth the friction.** Superseded designs get consulted more often than expected: when a new approach hits a wall the old one had already solved, when a regression traces to behaviour that was intentional under the previous design, or when the reason an approach was abandoned turns out to answer a current question.

Deleting a section makes it unrecoverable *in practice*. It survives in version history, but nobody performs archaeology on a deleted documentation section — the cost of finding it exceeds the cost of re-deriving it badly, so it gets re-derived badly.

**When retiring:** move the section here whole, add the note below, and remove it from the current specs in the same commit.

---

<!-- SAMPLE — delete when retiring the first real spec. -->

## Sample — Work Queue (timer-flush design)

> **Retired YYYY-MM-DD.** Superseded by write-through persistence — see `latest-implementation-specs.md` → "Work Queue". **Reason:** the flush interval had to be shorter than an idle threshold that is not contractual and has changed at least once. The design was correct only by coincidence. **Revisit if:** the runtime ever provides a durable termination signal, which would make a timer-based approach safe again.

*(the retired section's original text follows, unmodified)*
