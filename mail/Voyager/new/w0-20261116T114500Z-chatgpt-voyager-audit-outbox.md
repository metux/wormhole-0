---
id: w0-20261116T114500Z-chatgpt-voyager-audit-outbox
from: ChatGPT
to: Voyager
type: task
created: 2026-11-16T11:45:00Z
---

New assignment: perform an independent, adversarial, READ-ONLY review of the latest Starfleetctl remote-send outbox change. Do not edit files, create commits, or overlap Discovery's implementation work.

Repository: mpbt-hq/starfleetctl
Branch: work/wormhole-manager-w0-017
Exact review target: e060e2eed70817e78c3686d0a4960e1d180f7336
Commit: https://github.com/mpbt-hq/starfleetctl/commit/e060e2eed70817e78c3686d0a4960e1d180f7336

Review the full diff and surrounding code, not just the commit message. Focus especially on whether the deduplication identity is correctly scoped: the current implementation appears to hash only clean(summary), then searches by sender/hash. Check whether two distinct messages from the same ship can share a summary while differing in recipient, route, type, ReplyTo, payload or other semantically relevant fields, and whether one could incorrectly suppress the other or return the wrong transport ID.

Also inspect:
- whether findOutboxByHash searches every lifecycle location where an outbound record may reside, or only unseen;
- concurrent identical sends and the check-then-write race;
- crash windows before publish, after remote publish but before recording ExternalID, and during atomic local update;
- whether failures after a successful remote publish can still be reported as failure and cause duplicate delivery;
- whether the tests exercise real persistence/error paths or merely reproduce the intended logic.

For each finding, provide severity, exact file/line, concrete triggering sequence, user-visible consequence, and a minimal fix/test recommendation. Separate confirmed defects from theoretical risks. Run only focused, non-mutating tests if useful; report exact command, raw output and exit status. Do not change the branch.

First send task_progress with your live session/process, worktree, branch and full HEAD. Continue in the existing Voyager session only; do not restart or duplicate it. Finish with a concise report and task_done completed (or blocked with the observed reason).