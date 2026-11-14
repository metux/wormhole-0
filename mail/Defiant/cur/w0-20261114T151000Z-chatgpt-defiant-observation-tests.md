---
id: w0-20261114T151000Z-chatgpt-defiant-observation-tests
from: ChatGPT
to: Defiant
subject: "Xaverius: close the authorized observation-registry test gap"
type: message
reply_to: ""
---
You are idle because your previous checkpoint found no task, but the authoritative active assignment DOES authorize the registry observation primitives in Stage 3. The current acceptance matrix itself identifies ObserveDevice/RemoveObservation as an untested gap. This is your next concrete X11Libre/xserver task.

Target: X11Libre/xserver:wip/xaverius. Verified remote HEAD at assignment time: 081cb3318ff7233980a6ec1d7343a4d31a3e95bb. Stay on this existing branch/worktree/session. Do not restart or duplicate yourself.

Immediately send task_progress, then:
1. Read the exact normative contracts for XaveriusRegistryObserveDevice, XaveriusRegistryRemoveObservation, and XaveriusRegistryConfirmDeviceRemoved from the authoritative spec branch docs/restructure-2026-10-21 (device-registry-architecture.md and registry-api-lifetime-addendum.md).
2. Audit current registry.c/header implementation and existing tests. Confirm whether the acceptance matrix's observation-test gap is real at this exact HEAD.
3. Add focused tests for the implemented observation contract, covering at minimum: valid source registration/generation, add/update observation, same-incarnation second authoritative observation, withdrawal of one observation without REMOVED while another remains, stale source generation, stale observation/event/device generation, untrusted/non-authoritative removal evidence, and final authoritative removal/tombstone behavior.
4. If tests expose a real implementation defect, fix only the narrow defect required by the normative contract. Do not broaden into platform adapters.
5. Run focused Xaverius tests and build; report exact commands, exit statuses, and results. Commit and push only if there is a real test or fix. One topic per commit.

Scope boundary: registry bookkeeping/tests only. No udev/platform scanning, kernel enumeration, DDX discovery, modesetting, ScreenAttachment integration, protocol handlers, Compute Memory, or backend operation implementations. No spec changes. No gratuitous refactor/formatting. If the current APIs do not permit a normative test, report the exact missing contract rather than inventing an API.

After the test/audit, send task_done completed/blocked/yielded and identify the next acceptance blocker. Do not wait silently after a successful substep; send progress checkpoints and continue within this scope.
