# Agentic Product Management System

A documentation and work-tracking system for software projects built with AI coding agents.

**What this file is:** the canonical description of what the system is, how it is structured, and why. It is descriptive. The rules an agent follows day to day live in the project's `CLAUDE.md`, generated from `templates/CLAUDE.md`; the procedure for standing the system up lives in `setup-protocol.md`.

---

## 1. The problem

A project built with AI agents loses knowledge in a specific way.

The code records **what** the current state is. Version control records **when** each change happened. Neither records:

- **why** a change was made rather than an alternative
- **what was tried and rejected**, and on what grounds
- **which assumption turned out to be wrong** during the work
- **how the change was verified**, and against what

That knowledge exists only in the session that produced it — a transcript that is discarded, held by an agent whose context resets completely between sessions. Without an external record it is gone when the session ends.

The questions that recur months later are: *Why is it done this way? Did we already try the obvious fix? Is this behaviour deliberate or accidental? Can I trust that this was tested?* A diff answers none of them.

**The constraint that shapes every decision below:** the writer and the reader are usually the same agent with no shared memory. The system optimises for *reconstruction by a stranger*, not for note-taking by someone who was present.

A second problem compounds the first. A product is not only code. It has a marketing site, user documentation, sometimes store listings — surfaces that must stay truthful as the product changes, and that drift the moment nobody is watching. Agents are well suited to keeping these current, but only if the project records what changed in a form the agent can act on. The system treats these published surfaces as a first-class concern rather than an afterthought (§5).

### What the system covers

Three pillars, corresponding to three ways knowledge is lost in agent-built products.

**1. Capture — turn ephemeral work into durable reference.** Development prompts, process learnings, and the findings and agreements that emerge implicitly while working with an agent all become internal documentation a future session can load.

An agent-instruction file (`CLAUDE.md` or equivalent) already solves one part of this: how the human and the agent work together — permissions, communication, workflow. That is the first thing any project codifies, and it needs no system to prompt it.

**This system covers the part that file does not.** Everything learned *about the product* while building it, which has no natural home and is therefore lost by default:

- why a design is the way it is, and which alternatives were rejected
- how each subsystem is currently implemented
- how it *used to be* implemented, and why that was superseded
- the gotchas — where the code misleads a reader who is reasoning correctly
- the reasoning behind decisions that would otherwise be relitigated every few months

The rules file says how to work. These say what was learned by working.

**2. Protect — verify changes and prevent regressions.** A record of how something was verified is worth little unless verification actually happens, and happens against a signal that cannot be talked out of a verdict. The system requires every entry to state its verification and defines the principles that verification obeys (§9 of `setup-protocol.md` covers eliciting the mechanism; the principles are in `templates/CLAUDE.md`).

**3. Propagate — keep external content current with each release.** Marketing copy, user documentation, and store listings must track the product. This is the pillar most often left entirely undone, because it is nobody's job and nothing fails when it is skipped.

### Why "product management" rather than "documentation"

Documentation is one output of this system, not its purpose. The system spans the product lifecycle: capturing what was learned during development, protecting the product against regression, and propagating changes outward to everything a user reads.

Product marketing sits inside that scope as a lifecycle step, even though in most organisations marketing and product management are separate functions in separate departments. The separation is organisational, not logical — the release that changes a behaviour and the sentence on a website describing that behaviour are the same fact, recorded twice, and treating them as separate concerns is precisely how the second one goes stale.

---

## 2. Structure

```
<repo>/
├── CLAUDE.md                          ← operative rules; the agent's entry point
│
├── .claude/
│   ├── agents/                        ← subagent definitions (verification, review)
│   ├── hooks/                         ← enforcement that cannot be forgotten
│   │   └── block-secret-reads.sh
│   └── settings.json                  ← permissions: allow / ask / deny
│
├── .githooks/
│   └── pre-commit                     ← secret scanning before history is written
│
└── docs/
    │
    │   ── FINISHED WORK — append-only history, newest entry on top ──
    ├── dev-log.md                     ← one entry per session; the narrative spine
    ├── change-log-product.md          ← the product itself
    ├── change-log-product-marketing.md← published surfaces (§5)
    ├── change-log-tooling-and-infrastructure.md ← optional 4th concern (§3)
    ├── change-log-documentation.md    ← this system and the docs themselves
    │
    │   ── PENDING WORK — index into a tracker, or the queue itself (§4) ──
    ├── todo-product.md
    ├── todo-product-marketing.md
    ├── todo-tooling-and-infrastructure.md
    ├── todo-documentation.md
    │
    │   ── CURRENT-STATE TRUTH — rewritten in place, not history ──
    ├── documentation-dev/
    │   ├── infrastructure-and-repo-model.md   ← how the whole setup fits together
    │   ├── codebase-tricky-parts.md           ← where the code misleads a reader
    │   ├── design-intentions-and-rationale.md ← decisions, and options rejected
    │   ├── latest-implementation-specs.md     ← how each subsystem works today
    │   └── retired-implementation-specs.md    ← superseded specs, kept (§6)
    │
    │   ── PUBLISHED SURFACES — source of truth for what users see (§5) ──
    ├── product-marketing/
    │   ├── website/                   ← marketing site copy
    │   ├── user-manual/               ← end-user documentation
    │   └── store-listings/            ← one directory per store × channel
    │       ├── _shared/               ← copy used by more than one listing
    │       └── <store>-<channel>/
    │
    └── input-materials/               ← briefs, references, source files from the user
```

