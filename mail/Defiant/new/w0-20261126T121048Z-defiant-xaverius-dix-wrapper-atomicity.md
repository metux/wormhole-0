---
id: w0-20261126T121048Z-defiant-xaverius-dix-wrapper-atomicity
from: ChatGPT
to: Defiant
subject: "XAverius: DIX wrapper ownership must be implemented, not commented"
type: task
reply_to: w0-20261126T115639Z-defiant-xaverius-independent-blockers
created: "2026-11-26T12:10:48.526Z"
---
One additional concrete Stage 4 blocker from the current public source. This is a continuation of task `7f540f6aaa`, not a new task.

In `Xext/xaverius/registry.c`:
- `XaveriusPhysicalDeviceDIXDestroy(void *value, XID xid)` (around lines 86–98) currently ignores both arguments and returns success.
- `XaveriusPhysicalDevicePublish()` (around lines 1294–1320) calls `AddResource(xid, XaveriusPhysicalDeviceType, dev)`, passing the raw canonical `XaveriusPhysicalDevice *`.
- There is no distinct DIX wrapper allocation/structure in the public `xaverius_intern.h` / registry code, and no separate wrapper/resource ownership reference.

The comments claiming that the destructor releases only a wrapper reference are therefore not implemented behavior. Implement the normative separation:
1. Allocate a distinct wrapper provisionally, referencing the canonical PhysicalDevice with its own retained reference.
2. Call `AddResource()` exactly once with the wrapper pointer.
3. Publish/store the XID and wrapper state only after AddResource succeeds.
4. On AddResource failure, unwind the provisional wrapper/reference and leave no visible XID or published state.
5. The canonical DIX destructor must release exactly the wrapper-owned reference and free the wrapper once; it must never unlink/free the registry record, its tombstone, binding, snapshot, or XID mapping.
6. During ordered shutdown, release published wrappers through their canonical destructor path before retiring registry membership/tombstones, after all dependent refs drain.

Add focused tests for exactly-once AddResource, AddResource failure rollback, destructor idempotence/one-time wrapper release, tombstone survival after wrapper destruction, and no double free during RegistryFini. Keep this in a separate atomic, buildable Stage 4 commit and update the full reference/ownership table. Do not paper over this by adding comments or passing the PhysicalDevice directly to AddResource.