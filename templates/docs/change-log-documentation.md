# Change Log — documentation

Append-only history of changes to the documentation system itself — this system's files, CLAUDE.md, agent and hook configuration, the log files. **Newest entry on top.** Implementation detail behind the `dev-log.md` executive summary for sessions that touched this concern; cross-referenced by date. Pending work lives in `todo-documentation.md`.

**Append-only.** Entries are never rewritten. A correction is a new dated entry that supersedes — editing a past entry destroys the record of what was believed at the time, which is frequently what someone is looking for.

**Every entry carries:** the symptom or goal in plain terms as a leading bolded sentence; the mechanism and why this fix over the alternatives; **assumptions that turned out wrong**; and how it was verified, with named checks and a named control.

---

<!-- SAMPLE ENTRY — delete when writing the first real entry. -->

## Sample — YYYY-MM-DD — System installed

Branch `chore/apms-setup`. Executive summary in `dev-log.md`.

### `CLAUDE.md` — **new**

- Generated from the template. Boilerplate sections adopted unchanged.
- **Answers sourced from the repository, not the user:** branch topology (from CI trigger conditions), development commands (package manifest), environment prerequisites (CI setup steps).
- **Answers requiring the user:** the Tier-2 irreversibility list, and the published-surface inventory.
- **Adopted as an explicit default:** semantic versioning with human-controlled bumps. Flagged as a default rather than a decision, so it can be corrected.

### `docs/` — **new**

- Log, todo, and reference-doc structure created from templates. Sample content deleted at first real use.
- **Deliberately not created:** *(name any bucket skipped, and why — an empty change log reads as "nothing changed here" rather than "not tracked")*.

### Still open

- *(anything left undecided at setup, so it is visible rather than forgotten)*
