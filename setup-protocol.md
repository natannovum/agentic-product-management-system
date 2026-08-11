# Setup Protocol

The procedure for standing up the Agentic Product Management System in a project. Written for the agent doing the installation.

Read `system.md` first — it explains what is being built and why. This file is the how.

---

## The three-step rule

Every project-specific decision below follows the same order. **Do not skip ahead — each step is cheaper and more accurate than the one after it.**

1. **Harvest.** Does the project already have a `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`, or similar? Rules already written down are rules already agreed. Transfer them; do not re-derive or silently reword them.
2. **Inspect.** Can the repository or its infrastructure answer this? Continuous-integration configuration, manifests, lockfiles, deployment workflows and hook scripts are authoritative and free to read. An agent that asks the user something the repository states plainly is spending the user's attention badly.
3. **Interview.** Ask only what remains. Batch the questions; do not drip them one per turn.

**Where a step yields a partial answer, keep it and narrow the next step to the gap.** Where a default exists, adopt it *explicitly and visibly* — write it into `CLAUDE.md` and say you defaulted, so it can be corrected. A silent default is indistinguishable from a decision.

---

## Order of installation

1. **S1 — secret inventory**, and the enforcement hooks. First, always. It is the only item where being late is unrecoverable.
2. **S4 — irreversibility inventory.** The item most likely to need a real conversation, and the one with no safe default.
3. **S2, S3** — concern buckets and pending-work mode. These determine which files get created.
4. Copy `templates/` into place, minus any bucket the project does not have.
5. **S5–S12**, as their answers arrive. Write each into `CLAUDE.md` as it is settled.
6. **Backfill nothing.** Start the logs from today; note in the first session entry that prior history exists only in version control.

---

## S1 — Secret inventory

**Needed:** what counts as radioactive in this project, so the handling rules and the enforcement hook have concrete targets.

**Harvest:** existing rules about credential handling in any agent-instruction file.

**Inspect:** `.gitignore` entries for credential files; CI secret **names** (names only); environment-variable references in configuration; credential-shaped paths.

**Ask:** *"What credentials exist for this project, and where do they live?"* — **names and locations only. Never ask for a value, and refuse one if offered.**

**Good answer:** credential names and locations, plus which are shared across environments and which are per-environment. A shared credential has a wider blast radius and should be flagged as such.

**Default:** treat as radioactive any `.env*` file, any private key, any token or API key, any CI secret, any registry or publisher credential, and any command that dumps an environment wholesale. Extend from there — never narrow.

**Also install:** the pre-commit secret scanner and the pre-tool-use guard. These are stack-independent. Run the guard's test suite after adapting its patterns.

---

## S2 — Concern buckets

**Needed:** how many concerns the logs split into.

**Harvest:** any existing changelog and what it mixes together.

**Inspect:** does the project have published surfaces — a marketing site, user documentation, store or marketplace listings? Look for a `www/` or `site/` directory, a docs site generator, store metadata files, or deployment workflows targeting something other than the application.

**Ask:** *"Besides the code and the internal documentation, what does a user see that we'd want a change history for?"*

**Good answer:** a concrete list of surfaces, each with where it currently lives and who edits it.

**Default:** three — product, product marketing, documentation. Drop product marketing if there is genuinely no published surface. **Do not create a bucket speculatively:** an empty change log reads as *"nothing has changed here"* rather than *"this is not tracked"*, which is worse than its absence.

---

## S3 — Pending-work mode

**Needed:** whether pending work is tracker-backed (Mode A) or self-contained (Mode B). See `system.md` §4.

**Harvest:** existing conventions for referencing work items.

**Inspect:** an issue tracker linked from the README, referenced in commit messages or pull-request templates, or present as a repository integration. Check commit history for ticket-ID patterns.

**Ask:** *"Do you use an issue tracker — Linear, Jira, GitHub Issues, Notion, Trello, anything? If so, is that where you want pending work to live?"*

**Good answer:** one named system, the ticket-ID format, and the URL pattern, so the index files can link correctly.

**Default:** Mode A if a tracker exists, Mode B otherwise. Mode B is a fallback and its files must say so — see the templates.

**If Mode A:** the `todo-*.md` files still get created, as committed indexes into the tracker. Write the three Mode A rules from `system.md` §4 into `CLAUDE.md`; they are what stop the tracker becoming a place rationale goes to disappear.

---

## S4 — Irreversibility inventory (Tier 2)

**The most consequential item here. Do not guess at it.**

**Needed:** the concrete list of actions in this project that cannot be undone with version control or an existing backup.

**Harvest:** any existing rules about what requires approval.

**Inspect:** deployment and release workflows; publishing or submission steps; anything touching credentials, signing keys, or a registry account; database migrations; anything writing to a live datastore or third-party system.

