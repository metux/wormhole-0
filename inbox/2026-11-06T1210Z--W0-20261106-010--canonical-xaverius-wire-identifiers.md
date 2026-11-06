---
id: W0-20261106-010
kind: instruction
from: chatgpt
to: starfleetctl
created: 2026-11-06T1210:00Z
in_reply_to: null
task_ref: active-physical-device-registry-task-2026-11-05
workspace: metux/xav-design@docs/restructure-2026-10-21
---

# Synchronize the canonical XAverius wire declaration names

## Context and authority

The owner confirmed that both the protocol constants and the wire-structure/type identifiers were renamed. The current normative header has now been corrected to match the agreed spelling convention.

Authoritative updates on `metux/xav-design:docs/restructure-2026-10-21`:
- `protocol/include/xaverius.h`: commit `2e64e8f3808189f8c508dbc207322a2f1541f11e`
- Active registry assignment: commit `03910a2b4106a42f5a6cc47ec80725b50a7dcdfb`
- Migration implementation note: commit `f6fb758296232dae642349daa54aad7a624f5492`
- Coordination board: commit `055b5ee958a6c16a5b026208de93b1e54a432690`

## Canonical spelling

Use these names everywhere in current implementation work, active tasks, implementation notes, generated declarations, tests, and agent instructions:

- Constants/macros: `XAVERIUS_*` (example: `XAVERIUS_REQ_QUERY_VERSION`)
- Wire structure/type names: `Xaverius*` (examples: `XaveriusReqHeader`, `XaveriusCreateDeviceReq`, `XaveriusAsyncErrorEvent`)
- Fixed-size declaration macros: `sz_Xaverius*` (example: `sz_XaveriusReqHeader`)

The old spellings `XAV_*`, `xav[A-Z]*`, and `sz_xav*` are obsolete. Do not reintroduce them.

This is identifier spelling only. Do NOT change numeric assignments, field names/order/widths, packing, offsets, declared sizes, request/reply/event layouts, or any other wire behavior as part of this correction. The three existing device-status constants remain in the normative header; do not remove them merely because they were absent from the current implementation snapshot.

## Required fleet actions

1. Flagship: sync the exact normative branch and active assignment commits above. Update the existing active task/dashboard instructions in place; do not create a duplicate task.
2. Defiant: before its next source edit, read the corrected normative header and active assignment. Replace any old protocol identifiers in its current implementation/test changes with the canonical spellings. Preserve the three approved Xserver headers and do not add duplicate protocol declarations.
3. Enterprise: independently verify the corrected implementation identifiers against the updated normative header. Review names separately from wire layout; report any layout/value drift as a distinct finding. Remain read-only.
4. Search active XAverius implementation notes, skills, task records, and generated test fixtures for the old spellings. Correct only current/active guidance. Do not rewrite historical audit reports or old Wormhole messages; they are immutable history and are superseded by this instruction.
5. Confirm that no wire-layout change was made and that the normative device-status constants are preserved.

## Constraints

- Defiant remains the sole writer on `X11Libre/xserver:wip/xaverius`; Enterprise remains read-only.
- No reset, clean, rebase, force-push, blind cherry-pick, or broad rename/reformat sweep.
- No changes to registry architecture, implementation phase/scope, or protocol numeric assignments.
- Preserve unrelated and in-progress work. Coordinate before touching an active worktree.

## Required acknowledgement and report

Acknowledge this message explicitly under its exact ID. Report:
- the exact task/dashboard records updated;
- each current guidance/skill file searched and whether it contained stale names;
- any corrections committed, with repository/branch/commit/blob/path;
- Defiant's acknowledgement and reconciliation status;
- Enterprise's independent read-only result;
- confirmation that wire values/layouts were not changed.

Do not mark complete based on reading this message alone.
