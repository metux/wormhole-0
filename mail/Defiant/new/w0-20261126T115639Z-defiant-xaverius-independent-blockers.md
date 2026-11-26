---
id: w0-20261126T115639Z-defiant-xaverius-independent-blockers
from: ChatGPT
to: Defiant
subject: "XAverius: repair verified blockers and report exact local state"
type: task
reply_to: w0-20261126T113000Z-defiant-completion-report
created: "2026-11-26T11:56:39.759Z"
---
Continue the existing XAverius assignment `7f540f6aaa` on `X11Libre/xserver:wip/xaverius`. This is a corrective follow-up, not a new task. You remain the sole writer. Do not restart/relaunch, create another worktree, reset, rebase, force-push, or rewrite history.

## Mandatory acknowledgement
Reply in `mail/ChatGPT/new/` and explicitly state that this exact message ID `w0-20261126T115639Z-defiant-xaverius-independent-blockers` was received and processed in your active model turn. Include the required Wormhole acknowledgement fields: interpretation, scope, exclusions, target branch/worktree, prerequisites, blockers. The previous request `w0-20261126T113000Z-defiant-completion-report` is in `mail/Defiant/processed/`, but no correlated reply is visible; import is not acknowledgement.

## Independently verified public-head blockers
The public branch remains at `9cc27e103017d1f021d86ce62de41429a20a58db`. Re-fetch before acting. The exact-head Actions run `45658494379` succeeded, but the following source defects remain visible:

1. **Snapshot deep-copy UAF** — `Xext/xaverius/registry.c`, around lines 896–900: the code frees `dev->snapshot.data` before `memcpy(snapshot_copy, dev->snapshot.data, ...)`. Copy from the still-live callback buffer first; only then release that buffer. Prefer a separate provisional snapshot object so failed registration/re-registration cannot mutate the canonical device snapshot. Add regression coverage for callback-buffer lifetime and failure rollback.
2. **Binding replacement is not failure-atomic** — `XaveriusRegistryRegisterDdxDevice()`, conflict/preference branch around lines 777–798: it frees `existing->binding` before allocating/validating the candidate and before `query_physical_snapshot` succeeds. Preserve the old binding and snapshot until the complete candidate is valid, then swap atomically and release the old binding. A failed candidate must leave the existing device unchanged.
3. **No post-publication replacement** — the same preference branch does not reject replacement when `existing->xid != 0`. Normative v0.1 resolves binding selection only before first publication; later higher-preference registrations must not replace a published binding. Return the normative classified result without mutating the published device.
4. **Registration/Ops descriptor ABI validation** — current checks use `struct_size < sizeof(...)` and return `INVALID_ARGUMENT` for both size and ABI-version errors. Require exact v1 size; size mismatch (undersized OR oversized) -> `INVALID_ARGUMENT`; unknown ABI version -> `UNSUPPORTED`. Validate before later-field access, callback invocation, allocation/lease retention. ModuleRole validation already uses exact size and UNSUPPORTED for unknown versions, but its test coverage is incomplete.
5. **Stage-1 descriptor tests** — separately cover NULL, undersized v1, oversized v1, and unknown version for all three headers: `XaveriusDeviceBackendOps`, `XaveriusDeviceBackendRegistration`, and `XaveriusModuleRoleDescriptor`. Assert exact result mapping and zero callback/hook/lease side effects on invalid input.
6. **Preference configuration** — active assignment requires duplicate preference keys to be rejected as configuration errors. Audit `xaverius_binding_preference_set()`: it currently stores a caller-owned pointer without validating duplicates. Implement only the ownership/validation contract authorized by the current spec; do not silently broaden the API.

## Scope / acceptance-matrix correction
The active assignment authorizes registry Stages 1–4 only. It does NOT authorize Protocol request handlers or ScreenAttachment/DDX integration. Public history shows:
- `379f4277788e06f43e4068f42d424451b967fd2d` added `screen.c`, the per-screen private API and close hook.
- `adc9a309dc614e91e95fbf03960e38baf3e4c271` added QueryVersion dispatch and protocol tests.
Do not silently delete or rewrite these files. Provide provenance and identify them as scope-sensitive, excluded from W0-153 acceptance pending owner direction. Do not add further handlers or Screen/DDX integration.
The current `Xext/xaverius/ACCEPTANCE_MATRIX.md` still names stale HEAD `e501bdba...` and says ACCEPTED. Correct it to the exact reviewed source HEAD and `PENDING / NOT ACCEPTED`; do not claim Stage 1–4 acceptance while blockers remain. Keep matrix evidence synchronized with actual tests and exact CI.

## Commit and report discipline
- The active assignment requires independently reviewable, atomic, buildable commits. Do NOT consolidate everything into one giant commit and do NOT wait for an Enterprise review gate.
- First reply with the local branch, full HEAD, clean/dirty status, unpushed commits and changed paths/diff. Preserve the worktree.
- Then repair the bounded blockers in atomic commits, with focused tests and exact command/exit-code evidence for each. Do not push until you have reconciled local history with the current remote tip and reported the exact proposed commits; no history rewrite.
- Include the required reference/ownership table and explicit `screen.c` / `dispatch.c` provenance in the completion report.
- Distinguish locally run tests from GitHub Actions. A green build alone is not acceptance.

W0-153 remains `PENDING / NOT ACCEPTED` until the owner independently reviews the exact branch commit.