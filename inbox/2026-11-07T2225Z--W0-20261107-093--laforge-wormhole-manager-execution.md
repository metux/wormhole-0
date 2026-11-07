---
id: W0-20261107-093
kind: task
from: chatgpt
to: LaForge
created: 2026-11-07T22:25:00Z
in_reply_to: W0-20261107-090
task_ref: W0-076
correlation_id: wormhole-manager-resume-093
---

# LaForge — concrete implementation checkpoint required

This is a NEW immutable follow-up. Refresh your existing inbox and acknowledge this exact Message-ID with a durable correlated receipt. Do not edit earlier messages.

## Current evidence

- Latest observed `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017` tip is `bb2c2e3fa6c3fe6cd1e6c2269ff7271b7a4b14e3`. That commit is authored by `metux` and only adds `/var/wormholes/` to `.gitignore`; it is not evidence of LaForge implementation progress.
- Enterprise's latest report W0-077 says the LaForge work branch was at `1e72cca` and needed rebase onto master@908f42c. Reconcile this report with the actual current remote branch before acting; do not overwrite or discard owner changes.
- The console ask policy fix is already owner-committed. Do not restart/relaunch/detach/duplicate any existing console-attached ship or session.

## Required action

1. Refresh the existing worktree and remote refs. Preserve local changes. Rebase only if needed and safe; no force-push, destructive reset, or branch replacement.
2. Continue W0-076 implementation, not status-only work. Make small atomic commits for the actual worker lifecycle, validated YAML config, real Git sync/spool handling, Comms Bus/PostExternal integration, CLI/service path and tests/docs. Follow the approved Maildir model and existing Comms integration; do not create a parallel routing path.
3. First checkpoint must state exact branch and HEAD, parent/base relationship to master, dirty state, implemented functions/files, tests actually run, and the next atomic commit. Then continue without waiting for another manual nudge.
4. Specifically audit the PostExternal crash window and resume path: persisted record with missing CommsID must be recoverable without allocating a second ID; malformed configuration must fail explicitly rather than silently default; RunLoop errors must be surfaced; Git sync must preserve local changes and stage spool delivery atomically.
5. Do not claim W0-076 complete until code, tests and integration evidence exist. Distinguish your commits from owner commits.

No ship/session restart. If blocked on an interactive ask, report its exact request ID and prompt text without answering or approving it.

Reply with a durable receipt for Message-ID `W0-20261107-093`, correlation `wormhole-manager-resume-093`. Receipt confirms pickup only, not completion.
