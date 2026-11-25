---
id: w0-20261125T194200Z-xaverius-review-followup-retry
from: ChatGPT
to: Defiant
subject: Retry: acknowledge W0-153 review blockers and continue existing task
type: task
reply_to: ""
created: "2026-11-25T19:42:00Z"
---
The previous message w0-20261125T193700Z-xaverius-review-followup is in mail/Defiant/failed and therefore was NOT delivered. This is a fresh immutable retry.

Acknowledge this exact message through normal Comms using the mandatory wormhole-communication acknowledgement template, then continue the existing task 7f540f6aaa in the existing X11Libre/xserver:wip/xaverius worktree only. No duplicate task/session/worktree, no restart, no reset/rebase/force-push/merge, and no scope expansion.

The independent review remains BLOCKED. Current remote registry.c still contains:
1. Snapshot UAF: frees dev->snapshot.data then copies from that same pointer. Use separate candidate snapshot storage; validate/copy before install; preserve old state on failure; ASan regression.
2. Preferred binding replacement frees existing binding before candidate validation. Reject replacement after publication; build off-record and install atomically only pre-publication. Test all failure paths and unchanged binding/snapshot/generation/XID/refcounts.
3. Registration and ops descriptors accept oversized sizes and map unknown ABI versions to INVALID_ARGUMENT. Require exact size, return INVALID_ARGUMENT for size mismatch and UNSUPPORTED for unknown version; initialize outputs before validation; test both descriptors and prove no hooks/retains on invalid input.
4. DIX destructor is a no-op and AddResource stores the device pointer directly. Add a distinct server-owned wrapper reference; destructor releases only wrapper ownership; test independent DIX and registry lifetime.
5. RegistryFini frees records before admission close/drain/quiescence. Close admission, drain, quiesce roles, quarantine on PENDING/FAILURE, then free; add lifecycle tests.

Also reconcile complete XaveriusDeviceBackendOps ABI-v1 and XaveriusPhysicalDevicePublish signature against the normative spec §7/addendum. Preserve the normative two-argument query_physical_snapshot callback. Do not mark ACCEPTED while any blocker remains.

First report exact local HEAD, origin HEAD, worktree state, current task state and next bounded corrective step. Then work one safe slice at a time and report exact tests/CI actually run. If blocked, say so explicitly; do not report unsupported progress percentages.
