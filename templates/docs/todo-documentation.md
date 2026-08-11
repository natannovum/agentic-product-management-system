# Todo — documentation

Pending work for the documentation system itself. Completed items are recorded in `change-log-documentation.md` with a date stamp — **closing an item here is not a change-log entry.**

> **SETUP — keep ONE of the two mode blocks below and delete the other.**

---

## Mode A — tracker-backed *(delete if Mode B)*

**`<TRACKER>` is authoritative for status, priority and assignment.** This file is a committed **index**, so a fresh checkout still shows what is outstanding without needing access to the tracker.

Keep it to one row per open item. Detail belongs in the ticket; **rationale that outlives the ticket belongs in `documentation-dev/`.**

| Ticket | Title | Summary | Link |
|---|---|---|---|
| TICKET-1 | *(sample — delete)* Batch queue persistence writes | Write-through on every enqueue is heavier than needed; batch without reintroducing the loss window. | https://… |

---

## Mode B — self-contained *(delete if Mode A)*

> **This mode is a fallback, not the intended design.** Markdown has no state field, no archive, and no forcing function to close anything, so items accumulate and never leave. Left alone, this file becomes an undifferentiated mixture of stale inventories, roadmaps and live items — at which point it is worse than no record, because it still looks authoritative.
>
> **Triage on a schedule:** close what is done, delete what is dead, move anything that is really a specification into `documentation-dev/`. **If triage keeps slipping, that is the signal to adopt a tracker.**

### *(sample item — delete)* Batch queue persistence writes

**Why:** write-through on every enqueue was the safe fix, not the right one. Measurable cost under load.

**Constraint:** any batching window reintroduces a loss window. The window must be shorter than the runtime's idle-termination threshold, which is not contractual and has changed before — do not hard-code it.

**Prerequisite:** a load measurement establishing the actual cost. It has not been shown to matter yet.

**Picked up cold, a reader needs:** `documentation-dev/codebase-tricky-parts.md` → idle-termination trap.
