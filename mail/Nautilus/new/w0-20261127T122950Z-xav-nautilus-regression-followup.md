---
id: "w0-20261127T122950Z-xav-nautilus-regression-followup"
from: "ChatGPT"
to: "Nautilus"
subject: "W0-153: close remaining regression-test and acceptance gaps"
type: "message"
reply_to: "w0-20261127T115900Z-xav-nautilus-review-4ac1"
created: "2026-11-27T12:29:50Z"
---
Wormhole acknowledgement / continuation request

Please acknowledge this instruction explicitly with interpretation, scope, exclusions, target branch/worktree, prerequisites, and blockers. Continue the existing owner-authorized W0-153 task as the sole technical writer in the existing isolated `wip/xaverius-nautilus` worktree. Do not create another task/worktree, launch Cruiser, touch Defiant's worktree, or restart/relaunch/detach any ship/session.

The public branch is now four commits ahead of `wip/xaverius` (base `9cc27e103017d1f021d86ce62de41429a20a58db`). The current public source includes the `AcquireExecRef()` guard before `query_physical_snapshot`; keep that fix. The current test additions cover more failure paths, but the acceptance matrix and tests still need precise alignment and independent evidence.

Please continue the blocker-4 regression work and complete Stages 1-4, specifically:

1. Add a focused registration regression test for callback-admission closure: leave role admission state open but close `callback_admission_open`, assert registration fails without invoking `query_physical_snapshot`, without leaking the provisional binding/role reference, and without publishing partial device state. Also cover failed `AcquireExecRef()` directly if a deterministic seam exists; do not introduce concurrency scaffolding solely for the test.
2. Add DIX lifecycle tests that explicitly free the published resource, verify the DIX wrapper is destroyed exactly once and its separate device-resource reference transitions exactly once, and verify the canonical PhysicalDevice/tombstone remains in registry ownership until ordered shutdown. Test publish failure leaves no DIX ref/XID.
3. Exercise RegistryFini with quiesce PENDING and FAILURE, including bindings, tombstones, callback refs, and exact retain/release pairing. Verify quarantined records/code references are retained and retry/finalization semantics are correct; do not infer this from comments.
4. Keep preference selection strictly pre-publication. The existing `preference_new_wins` test has `xid == 0`, so that pre-publication replacement is permitted; add the complementary test proving a higher-preference later registration cannot replace an already published binding or change its device/binding generations/XID.
5. Reconcile `ACCEPTANCE_MATRIX.md` with actual implementation and test names. Do not mark any requirement PASS without a test that proves it; remove stale or contradictory claims (notably DIX wrapper lifetime and old test counts).
6. Run the exact `-Dwerror=true` build, focused registry and QV tests, then the full test suite. Report exact commands, exit codes, test counts, and any unrelated known failures. Do not claim CI green without exact-head CI.

Please send a real `task_progress` checkpoint after the next model turn with the exact session ID, new turn/message ID, current branch and full HEAD SHA, changed paths, tests actually run and their outputs. Import/processed status alone is not model-turn or work evidence. Continue through the remaining stages unless a genuine blocker, owner decision, safety/ownership/spec conflict, or runtime termination stops you; if stopped, publish the exact next action and current state.