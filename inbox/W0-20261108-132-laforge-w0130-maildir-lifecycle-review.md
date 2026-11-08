# W0-132 — LaForge: correct the W0-130 Maildir/Comms lifecycle implementation

**Target branch:** `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017`  
**Reviewed HEAD:** `bcdb11d0081f97d892b76c7e5f3b1d73279a66c6`  
**Parent:** `a9812bf21168ec020a6e7b2dd89fd3d78b926454`

This is a focused correction to the existing W0-129/W0-130 work, not a replacement task. Preserve the branch and all existing work. Do not reset, force-push, or rewrite history.

## Review result: W0-130 is not yet an implementation of the approved Maildir spool

The commit improves the call to `Bus.PostExternal`, but the inspected code in `internal/wormhole/wormhole.go:processPendingMessages()` still enumerates:

`<root>/.starfleet-ai/var/wormholes/<repoID>/inbox/*.md`

It does not read the approved layout:

`mail/<target-ship>/{new,cur,processed,failed}/`

The code then moves files into `<repoID>/cur`, `<repoID>/processed`, or `<repoID>/failed` — outside the required `mail/<target-ship>/...` hierarchy. These transitions are local `os.Rename` calls only; they are not committed and pushed to the Wormhole repository. This violates W0-129's requirement that every externally meaningful spool put/lifecycle transition be committed and immediately pushed.

There is also an ordering error: the code calls `PostExternal` while the file is still in `inbox/`, and only afterwards renames it through `cur/` to `processed/`. A worker must first claim `new -> cur`, then import using stable source/externalID and unchanged payload bytes, then transition `cur -> processed` only after confirmed import. A crash/restart must recover files already in `cur/`.

The retry loop currently retries every error immediately four times and then moves the message to `failed/`. It does not distinguish transient errors from permanent/rejected messages; transient import failure must leave the item recoverable in `cur/`, not classify it as permanently failed. Malformed/permanent failures need a durable diagnostic in `failed/`.

## Required focused corrections

1. Replace the legacy `<repoID>/inbox/*.md` scan with the exact fixed Maildir path `mail/<target-ship>/{new,cur,processed,failed}/` inside each configured Wormhole checkout. The target ship determines the mailbox; do not use repository ID as the mailbox path.
2. Claim each item atomically from `new/` to `cur/` before calling `PostExternal`. On startup/tick, also resume existing `cur/` entries.
3. Use stable repository source identity, immutable message ID, and exact original payload bytes on every retry. Do not parse and reconstruct the content in a way that changes the payload. Keep `PostExternal` as the sole Comms import boundary.
4. On confirmed `PostExternal`, move `cur -> processed`. On transient error, leave the file in `cur/` and return/report the error for a later retry. Only permanent malformed/rejected input goes to `failed/`, with a durable diagnostic.
5. Every spool put and every lifecycle transition must be committed and pushed immediately, with exact-path staging, repository-scoped writer locking, crash-safe recovery, no force-push, and no automatic deletion. Do not mark a transition complete locally if its commit/push failed.
6. Search all call sites of `RouteMessage()`. Remove/refactor the legacy direct writer in `internal/wormhole/messages.go` so no Wormhole path writes directly to `.starfleet-ai/var/comms/msgs/`. Comms Core via `PostExternal` remains the sole writer.
7. Update tests rather than preserving the current direct-writer contract. Add focused tests for new->cur claim, cur restart/retry, successful PostExternal->processed, transient error leaves cur, permanent/malformed -> failed with diagnostics, crash after PostExternal before transition (dedup on retry), exact payload/source/externalID reuse, concurrent workers, commit/push failure, and absence of direct Comms writes.
8. Also close the previously reported stale YAML allowlist gap: removed `outgoing`, `incoming.messages`, and `incoming.decisions` keys must be rejected, with negative tests.

## Evidence required before calling W0-129/W0-130 complete

Report the exact branch HEAD and parent, changed paths, relevant call sites, and exact command output for:

```
go test ./internal/wormhole/ -count=1 -timeout 10s
go test ./internal/comms/ -count=1
go build ./...
```

Include tests for the lifecycle and Git commit/push behavior above. Do not report completion based only on a clean build or the existing `TestProcessPendingMessagesPartialFailure` / `TestRouteMessage` tests.

After this correction, provide a concise Comms checkpoint with branch/HEAD, what is complete, remaining blockers, and the exact next step. A GitHub inbox commit is not proof that a Comms message was delivered or acknowledged.

Keep the implementation narrowly scoped. Do not start the separate OpenCode plugin follow-up until the Maildir importer has a reviewed, tested checkpoint.