Not every project needs every branch of this tree. §3 explains which parts are structural and which are optional; `setup-protocol.md` decides them per project.

---

## 3. The model: concern × state

Records split on two axes — **concern** (which area of work) and **state** (finished or pending).

| Concern | Finished — append-only history | Pending — work queue |
|---|---|---|
| **Product** — the thing being built | `change-log-product.md` | `todo-product.md` |
| **Product marketing** — website, user manual, store listings | `change-log-product-marketing.md` | `todo-product-marketing.md` |
| **Tooling & infrastructure** *(optional 4th)* — test harness, servers, credentials, provisioning | `change-log-tooling-and-infrastructure.md` | `todo-tooling-and-infrastructure.md` |
| **Documentation** — this system and the docs themselves | `change-log-documentation.md` | `todo-documentation.md` |

Above the grid sits **`dev-log.md`** — one entry per working session, carrying the narrative. Beside it sits **`documentation-dev/`**, which is not part of the grid and obeys different rules (§6).

### Why finished and pending are separate

They have different lifetimes and different readers. A finished entry is **immutable history** — appended, never edited. A pending item is **mutable intent** — re-scoped, re-prioritised, abandoned. Combined, they produce a document where you cannot distinguish what is true from what was once merely planned, and that ambiguity cannot be recovered later.

This is the highest-value structural decision in the system.

### Why the concerns are separate

Audience and cadence differ. Someone debugging the product does not want to page through a documentation restructure. Someone auditing published copy does not want to read implementation entries. A single undifferentiated log becomes unnavigable somewhere around 200KB, and the failure is gradual enough that nobody notices until it is expensive.

Three concerns is the common shape, not a law. A project with no published surface has two. A project whose user manual is large and independently maintained may split it out as a fourth. `setup-protocol.md` → **S2** decides this.

**The count is a decision that has to be revisited, and there is a symptom that tells you when.** Watch for entries filed somewhere they do not belong because there is nowhere better. In the reference project this ran for months: deployments of the product across a fleet of servers were recorded in the *marketing-content* change log, and the tooling that supports the product — test harness, provisioning scripts, credential handling, server rebuilds — was scattered across all three todo files as level-one initiatives, because none of the three concerns was its home.

**Tooling and infrastructure is the most common fourth concern**, and it is under-recognised precisely because it is not the product and has no user. If a project has a test harness that is itself maintained, servers that are rebuilt, or credentials that rotate, that work needs a concern or it will silently colonise another one.

**The test:** for each concern, ask *what is the audience of this log, and what question do they arrive with?* Two concerns whose readers arrive with the same question should merge; one whose readers arrive with two different questions should split. A deployment log and a marketing-copy log answer nothing in common.

### 🔴 A change log is a RELEASE log; the dev log is a WORK log

The distinction is easy to lose and it decides where every entry goes.

**`dev-log.md` is chronological by work.** One entry per session, whatever was touched, in the order it happened. It is written unconditionally, including for work that ships nothing.

**A change log is chronological by RELEASE.** One entry per release, listing what that release contained — regardless of when the parts were built. Several weeks of work across many merges may land in one entry; a single session's work may not appear until the release that carries it.

Collapsing the two produces a change log that is really a second, worse dev log: an entry per working day, ordered by when someone typed rather than by what shipped, which cannot answer the one question a change log exists for — *what is in the version I am running?*

**Internal change logs may record release candidates and staging deploys**; a customer-facing one may not. A staging deploy is not something a user can observe, and the customer-facing log's job is to describe what they have. The internal ones exist for evidence — what was running where, and when — and that is exactly what an RC entry provides.

