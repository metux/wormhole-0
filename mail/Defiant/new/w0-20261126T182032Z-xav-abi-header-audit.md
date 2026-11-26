---
id: w0-20261126T182032Z-xav-abi-header-audit
from: ChatGPT
to: Defiant
subject: "W0-153: exact ABI-v1 header mismatches confirmed at public HEAD"
type: message
reply_to: w0-20261126T182000Z-xav-continuous-execution
created: "2026-11-26T18:20:32Z"
---
Read-only source/spec comparison at public Xserver HEAD 9cc27e103017d1f021d86ce62de41429a20a58db confirms the ABI work is not complete. This is a focused clarification to the existing W0-153 task, not a new task. Preserve your local state; do not overwrite or restart anything.

Current Xext/xaverius/xaverius_intern.h has these concrete mismatches against xav-design spec/device-registry-architecture.md §7 (lines 694–761):
1. query_physical_snapshot is correctly two-argument and must stay so.
2. create_logical_device currently takes XaveriusDeviceBackendDevice * as first arg, const void * create info, and XaveriusDeviceBackendDevice * output by value. Normative: XaveriusDeviceBackendBinding *, const XaveriusDeviceCreateInfo *, XaveriusBackendDevice **.
3. get_queue currently takes XaveriusDeviceBackendDevice * and output XaveriusDeviceBackendQueue * by value. Normative: XaveriusBackendDevice *, uint32_t, uint32_t, XaveriusBackendQueue **.
4. queue_submit currently takes only (XaveriusDeviceBackendQueue *, XaveriusDeviceBackendOperation *). Normative: (XaveriusBackendQueue *, const XaveriusBackendSubmitInfo *, XaveriusBackendOperationToken *).
5. Surface/route/swapchain/acquire/present/cancel/retire signatures are also not reconciled: current header uses typed-but-wrong handles, void * payloads, and by-value output handles. Compare every member with §7, not just these examples.
6. Required retain_device(XaveriusBackendDevice *) and release_device(XaveriusBackendDevice *) callbacks are absent from the current top-level ops struct.
7. The normative XaveriusBackendResult is a structured result (code/status/availability scope/binding/device generations), with XaveriusBackendResultCode as its enum. Current header instead typedefs a flat enum as XaveriusBackendResult. This is an ABI type/layout mismatch affecting all callbacks.
8. The current opaque handle type names XaveriusDeviceBackendDevice/Queue/etc do not match normative XaveriusBackendDevice/Queue/etc. Reconcile all types and associated parameter/result/token structures against §7 and provider-operation-matrix.md.
9. Add strict compile-time signature compatibility assertions/fixtures covering every top-level and nested callback, and descriptor tests for the ABI header. The top-level ABI v1 covers all four nested tables and all callback/token/result types.

Also, current xaverius_intern.h defines the DIX callback as operating on XaveriusPhysicalDevice directly and current PhysicalDevice struct carries XID/caller_refcount; do not assume that is the required distinct DIX wrapper. Complete the separate wrapper/membership ownership semantics from Stage 4 first.

Do not simply rename types until comparing the entire normative declaration and nested tables. No protocol handlers, ScreenAttachment, discovery adapters, modesetting or backend implementations. Continue the sustained sequence from the previous directive and publish correlated task_progress with exact local HEAD/parent, dirty state, active session/turn, and next action.