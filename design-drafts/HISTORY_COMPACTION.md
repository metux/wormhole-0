# Design plan: Wormhole Git history compaction

Status: **PROPOSED — implementation deferred until current performance work is complete**

Owner: Wormhole manager in `mpbt-hq/starfleetctl` (`internal/wormhole`)

This document describes a future maintenance capability. It does not authorize running compaction, force-pushing, deleting refs, or changing current daemon behavior.

## 1. Goal and non-goals

### Goal

Bound the amount of Git history that active Wormhole clients must fetch and maintain, while preserving the exact current repository tree and all active Maildir state.

Compaction must rewrite the remote branch so that the new branch history has a new root commit. Merely using `git clone --depth` or `git fetch --depth` is insufficient: shallow clients reduce their local history, but do not shorten the history stored/reachable from the GitHub branch.

### Non-goals

- Do not compact the frozen legacy `inbox/` or `outbox/` archives separately or replay their contents.
- Do not change message IDs, envelope bytes, Maildir semantics, Comms ingestion, delivery guarantees, or routing.
- Do not make compaction a prerequisite for normal sync.
- Do not run compaction as part of ordinary send/import operations.
- Do not promise physical erasure of Git objects from GitHub storage. Rewriting refs removes commits from the active branch's reachable history, but GitHub may retain unreachable objects, cached views, PR refs, forks, or other references. Strict removal of sensitive data is a separate GitHub-side process.

## 2. Placement and ownership

Implement this inside the Wormhole manager, not in individual ship clients and not as an unrelated cron job.

Relevant existing code:
- `internal/wormhole/wormhole.go`: daemon lifecycle, RunLoop and repository-level orchestration.
- `internal/wormhole/git_sync.go`: Git command execution, timeouts, fetch/push and retry behavior.
- `internal/wormhole/spool.go`: repository-scoped spool operations and writer serialization.
- `internal/wormhole/cli.go`: maintenance CLI surface.
- `doc/wormhole.md`: operator documentation and recovery procedure.

The daemon already owns repository synchronization and the remote Maildir lifecycle. It is therefore the correct place to coordinate a repository-wide freeze, preflight, rewrite, verification, and client recovery.

## 3. Proposed behavior

Compaction is a distinct maintenance operation with three modes:

1. **inspect / dry-run** — report eligibility, current remote head, commit count, repository size if available, pending mailbox counts, and blockers. No writes.
2. **compact** — explicit operator-triggered operation in the first implementation. It performs the full guarded protocol below.
3. **automatic policy** — later phase only, after measurements establish useful thresholds and safe operational behavior. Disabled by default.

Do not select a fixed threshold such as 500 commits yet. First collect real repository growth and sync/fetch cost after the existing performance problems are resolved. A future policy may use commit count, pack/repository size, or measured fetch duration; it should use hysteresis and a minimum interval to avoid repeated rewrites.

Compaction should initially be limited to the configured canonical writable repository and its configured branch. Read-only repositories/clients must never initiate it.

## 4. Safety protocol

A history rewrite is a repository-wide maintenance event, not an ordinary local Git operation. A local mutex alone is insufficient because several workspaces may push to the same remote.

### 4.1 Global maintenance exclusion

Before implementation, add a distributed maintenance exclusion protocol understood by every Wormhole writer:

- Acquire a repository-scoped maintenance lock using an atomic remote operation (for example, a dedicated lock ref under `refs/wormhole/` created without force). The lock record identifies repository, operation ID, owner instance, acquisition time, and intended branch.
- Every send, sync, lifecycle transition, and push path must check the lock before starting a write. Writers that observe it must pause/retry without modifying or pushing the repository.
- Only one compaction can own the lock. Never steal an apparently stale lock automatically: require explicit operator inspection/recovery, because an expired-looking lock does not prove its owner is dead.
- Release the lock only after postflight verification and client recovery instructions/state are durably recorded.
- The lock protocol must be deployed to all writers before enabling compaction. An older daemon that ignores the lock can invalidate the safety guarantees.

If a robust lock cannot be implemented with the existing Git transport/auth model, compaction must remain an offline maintenance command requiring all writers to be stopped. Do not substitute a process-local mutex and call it distributed safety.

### 4.2 Preflight checks

Abort without rewriting if any condition is not satisfied:

- The remote, branch, and current remote head are resolved unambiguously.
- The repository is configured writable and is not in a read-only mode.
- No other maintenance operation or writer is active.
- All relevant local Git worktrees are clean: no staged, unstaged, untracked transport files, merge/rebase/cherry-pick in progress, or uncommitted lifecycle moves.
- No local commits are ahead of the remote and no unexplained divergence exists.
- All mailboxes have no unprocessed `new/` messages and no claimed `cur/` messages. In-flight sends/imports must finish before the freeze can proceed.
- Pending lifecycle transitions are committed and, for a writable canonical client, pushed and confirmed on the remote.
- The frozen legacy top-level `inbox/` and `outbox/` remain untouched; they are not scanned or replayed.
- A fresh fetch confirms the expected remote head. Record that exact SHA as the compare-and-swap expectation.
- The operator sees a dry-run summary and explicitly confirms the destructive rewrite.

A failed preflight is a clean abort. Never silently move pending messages to another state to make compaction eligible.

### 4.3 Construct the replacement history

- Fetch the exact remote branch head and record its tree SHA.
- Create a new root commit with **no parent**, using exactly the current branch tree. This is the essential history truncation: the content tree is preserved, while all parent links to the old history are removed.
- Use a temporary candidate ref for validation. Do not first update the canonical branch.
- Verify that the candidate commit has zero parents and exactly the recorded tree SHA.
- A temporary backup ref/branch pointing to the pre-compaction head is allowed and is recommended for the initial rollout as a recovery measure. It must be clearly named with the operation ID and creation time, excluded from normal client fetch refspecs, and have an explicit expiry/deletion deadline. It must not become a permanent archive. Delete the temporary ref after the agreed rollback window and verify that no other branch or tag managed by us still points into the old history. Until deletion, old commits remain reachable through that backup ref and may be fetchable by someone who knows its name.

### 4.4 Publish with compare-and-swap

- Re-read the remote canonical branch immediately before publication.
- Update it using a force-with-lease equivalent, requiring the remote head to equal the preflight SHA. Never use an unconditional force push.
- If the head changed, abort and restart preflight; do not overwrite concurrent work.
- If branch protection or GitHub policy rejects the rewrite, stop and report the blocker. Do not weaken repository protections automatically.
- Remove the temporary candidate ref after successful verification.

### 4.5 Postflight verification

- Fetch the canonical branch again from the remote.
- Verify its head is the candidate SHA, it has no parents, and its tree SHA equals the pre-compaction tree SHA.
- Verify the committed Maildir tree and all current repository files are byte-for-byte unchanged at the tree level.
- Verify the remote branch's reachable commit count is one (or the explicitly chosen retained-window count if a future design changes the policy).
- Record old head SHA, new head SHA, old/new reachable commit counts, tree SHA, timestamp, operator/daemon identity, and verification result in daemon logs/state outside the rewritten Git history as well as the post-compaction operational report.
- If any postflight check fails, stop all writers and require operator recovery. Do not automatically attempt a second force-push.

## 5. Client recovery after compaction

A client with the old history must not perform an ordinary fetch+rebase and then push. That can reintroduce the old commits or produce a confusing unrelated-history merge.

After successful compaction:

- All clients detect that the remote branch is not a descendant of their local branch.
- They enter a dedicated `history-rewritten` recovery path and refuse pushes until reconciled.
- Clean clients discard the old local branch history and reset/reclone from the new remote head.
- Any client with local uncommitted or unpushed transport state is quarantined for manual reconciliation; never discard it automatically.
- Read-only clients also refresh to the new history. Their local lifecycle commits must not be rebased from the old lineage onto the new root.
- Preserve clone-local daemon state only where its semantics are independent of Git commit ancestry (for example, per-mailbox initial-scan markers). Validate this explicitly; do not reset Comms deduplication or message identity state.
- After refresh, run a normal read-only verification sync before re-enabling writes.

A post-compaction generation marker may be useful for fast detection, but the authoritative test remains the branch ancestry/head relationship, not timestamps alone.

## 6. GitHub and retention limitations