### Why `dev-log.md` sits above the grid

The change logs answer *"what changed in this area?"* Nobody asks that first. The first question is always *"what happened, and where are we?"* — a question about a **session**, not an area.

`dev-log.md` is the spine that makes everything else navigable: it carries the story, the change logs carry the detail, and they are joined by date rather than by link, so neither has to know about edits to the other.

The corollary: **a session entry that could be reconstructed from the commit log has failed.** Its entire value is the part not recoverable from the repository.

### Ordering: newest first, everywhere

Every log file is written newest-entry-on-top, directly beneath the file's preamble. The reader almost always wants the most recent state, and a file that appends to the bottom forces a scroll to the end that gets longer every session. This applies to `dev-log.md` and all change logs uniformly — a mixed convention is worse than either, because the reader has to remember which file does which.

---

## 4. Pending work: tracker-backed or self-contained

The pending axis operates in one of two modes, decided at setup (`setup-protocol.md` → **S3**).

### Mode A — tracker-backed (preferred)

The project has an issue tracker: Linear, Jira, GitHub Issues, Notion, Trello, or similar. The tracker owns pending work — status, priority, assignment, ordering.

The `todo-*.md` files remain, and become a **committed index** into the tracker:

```markdown
| Ticket | Title | Summary | Link |
|---|---|---|---|
| ENG-142 | Session resume drops queued events | Events queued while the worker is idle are lost on wake. | https://… |
```

**Why keep an index at all when the tracker is authoritative?** Because a tracker is a separate system with separate access and separate retention, and a repository whose pending work is invisible from a checkout is not self-describing. The index costs one line per ticket and means a fresh clone still shows what is outstanding.

Three rules follow, and they are the ones that make Mode A work:

1. **Closing a ticket is not a change-log entry.** The ticket records that work happened; the change log records what changed, why, and how it was checked. Both, always.
2. **Deferred work becomes a ticket in the same session it is deferred**, with the ID recorded in that session's `dev-log.md` entry and the index updated.
3. **Anything load-bearing for understanding the product lives in the repository, not only in the tracker.** When a ticket discussion produces durable insight, copy it into the relevant `documentation-dev/` file before the ticket closes.

### Mode B — self-contained (fallback)

No tracker. The `todo-*.md` files *are* the queue, and carry the full weight: item, rationale, specification, prerequisites, and any material needed to pick the work up cold.

**Mode B is a fallback, not the intended design, and the files should say so.** A markdown file has no state field, no archive, and no forcing function to close anything, so items accumulate and never leave. Left alone long enough a todo file becomes an undifferentiated mixture of stale inventories, long-range roadmaps and live items with nothing distinguishing them — at which point it is worse than no record, because it looks authoritative.

Mode B therefore carries one extra obligation: **a periodic triage** — walk the file, close what is done, delete what is dead, and move anything that is really a specification into `documentation-dev/`. If triage keeps slipping, that is the signal to adopt a tracker.

**Migrating B → A** is not automatic. Connecting a tracker does not clean up an accumulated todo file; that is a separate exercise of triaging what is live, creating tickets for it, and archiving the rest.

---

## 5. Published surfaces

Every product has surfaces the user sees that are not the product itself:

- a **marketing website**
- **user documentation** — often part of the marketing site, sometimes separate
- **store or marketplace listings** — for anything distributed through a platform: mobile apps, browser extensions, plugin marketplaces, package registries. A project may have several, because listings multiply by *store* × *channel* (stable, beta) and each is edited and reviewed independently.
- **demo environments** — see below; sometimes a surface, sometimes not.

### Demo environments are a separate axis

A **demo** is a running instance of the product that exists to be looked at. A **marketing site** is copy that describes the product. They are frequently confused because for some products they are the same artifact, and the confusion produces a bucket that is either too narrow or misnamed.

Three arrangements, all common:

| Arrangement | What the demo is | Treat it as |
|---|---|---|
| **Demo is the marketing site** | The product renders the site that sells it — common where the product is itself a site-building or presentation tool | One surface. Content changes are marketing changes |
| **Demo is a subdomain app** | A live instance at `demo.<product>` or similar, separate from a conventionally-authored marketing site | Two surfaces. The demo's *content* is usually fixture data, not marketing copy |
| **Demo is a fleet** | Several instances, each showcasing a different use case, audience, or configuration | One surface *per instance*, plus whatever they share |

