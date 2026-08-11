# Latest Implementation Specs

**Current-state truth, rewritten in place.** How each subsystem works *today*. Read the relevant section before touching that area.

**Keep headings stable** — `CLAUDE.md` and other documents point at sections here by name, and nothing verifies those pointers. **When renaming a heading, search the repository for the old name and repair every hit in the same commit.**

**When a section is superseded, move it to `retired-implementation-specs.md`** with a note naming what replaced it. Do not delete it.

---

<!-- SAMPLE — delete when adding the first real spec. -->

## Sample — Work Queue

**Purpose.** Holds commands between receipt and execution across a lifecycle that may terminate the host process at any idle moment.

**Mechanism.** Enqueue writes through to durable storage before acknowledging. On wake, the queue rehydrates before accepting new work — ordering is preserved because the store is append-only and read back in insertion order.

**Constraints.**
- No reliable termination signal; nothing may depend on shutdown running.
- The idle threshold is not contractual — never hard-code it.

**Known cost.** Write-through on every enqueue is heavier than necessary. Batching is deferred pending a measurement that it matters.

**Related:** `codebase-tricky-parts.md` → idle-termination trap; `design-intentions-and-rationale.md` → persistence at write time.