**Ask:** *"Which actions here can't be undone — where a mistake reaches users, or destroys something no backup covers?"* Then per candidate: *"If this went wrong at 2am, what's the recovery path?"* An action with no clean answer is Tier 2.

**Good answer:** each item paired with **why** it is irreversible. The reason is what lets the boundary be applied to situations nobody listed.

**Almost always Tier 2:** deploying or publishing anything users see; direct edits to a live content system; anything touching signing keys or a publisher account; force-push, hard reset, remote branch deletion; changes to CI, build, or release automation; anything touching user data or telemetry; merges to a branch that triggers a deployment.

**Default: none.** This item has no safe default. An incomplete list is worse than an absent one because it reads as authoritative and the gap is invisible. If the user cannot answer fully, record what is known, **mark the list explicitly incomplete**, and treat anything unlisted-but-suspicious as Tier 2 until classified.

---

## S5 — Branch strategy

Two separable questions. Do not conflate them.

### Naming convention — safe preset

`feature/*`, `fix/*`, `docs/*`, `chore/*` for working branches. Widely used, mirrors the Conventional Commits vocabulary, and costs nothing if the project has no strong opinion. **Adopt unless the repository shows a different convention already in use** — in which case keep theirs; consistency with existing history beats consistency with this document.

### Topology — elicited

Whether the project runs a single integration branch, or two tiers (an integration branch plus a release branch), is **not** a universal practice and must not be presumed.

**Inspect first, and this is usually decisive:** CI trigger conditions state authoritatively which branches cause deployments. Read `on: push: branches:` and equivalent. A branch that triggers a deploy is an environment boundary; a branch that triggers nothing is a convenience.

**Ask only what CI cannot show:** *"Is there an environment that should always be working — something you or others test against? And should that be fed from its own branch?"*

**Good answer:** a table of branch → purpose → what happens on merge. The third column is what connects this to S4.

**Default:** single integration branch plus working branches, releases cut by tag. **Do not import a "the integration branch must stay release-quality" rule unless something concrete depends on it** — a shared test environment, a staging deploy, other people's work. Without such a dependency the constraint is ceremony and will be resented and then ignored.

---

## S6 — Release model and trigger phrases

**Needed:** how versions are decided and cut, and which user phrasings activate which workflow.

**Harvest:** existing release documentation.

**Inspect:** version field and its history; tags; release automation; existing changelog heading conventions.

**Ask:** *"Walk me through what happens between 'this is done' and 'users have it.'"*

**Good answer:** the version scheme, who decides a bump, what automation does versus what a human does, and the phrases that mean *"cut a release"* as distinct from *"this increment is finished."*

**Two mechanisms port regardless of the model, and should be written into `CLAUDE.md`:**

- **Trigger phrases, mapped explicitly**, with the instruction to ask rather than guess when a phrase is ambiguous. This is what stops *"done with this"* being read as *"ship it."* Treat the agreed phrases as **literal** — near-synonyms are not the phrase.
- **Humans decide version bumps.** Automation deploys; it does not decide what it is deploying.

**Default:** semantic versioning, human-controlled bumps, changelog entries grouped by version. Confirm rather than assume — distribution platforms often impose constraints such as monotonic versions or numbers that can never be reused.

---

## S7 — Verification

**Needed:** how a change is judged correct, and what the deterministic signal is.

**Inspect:** test suites, CI check definitions, linters, type checking, snapshot or visual-regression harnesses, coverage configuration.

**Ask:** *"When you say something works, what convinced you?"* — this surfaces the real signal, which is often not the documented one.

**Good answer:** a named, mechanical pass/fail, plus honest scope on what it does not cover.

**Four principles port regardless of mechanism:**

- **The agent that wrote the change does not grade it.** Verification goes to a fresh context.
- **Judge against a deterministic signal**, not by reasoning about output from scratch.
- **Circuit breakers on the fix loop:** stop and escalate on three failed fix rounds, on oscillation (a fix re-breaks a passing check), when the fix needs a product judgment call, or when the specification is genuinely ambiguous. Otherwise keep fixing silently — do not ask permission to continue.
- **Test intensity tiers by change type:** documentation and version bumps get no regression run; a localised change gets a targeted subset; a change to a foundational file gets the full matrix.

**Default:** if there is no mechanical signal, **say so plainly** rather than describing a process that does not exist, and record establishing one as the first documentation todo.

---

## S8 — Development environment

**Needed:** how to run, build, watch and test locally, and what breaks if the environment is wrong.

**Inspect:** package manifest scripts, container definitions, runtime version files, README setup, and **CI setup steps — which are a working environment specification by construction.**

**Ask:** *"What isn't written down that a new machine would get wrong?"*

**Good answer:** exact commands, one block per process that must run concurrently, and **the failure signature of a wrong runtime version.** The costly cases are the ones that fail confusingly rather than cleanly — a process that crashes while its supervisor reports success costs an hour undocumented and ten seconds documented.

