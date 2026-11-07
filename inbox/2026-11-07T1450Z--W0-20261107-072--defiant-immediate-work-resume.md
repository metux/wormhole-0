---
id: W0-20261107-072
kind: task
from: owner-coordination
to: defiant
created: 2026-11-07T14:50Z
priority: urgent
workspace: X11Libre/xserver
branch: wip/xaverius
correlation_id: W0-20261107-072
supersedes: none
---

# Immediate resume — no idle/status-only cycles

The owner reports that you have again stopped making progress. This is an execution escalation, not a change to the technical assignment.

Continue the existing active PhysicalDevice registry task in the existing console-attached session and worktree. Do not restart, relaunch, detach, duplicate, or replace the ship/session.

## Do this now

1. Read the latest authoritative assignment and current source branch tip. The known tip at escalation time is `e15e2331eda42ac975cfe7c47cda423dd65aef77`; verify before editing.
2. Inspect the exact current compiler failures already recorded in the active task. Do not spend another cycle repeating status checks or restating known defects.
3. Make the smallest real source correction for the immediate compile/control-flow/type/declaration-order blockers (the already-defined first commit boundary). Preserve scope: no snapshot/lifetime/publication redesign in this repair commit; no discovery, ScreenAttachment, protocol handlers, or broad cleanup.
4. Run the exact Ubuntu and SDK `-Werror` builds. Fix failures caused by this commit until both pass or a concrete external blocker is proven.
5. Commit the repair and push the existing branch. Record full commit and parent SHAs, exact changed files, commands, exit codes, and CI run URL.
6. Continue to the next authorized registry stage after a successful checkpoint. Do not stop merely because one commit or build succeeded.

## Anti-idling rule

A `git status`, branch listing, progress claim, repeated audit, or statement that you are “working” is not a checkpoint. A checkpoint must identify a tangible source diff, commit SHA, test result, or exact blocker. After each successful substep, proceed immediately to the next substep without waiting for another prompt. If no protected question or external dependency blocks you, make the next code change now.

If genuinely blocked, provide the exact blocking condition and preserve the exact prompt/request ID and text. Do not answer or approve protected questions yourself. Keep all existing work intact.

A task file being present in Wormhole is not proof you received it. Acknowledge this task with its correlation ID through the established durable receipt path once received, then continue execution.
