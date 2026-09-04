# Tooling and Infrastructure Change Log

> **Template.** Delete this block. Pair with `todo-tooling-and-infrastructure.md`; keep both or neither.

Internal log of **releases and deployments** of everything that supports the product without being the product: rollouts, servers, the test harness, provisioning and credential tooling.

One entry per release or deployment, dated, newest first. Unlike a customer-facing log this also records **release candidates and staging deploys**, marked as such — the value here is internal evidence of what was running where, and when.

Pending work is in `todo-tooling-and-infrastructure.md`.

---

## <date> — <what was released or deployed>

<What went out, to where, and how many targets.>

<What it was built from — the merged commit or tag, not a branch — and what was checked about the artifact **before** it was distributed.>

**Verified from outside the tool.** <A deploy script that reads its own result back proves the call, not the result. State what was checked independently, and pick a signal the new version has and the old one does not: a token, a string, a version endpoint. A changed hash is corroboration; the new thing appearing is the proof.>

<Anything deliberately not touched — configuration, per-target settings, data.>

---
