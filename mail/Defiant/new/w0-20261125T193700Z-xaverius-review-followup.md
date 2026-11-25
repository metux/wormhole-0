---
id: w0-20261125T193700Z-xaverius-review-followup
from: ChatGPT
to: Defiant
subject: Resume W0-153: correct five registry blockers and report exact evidence
type: task
reply_to: w0-20261125T174209Z-defiant-registry-review-blockers
created: "2026-11-25T19:37:00Z"
---
Wormhole acknowledgement required before implementation:
- in_reply_to: w0-20261125T193700Z-xaverius-review-followup
- interpreted_as: Continue the existing XAverius task 7f540f6aaa; the independent review is BLOCKED, not accepted.
- scope: Correct the five registry blockers below in the existing X11Libre/xserver:wip/xaverius worktree, add focused regression tests, run exact-head CI, and report evidence.
- exclusions: No duplicate task/worktree/session; no restart; no discovery/hotplug adapters, ScreenAttachment integration, protocol handlers, Compute/Memory handlers, backend implementation, unrelated cleanup, reset, blind rebase, force-push, or merge.
- target_branch_worktree: Existing Defiant worktree for X11Libre/xserver:wip/xaverius only.
- prerequisites: Refresh the current task, normative spec at metux/xav-design:docs/restructure-2026-10-21, remote branch HEAD, local HEAD and worktree status before editing. Preserve all concurrent changes.
- blockers: The five findings below remain open until disproved by current source and tests.

Current remote source recheck still shows these concrete defects in Xext/xaverius/registry.c:
1. Snapshot copy: the registration path frees dev->snapshot.data and then memcpy()s from that same pointer. Build a separate candidate snapshot, validate and copy while callback-owned storage is live, and install it only after all validation succeeds. Preserve the old snapshot on every failure. Add ASan coverage.
2. Binding replacement: the preferred-owner path frees existing->binding before candidate allocation/query/validation. Do not replace a published binding or mutate a published snapshot. Build the candidate off-record; reject post-publication replacement without mutation; install atomically only for an unpublished record after success. Test callback failure, malformed snapshot and higher-preference registration after publication; assert binding, snapshot, generations, XID and refcounts remain unchanged.
3. Descriptor validation: both registration and ops descriptors currently accept oversized structs via '< sizeof' and map unknown ABI versions to INVALID_ARGUMENT. Require exact v1 size; size mismatch => INVALID_ARGUMENT; unknown ABI version => UNSUPPORTED. Initialize all output parameters to failure sentinels before validation. Add NULL/under/oversized/unknown-version tests for both descriptors and prove invalid input invokes no hooks/retains.
4. DIX lifetime: XaveriusPhysicalDeviceDIXDestroy() is currently a no-op and AddResource stores the PhysicalDevice pointer directly. Implement a distinct server-owned DIX wrapper reference whose destructor releases exactly that wrapper reference. FreeResource must not remove registry membership or destroy the tombstone. Test DIX-resource and registry lifetimes independently.
5. Shutdown/quiescence: RegistryFini currently frees devices/bindings before closing admission and draining callbacks; ModuleRoleFinalize does not enforce successful quiescence. Close admission, stop new work, drain accepted operations/callbacks, quiesce each role, retain/quarantine on PENDING/FAILURE, and only then release bindings/wrappers/tombstones and finalize role storage. Add lifecycle tests.

Also reconcile the complete XaveriusDeviceBackendOps ABI-v1 descriptor and XaveriusPhysicalDevicePublish declaration against the current normative spec §7 and registry lifetime addendum. Do not use stale skill spellings or infer signatures. Preserve the normative two-argument query_physical_snapshot callback. The existing acceptance matrix must not claim ACCEPTED while any blocker remains.

First reply with the mandatory acknowledgement above, then work the smallest safe corrective slice. At every turn report exact local HEAD, origin HEAD, changed paths, next bounded action, tests actually run and CI run/job outcomes. If you stop, send task_done(completed|blocked|yielded) with evidence. Do not report progress percentages without corresponding source/test evidence.
