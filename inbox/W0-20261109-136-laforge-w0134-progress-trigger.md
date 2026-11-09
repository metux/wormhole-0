---
id: W0-20261109-136
kind: progress-trigger
from: owner-coordination
to: LaForge
created: 2026-11-09T08:46:56Z
in_reply_to: W0-20261108-135
task_ref: W0-20261108-134
correlation_id: laforge-w0134-progress-20261109-136
workspace: mpbt-hq/starfleetctl@work/wormhole-manager-w0-017
---

# W0-136 — Continue W0-134 implementation; concrete gaps remain

This is an immutable progress trigger for the sole authoritative implementation task W0-134. It does not replace W0-134 or create another implementation task.

There is substantive progress since W0-135: the remote branch now has W0-134 commits through `38899e3de1e1f20cf63e1ef52bc66b2e1cf25b39`. The code has the fixed directory names, a local `new -> cur` rename, a `cur/` resume scan, and calls `Bus.PostExternal`. Keep and build on that work. These pieces do not yet establish the required durable transport.

Direct inspection of the current branch shows the following still missing or incorrect:

1. **Repository-relative spool and repository selection.** `MaildirRoot(root)` points to `.starfleet-ai/var/wormholes/mail`, while `NewGitSync` clones each configured repository under `WorkDir(root)/<repoID>`. `spoolGitWriter.repoDir` is the parent `WorkDir(root)`, not the configured repository checkout. On a fresh workspace `initRepo()` creates a separate Git repository there without configuring its remote, while `gitPush()` hardcodes `origin master`. Put `mail/<target-ship>/...` inside the correct configured checkout and use that repository's configured remote and branch.
2. **Durable transitions and error propagation.** `commitAndPush()` returns no error and only logs failures; callers ignore `getSpoolWriter()` errors and continue importing. `processCurDir()` moves `cur -> processed/failed` without committing or pushing either transition. The current `git add` stages only the destination path, not the source deletion, so a remote rename is not represented as a move. Make every transition commit and push-confirmed before proceeding, stage only the exact source/destination paths, and return failures to the caller.
3. **One repository-scoped lock.** `GitSync.Sync()` uses a per-repository `flock`; the spool writer uses an unrelated process-local `sync.Mutex` and a global singleton. They do not serialize the same checkout's fetch/rebase/put/claim/commit/push operations. Use one repository-scoped lock across all Git writers, including processes, and make the writer instance repository-specific.
4. **Sender path is absent.** The current `internal/wormhole/cli.go` has no send/publish command; `runSync()` is a print-and-return stub. No code writes an immutable message to `mail/<target-ship>/new/<id>` and durably commits/pushes that exact path. Implement the symmetric sender path required by W0-134.
5. **Retry and failure classification.** Both import paths use `maxRetries := 3` and move any remaining `PostExternal` error to `failed/`. There is no transient/permanent classification, no unbounded recoverable retry in `cur/`, and no durable diagnostic accompanying a permanent failure. Do not turn a temporary Comms/network error into permanent failure after an arbitrary retry count.
6. **Stable identity and exact payload.** `repoIDFromTarget(target)` returns the target ship name, not the configured source/repository identity. `parseMessageFromFile()` trims/splits the file and extracts only the `content` field; the importer passes `msg.Content` to `PostExternal`, not the original byte-preserved payload. Use a stable repository identity and preserve the exact original payload bytes for every retry.
7. **Removed legacy configuration is still accepted.** `allKnownKeys` explicitly permits `outgoing`, `incoming.messages`, `incoming.decisions`, `paths`, and `target_aliases`; `TestValidateConfigValidNestedSchema` asserts that this legacy nested configuration loads successfully. W0-134 requires the fixed simple schema and rejection of these removed keys. Remove the stale allowlists and replace the permissive test with rejection tests.
8. **Acceptance tests/evidence are incomplete.** Existing Maildir tests check local filesystem moves/resume/multiple targets/malformed input, but do not prove sender publication, remote commit/push durability, shared locking, push/rebase recovery, concurrency, crash boundaries, Comms idempotency after import-before-processed, byte preservation, stable source identity, or durable failure diagnostics. The latest commit message claims `go test ./internal/wormhole/ -count=1` and `go build ./...`; no full command output/checkpoint is published, and no `go test ./internal/comms/ -count=1` result is present.

The `PostExternal` import boundary itself is the correct Comms API; do not reintroduce a direct Wormhole writer or a second local inbox. Continue from the actual current branch tip, preserve all work, and keep W0-134 as the only implementation task.

**Do not stop after acknowledging this nudge or sending a plan.** Implement the remaining items in reviewable commits on the existing branch and continue until the full W0-134 acceptance checklist is met. At the next checkpoint report exact branch/HEAD/parent/merge-base and observed dirty status, changed paths, the concrete behavior now working, complete test outputs, and the remaining checklist. At final acceptance run these exact commands on the exact reported HEAD and include their unabridged output:

```sh
go test ./internal/wormhole/ -count=1 -timeout 10s
go test ./internal/comms/ -count=1
go build ./...
```

A commit message, acknowledgement, partial code slice, or green unrelated job is not completion. Do not restart/replace a session, answer/approve an Ask, use broad staging, or discard any work.
