# Product Marketing — source of truth for published surfaces

**The repository is the source of truth; every publishing destination is a deployment target.**

A dashboard, content system, or store console has no meaningful version history, no review step, and no diff. Copy edited there directly is unrecoverable and unreviewable. Copy edited here gets both, and publishing becomes a deployment rather than an act of authorship.

## Structure

```
product-marketing/
├── _shared/              copy used by more than one destination
├── website/              marketing site copy
├── user-manual/          end-user documentation (if not part of the site)
└── store-listings/
    └── <store>-<channel>/    one directory per store × channel
        ├── listing.md        title, summary, description, category
        ├── declarations.md   permissions, data collection, pricing, platforms
        └── assets.md         screenshot / promo inventory and what each shows
```

**One directory per store *and* channel.** Stable and beta are separate entries — each is edited, reviewed and published independently, and they legitimately differ.

**Hold the exact copy as published**, so comparing against the live destination is mechanical rather than a judgment call. Where copy is genuinely shared, keep it in `_shared/` and reference it — duplicated copy is what drifts.

## Rules

1. **A change to `_shared/` states which destinations it was pushed to.** Updated here but applied to only some destinations is the likeliest drift source, and invisible without this.
2. **Reconcile on a schedule** and record the result, **including "no drift."** A reconciliation that is not recorded did not happen.
3. **Where channels deliberately differ, say so in the file** — or the next reconciliation "fixes" a deliberate difference.
4. **Declarations are not discretionary.** Anything altering what a listing must declare — permissions, data collected, pricing, supported platforms — makes that text stale immediately. This belongs in the pull-request checklist, because the alternative is discovering the mismatch during a platform review.

## Delete what does not apply

A project with no store presence deletes `store-listings/`. A project whose manual is part of the marketing site deletes `user-manual/`. **Do not keep empty directories** — they read as "nothing here yet" rather than "not applicable."
