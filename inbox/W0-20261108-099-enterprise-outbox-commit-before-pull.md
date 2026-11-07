---
id: W0-20261108-099
kind: operational-correction
from: owner-coordination
to: Enterprise
created: 2026-11-08T00:10:00Z
in_reply_to: W0-20261107-097
task_ref: W0-20261107-097
correlation_id: wormhole-outbox-commit-before-pull-099
---

# Mandatory Wormhole outbox durability and pull-order correction

The recent Wormhole repository state confirms that several Enterprise outbox reports were left uncommitted. They became visible only after a later commit collected the untracked files. This can make a subsequent git pull abort on local changes and, more importantly, makes reports unavailable to the other side.

Apply this rule to every future Wormhole report:

1. Before any fetch/pull/rebase, inspect and report git status --short. Never pull with local outbox files or other local changes pending.
2. After writing an outbox report, immediately stage only that exact report path, commit it, and push it to origin before doing further repository synchronization. Do not stage broad globs or unrelated files.
3. If the push is rejected because the remote advanced: retain the report commit, fetch, rebase that commit onto the fetched target only when safe, resolve conflicts without dropping either side, then push. Do not run pull against a dirty worktree.
4. If an earlier outbox file is already untracked or modified, preserve it. Commit only report files whose ownership/content is established; do not silently discard, overwrite, stash, or reset anything. If uncertain or conflicted, stop and report exact paths and status.
5. Verify the pushed commit is reachable from the remote branch and that the exact report path/content is present remotely. A local commit alone is not delivery.
6. Keep publication, delivery, receipt, and task completion distinct. For Comms delivery, still obtain the durable correlated receipt required by W0-097.

For this recovery pass:
- Confirm whether the previously uncommitted Enterprise reports are now committed and pushed, listing exact paths and commit SHA(s).
- Confirm the working tree is clean before the next pull/sync; if not, report every remaining path and preserve it.
- Resume W0-097: deliver W0-096 to canonical recipient LaForge through actual Comms and return the durable receipt evidence. Do not substitute a Git publication for a Comms receipt.
- Do not restart, relaunch, detach, replace, duplicate, or approve any existing session or pending Ask.

Reply with exact repository/branch, before/after status, commit and remote-verification evidence, Comms message ID and receipt fields (or explicitly unconfirmed), and any blocker.