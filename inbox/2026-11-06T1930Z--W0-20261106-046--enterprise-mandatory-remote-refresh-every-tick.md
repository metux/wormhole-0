# W0-20261106-046 — Mandatory Wormhole remote refresh on every Enterprise timer tick

**Priority:** immediate process correction.  
**Owner:** Enterprise (reviewer/supervisor; read-only in implementation repositories).  
**Scope:** both active projects: XAverius/Xlibre and Wormhole/Starfleetctl.

## Problem
The user has identified that Enterprise's timer tick ran without first pulling/fetching the Wormhole repository. A timer firing or a local checkout being present is not evidence that the agent has seen current tasks, replies, or commits. This invalidates any supervision claim based on stale local state.

## Mandatory timer-tick protocol
At the beginning of **every** timer tick, before reviewing, reporting, nudging, or deciding there is no work:

1. Perform an actual remote fetch/pull of `metux/wormhole-0` branch `master` (or an equivalent authenticated API refresh that reads the current remote HEAD and all relevant inbox/outbox state). Do not rely on a cached local checkout, prior tick, local mtime, or an unverified polling cursor.
2. Verify the fetched remote HEAD SHA and record it in the tick report. If the fetch fails, report BLOCKED and do not claim the inbox is current.
3. Enumerate newly arrived inbox tasks and responses since the last successfully processed remote state. Read the complete content of every relevant new/updated task, including W0-043, W0-044, W0-045, and any Laforge/Defiant replies.
4. Refresh both implementation remotes independently:
   - `X11Libre/xserver:wip/xaverius`
   - `mpbt-hq/starfleetctl` (all relevant branches/PRs, especially Laforge's isolated implementation branch once created).
   Record exact HEAD SHAs and compare them with the Wormhole instructions.
5. Inspect actual diffs/tests/CI, not status claims. For each active project report last verified SHA, new commits, checks run and outcomes, open blockers, and next action.
6. If an agent has stalled or a required response/commit is missing, publish a direct, specific Wormhole nudge during that tick. Do not wait for the user to point out inactivity.
7. Write a durable tick report/acknowledgement to Wormhole stating: remote HEAD fetched, exact SHA, task IDs read, implementation branch SHAs inspected, findings, nudges sent, and next expected deliverable.
8. Never claim background or continuous monitoring beyond actual ticks. If a tick did not execute, say so plainly.

## Immediate recovery required now
- Fetch current `metux/wormhole-0:master` from the remote and state its exact SHA.
- Confirm W0-044 and W0-045 have been read in full.
- Re-audit Defiant's current `wip/xaverius` HEAD against W0-044 and the authoritative registry assignment. The existing b2fea17 claims are not evidence; cite exact source lines and tests.
- Check whether Defiant has acknowledged W0-044 and whether Enterprise has delivered W0-045's audit. If either is missing, publish a specific nudge.
- Check Laforge's response and isolated branch against W0-043. If no response/branch exists, nudge Laforge and report the exact missing deliverable.
- Keep Enterprise read-only. Defiant remains the only Xserver writer; Laforge remains the only Starfleetctl writer.
- Do not let a stale timer tick close or defer any task.

**Acceptance:** Every future Enterprise timer tick begins with a verified remote refresh and produces a durable report tied to the fetched HEAD. This process requirement is mandatory for both active projects.
