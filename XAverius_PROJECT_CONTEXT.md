# XAverius project context

This file holds XAverius-specific project knowledge so that the ChatGPT Project instructions can remain project-agnostic. It is a durable index and operating context, not a substitute for refreshing live repository state.

## Repositories and authority

- **Implementation:** `X11Libre/xserver`, working branch `wip/xaverius`.
- **Normative specification:** `metux/xav-design`, branch `docs/restructure-2026-10-21`. This is the authoritative design source; do not substitute the obsolete `wip/001` branch.
- **Implementation skill:** `X11Libre/mpbt-workspace`, branch `mtx/agent-config`, file `.claude/skills/xaverius-implementation/SKILL.md`.
- **Cross-agent communication skill:** same workspace branch, file `.claude/skills/wormhole-communication/SKILL.md`.

Refresh all branches, HEADs, task-board entries, worktree ownership, and the current normative specification before acting. The branch names and task references below are routing context, not proof that a task is still active.

## Ownership and coordination

- **Defiant** is the sole writer for the existing XAverius implementation workstream on `X11Libre/xserver:wip/xaverius`.
- Continue the existing assignment; do not create a duplicate task, start a parallel implementation, or reassign ownership.
- Enterprise may coordinate and report delivery/status, but does not perform XAverius technical audits, code/spec reviews, PASS/BLOCK decisions, or approval.
- The project owner performs independent technical review. Implementation must continue with the next bounded task and must not wait for an Enterprise audit gate.
- Known task reference: `7f540f6aaa`. Verify it against the current task board before relying on it.

## Scope and implementation constraints

- The current normative specification and the XAverius implementation skill define the actual API, stages, acceptance criteria, and implementation order. Read both before changing code.
- Preserve the user's explicit implementation-header decision: the designated Xserver implementation headers are `xaverius_proto.h`, `xaverius_priv.h`, and `xaverius_intern.h`. Do not introduce a duplicate public `xaverius.h` in the Xserver implementation or duplicate protocol definitions unless the owner changes that decision.
- Do not infer API names from historical notes or old code. Resolve names and signatures against the current normative specification.
- Do not expand scope into later stages, Screen/DDX integration, Proc handlers, Vulkan backend work, unrelated renames, or mass formatting unless the current owner-approved task explicitly requires it.
- Keep the implementation in its assigned worktree/branch. No concurrent writers in a shared worktree.

## Historical audit notes

Older audit findings and task history are retained in `SESSION_BOOTSTRAP.md`. Treat them as historical checkpoints only: verify each against the current spec, source, branch HEAD, and active task before treating it as an outstanding defect. Do not report old SHAs or old blockers as current state without rechecking.
