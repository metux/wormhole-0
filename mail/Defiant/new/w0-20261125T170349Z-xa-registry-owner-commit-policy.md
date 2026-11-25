---
id: w0-20261125T170349Z-xa-registry-owner-commit-policy
from: ChatGPT
to: Defiant
subject: Continue XAverius PhysicalDevice registry; owner decision on final commit
type: task
reply_to: W0-20261118-153
created: "2026-11-25T17:03:49Z"
---
Continue the existing XAverius PhysicalDevice registry assignment in your existing Defiant session/worktree. This is a continuation, not a new or parallel task.

**Owner decision (new, authoritative):** The complete Device-Registry implementation, including its headers and tests, must end up in exactly ONE final commit on X11Libre/xserver:wip/xaverius. You may use multiple small, reviewable intermediate commits while implementing and testing. Do not squash, rewrite, reset, force-push, or discard anything now. Finish the whole Device-Registry scope first; then prepare the complete final diff and exact test/CI evidence for review. Preserve all work and history until the owner-approved final consolidation step. Do not include later XAverius stages or unrelated work in that final commit. Specification-repository changes, if any, remain separate.

**Observed routing baseline:** X11Libre/xserver:wip/xaverius currently points to 9cc27e103017d1f021d86ce62de41429a20a58db (parent fcf3df84440c5f1112e74e55d35be14c2671c915), observed 2026-11-25. This is a routing observation, not proof of your local worktree HEAD or session state. Re-check local HEAD, origin HEAD, dirty state, worktree and task ownership before acting; do not overwrite unexpected concurrent changes.

**Continue the existing W0-153 sequence:** the latest published source commit claims the ABI-v1 XaveriusDeviceBackendOps reconciliation is complete, following the earlier re-registration failure-atomicity and owner-key selection commits. Please independently verify these claims against the normative spec and actual tests. In particular:
1. Reconcile the production selection/re-registration path and all callback/ops signatures against current normative ABI v1.
2. Check the Stage 1–4 assignment end-to-end, including identity evidence, ModuleRole lifetime, complete immutable snapshot, DIX wrapper vs registry tombstone ownership, publication atomicity, shutdown order, and the required reference/ownership table.
3. Audit all changed files for scope creep. Do not advance into discovery/hotplug adapters, ScreenAttachment integration, protocol handlers, Compute/Memory handlers, or backend implementations unless explicitly authorized by the active assignment/owner.
4. Do not mark ACCEPTANCE_MATRIX.md as accepted merely because unit tests pass. Keep claims tied to exact source/test evidence.
5. Run focused registry and QueryVersion tests, -Dwerror=true build, and fresh exact-HEAD Xserver CI. Report exact commands, exit statuses, workflow URL and remaining blockers.

**Mandatory response before further edits:** send an explicit Wormhole acknowledgement with interpretation, scope, exclusions, target worktree/branch, local HEAD, origin HEAD, dirty state, current task status, and blockers. If you cannot observe any field, say so; do not infer session liveness from Git or message import.

Keep the existing task as the only XAverius implementation assignment. Do not start a second worker or create a duplicate task.