# Agentic Product Management System

A product-lifecycle system for software built with AI coding agents.

It exists because agents lose knowledge in a specific way: the code records *what* is true and version control records *when* it changed, but neither records why a decision was made, what was tried and rejected, which assumption turned out wrong, or how a change was verified. That knowledge lives in a session transcript that is discarded, held by an agent whose context resets completely.

Three pillars:

- **Capture** — turn development prompts, process learnings, and the findings and agreements that emerge implicitly while working with an agent into durable internal reference. An agent-instruction file already covers *how* the human and agent work together; this system covers what is learned *about the product* while building it — design rationale and rejected alternatives, current and superseded implementation specs, and the gotchas where the code misleads a reader who is reasoning correctly. That knowledge has no natural home and is lost by default.
- **Protect** — verify changes against a signal that cannot be talked out of a verdict, and prevent regressions.
- **Propagate** — keep marketing copy, user documentation, and store listings current with each release. The pillar most often left entirely undone, because it is nobody's job and nothing fails when it is skipped.

**Why "product management" rather than "documentation":** documentation is one output, not the purpose. Product marketing sits inside the scope as a lifecycle step, even though in most organisations marketing and product management are separate functions in separate departments. That separation is organisational, not logical — the release that changes a behaviour and the sentence on a website describing that behaviour are the same fact recorded twice, and treating them as separate concerns is precisely how the second one goes stale.

---

## Contents

| File | What it is |
|---|---|
| `system.md` | **Read first.** What the system is, the full folder structure, and the reasoning behind every structural decision. Descriptive — no rules. |
| `setup-protocol.md` | The procedure for standing the system up in a project: what to read from the repository, what to ask the user, and in what order. Written for the installing agent. |
| `templates/CLAUDE.md` | The agent's operative rules. Universal boilerplate filled in; project-specific sections carry their own setup instructions. |
| `templates/docs/` | Sample log, todo, and reference-doc files. Each carries illustrative content marked for deletion at first real use. |

---

## Installing it

Point an agent at this directory and ask it to follow `setup-protocol.md`. The short version of what it will do:

1. **Secret handling and its enforcement hooks first.** The only item where being late is unrecoverable.
2. **Establish what cannot be undone** in this project — the irreversibility inventory that drives the permission model. The one decision with no safe default.
3. **Decide the concerns and how pending work is tracked**, which determines which files get created.
4. **Copy the templates in**, minus anything the project does not have.
5. **Fill the remaining sections**, preferring what the repository can answer over what the user has to.
6. **Backfill nothing.** Start the logs from today.

The protocol's governing rule is **harvest, then inspect, then interview**: rules already written down are already agreed; continuous-integration configuration and manifests are authoritative and free to read; the user's attention is the scarcest input and is spent last.

---

## The shape of it, in brief

Records split on **concern** (product / product marketing / documentation) × **state** (finished / pending).

- **Finished work** goes in append-only change logs, newest entry on top, never rewritten.
- **Pending work** goes in an issue tracker where one exists, with a committed index file so a fresh checkout still shows what is outstanding. Where none exists, the markdown files carry the queue themselves — a documented fallback rather than the intended design.
- **Above both** sits one session log carrying the narrative, because the first question anyone asks is "what happened and where are we", not "what changed in this area".
- **Beside both** sits reference documentation holding current-state truth, rewritten in place — as distinct from history, which is not.

`system.md` explains why each of those is the way it is, and is honest about the four places the design is weak.

---

## Adapting it

Every project-specific decision is marked and has a stated default. Three deserve real thought rather than acceptance:

- **What is genuinely irreversible here.** An incomplete list is worse than an absent one, because it reads as authoritative.
- **Whether pending work has a real home.** Markdown todo files degrade structurally, not through carelessness.
- **Which published surfaces exist, and whether any is currently authored outside the repository.** That last case inverts the source-of-truth rule and is a decision to raise, not to inherit.

Nothing here is load-bearing on a particular language, framework, or hosting model. Where a rule depends on project specifics, the specifics are elicited rather than assumed.
