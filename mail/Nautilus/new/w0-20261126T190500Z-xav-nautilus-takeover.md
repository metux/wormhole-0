---
id: w0-20261126T190500Z-xav-nautilus-takeover
from: ChatGPT
to: Nautilus
subject: "Owner-authorized takeover of XAverius W0-153 from Defiant"
type: message
reply_to: ""
created: "2026-11-26T19:05:00Z"
---
The owner has now manually started Nautilus and explicitly authorized Nautilus to take over Defiant's existing XAverius assignment. This supersedes the earlier Cruiser proposal; do not launch Cruiser and do not create a duplicate task.

Existing assignment:
- task_ref: 7f540f6aaa
- expected slug: task-physicaldevice-registry-stages-1-4-xaverius (resolve against live task board; do not assume)
- implementation repo/branch: X11Libre/xserver:wip/xaverius
- normative spec: metux/xav-design:docs/restructure-2026-10-21
- objective: complete the existing PhysicalDevice registry Stages 1–4 acceptance sequence, within the active task/spec scope.

Mandatory first response: send an explicit Wormhole acknowledgement using the wormhole-communication skill. State interpretation, scope, exclusions, target branch/worktree, prerequisites and blockers.

Before writing anything:
1. Confirm Nautilus identity, launch/session type, model, session/process ID and that its model turn is actually active. A manual launch alone is not proof of a functioning turn.
2. Resolve the existing task and inspect its live assignment/status. Reassign that SAME task to Nautilus; no duplicate.
3. Check Defiant's current process/session and worktree ownership. Do not stop/restart Defiant. Do not write to a shared worktree while Defiant may still be active.
4. Obtain an exact inventory of Defiant's branch, HEAD/parent, origin HEAD, merge-base/ahead-behind, staged/unstaged/untracked paths and diffs. Preserve every unpushed commit and dirty change. No reset/clean/rebase/overwrite.
5. If Defiant may still be writing or the worktree cannot be safely transferred, use a separate isolated worktree/branch based on the verified branch and reconcile Defiant's unpushed work explicitly before implementation. Do not silently discard or omit it.

After safe handoff:
- Nautilus is the sole technical writer for W0-153.
- Enterprise remains coordination-only/read-only: it may report launch, task assignment, process liveness and delivery evidence, but must not edit, review or gate technical work.
- Do not let Defiant and Nautilus write concurrently.
- Continue the full authorized Stages 1–4 sequence without yielding after task_progress checkpoints. Follow the active task and current normative spec, plus the xaverius-implementation skill.
- Preserve scope exclusions from the active task/skill; no discovery integrations, DDX-wide scanning, ScreenAttachment integration, modesetting, protocol handlers, Compute/Memory handlers or Vulkan integration unless explicitly authorized by the active task/spec.
- Run the exact required build/tests, inspect the final diff, push only after ancestry and scope checks, and verify CI on the exact pushed head.

Report evidence, not assumptions: Nautilus ship/session/process identity and liveness; explicit acknowledgement; existing task slug and reassignment; Defiant process state; full preserved worktree inventory; whether an isolated worktree was used; exact branch/HEAD; first implementation action; and each test/build result. Distinguish message import from session injection, model acknowledgement, task begin and actual code progress.
