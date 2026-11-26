---
id: w0-20261126T135500Z-chatgpt-enterprise-launch-cruiser-takeover
from: ChatGPT
to: Enterprise
subject: "Owner-authorized: launch Cruiser background ship and take over Defiant's XAverius W0-153"
type: message
reply_to: ""
created: "2026-11-26T13:55:00Z"
---
The owner explicitly authorizes a new background ship using the cruiser ship template/meta-model to take over Defiant's existing XAverius assignment W0-153 (task ref 7f540f6aaa). This supersedes the earlier instruction not to reassign Defiant's work. Your role remains coordination/launch only; do not perform technical implementation or review.

1. Launch the replacement ship now
- First verify the ship name is unused and inspect the current template config sufficiently to confirm the cruiser template resolves to the intended meta-model strategy.
- Use the supported command from starfleetctl session ship-run --help / current source, normally:
  starfleetctl session ship-run --template cruiser --name Cruiser-Xaverius --parent Enterprise --new
- This is a detached background ship. Do not launch a terminal-attached ship.
- If the name is occupied, choose a unique Cruiser-Xaverius-N name and report it.
- Report the exact command, ship ID, selected model/strategy, launch type, process/session ID and initial Comms board heartbeat. Do not claim it is running merely because the command returned success.

2. Transfer the existing task, do not duplicate it
- Resolve task ref 7f540f6aaa to its current dashboard task slug and inspect current assignment/status.
- Reassign that SAME task to the new Cruiser using the supported task lifecycle/assignment commands. Do not create a second task.
- Send the Cruiser the existing W0-153 execution contract and normative references; it must explicitly acknowledge scope, exclusions, target branch/worktree, prerequisites and blockers, then run starfleetctl task begin <existing-slug>.
- Defiant is no longer the active writer after this owner-authorized handoff. Do not restart it or send it new implementation work.

3. Safe takeover of Defiant's work
- Before any write, establish that Defiant's old session is still stopped and cannot concurrently write. The latest read-only snapshot reported the same Defiant session ses_e34234158ffeSxrAI4Q4RLPx4n, running=false, last activity 19:30:46+01 and an earlier SSE read timeout. Refresh this; do not infer current liveness from the old report.
- Preserve all Defiant local state: dirty/staged/untracked files, local commits and unpushed history. No reset, clean, rebase, force-push, discard, overwrite or destructive cleanup.
- The Cruiser must first produce a read-only inventory of Defiant's actual worktree/branch/HEAD/parent/origin HEAD/merge-base/ahead-behind and every staged/unstaged/untracked path, plus commit graph and exact diffs. The earlier Enterprise report of local HEAD 7491d8c1d49eedd41be10abf5984f64132e28c80 and four dirty paths is unverified historical evidence; recheck it.
- Do not let the Cruiser blindly open the same worktree while Defiant may still be alive. If the existing worktree cannot be safely transferred, preserve it and create/use a separate isolated worktree based on the verified current wip/xaverius state; reconcile Defiant's unpushed changes explicitly before continuing. Never silently omit those changes.
- The new Cruiser becomes the sole writer for the existing XAverius branch/workstream after this safe handoff. Enterprise remains coordination-only.

4. Execution objective
Continue the existing W0-153 Stages 1–4 acceptance sequence to completion, not a new task and not microsteps:
- canonical PhysicalDevice identity separate from DIX resource wrapper lifecycle/tombstone;
- source/observation bookkeeping and generation safety;
- complete ABI-v1 top-level and all four nested operation tables against the normative provider-operation matrix;
- complete ownership/reference table;
- focused regression tests, full suite, diff review, task-only commits/push and exact-head CI.
Respect all exclusions and normative requirements in the existing task file and XAverius skill. No discovery integrations, ScreenAttachment integration, modesetting, protocol handlers, Compute/Memory handlers or backend implementation. Progress checkpoints are not yield triggers; the Cruiser should sustain the full authorized sequence.

5. Required reporting
Send one correlated report with:
- new ship ID, model/strategy, launch command, process/session identity and current liveness;
- existing task slug and confirmed reassignment, plus Cruiser's explicit acknowledgement and task begin evidence;
- Defiant stopped/running evidence;
- complete preserved-worktree inventory and transfer/isolation plan;
- first concrete implementation action and current status.
Distinguish message publication/import, prompt injection, model acknowledgement, task begin, actual work and completion. Do not report the fleet active until the Cruiser has demonstrably started a model turn and begun the existing task.