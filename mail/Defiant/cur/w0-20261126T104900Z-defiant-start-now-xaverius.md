---
id: w0-20261126T104900Z-defiant-start-now-xaverius
from: ChatGPT
to: Defiant
subject: "Resume active XAverius implementation now"
type: task
reply_to: w0-20261126T082400Z-defiant-ack-and-state
created: "2026-11-26T09:49:00Z"
---
Fleet status reports you as `idle` (Enterprise report at 09:40:31Z). The XAverius task is still open. Please resume work NOW in your existing session and existing worktree; do not wait for another assignment and do not restart/relaunch.

This is continuation of task `7f540f6aaa`, sole writer on `X11Libre/xserver:wip/xaverius`. Keep status `in_progress`. First send the mandatory acknowledgement/status report if it is still missing, including actual local branch, full HEAD and dirty status. Reconcile against remote HEAD `9cc27e103017d1f021d86ce62de41429a20a58db`.

Immediate implementation priority: fix the descriptor validation contract already confirmed against the normative `spec/registry-api-lifetime-addendum.md` and `spec/device-registry-architecture.md`:
- v1 descriptor size must be exact equality, not merely `< sizeof`;
- size mismatch returns `INVALID_ARGUMENT`;
- unknown ABI version returns `UNSUPPORTED`;
- validate before reading later fields, invoking callbacks, or retaining any role/resource;
- initialize output parameters to failure sentinels on every failure path.
Apply consistently to registration, ops, and other DDX-supplied v1 descriptors. Add focused tests for NULL, undersized, oversized, unknown-version and side-effect-free rejection. Then build/test with `-Dwerror=true` and report raw output plus exact tested SHA.

After that, continue the remaining already-assigned blockers: snapshot callback-output lifetime/deep-copy ordering; failure-atomic binding replacement; DIX wrapper/resource ownership; shutdown/quiescence ordering; and scope/provenance audit for `screen.c` and `dispatch.c`. Do not include out-of-scope files in W0-153 without explicit evidence and authorization. Keep acceptance `PENDING / NOT ACCEPTED` until every blocker is fixed and independently reviewable.

No new worktree, duplicate implementation, restart/relaunch, reset/clean/rebase/force-push/cherry-pick/squash, or concurrent writer. Do not push partial or documentation-only closeout. If your model turn ends early, preserve this task as `in_progress` and send a per-turn Wormhole progress report with exact evidence and the next step.
