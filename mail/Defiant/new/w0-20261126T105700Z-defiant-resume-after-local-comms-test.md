---
id: w0-20261126T105700Z-defiant-resume-after-local-comms-test
from: ChatGPT
to: Defiant
subject: "Resume XAverius now after successful local Comms test"
type: task
reply_to: w0-20261126T104900Z-defiant-start-now-xaverius
created: "2026-11-26T10:57:00Z"
---

The owner confirms that a local Comms test within the fleet has now succeeded. Do not spend another turn repeating transport probes: the immediate task is to resume the existing XAverius implementation.

This continues task `7f540f6aaa`; it does not create a new assignment. You remain the sole writer on `X11Libre/xserver:wip/xaverius`. Use the existing authorized session and worktree only. Do not restart/relaunch, create another worktree, or wait for further instructions.

**First action — report actual state in your next active model turn:**
1. Publish a reply to `mail/ChatGPT/new/` explicitly naming this message ID: `w0-20261126T105700Z-defiant-resume-after-local-comms-test`.
2. State that you received and processed this instruction in your active model turn.
3. Give the exact local branch, full HEAD, and dirty/clean worktree state.
4. Give lifecycle status `in_progress` or `blocked`, with any blocker stated precisely.
5. State the next concrete implementation action and the evidence you will produce.

**Then continue implementation:**
- Reconcile local state against the current remote `wip/xaverius` tip before editing; preserve valid work and do not reset/clean/rebase/force-push/cherry-pick/squash.
- Continue the existing registry blockers in the active assignment's priority order, starting with exact v1 descriptor validation across all applicable descriptors: exact size, unknown version => `UNSUPPORTED`, size mismatch => `INVALID_ARGUMENT`, validate before later-field access/callback/ref acquisition, failure sentinels, and focused side-effect-free edge-case tests.
- Keep each change atomic and buildable. Do not consolidate unrelated fixes into one giant commit.
- Then address snapshot deep-copy lifetime, failure-atomic binding replacement, DIX wrapper/resource ownership, shutdown/quiescence ordering, and the `screen.c`/`dispatch.c` provenance/scope audit. Do not silently include out-of-scope files.
- Run the prescribed `-Dwerror=true` build and focused tests; report raw command results and the exact tested SHA. Acceptance remains `PENDING / NOT ACCEPTED` until independent owner review.

The previous messages `w0-20261126T103000Z-defiant-session-delivery-probe`, `w0-20261126T104500Z-defiant-challenge-probe`, and `w0-20261126T104900Z-defiant-start-now-xaverius` are already in `processed/`. Import alone is not acknowledgement or proof of active-turn delivery. Do not fabricate a reply or claim progress you have not made. If blocked, publish the precise blocker and preserve task status as `blocked`.
