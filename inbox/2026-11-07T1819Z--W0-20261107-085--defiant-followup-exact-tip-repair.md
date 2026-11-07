---
id: W0-20261107-085
kind: task-followup
from: owner-coordination
to: defiant
created: 2026-11-07T18:19:00Z
in_reply_to: W0-20261107-075
task_ref: XAverius PhysicalDevice registry / W0-016 §6
workspace: X11Libre/xserver@wip/xaverius
---

# W0-085 — New immutable follow-up: exact-tip blockers and CI failure

This is a NEW message. It supplements W0-075; it does not edit, erase, or silently replace that original assignment. Read this entire message even if W0-075 was already acknowledged.

## Latest owner-verified evidence

At the fetched branch tip `80b411f6032c6176303039d94dcf2615520087e9`:

1. `XaveriusRegistryRegisterDdxDevice()` is split by a duplicate comparator definition; the translation unit is malformed.
2. The exact-replay path reads `binding->binding_generation` before `binding` is allocated. Replay must compare the existing live registration against immutable supplied ownership data and allocate nothing.
3. `xaverius_create_ordering_key()` returns one mutable static `key_buf[128]`, so registrations overwrite each other's keys; it silently truncates longer keys.
4. The new-registration success path appends/returns `dev` without assigning `dev->binding = binding`.
5. Snapshot callback use does not populate `snapshot_size`, although later code checks/copies it. Do not copy an unknown-size object.
6. Post-`XaveriusModuleRoleRetain(role)` failure paths do not consistently release the role; identity-evidence allocation failure is one observed leak path.
7. DIX destruction still unlinks/frees canonical registry state, contrary to the tombstone lifetime rule.
8. RegistryFini's complete tombstone-drain/ordered-shutdown behavior remains unproven.

Exact-tip Actions run [42390719664](https://github.com/X11Libre/xserver/actions/runs/42390719664) completed **failure** for that HEAD. Multiple build jobs failed and conformance jobs were skipped. Do not report CI as green or infer a particular diagnostic without reading the relevant job logs.

The authoritative implementation note on `metux/xav-design:docs/restructure-2026-10-21` has also been refreshed with these findings. Treat the normative spec and that active task as authority.

## Required immediate execution

- Refresh the actual remote tip and current CI before editing.
- Repair the translation-unit/control-flow and type/API blockers first, narrowly and in a buildable commit.
- Do not read a provisional binding before allocation; exact replay must not allocate.
- Resolve owner_key only through the ModuleRole-owned model; do not invent registration fields.
- Do not silently truncate canonical ordering keys; implement the normative normalized tuple with correct owned lifetime.
- Establish correct binding ownership on successful registration.
- Keep snapshot ABI, tombstone/DIX lifetime redesign, and publication redesign in separate reviewable commits.
- Run focused tests, Ubuntu and SDK `-Werror` builds, and inspect the exact new workflow's jobs. Record commands, exit statuses and full commit/parent SHAs.
- Continue through the already-authorized registry stages without waiting for another manual nudge, stopping only at a concrete blocker requiring owner input.

## Acknowledgement and reporting

Acknowledge this exact message ID `W0-20261107-085` through the durable correlated receipt path. Then provide a progress checkpoint with current HEAD, changed files, actual test results, and next action. If blocked by an Ask, preserve its exact prompt text and request/correlation ID; do not approve it.

Do not restart, relaunch, detach, duplicate or replace the existing console-attached Defiant session. No force-push, history rewrite, broad cleanup, Stage 5, ScreenAttachment, discovery, protocol handlers or backend work.
