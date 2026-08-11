# Infrastructure and Repo Model

**Current-state truth, rewritten in place.** How the whole setup fits together — the knowledge a new contributor or a fresh agent session cannot reconstruct from any single file.

**Why this document earns its place:** infrastructure knowledge is the most expensive kind to lose and the least likely to be written down, because whoever set it up did not need notes. It is also where an agent, lacking that memory, can do the most damage by guessing.

**Credentials appear here by name and location only. Never a value.**

---

## Environments

<!-- SAMPLE — replace. Mark inferred rows as inferred. -->

| Environment | Purpose | Reached by | Who can access |
|---|---|---|---|
| Local | Development | `<command>` | Anyone with the repo |
| Staging | Pre-release testing | Merge to `<branch>` | Team |
| Production | Live users | `<release process>` | Owner only |

## Branch → environment mapping

Which branch reaches which environment, **and by what mechanism**. Read from CI trigger conditions; these are authoritative.

| Branch | Triggers | Reaches |
|---|---|---|
| | | |

## Hosting and services

Where things run; third-party services depended on and what each provides. Note anything with a single point of failure.

## Build and release pipeline

What is automated, what is manual, and **what the manual steps are**. A manual step that only one person knows is an outage waiting for a holiday.

## Credentials — names and locations

| Name | Lives in | Scope | Shared across environments? |
|---|---|---|---|
| | | | |

Shared credentials have a wider blast radius; flag them.

## Known manual steps and their failure modes

Anything relying on someone remembering. For each: what it is, what happens when it is forgotten, and how you would notice.

## Recovery

What backups exist, where, how to verify one is good, and what is **not** covered.
