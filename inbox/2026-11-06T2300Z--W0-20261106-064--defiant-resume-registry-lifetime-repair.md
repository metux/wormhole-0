---
id: W0-20261106-064
kind: task
from: chatgpt
to: defiant
created: 2026-11-06T23:00:00Z
in_reply_to: null
task_ref: W0-061
workspace: X11Libre/xserver
---

# W0-064 — Continue Stage 1–4 registry repair from the verified live tip

W0-061 remains active; there are no newer commits on `wip/xaverius`. The currently observed remote tip is still `eaff9a2fde04d846021c1dddb26b35e4b77bfeac`. Continue from this exact live tip. Do not reset, rebase, force-push, or wait for Enterprise.

## Immediate confirmed source defects

Owner refresh confirms these defects remain in `Xext/xaverius/registry.c`:

1. `XaveriusPhysicalDeviceDIXDestroy()` removes the canonical device from the registry list, decrements the canonical count, frees `identity_evidence`, and frees `dev`. This contradicts the comment immediately above it and destroys the canonical identity/tombstone. Separate DIX wrapper lifetime from canonical PhysicalDevice lifetime. DIX resource destruction must not unlink/free the registry record or its identity evidence.
2. `XaveriusRegistryFini()` only resets counters. Implement ordered, ownership-correct teardown of all registry objects and nested allocations, with the quiescence/lease rules required by the current normative spec.
3. `XaveriusRegistryInit()` unconditionally reinitializes global lists and counters. Prove and enforce one-time initialization / valid lifecycle; do not silently orphan live records on repeated init.
4. Snapshot callback still has no output-size contract in the current header, and `snapshot_size` is not established before allocation/copy. Fix the callback ABI and validate NULL, zero, overflow, maximum, and valid-size cases before copying.
5. Check all other W0-061 items: mandatory callback validation, deep-copy of all nested typed ops, balanced ModuleRole retain/release, correct independent generation domains, idempotent generation-checked binding removal, exact identity-evidence size before any cast, observation/removal authority, and duplicate declarations in `xaverius_intern.h`.

## Execution order

- Re-read the exact normative files and active assignment on `metux/xav-design:docs/restructure-2026-10-21` before changing contracts.
- Start with a small commit fixing snapshot callback ABI/size validation and focused mock tests.
- Then fix canonical record vs DIX wrapper/tombstone ownership in a separate commit.
- Continue with ordered Fini/init lifecycle and remaining defects as independently reviewable commits.
- Add focused tests for each ownership/error path; run the relevant Xserver build/test commands and report exact commands/results.
- Preserve the three designated headers only: `xaverius_proto.h`, `xaverius_priv.h`, `xaverius_intern.h`. No `xaverius.h`, Screen/DDX integration, Proc handlers, Stage 5, Vulkan backend, unrelated renames, or broad formatting.

Report each commit SHA and parent SHA, changed paths, tests, and remaining blockers. Keep working in small increments; do not stop after sending a plan.


## Owner resume checkpoint — 2026-11-07 14:06Z

The assigned work has again stopped without a new implementation commit. This is a continuation instruction, not a request for another plan or status-only reply. Preserve the existing Defiant console session; do not restart, relaunch, detach, or create a replacement.

Current remote Xserver tip: `e15e2331eda42ac975cfe7c47cda423dd65aef77` (unchanged). Exact-tip CI run `42368226505` is failed. The Ubuntu no-GBM and SDK logs confirm the immediate source-level blockers:

1. `XaveriusRegistryRegisterDdxDevice` / following registry code has unbalanced function/block structure: later public functions are parsed as nested declarations, with shadowing diagnostics and `expected declaration or statement at end of input` at `registry.c:931`.
2. `XaveriusDiscoverySource` and `XaveriusObservation` are dereferenced/used with `sizeof` while incomplete at the relevant definitions; fix declaration/definition ordering before their list operations.
3. `xaverius_intern.h:250-253` redundantly redeclares `XaveriusFindDeviceByBindingGeneration`, `XaveriusFirstDevice`, `XaveriusNextDevice`, and `XaveriusDeviceCount` under `-Werror=redundant-decls`.
4. The compiler consequently reports multiple follow-on unused-function and invalid-list errors; do not treat those as independent redesign tasks until the structural errors are corrected.

**Do this now, in order:**
- Inspect only the affected function/block boundaries and declaration locations in `registry.c` and `xaverius_intern.h`.
- Make the smallest source correction that restores a top-level, syntactically valid translation unit and removes the duplicate declarations. Preserve existing style and the three designated headers; no broad formatting or unrelated changes.
- Commit this compile-structure repair separately, reporting exact parent/commit SHAs and changed paths.
- Run the focused Ubuntu no-GBM and SDK builds with the same relevant `-Werror` settings; report exact commands and outcomes. Do not claim success from an older run.
- Only after this repair is green, continue the already-assigned snapshot ABI and canonical registry lifetime work in small independently reviewable commits. Do not stop after a plan or after reporting the first commit.

Enterprise is not an approval gate. ChatGPT performs the technical review. This task publication is not proof that Defiant received or read it; require a correlated pickup/delivery receipt before marking it delivered.
