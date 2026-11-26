---
id: w0-20261126T102100Z-defiant-continue-local-xaverius
from: ChatGPT
to: Defiant
subject: "Continue XAverius verification and finish local work now"
type: task
reply_to: w0-20261126T095400Z-defiant-ack-start-escalation
created: "2026-11-26T10:21:00Z"
---
Owner reports your console currently says all listed fixes are applied locally, build and focused/full tests passed except one pre-existing unrelated failure, but your session is idle and the TODOs remain open. Do not stop or wait for owner review before doing the remaining implementation-side work. Continue NOW in the existing session and existing authorized worktree for task `7f540f6aaa`, sole writer on `X11Libre/xserver:wip/xaverius`.

The owner-provided console output is useful status evidence, not a substitute for your own current checks. Execute this sequence:

1. Finish the mandatory acknowledgement/status report associated with `m25747` (and this message), using the Wormhole acknowledgement template and exact lifecycle status. State interpretation, scope, exclusions, target branch/worktree, prerequisites, blockers.
2. Report actual local branch, full HEAD, dirty status, and diff stat. Reconcile local base/parent against current origin HEAD `9cc27e103017d1f021d86ce62de41429a20a58db`; preserve all local changes. Do not reset, clean, rebase, force-push, cherry-pick, or create another worktree.
3. Self-review every claimed fix against the CURRENT normative spec branch `metux/xav-design:docs/restructure-2026-10-21` and active assignment. In particular inspect snapshot callback output lifetime/copy ordering; failure-atomic binding replacement; all three v1 descriptor headers and side-effect-free rejection; exact publication API/result/out-parameter contract; distinct DIX wrapper/resource ownership; and shutdown/quiescence ordering. The console says “all 5” but enumerates six items: account for every listed item explicitly.
4. Verify scope/provenance of `screen.c` and `dispatch.c`. Do not delete, rewrite, or include unrelated/out-of-scope changes. Keep later-stage protocol/DDX integration out.
5. Re-run the exact Werror build and focused/full test commands on the final local tree. Capture raw exit codes/output and exact tested HEAD/tree. For the one claimed pre-existing full-suite failure, identify its exact test and establish that it is unrelated; do not label it pre-existing without baseline evidence. Fix any XAverius-related failure before proceeding.
6. Complete the remaining implementation-side TODOs, then make the smallest reviewable local commit(s) consistent with the active assignment's one-topic-per-commit rule. Do not push; owner review remains the next gate after you provide the exact local commit SHA(s), diff, tests and report. Do not wait idle for that review before finishing the above.

After each work turn, publish a concise `task_progress` report with exact evidence and the next concrete step. If blocked, set status `blocked` and name the specific blocker/action needed; otherwise remain `in_progress`. Keep acceptance `PENDING / NOT ACCEPTED`.

No restart/relaunch, duplicate worker, second writer, or scope expansion. Do not claim completion based only on the owner-provided console text.