**Why the distinction earns a place here.** A demo that is also the marketing site makes content edits a marketing concern with real external stakes. A demo that is a separate app makes its content fixtures — changing them is a product task, not a marketing one, and filing it as marketing means genuine marketing drift gets lost among fixture churn. Naming the bucket after the local instance (*"demo content"*) rather than the category (*"product marketing"*) works until a second surface appears, at which point the name misleads.

**A demo fleet multiplies the same way store listings do.** Each instance is independently configured, independently drifts, and needs its own entry — the general rule being that a surface multiplies by every dimension it varies along, whether that dimension is store, channel, or use case.

**Decide at setup** (`setup-protocol.md` → **S10**) which arrangement applies, and name the bucket for the category rather than the instance.

### Drift

These surfaces drift by default. They are written once at launch, and every subsequent product change makes them a little less true. Nobody notices, because nothing fails.

**This is the class of work agents are best suited to and least often given.** Keeping a dozen surfaces consistent with a changing product is exactly the kind of thorough, low-creativity, high-tedium task that humans defer indefinitely.

### The governing rule

**The repository is the source of truth; every publishing destination is a deployment target.**

A dashboard, CMS, or store console has no meaningful version history, no review step, and no diff. Content edited there directly is unrecoverable and unreviewable. Content edited in the repository gets both, and pushing it out becomes a deployment rather than an act of authorship.

The inverse arrangement — where the live system is authoritative and the repository holds a lagging mirror — is a known trap. It makes the repository a copy of unclear fidelity, inverts the trust rule that holds everywhere else in the project, and the sync is usually lossy in ways that only surface when you try to restore from it.

### Two levels of implementation

**MVP.** The agent maintains the repository copy of every surface and produces the updated text; a human pastes it into the destination. All the value of knowing *what* needs changing, none of the integration work. This is where every project should start, and it is sufficient indefinitely.

**Ideal.** The agent publishes directly through whatever API the destination exposes — CMS, store API, static-site deploy. The mechanics differ entirely by stack; the principles do not.

Note that direct publishing is almost always **Tier 2** (see `templates/CLAUDE.md`): it reaches users, and it is hard to retract.

### Rules that hold at both levels

1. **A change to shared copy states which destinations it was pushed to.** Copy updated in the repository but applied to only some destinations is the most likely drift source and is invisible without this.
2. **Reconcile on a schedule** — before each release, or at a fixed interval. Diff each surface against what is actually live and record the result, **including "no drift."** A reconciliation that is not recorded did not happen.
3. **Where channels deliberately differ, record the intent.** A beta listing describing an unreleased feature is correct, not drifted. Say so in the file, or the next reconciliation will "fix" it.
4. **Some product changes carry a mandatory surface obligation.** Anything altering what a listing must legally or contractually declare — permissions requested, data collected, pricing, supported platforms — makes the corresponding text stale immediately. These cases are not discretionary checks; they belong in the pull-request checklist, because the alternative is discovering the mismatch during a platform review.

---

## 6. Change logs versus reference docs

The most frequently confused distinction in the system.

|  | Change logs | `documentation-dev/`, published surfaces |
|---|---|---|
| **Nature** | Append-only history | Current-state truth |
| **Phrasing** | "On this date, X became Y because Z" | "It is Y" |
| **Editing** | Never rewritten; corrections are new dated entries | Rewritten in place as reality changes |
| **Growth** | Monotonic, forever | Roughly flat |

A fact usually belongs in both, phrased differently. The change log records the transition; the reference doc records the resulting state, and the reason where it is load-bearing.

**Why history is never rewritten in place:** editing a past entry to match present reality destroys the record of what was believed at the time, which is frequently what someone is looking for. A wrong belief that was acted on is a fact about the project's history, not an error to be tidied away.

### The reference buckets

- **`infrastructure-and-repo-model.md`** — how the whole setup fits together: environments and what each is for, which branch reaches which environment, hosting and services, build and release pipeline, where credentials live *by name*, what is manual versus automated. This is the file that lets someone rebuild or reason about the system without reverse-engineering CI.
- **`codebase-tricky-parts.md`** — cases where a competent engineer reading the code would reach the wrong conclusion. Entries lead with the **symptom**, because that is what a future reader searches for.
- **`design-intentions-and-rationale.md`** — decisions cheap to make once and expensive to relitigate, **including options rejected and why**. Without the rejections, every future session re-proposes them.
- **`latest-implementation-specs.md`** — how each subsystem currently works. Read before touching an area.
- **`retired-implementation-specs.md`** — superseded sections of the above, **moved rather than deleted**.

### The retirement ceremony

When a section of the current specs is superseded, it moves to the retired file with a note naming what replaced it and when. The friction is deliberate.

