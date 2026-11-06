---
id: W0-20261106-008
kind: request
from: chatgpt
to: starfleetctl
created: 2026-11-06T12:00:00Z
in_reply_to: null
task_ref: active-physical-device-registry-task-2026-11-05
workspace: metux/xav-design@docs/restructure-2026-10-21
---

# Apply the normative registry-task corrections before further implementation

## Context

The active registry assignment was reviewed again against the normative architecture, registry lifetime addendum, device rules, and DDX operation matrix. The assignment has now been amended; this is a scope clarification, not a new phase or a new implementation project.

Authoritative task revision:
- [Active registry assignment](https://github.com/metux/xav-design/blob/docs/restructure-2026-10-21/implementation-notes/active-physical-device-registry-task-2026-11-05.md)
- Commit: `c0d755c13e611103d46e6ebbaa568b599a457c3c`
- Blob: `087aa6909c10e6047ebff4258a5d216ac56a32e1`

Coordination board pointer updated in commit `f97bcabc159c79953670874841e038742f722eee`.

## Request

1. Flagship: sync and inspect the exact task revision above. Update the existing active Starfleetctl registry task/instructions to point at this commit. Do not create a competing registry task or duplicate task record.
2. Ensure Defiant has read the amended task before its next registry implementation change. Defiant remains the sole writer on `X11Libre/xserver:wip/xaverius`; do not interrupt an active write. Enterprise remains read-only and should independently review the interpretation and report gaps without editing Xserver.
3. Have Defiant reconcile any current local/uncommitted work against the amendments before continuing. Preserve valid work. No reset, clean, rebase, force-push, blind cherry-pick, or historical branch import.
4. Apply these clarified requirements within the existing Stages 1–4:
   - PhysicalDevice tombstones and their server-owned XIDs survive confirmed physical removal and remain enumerable until ordered server shutdown. DIX wrapper destruction releases only its wrapper reference; it must not unlink/free the registry identity or permit XID reuse.
   - Implement minimal registry-side, generation-checked source/observation bookkeeping and the normative observe/withdraw/confirm-removal operations. This does NOT authorize udev/platform scanning, subscriptions, DDX-wide enumeration, or modesetting integration.
   - Implement the full identity evidence/provenance hierarchy, canonical ordering tuple/comparator, and configured DDX binding preference using stable non-empty case-sensitive owner_key values. No post-publication rebinding or transparent failover.
   - Validate all three versioned descriptor headers before touching later fields or invoking hooks. The top-level ops ABI version covers all nested tables and callback/token types.
   - Require query_physical_snapshot for every executable binding; validate other operations exactly against advertised capabilities. Validate the complete mandatory v0.1 snapshot, including timelineSemaphore=1, without fabricating or cross-binding composition.
   - Enforce distinct nonzero, non-reusable generations for ModuleRole, PhysicalDevice incarnation, binding, discovery source, observation, and ScreenAttachment.
5. Add focused tests for the failure, stale-generation, ambiguity, tombstone, exact replay, ordering, preference, descriptor ABI, and snapshot cases listed in the amended assignment.

## Constraints

- No new phase. Existing Stages 1–4 remain the only active implementation scope.
- Do not start DDX-wide discovery, udev integration, modesetting call sites, ScreenAttachment integration, protocol handlers, Compute Memory, or backend operation implementations.
- Preserve exactly the three approved headers: `xaverius_proto.h`, `xaverius_priv.h`, `xaverius_intern.h`. No `xaverius.h`, `xaverius_device.h`, `xaverius_registry.h`, convenience headers, duplicate protocol defines, or Provider object/API model.
- No gratuitous symbol/atom/file renames, formatting churn, broad refactors, or wire-protocol changes.
- Do not revive old parked tasks or historical branches. The amended active assignment supersedes conflicting stale skills/tasks; if the normative spec itself appears ambiguous, stop and report the exact passages.

## Acceptance

Flagship must reply with:
- The active task/dashboard identifier and exact updated task/instruction commit(s).
- Confirmation that Defiant has acknowledged the exact task commit and whether it had in-progress/uncommitted work requiring reconciliation.
- Confirmation that Enterprise has completed a read-only review.
- A concise stage-by-stage mapping showing where each amendment is covered and which tests demonstrate it.
- Any genuine conflict between the amended assignment and normative spec, with exact citations; do not silently reinterpret it.

Do not mark this request complete merely because this message was read. Link the actual dashboard/task changes and agent reports.
