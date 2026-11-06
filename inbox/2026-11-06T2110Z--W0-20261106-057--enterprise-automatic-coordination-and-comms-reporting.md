# W0-057 — Enterprise: execute local coordination and report through Wormhole

**Owner:** Enterprise  
**Role:** local coordination and Comms routing only  
**Priority:** immediate  
**Created:** 2026-11-06

## Purpose

Continue the coordination loop without waiting for another manual prompt. This is a coordination task, not a technical audit or implementation task.

## Required actions

1. Refresh/pull `metux/wormhole-0:master` before acting. Read the latest bootstrap and inbox tasks.
2. Through the established local Comms mechanism, deliver the currently active instructions to the implementers:
   - Defiant: W0-050 and W0-052 (registry corrections on `X11Libre/xserver:wip/xaverius`).
   - Laforge: W0-051 and W0-054 (Wormhole implementation corrections on its isolated Starfleetctl branch).
   - Also deliver W0-055 and W0-056 to the local coordination process as applicable.
3. Explicitly tell Defiant and Laforge to continue independently in small, focused commits, push their branches, and report commit SHAs. They must not wait for an Enterprise technical audit or an owner approval between implementation steps.
4. For Laforge, specifically communicate that commit `94af3ab07b5188e6a994dd98c40a93c2365355da` is an unaccepted draft. The first required correction is real integration with the existing `internal/comms.Bus` via its supported external-posting API; local filesystem inbox mirroring is not remote delivery. Subsequent work must address durable/idempotent delivery, W0-026 identity/sequence/recipient rules, real worker/daemon lifecycle, config/CLI/service integration, and fault/race tests. Keep these as small separately committed packages.
5. For Defiant, specifically communicate that the first priority is the snapshot callback ABI/size contract and tests, followed by callback validation/deep-copy, complete cleanup and ModuleRole accounting, replay/removal generation correctness, canonical device lifetime/tombstones/ordered finalization, Stage 3 authority, and duplicate declarations. No Stage 5, Screen/DDX integration, Proc handlers, Vulkan backend, unrelated renames, or mass formatting.
6. Establish and maintain the requested real Comms feedback loop. Send a targeted status message only to a verified configured recipient; do not guess an identity or destination. Report the actual Comms message ID, recipient/route, send result, and acknowledgement status.
7. Write a durable coordination report into Wormhole with:
   - local worktree paths, branch names, current HEAD SHAs, dirty/unpushed state;
   - which tasks were actually delivered, to whom, by which Comms route, and when;
   - implementer acknowledgements (or explicitly none);
   - the current Comms feedback-loop state and message ID;
   - blockers requiring owner action.
8. Repeat coordination when new owner tasks arrive or implementers publish new commits. Keep reports factual and distinguish task publication from actual delivery and acknowledgement.

## Boundaries

- Enterprise performs local coordination, message delivery, nudging, and factual status reporting only.
- Do **not** perform technical audits, acceptance decisions, code changes, or merge decisions. The owner performs all technical audits, including every Laforge branch/commit and all Defiant registry commits.
- Do not claim delivery, acknowledgement, or a Comms send unless it actually happened and can be evidenced.
- Do not rewrite history or push to protected/master branches.
- If a local Comms route or worktree is unavailable, report the exact limitation and the next concrete action; do not silently substitute a filesystem write for remote Comms delivery.
