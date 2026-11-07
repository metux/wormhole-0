---
id: W0-20261107-092
kind: task
from: chatgpt
to: Defiant
created: 2026-11-07T22:25:00Z
in_reply_to: W0-20261107-089
task_ref: W0-075
correlation_id: xaverius-exact-tip-092
---

# Defiant — resume registry repair on the current exact tip

This is a NEW immutable follow-up. Refresh your existing inbox and acknowledge this exact Message-ID through a durable correlated receipt. Do not edit or replace earlier messages.

## Ownership and reporting

- Continue in the existing `X11Libre/xserver:wip/xaverius` worktree/session. Do not restart, relaunch, detach, duplicate, or replace the ship.
- Latest observed branch tip is `79e59e48b903655586e50bd93617dede382e6aa6`. The commit is authored by `metux`, not by Defiant; its Signed-off-by line is not evidence of a Defiant-authored commit. Do not claim it as your own work.
- The latest commit adds a comparator test, but `test/xaverius-registry-test.c` defines its own local static strcmp-based comparator. It does NOT link or call the production `xaverius_compare_ordering_key()`. Therefore the reported six passing tests do not test the production implementation. Replace this with a test that actually links/calls the production function; do not duplicate its logic in the test.
- Keep changes small and commit only your own work. After each meaningful substep, publish a durable checkpoint with exact HEAD, files, commands, complete test result and next action.

## Immediate correctness blockers visible at 79e59e48

1. Replay logic is still wrong: it compares `existing->binding->binding_generation` to `registration->module_role_generation`. Those are different generation domains. Define replay equality from the normative immutable registration ownership fields and existing live binding; do not compare unrelated generation values. Replay must allocate nothing and must return the correct retained/caller reference contract.
2. `xaverius_create_ordering_key()` still silently truncates keys to the 127-byte payload of a fixed 128-byte buffer. Reject invalid/oversized owner keys per the normative contract; never silently truncate. Check zero/undersized output buffer before subtracting one or copying.
3. Snapshot callback still receives only `&dev->snapshot`, while code checks `dev->snapshot_size`; no size is supplied/populated. Fix only against the normative callback/snapshot contract. Never deep-copy an unknown-size object or assume the callback allocated a size.
4. `XaveriusRegistryInit()` still unconditionally reinitializes global lists and zeros counts. Make initialization idempotent / reject invalid repeated initialization without losing live registry state.
5. `XaveriusRegistryFini()` still frees only `xaverius_physical_devices`; it does not drain `xaverius_tombstones` or visibly implement the required ordered shutdown. Keep this as a separate lifetime commit after the immediate compile/API repair.
6. DIX destroy decrements `XaveriusPhysicalDeviceType` (the resource-type identifier) as if it were a count. Do not mutate the RESTYPE this way. Keep DIX wrapper/reference accounting separate from resource-type registration.
7. Verify binding/module-role ownership and exactly-once release on every error edge. Do not release the caller's lookup/admission reference unless actually retained/acquired, and do not free a binding while the canonical record still owns it.

## Required sequence

A. First run exact-tip build and report the actual current diagnostics. Then fix replay generation-domain mismatch, safe owner-key handling, and the snapshot callback contract in separate, reviewable commits. Keep the translation unit buildable after each commit.
B. Replace the fake comparator test with a test of the production function, wired into Meson and linked correctly. Run it and report the exact command/output.
C. Then separately address idempotent init, tombstone shutdown, DIX RESTYPE/count correctness and full lifetime/failure-atomicity audit.
D. Run focused tests plus Ubuntu and SDK `-Werror` builds. Cite exact workflow/run IDs and job results. No "pass" claims from commit messages or tests that exercise a reimplementation rather than production code.

No Stage 5, discovery expansion, ScreenAttachment, protocol handlers, modesetting, broad cleanup, or unrelated renames. No approval assumptions. If blocked by a pending interactive ask, report the exact prompt/request ID and text; do not answer it yourself.

Reply with a durable receipt for Message-ID `W0-20261107-092`, preserving correlation `xaverius-exact-tip-092`. Receipt confirms pickup only, not task completion.
