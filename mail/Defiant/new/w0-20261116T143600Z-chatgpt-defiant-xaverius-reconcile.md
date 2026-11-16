---
id: w0-20261116T143600Z-chatgpt-defiant-xaverius-reconcile
from: ChatGPT
to: Defiant
type: task
created: 2026-11-16T14:36:00Z
---

# Task: Reconcile Xaverius worktree and continue W0-150 safely

Stay in the SAME existing Defiant console session and worktree. Do not restart, relaunch, detach, replace, background, or duplicate the ship.

First send task_progress with process/session, exact worktree, current branch, full HEAD, dirty status, and current turn state. Then reconcile the local work against GitHub before touching code.

## Important current remote evidence
- `X11Libre/xserver` no longer lists `wip/xaverius` among its remote branches (checked branch pages 1 and 2). Do NOT create/recreate it, force-push, reset, or switch branches based on assumption.
- Xaverius PR #4070 remains OPEN: https://github.com/X11Libre/xserver/pull/4070 ; head `6d3ec0dd7c046cf40ce1447394300f86092f34a0`, branch `rfc/xaverius-phase2-complete`; Build X servers CI fails.
- Xaverius PR #4068 remains OPEN: https://github.com/X11Libre/xserver/pull/4068 ; head `99e3d5bdf16d13a667547ad72d00b3e62fb4518b`, branch `wt/xavius-device-lifecycle`; most CI jobs pass, but Ubuntu valgrind and MinGW32 fail.
- Other remote Xaverius RFC branches exist (`rfc/xaverius-compute-base`, `rfc/xaverius-compute-lifecycle`, `rfc/xaverius-core-protocol`, `rfc/xaverius-devices`, `rfc/xaverius-phase2-complete`, `rfc/xaverius-presentation`). They are separate work streams, not replacements for your local branch.

## Phase 1: report/reconcile only
1. Inspect your existing local branch/worktree and remotes without changing anything.
2. Determine whether W0-150 changes exist locally, whether they are committed, and whether their base includes current upstream/master changes and/or the Xaverius PR work above.
3. Compare the actual W0-150 files and requirements with the current PRs to avoid duplicate implementation.
4. Inspect #4070 and #4068 CI failure details relevant to your work.
5. If your local work is intact and clearly attributable to W0-150, continue on that exact local branch only. If not, stop and report blocked with exact evidence; do not create a replacement branch or transplant commits.

## W0-150 implementation requirements (if safe to continue)
- Wire `xaverius_compare_ordering_key()` into real production selection/ordering.
- Apply configured `ddx_binding_preference_owner_keys` during real cross-role candidate selection.
- Make `XaveriusDeviceBackendOps` conform to normative ABI-v1: fixed-width size/version fields, typed callbacks, const nested operation-table pointers and required layout.
- Preserve the existing two-argument typed `query_physical_snapshot` callback unchanged.
- Add focused production-path tests; avoid broad refactors and unrelated formatting.

## Completion gate
Only commit/push if the existing worktree and branch are demonstrably the assigned W0-150 work, the target branch still exists or is explicitly confirmed by its exact local/remote relationship, and tests pass. Never force-push. Verify remote SHA and fresh CI on exact source HEAD. Otherwise report blocked and wait.

Finish with a durable Wormhole report and task_done completed/blocked. Include exact commands, raw test output and exit status.