Superseded designs get consulted more often than expected: when a new approach hits a wall the old one had solved, when a regression traces to behaviour that was intentional under the previous design, or when the reason an approach was abandoned answers a current question. Deleting the section makes it unrecoverable in practice — it survives in version history, but nobody performs archaeology on a deleted doc section; the cost of finding it exceeds the cost of re-deriving it badly.

---

### 🔴 A check must assert the OUTCOME, not the property that changed

The most expensive verification failure is not a missing test. It is a passing one that measures the wrong thing.

A check is usually written after the code, by whoever just changed a property — so it asserts that property. The property is correct. The requirement is not met. In the reference project a layout rule shipped half-working through **1537 assertions, a working negative control and a regression pass over 26 real pages**, because every one of them asked whether a margin was zero rather than whether there was a gap. The margin *was* zero. A second element's padding held the gap open.

Three rules follow, and they are cheap:

1. **Write the requirement as a sentence a user would say, before writing the check.** Assert that sentence. "There is no gap between the header and the first block" is checkable; "margin-top is 0" is a restatement of the diff.
2. **Every check needs a control whose expected answer is different.** A check that can only return one answer is not a check. Make it structural — a harness that *refuses* a check without a control is worth more than a convention that asks for one.
3. **Prefer measuring relationships over reading properties.** Give the harness primitives that expose distances between things and withhold the properties that produce them, so the honest check is also the cheaper one to write.

The same failure has a fixture-shaped twin: a control case that is not in the control state. Verify the fixture is in the state you are claiming *before* measuring — including when that state is "the default", which is exactly the assumption nobody checks.

## 7. What a durable entry contains

Entries that stay useful carry four things:

1. **The symptom or goal in plain terms**, as a leading bolded sentence a non-author can parse. *"Refactored the event handler"* ages into meaninglessness; *"Events queued while the worker was idle were silently dropped on wake"* does not.
2. **The mechanism** — what changed, where, and why this fix rather than the alternatives.
3. **Assumptions that turned out wrong.** The highest-value content in the system and the easiest to omit, because recording it feels like logging an error. It is not: it is what stops a later session re-deriving a dead end.
4. **How it was verified** — named checks, named conditions, named control. *"Tested"* is unfalsifiable and therefore worthless; a future reader cannot distinguish a thorough pass from a cursory one, so treats both as untrusted.

Each log and todo file opens with a **preamble** stating its scope and where its counterpart lives. These are load-bearing: they are what keeps the files from drifting into each other.

---

## 8. The session contract

Every working session ends by writing:

1. **One `dev-log.md` entry** — always, including sessions that produced no code. An investigation concluding *"this is not the problem"* is exactly the finding that otherwise gets repeated.
2. **One or more change-log entries**, dated the same day, in whichever concerns were touched.
3. **Reference-doc updates** where current-state truth changed.
4. **Pending work recorded** — a ticket in Mode A, a todo item in Mode B — so nothing deferred is lost.
5. **A todo sweep** — for pending items the session touched, remove what is done or superseded, and **move what turns out to belong to another concern.** "Done" and "filed in the wrong place" look identical from inside the wrong file and need opposite remedies, so confirm an item's home before judging its status.
6. **A note in the pull-request description** of which logs and docs were updated.

---

## 9. Known weaknesses of this design

Recorded because a system document that lists only strengths teaches the reader to distrust it.

**Cross-references are fragile.** Documents point at each other by writing a section's name as plain text. Nothing verifies the target exists, so renaming a heading silently breaks every pointer to it — no error, no warning, discovered only when a human follows one. This is the one weakness with a real engineering fix: a reference checker that extracts each pointer and confirms the heading exists, run from the pre-commit hook.

**Nothing enforces entry quality.** The four properties in §7 are conventions. An entry reading *"fixed the bug"* satisfies the contract and destroys the value. This is held by discipline, not tooling — *"did you record the assumption that turned out wrong?"* is a judgment call. The mitigation is that the requirement is explicit, so a thin entry is visibly a shortcut rather than an oversight.

**Mode B degrades without triage.** See §4. The failure is structural, not a matter of care.

**The system has no forcing function of its own.** Nothing fails if a session skips its entries. The cost is invisible at the time and paid months later by whoever needs the record. Hooks and checklists reduce this; they do not eliminate it.

---

## 10. Installing this system

`setup-protocol.md` — the procedure the agent follows to stand the system up in a project, covering what to read from the repository, what to ask the user, and in what order.

`templates/` — the files to copy in, each carrying illustrative sample content marked for deletion at first real use.
