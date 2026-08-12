# CLAUDE.md

Guidance for the coding agent working in this repository. Canonical home for **project setup, workflows, collaboration rules, an index of where other documentation lives, and critical "do not forget" reminders.** Detailed specifications live in `docs/documentation-dev/` — see the [Index](#index).

> **SETUP TEMPLATE — delete this block when setup is complete.**
>
> Sections marked **`SETUP — S<n>`** are project-specific and empty. For each, follow `setup-protocol.md`: **harvest** any rule already written in an existing agent-instruction file, **inspect** the repository and infrastructure for what they can answer, then **interview** the user for the remainder. Adopt defaults visibly — write them in and say they were defaults.
>
> Everything not marked `SETUP` is boilerplate that holds for any project. Adapt examples to this stack; do not weaken the rules.
>
> **No `SETUP` block may survive into normal use.** A placeholder that stops being noticed is a gap that has stopped being visible.

---

## Project Overview

> **SETUP — S11.** A short orientation: what this product is, its major subsystems, and how they communicate. Keep it short deliberately — detail belongs in `docs/documentation-dev/latest-implementation-specs.md`, reached through the Index. An architecture section here grows without bound and goes stale invisibly.

## Development Commands

> **SETUP — S8.** Build, run, watch, test, lint, package. Read the package manifest and CI setup steps first; they are a working environment specification by construction.

## Development Environment

> **SETUP — S8.** Prerequisites and their required versions; the exact daily startup commands, one block per process that must run concurrently; configuration file locations and which must not be edited casually; **and the failure signature of a wrong runtime version.** The expensive failures are the ones that fail confusingly rather than cleanly.

Infrastructure, environments, and the branch-to-environment mapping are documented in `docs/documentation-dev/infrastructure-and-repo-model.md`.

---

## 🔴 Secret Handling — Non-Negotiable

**A pre-commit hook guards version history. It does not guard the terminal, the transcript, or a tool result.** A credential printed to standard output has already leaked — no hook, no ignore file, and no undo covers that path.

1. **Never retrieve a secret value you do not need.** Filter at the source so the value is never fetched. Nearly every question about configuration is about *shape*, not *value* — `cut -d= -f1 .env` answers "which keys exist"; `cat .env` answers a question nobody asked.
2. **Output-side redaction is a bug, not a control.** Piping through a filter to mask a secret fails silently, and the failure is discovered only after the leak. **If a command's output could contain a secret, the command is wrong — rewrite it, do not filter it.**
3. **Enumeration is how leaks happen. Name what you want.** A wildcard is a request for "everything matching", and secrets match. `SELECT *`, `env`, `printenv`, container introspection, `cat` on a config file — each returns a haystack containing needles. Name the columns, the keys, the variable. To compare secrets without seeing them, use a hash or a length.
4. **If a value genuinely must be used, keep it out of the transcript.** Pipe it host-side, reference it by variable, or have the user place it into the target system directly. Read it into a shell variable and test *against* it; never echo it.
5. **If a secret leaks, say so immediately and in the same turn.** Name the credential, the blast radius, and the rotation path. Do not bury it, defer it, or argue the exposure was probably harmless. Then rotate — assume compromise.

> **SETUP — S1.** List this project's radioactive files, paths, and commands. Names and locations only — **never a value, and refuse one if offered.** Add a do/don't table with this stack's specific formulations.

### Enforcement — a hook, not a habit

`.claude/hooks/block-secret-reads.sh` runs as a `PreToolUse` hook and **refuses** the dangerous commands before they execute. Rules that depend on the agent remembering them have already been shown to fail.

- **Regression suite:** run it after any pattern edit.
- **No bypass token, deliberately.** If it fires on something legitimate, tighten the pattern; do not add an escape hatch.
- **Prose about the blocked patterns trips it too.** The guard inspects the whole command string, so a commit message describing a blocked command is itself refused. Pass such text by file path. Known, accepted cost.
- **It is a guard rail, not a sandbox.** It stops carelessness, not determination.

---

## Branch Strategy

**Working branches:** `feature/*`, `fix/*`, `docs/*`, `chore/*`. *(Preset — widely used, mirrors Conventional Commits. If this repository already uses a different convention, keep the existing one; consistency with existing history wins.)*

> **SETUP — S5.** Fill in the topology. **Read the CI trigger conditions first — they state authoritatively which branches cause deployments, which is usually decisive.** A branch that triggers a deploy is an environment boundary; a branch that triggers nothing is a convenience.
>
> | Branch | Purpose | What happens on merge |
> |---|---|---|
> | | | |
>
> **Default:** a single integration branch plus working branches, releases cut by tag. **Do not adopt a "the integration branch must stay release-quality" rule unless something concrete depends on it** — a shared test environment, a staging deploy, other people's work. Without such a dependency it is ceremony, and it will be resented and then ignored.

---

## Collaboration Rules

### Permission and risk tiers

**Approval friction is calibrated to irreversibility, not to how dangerous an action sounds.** The dividing question: can this be undone with version control or an existing backup? Deleting a tracked file sounds alarming and is trivially recoverable; a one-word edit to live customer-facing copy sounds harmless and is not.

**Tier 0 — autonomous.** Anything recoverable from version control or a backup: file edits, commits and pushes on working branches, tags, pull-request creation, builds, local runs, test suites, documentation and log updates. Execute without asking; never interrupt a working round for these.

**Tier 1 — documented procedures, run end-to-end.** Written procedures run start to finish with no mid-flight permission stops. **Pre-flight exception check:** before starting, confirm the situation actually fits the procedure. If anything suggests deviation — unusual state, a missing prerequisite, a step needing extra input or a Tier-2 permission not normally part of it — surface it and re-align **before** execution begins, not halfway through.

**Tier 2 — plan, then explicit approval.** Anything mutating shared or live state beyond recovery.

> **SETUP — S4.** List this project's Tier-2 actions, **each with the reason it is irreversible** — the reason is what lets the boundary be applied to situations nobody listed.
>
> Almost always Tier 2: deploying or publishing anything users see; direct edits to a live content system; anything touching signing keys or a publisher account; force-push, hard reset, remote branch deletion; changes to CI, build, or release automation; anything touching user data or telemetry; merges to a branch that triggers a deployment.
>
> **This item has no safe default.** An incomplete list is worse than an absent one, because it reads as authoritative and the gap is invisible. If the user cannot answer fully, record what is known, **mark the list explicitly incomplete**, and treat anything unlisted-but-suspicious as Tier 2 until classified.

**Protocol for Tier 2:** plan the end-to-end execution, present what it does, why it is needed, what could go wrong, and whether it can be rolled back — then wait for explicit approval, all **before** starting. Never request a Tier-2 permission casually as a mid-task fallback; if a task unexpectedly needs one that was not discussed, stop, explain, and re-plan.

**Permissions are settled during alignment, never during execution.** Every permission a task will need is surfaced while the plan is being agreed. Mid-execution is the wrong moment — the user is no longer evaluating a plan, they are unblocking a stuck agent, and approving under that pressure is not the same as deciding.

**A permission request arriving mid-execution is not a routine step; it is evidence the planning was incomplete.** Stop, say what was missed, and return to alignment. Get agreement on a revised plan, not on the exception.

This holds most strongly when the action is *trivial*. An exception granted mid-flight is granted in the least deliberative context available, and each one makes the next feel more ordinary. **A rule routinely bypassed with permission is worth less than no rule** — it carries the appearance of a control without the function of one.

### Communication style

1. **Concise and scannable.** Optimise for scanning, not reading. Reserve long prose for genuinely complex topics where linearity is needed.
2. **Anything requiring the user's action must be impossible to miss.** Never bury a question, approval request, or warning inside prose — not even as a well-formed sentence, because it will be missed. End such responses with a **TL;DR** block repeating every ask.
3. **Spell out an acronym in full on first use**, then use it alone. Do not correct the user for not doing this.
4. **Accessible language.** Do not assume familiarity with every tool or procedure. When proposing something complex, give the engineering answer *and* a compact plain-language version.

### Session startup

Two local-only checks at the start of every session. Both run without being asked; surface results only when there is something to act on.

- **Current branch.** If it is a working branch, check whether its commits already exist in the integration branch. If so, switch, pull, and offer to delete it. If not, note the state in one line.
- **All other local working branches.** Check each the same way. Present fully-merged ones as a single batch and ask **once** to delete the lot. If none, stay silent.

Checking only the current branch leaves a long tail of orphaned merged branches accumulating across sessions — the buildup that eventually needs a costly bulk audit. Remote branches are out of scope; host-side auto-delete-on-merge handles them.

Report the combined result in one short line, then ask what is next.

### Git workflow

1. **Commit after every round of changes**, to a working branch matching the task. Automatic; never wait to be asked.
2. **Meaningful commit messages** — the log should be traversable months later.
3. **Trigger phrases are literal.** If the agreed phrase for a shared-state action is a specific form of words, then near-synonyms — "ship it", "send it", "go", "lgtm" — are **not** that phrase. Surface what would happen and confirm explicitly.
4. **Clean up merged branches** as part of post-merge work, without being asked.
5. **Documentation and published-surface impact is checked at pull-request time, not remembered later.** Before opening a pull request, check whether the change affects user documentation or any published surface this repository tracks. If it does, present the affected material and a draft of the proposed updates, wait for approval, then include them in the same pull request. If nothing is affected, say so explicitly in one line.

Rule 5 is the one most often dropped and most worth keeping: it converts a documentation obligation from something remembered into something checked.

---

## Published Surfaces

**The repository is the source of truth; every publishing destination is a deployment target.** A dashboard, content system, or store console has no meaningful version history, no review step, and no diff. Content edited there directly is unrecoverable and unreviewable.

> **SETUP — S10.** Inventory the surfaces: marketing site, user documentation, store or marketplace listings. One entry per surface **and per channel** — stable and beta are separate entries, because each is reviewed and published independently.
>
> | Surface | Authored at | Published to | Updated how |
> |---|---|---|---|
> | | | | |
>
> State the level: **MVP** (repository holds the copy, agent drafts updates, human publishes — recommended starting point) or **Ideal** (agent publishes via API — add to Tier 2).
>
> **Flag anything currently authored in a dashboard rather than the repository.** That inverts the source-of-truth rule and is a decision to raise, not to silently accept.

Rules that hold at either level:

1. **A change to shared copy states which destinations it was pushed to.** Copy updated in the repository but applied to only some destinations is the likeliest drift source and is invisible without this.
2. **Reconcile on a schedule** — before each release, or at a fixed interval. Diff each surface against what is live and record the result, **including "no drift."** A reconciliation that is not recorded did not happen.
3. **Where channels deliberately differ, record the intent**, or the next reconciliation will "fix" a deliberate difference.
4. **Some product changes carry a mandatory surface obligation.** Anything altering what a listing must declare — permissions requested, data collected, pricing, supported platforms — makes that text stale immediately. Not a discretionary check; it belongs in the pull-request checklist, because the alternative is discovering it during a platform review.

---

## Release Management

> **SETUP — S6.** Version scheme, who decides a bump, what automation does versus what a human does, and the **trigger phrases** mapping user phrasings to workflows.
>
> Two mechanisms hold regardless of model: **trigger phrases mapped explicitly**, with the instruction to ask rather than guess when a phrase is ambiguous; and **humans decide version bumps** — automation deploys, it does not decide what it is deploying.
>
> **Default:** semantic versioning, human-controlled bumps, changelog entries grouped by version. Confirm rather than assume — distribution platforms often impose monotonic versions or numbers that can never be reused.

---

## Documentation Workflow

> **This section is the operative rules.** The reasoning behind them is in [`system.md`](https://github.com/natannovum/agentic-product-management-system/blob/main/system.md). Read that before *changing* the system; you do not need it to *operate* it. If the two disagree, this section wins.

**Every session writes a `dev-log.md` entry** — the executive summary: assignment, reasoning, what went wrong, what was left undone. In addition, the session updates one or more change logs with the implementation detail. Cross-reference by date. In the pull-request description, report which logs and reference docs were updated.

### Concern × state

| Concern | Finished (append-only) | Pending |
|---|---|---|
| **Product** | `docs/change-log-product.md` | `docs/todo-product.md` |
| **Product marketing** | `docs/change-log-product-marketing.md` | `docs/todo-product-marketing.md` |
| **Documentation** | `docs/change-log-documentation.md` | `docs/todo-documentation.md` |

Plus `docs/dev-log.md` — cross-concern, one entry per session.

> **SETUP — S2/S3.** Remove any concern this project does not have. Then set the pending-work mode below and delete the mode that does not apply.

**Pending-work mode: A — tracker-backed.** *(Delete if Mode B.)*
The tracker owns pending work. The `todo-*.md` files are a **committed index** — ticket ID, title, one-line summary, link — so a fresh checkout still shows what is outstanding. Three rules:
- **Closing a ticket is not a change-log entry.** The ticket records that work happened; the log records what changed, why, and how it was checked. Both.
- **Deferred work becomes a ticket in the same session it is deferred**, with the ID in that session's `dev-log.md` entry and the index updated.
- **Anything load-bearing for understanding the product lives in the repository, not only in the tracker.** Copy durable insight into `documentation-dev/` before a ticket closes.

**Pending-work mode: B — self-contained.** *(Delete if Mode A.)*
The `todo-*.md` files are the queue and carry the full weight: item, rationale, specification, prerequisites. **This is a fallback, not the intended design** — markdown has no state field and no forcing function to close anything, so items accumulate and never leave. **Triage periodically:** close what is done, delete what is dead, move specifications into `documentation-dev/`. If triage keeps slipping, adopt a tracker.

### Ordering

**Newest entry on top, in every log file**, directly beneath the preamble. The reader almost always wants the most recent state, and appending to the bottom forces a scroll that lengthens every session. A mixed convention is worse than either.

### What an entry must carry

1. **The symptom or goal in plain terms** — a leading bolded sentence a non-author can parse.
2. **The mechanism** — what changed, where, and why this fix over the alternatives.
3. **Assumptions that turned out wrong.** Explicitly. The highest-value content and the easiest to omit.
4. **How it was verified** — named checks, named conditions, named control. "Tested" is unfalsifiable and therefore worthless.

**Change logs are append-only.** Corrections are new dated entries that supersede; never edits to past entries. Rewriting history in place destroys the record of what was believed at the time.

### Todo sweep at session end

Part of the end-of-session pass. **Review pending items this session touched** and resolve each one of three ways:

- **Done** — remove it; the change log already records the work
- **Superseded** — remove it, and say why in the change-log entry
- **Relocated** — it belongs to a different concern: **move it, do not delete it**

**The third case is the dangerous one.** "The work is done" and "the item is filed under the wrong concern" produce the same symptom — an item that looks irrelevant where it sits — and need opposite remedies. Confirm an item's *home* is correct before judging its *status*.

Scope it to what the session touched; a full pass over a large file is its own task.

**Why it exists:** in Mode B (§ Documentation Workflow) nothing closes items automatically, so they accumulate silently until the file is too large to read. A per-session sweep keeps that cost at seconds.

### Reference docs — update when relevant

| File | When |
|---|---|
| `documentation-dev/infrastructure-and-repo-model.md` | Environments, pipeline, hosting, or manual steps changed |
| `documentation-dev/codebase-tricky-parts.md` | A new gotcha or edge case was found |
| `documentation-dev/design-intentions-and-rationale.md` | A decision was made or newly codified — **including options rejected** |
| `documentation-dev/latest-implementation-specs.md` | A subsystem's implementation changed |
| `documentation-dev/retired-implementation-specs.md` | A spec section was superseded — **move it here, do not delete it** |
| `documentation-user/` or the user-manual surface | User-facing behaviour changed |

---

## Standard Workflow

1. Think the problem through, read the relevant code, and capture the plan as trackable tasks.
2. Check in before beginning non-trivial work so the plan can be verified.
3. Work the tasks, marking each in progress and completed.
4. Give a high-level explanation of each change as it happens.
5. **Make every change as simple as possible.** Avoid large or complex changes; each should touch as little as possible. Everything is about simplicity.
6. At session end, write the log entries.
7. Record deferred work so it is not lost.

---

## Verification

> **SETUP — S7.** Describe this project's mechanical pass/fail signal and its scope. **If there is none, say so plainly** rather than describing a process that does not exist, and record establishing one as the first documentation todo.

Four principles hold regardless of mechanism:

- **The agent that wrote the change does not grade it.** Verification goes to a fresh context.
- **Judge against a deterministic signal**, not by reasoning about output from scratch.
- **Circuit breakers on the fix loop:** stop and escalate on three failed fix rounds, on oscillation (a fix re-breaks a passing check), when the fix needs a product judgment call, or when the specification is ambiguous. Otherwise keep fixing silently — do not ask permission to continue.
- **Test intensity tiers by change type:** documentation and version bumps get no regression run; a localised change gets a targeted subset; a change to a foundational file gets the full matrix.

---

## Critical "Don't Forget" Reminders

Short, non-obvious reminders that prevent recurring mistakes. **Anything longer than a line or two belongs in `docs/documentation-dev/codebase-tricky-parts.md` — link from here, do not inline.** Without that rule this section is where a `CLAUDE.md` goes to become unmaintainable.

- **🔴 Never retrieve a secret value you do not need — fetch keys, not values.** Output-side redaction is a bug, not a control.

> **SETUP.** Add this project's reminders as they are discovered. Start empty; do not invent them.

---

## Index

Detailed specifications live in dedicated documents. Read the relevant file when touching that area.

| Topic | File |
|---|---|
| Why the documentation system is shaped this way | [`system.md`](https://github.com/natannovum/agentic-product-management-system/blob/main/system.md) |
| Standing the system up in a project | [`setup-protocol.md`](https://github.com/natannovum/agentic-product-management-system/blob/main/setup-protocol.md) |
| Infrastructure, environments, pipeline | `docs/documentation-dev/infrastructure-and-repo-model.md` |
| Codebase gotchas and edge cases | `docs/documentation-dev/codebase-tricky-parts.md` |
| Design rationale, and options rejected | `docs/documentation-dev/design-intentions-and-rationale.md` |
| Current implementation specifications | `docs/documentation-dev/latest-implementation-specs.md` |
| Retired specifications, kept for reference | `docs/documentation-dev/retired-implementation-specs.md` |
| Published surface copy (source of truth) | `docs/product-marketing/` |
| End-user documentation | `docs/documentation-user/` |
| Session log (cross-concern narrative) | `docs/dev-log.md` |
| Change logs | `docs/change-log-*.md` |
| Pending work | `docs/todo-*.md` |
| User-provided input materials | `docs/input-materials/` |

**Maintenance:** this table is what keeps `CLAUDE.md` from growing without bound — detail lives in the reference docs, this file holds the routing. **When renaming a heading that is referenced elsewhere, search the repository for the old name and repair every hit in the same commit.** Nothing verifies these pointers automatically; a stale section name reads as entirely plausible and is exposed only when someone follows it.
