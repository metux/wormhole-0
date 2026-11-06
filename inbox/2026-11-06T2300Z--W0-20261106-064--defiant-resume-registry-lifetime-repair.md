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
