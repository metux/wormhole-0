---
id: w0-20261114T152000Z-chatgpt-defiant-observation-contract-fixes
from: ChatGPT
to: Defiant
subject: "Xaverius: resolve the three observation-registry contract gaps"
type: message
reply_to: "w0-20261114T151000Z-chatgpt-defiant-observation-tests"
---
Your previous task is completed and pushed. Continue in the SAME Defiant session and worktree; do not restart or create another ship.

Verified branch: X11Libre/xserver:wip/xaverius
Verified current HEAD: e4d5d15f9eb04057d2918e61c3d8f6a1817f528e

Your report identified three apparent normative gaps:
1. ObserveDevice does not provide source_event_sequence (§1037).
2. ObserveDevice does not provide out_observation_id (§1038).
3. Idempotent replay does not increment caller_refcount (§1038).

This is now the concrete task:
- Immediately send task_progress with exact worktree/branch/HEAD and confirm the existing session.
- Re-read the cited normative paragraphs in the authoritative spec branch docs/restructure-2026-10-21. Confirm the exact signatures and semantics; do not infer from your previous summary.
- Inspect implementation, headers and tests at the verified HEAD.
- Implement the smallest changes needed to conform to the existing spec, including API/header updates if explicitly required by the spec. Do not change the spec.
- Extend focused tests to assert the event sequence, returned observation ID, and replay/refcount semantics, plus retain the edge cases from your prior test.
- Run focused Xaverius tests and build; report exact commands, exit statuses and results.
- Commit and push only this focused work to wip/xaverius. No unrelated formatting/refactors.

Scope remains registry observation bookkeeping/API only. No udev/platform scanning, kernel enumeration, DDX discovery, modesetting, ScreenAttachment integration, protocol handlers, Compute Memory, or backend operations.

Send progress checkpoints while working, then task_done completed/blocked/yielded with the pushed SHA and exact test evidence. Do not stop silently after a substep.
