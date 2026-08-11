# Design Intentions and Rationale

**Current-state truth, rewritten in place.**

Decisions that are cheap to make once and expensive to relitigate. **Record the options rejected and why** — without that, every future session re-proposes them, and the same conversation is had until someone gives in out of fatigue rather than evidence.

---

<!-- SAMPLE — delete when adding the first real entry. -->

## Sample — Persistence at write time rather than at shutdown

**Decision.** State that must survive an idle period is written when it changes, not flushed on termination.

**Why.** The runtime gives no reliable termination signal. A shutdown hook is best-effort and silently does not run in the case that matters most.

**Rejected: flush on a timer.** Simpler and cheaper, but it leaves a loss window whose safe size depends on an idle threshold that is not contractual and has changed before. Trading correctness for throughput against an undocumented constant is not a trade worth making.

**Rejected: keep state in the caller.** Moves the problem rather than solving it, and every caller then needs the same knowledge.

**Revisit if:** the runtime gains a durable termination signal, or write cost is *measured* to matter. It has not been.
