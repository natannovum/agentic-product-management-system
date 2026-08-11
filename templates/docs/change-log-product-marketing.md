# Change Log — product marketing

Append-only history of changes to published surfaces — marketing site, user documentation, store or marketplace listings. **Newest entry on top.** Implementation detail behind the `dev-log.md` executive summary for sessions that touched this concern; cross-referenced by date. Pending work lives in `todo-product-marketing.md`.

**Append-only.** Entries are never rewritten. A correction is a new dated entry that supersedes — editing a past entry destroys the record of what was believed at the time, which is frequently what someone is looking for.

**Every entry carries:** the symptom or goal in plain terms as a leading bolded sentence; the mechanism and why this fix over the alternatives; **assumptions that turned out wrong**; and how it was verified, with named checks and a named control.

---

<!-- SAMPLE ENTRY — delete when writing the first real entry. -->

## Sample — YYYY-MM-DD — Feature description updated across listings

- **The "offline mode" paragraph described behaviour removed two releases ago.** Rewritten in `product-marketing/_shared/description.md`.
  - **Pushed:** `<store>-stable`, `<store>-beta`
  - **Staged, not pushed:** `website` — awaiting the next site deploy
  - Marketing site and user manual reviewed for the same claim; the manual's "Working offline" page also referenced it and was updated in the same pull request.
- **Reconciliation, all surfaces:** no other drift found. *(Recorded even when clean — a reconciliation that is not recorded did not happen.)*
