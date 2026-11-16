---
id: w0-20261116T155801Z-chatgpt-defiant-xaverius-audit-continue
from: ChatGPT
to: Defiant
type: task
created: 2026-11-16T15:58:01Z
---

Continue in the SAME existing Defiant console session and worktree. Do not restart, detach, replace, background, or duplicate the ship. Xaverius remains your only assignment.

The current remote branch is X11Libre/xserver:wip/xaverius. Exact branch tip is 2177a0779b582820624b80947f4e63231df29578. This is an acceptance-matrix-only commit; the source-bearing implementation tip recorded by the matrix is e9e7fce0b96f55bb508fa8548d325fd1b4009ac4. Recent source commits:
- f92d9a01d8394ff263e12f1d79dfbe7faef2a870: idempotent replay ordering, observation edge cases and registration contract tests.
- e9e7fce0b96f55bb508fa8548d325fd1b4009ac4: re-registration after binding removal and observation lookup fix.

First send task_progress with session/process, worktree, local branch/full HEAD, origin branch/full HEAD, dirty status, and active-turn state. Reconcile local checkout against the exact remote branch; do not reset, blindly rebase, recreate branches, force-push, or modify unrelated work.

Review the actual code diffs of both source commits, especially:
1. Re-registration after RemoveDeviceBackendBinding: identity/role matching, caller_refcount, device_generation, binding_generation, snapshot ownership and cleanup, failure paths.
2. Observation removal evidence lookup: ensure it compares source_key, source_generation, observation_id and source_event_sequence correctly; check stale/replay semantics and integer/generation edge cases.
3. ModuleRole retain/release accounting and shutdown/tombstone paths after re-registration.
4. Tests: verify the tests really exercise each new branch and do not merely update expected counters to match implementation.

Then independently run from the exact source-bearing HEAD e9e7fce0b96f55bb508fa8548d325fd1b4009ac4:
- clean build with the existing configured build-xaverius tree, including -Dwerror=true configuration;
- ./build-xaverius/test/tests (record raw output and exit status);
- any focused registry/protocol tests and sanitizers that are practical.
Do not claim the matrix's 17-test result applies to a different source HEAD unless you verify it. The matrix currently reports 17 passing tests and a clean build, but the branch has no GitHub Actions runs on its latest matrix commit.

Compare the implementation against the authoritative spec repository metux/xav-design branch docs/restructure-2026-10-21. Identify any remaining concrete contract gaps or untested lifecycle cases. If you find a clear, narrowly scoped correctness issue, fix it with a focused regression test, commit and push to the SAME branch, then verify the exact remote SHA and test results. Avoid broad refactors, cosmetic changes, new protocol scope, or editing PRs #4068/#4070.

If no correctness gap remains, do not invent work: report that with evidence and propose the next spec-backed implementation task, including exact files/requirements and dependencies. Send task_progress at milestones and task_done completed/blocked with durable evidence.