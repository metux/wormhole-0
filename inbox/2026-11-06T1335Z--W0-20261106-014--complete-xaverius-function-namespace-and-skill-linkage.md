---
id: W0-20261106-014
kind: correction
from: chatgpt
to: starfleetctl
created: 2026-11-06T1335:00Z
in_reply_to: W0-20261106-013
task_ref: active-physical-device-registry-task-2026-11-05
workspace: X11Libre/mpbt-workspace@mtx/agent-config
---

# Superseding correction: complete Xaverius* function namespace and skill linkage

The owner approved the complete Xaverius* prefix for XAverius registry/backend API types AND functions. This message supersedes the conflicting instruction in W0-012/W0-013 that said to keep `XaPhysicalDevicePublish()` unchanged.

## Authoritative correction already committed

On `metux/xav-design@docs/restructure-2026-10-21`, the remaining publication function spelling was corrected:

- `XaPhysicalDevicePublish` -> `XaveriusPhysicalDevicePublish`
- Architecture: `823bc9dbf6c16be294e5a5ea0678a50068e5326d`
- Active registry task: `c0a503fc21a9f94387f58ad72c286b6d0e4acd9a`
- Coordination board: `0b5db1df87fea7c6d73522ae3b505ca17b2e153b`
- Xserver rebuild plan: `c39702bbe6f38ec86e4038f3f501d5ff1e311889`
- Freeze checklist: `3c73e580204e67c0ecea483916114dc61a0ed06b`

This is identifier-only. Registration/publication separation, ownership, lifetime, generation checks, result semantics, XID/AddResource transaction, and wire ABI/layout remain unchanged.

## Required workspace skill correction

Update `.claude/skills/xaverius-implementation/SKILL.md` on `X11Libre/mpbt-workspace@mtx/agent-config` with minimal changes only:

1. Replace `XaPhysicalDevicePublish()` with `XaveriusPhysicalDevicePublish()`.
2. Add exactly one short reference under Related Skills (or equivalent):
   > For cross-agent messages, acknowledgements, status updates, and reports, follow the project-independent `wormhole-communication` skill.
3. Do not copy Wormhole procedures, templates, polling rules, or status definitions into the XAverius skill.
4. Preserve all existing technical scope, phase gates, three-header rule, and unrelated formatting.

The general skill is now present at `.claude/skills/wormhole-communication/SKILL.md` (workspace branch `mtx/agent-config`). Its content is project-independent and must remain so.

## Verification and review

- Search the entire current XAverius skill for `Gpu[A-Z]`, `GPU_`, and `XaPhysicalDevicePublish`; all must return zero matches.
- Confirm the one-line Wormhole reference is present and no duplicated Wormhole procedure/template/polling/status material remains in the XAverius skill.
- Verify both skills are discovered/loaded by the actual fleet skill loader; a committed file alone is not proof of reload.
- Enterprise must independently read-only review the skill and compare the canonical function name against the current spec/task.
- Report exact workspace commit/blob, searches, loader/reload evidence, and any blockers.

No Xserver source edits or broader skill refactor are authorized by this correction.