**Default:** transcribe the CI setup steps, and mark them unverified locally until confirmed.

---

## S9 — Infrastructure and repo model

**Needed:** how the whole setup fits together — the thing a new contributor or a fresh agent session cannot reconstruct from any single file.

**Inspect:** deployment workflows, hosting configuration, container and orchestration files, DNS or domain configuration if present, environment definitions, and the relationship between repositories if there is more than one.

**Ask:** *"If you had to rebuild this from scratch, what would you need to know that isn't in the repo?"*

**Good answer, written into `documentation-dev/infrastructure-and-repo-model.md`:**

- **Environments** — what exists, what each is for, who can reach it
- **Branch-to-environment mapping** — which branch reaches which environment, and by what mechanism
- **Hosting and services** — where things run, what third-party services are depended on
- **Build and release pipeline** — what is automated, what is manual, and what the manual steps are
- **Credentials by name and location** — never values (S1)
- **Known manual steps and their failure modes** — anything that relies on someone remembering

**Why this bucket earns its place:** infrastructure knowledge is the most expensive kind to lose and the least likely to be written down, because whoever set it up did not need notes. It is also the area where an agent, lacking that memory, can do the most damage by guessing.

**Default:** create the file and populate what CI and configuration reveal, marking clearly what is inferred versus confirmed.

---

## S10 — Published surfaces

**Needed:** the inventory of what users see, and where each is authored today. See `system.md` §5.

**Inspect:** marketing site source, documentation site generators, store or marketplace metadata files, any content directories not part of the application.

**Ask:** *"What does the outside world read about this product, and where does each of those live?"* Follow with: *"Who edits each, and is any of it edited directly in a dashboard?"*

**Good answer:** one entry per surface — what it is, where authored, where published, who owns it, and how it is updated today. For anything with multiple channels (stable and beta, or several stores), each channel is its own entry, because each is reviewed and published independently.

**Resolve the demo question explicitly** (`system.md` §5). Ask: *"Is there a demo, and is it the same thing as the marketing site or separate?"*

- **Demo is the marketing site** — one surface; content edits are marketing changes with external stakes.
- **Demo is a separate app** — two surfaces; the demo's content is usually fixture data, and filing it as marketing buries real marketing drift among fixture churn.
- **Demo is a fleet** of instances showcasing different use cases — one entry per instance, plus whatever they share. A fleet multiplies exactly as store listings do.

**Name the bucket for the category, not the local instance.** *"Demo content"* reads correctly until a second surface appears, and then misleads permanently.

**Then decide the level** (`system.md` §5):

- **MVP** — the repository holds the copy, the agent produces updates, a human publishes. **Recommend this as the starting point.** It captures the value immediately and needs no integration work.
- **Ideal** — the agent publishes directly through an API. Note that this is almost certainly Tier 2 (S4) and should be added there.

**Flag explicitly if any surface is currently authored in a dashboard rather than the repository.** That inverts the source-of-truth rule and should be raised as a decision, not silently accepted.

**Default:** create the bucket for surfaces that exist; MVP level; note anything dashboard-authored as an open item.

---

## S11 — Architecture orientation

**Needed:** enough structure for an agent to know where to look, and no more.

**Inspect:** directory layout, entry points, build configuration, module boundaries, dependency manifests.

**Ask:** *"What surprises people about how this is put together?"*

**Good answer:** a short orientation naming the major subsystems and how they communicate. **Keep it deliberately short** — detail belongs in `documentation-dev/` reached through the index, because an architecture section in `CLAUDE.md` grows without bound and goes stale invisibly.

**Default:** describe only what is verifiable from the repository, and mark anything inferred as inferred.

---

## S12 — Reference doc buckets

**The universal five** carry unchanged: infrastructure and repo model, tricky parts, design rationale, current specs, retired specs.

**Inspect:** recurring themes in issue history and pull-request discussion. Repeated explanations of the same thing indicate a missing bucket.

**Ask:** *"What do you find yourself re-explaining?"*

**Good answer:** additional buckets matching the project's real failure modes — platform compatibility notes, third-party review or compliance history, a registry of deployed instances, performance characteristics.

**Default:** the five, created as stubs with section skeletons rather than empty files.

---

## Finishing

Before declaring setup complete:

- Every section of `CLAUDE.md` is either filled or explicitly marked as deliberately empty. **No placeholder comments left in a live file** — a `<!-- FILL -->` that survives into normal use is a gap that has stopped being visible.
- Every default adopted is stated as a default in the text, not presented as a decision.
- The secret-handling hook's test suite passes.
- The sample content in each template has been deleted at first real use, or is still clearly marked as sample.
- The first `dev-log.md` entry records what was set up, which answers came from the repository versus the user, and what was left undecided.