This feature's goal is to make old commits unavailable through normal clones of the canonical branch after compaction. GitHub does not need to physically delete unreachable objects immediately. Temporary backup branches are permitted during a bounded rollback window; while they exist, their old commits remain reachable and may be fetchable through those refs. After the rollback window, delete the backup refs and verify that no branch or tag managed by us still exposes the old history. GitHub may retain unreachable objects internally, and this is acceptable.

Before enabling it on the production repository, inspect:
- all branches and tags, including temporary backup refs;
- open/closed PR refs and other GitHub-managed refs;
- forks and mirrors;
- cached commit/patch URLs and externally retained clones.

Do not claim secure erasure. If the requirement is removal of sensitive content rather than bounded operational history, use GitHub's sensitive-data removal/support process and separately handle forks/clones. Recreating the repository under a new identity may be an operational alternative, but is not part of this implementation plan.

## 7. Configuration and CLI proposal

Keep the initial interface small and explicitly opt-in. Proposed shape (names are provisional):

```yaml
wormholes:
  maintenance:
    history_compaction:
      enabled: false
      mode: manual
      # thresholds intentionally TBD after performance measurements
```

Proposed CLI:
- `starfleetctl wormhole history inspect [--repo ID]`
- `starfleetctl wormhole history compact --repo ID`

The compact command should require an interactive confirmation by default. A future automation mode must require a separate explicit configuration opt-in and must still honor every safety gate. Do not add a generic `--force` bypass.

## 8. Implementation phases

### Phase 0 — prerequisite: performance work (current priority)

No compaction implementation. Resolve the current performance problems, add/retain timing for clone/fetch/rebase/push and mailbox processing, and establish a stable baseline.

### Phase 1 — observability and dry-run

- Measure reachable commit count, repository/pack size where available, fetch duration, and sync duration.
- Count `new/`, `cur/`, `processed/`, and `failed/` entries without mutating them.
- Implement inspect/dry-run output and preflight diagnostics only.
- Add tests proving dry-run never changes refs, index, worktree, or mailbox contents.

### Phase 2 — distributed writer freeze

- Design and test the maintenance lock protocol.
- Gate every repository write path on the lock, including send, sync, claim/move commits, and retry pushes.
- Test multiple daemon instances/workspaces and lock contention.
- Keep the compaction operation unavailable until every writer path is gated.

### Phase 3 — local rewrite engine

- Build a replacement root commit from the current tree in an isolated local/bare test repository.
- Verify zero parents and identical tree.
- Add failure injection at every step; prove canonical ref remains unchanged until the guarded publication step.
- Test dirty worktree, pending messages, `cur/` claims, divergent local commits, and interrupted operations.

### Phase 4 — remote publication and client recovery

- Test force-with-lease publication against a local bare remote first.
- Test concurrent remote update rejection.
- Test fresh clones, stale writable clones, and read-only clones.
- Implement the dedicated history-rewritten detection and recovery path.
- Run an end-to-end test with multiple independent clients and a disposable GitHub repository before production use.

### Phase 5 — operator documentation and controlled manual rollout

- Document exact backup, stop/freeze, compaction, verification, and recovery steps.
- Run one explicitly approved manual compaction on a disposable repository.
- Only then consider production manual use.
- Automatic scheduling remains a separate future decision.

## 9. Acceptance criteria

- No compaction can run while messages are pending or a writer is active.
- Every writer honors the same repository-scoped maintenance lock.
- Dry-run is strictly read-only.
- Replacement branch has a new root and the exact pre-compaction tree.
- Publication is compare-and-swap guarded; concurrent pushes are never overwritten.
- Failed preflight or candidate validation leaves the canonical remote branch unchanged.
- Stale clients cannot accidentally reintroduce the old history.
- Both writable and read-only clients recover safely.
- Comms external-ingestion idempotency and Maildir lifecycle semantics remain unchanged.
- Legacy top-level archives remain untouched and are never replayed.
- Documentation clearly distinguishes branch-history truncation from guaranteed physical erasure on GitHub.

## 10. Deferred decisions

Do not settle these until Phase 0 measurements exist:
- commit-count / pack-size / fetch-duration thresholds;
- compaction cadence and hysteresis;
- whether a small retained commit window is ever preferable to a single new root;
- whether automatic compaction is worth the operational complexity;
- exact GitHub-side cleanup expectations and whether support intervention is required.

**Current decision:** keep this as a documented future feature. Resolve performance issues first; do not implement or activate history compaction yet.
