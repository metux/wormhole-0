---
id: W0-20261108-135
kind: implementation-followup
from: owner
to: LaForge
created: 2026-11-08T17:59:00Z
priority: urgent
task_ref: W0-20261108-134
correlation_id: laforge-w0134-followup-135
workspace: mpbt-hq/starfleetctl@work/wormhole-manager-w0-017
---

# W0-135 — Continue W0-134 implementation; no completion evidence yet

This is a progress trigger for the existing W0-134 task, **not a replacement task**.

I checked the authoritative remote branch and Wormhole reports. The branch is still at `421652ef8f2d037d1766bb110ba2bca0ef9a8565` (2026-11-08 13:04:27Z). Its latest commit is a gofmt/alignment-only change; it does not implement the required Maildir receiver lifecycle. The newest LaForge outbox report is still W0-114, which describes the older W0-111 YAML-validation/RunLoop work and points back to superseded W0-096. It is not a W0-134 implementation checkpoint and provides no current evidence for W0-134. Do not treat that report or the formatting commit as completion.

Continue coding on the existing branch from its actual current HEAD. Do not stop at this message, acknowledge it and wait, or send another plan. Work through W0-134's implementation order and acceptance checklist. In particular, the current verified evidence does not establish any of these required behaviors:

1. Fixed symmetric `mail/<target-ship>/{new,cur,processed,failed}/` layout and immutable sender publication to `new/`, with exact-path commit/push.
2. Receiver atomically claims `new -> cur` and commits/pushes that claim **before** import; startup and every cycle resume `cur/`.
3. Import only via `Bus.PostExternal(source, externalID, from, target, text)`, with stable source identity, immutable message ID, and byte-identical payload on retries.
4. Move `cur -> processed` only after confirmed import, then commit/push; transient errors remain recoverable without an arbitrary retry cap; permanent rejects move to `failed/` only with durable diagnostics.
5. Crash recovery across claim/import/processed boundaries, including idempotent retry after import but before the processed transition.
6. One repository-scoped writer lock covering all Git operations; exact-path staging; safe commit/push/rebase retry with no force push; durable recovery across local move, commit, and push failure points.
7. Remove/refactor reachable direct Wormhole writers such as `RouteMessage()` that write directly under `.starfleet-ai/var/comms/msgs/`; `PostExternal` must be the sole import boundary and Comms Core the sole message-record writer.
8. Keep the simplified fixed configuration and reject legacy nested path/alias settings.
9. Focused tests for every W0-134 case, including concurrency, restart recovery, byte preservation, idempotency, transient/permanent failure, and no direct writer.

Some of these may already exist in unreported local work; if so, preserve it and provide the exact code paths and tests as evidence. Do not claim they are done without inspection and proof.

**Immediate next actions:**
- Fetch the branch, inspect its actual HEAD/parent/merge-base and working tree, and preserve all work.
- Implement the next missing lifecycle slice (atomic claim + durable Git publication, then `cur/` recovery and `PostExternal` import); continue through the remaining checklist without waiting for another prompt.
- Add focused regression tests alongside each behavior.
- Commit and push each coherent slice to `work/wormhole-manager-w0-017`.
- After each substantive commit, publish a W0-134 checkpoint with exact SHA, parent, changed paths, behavior implemented, full test output, remaining checklist items, and the next concrete code action.

At final acceptance, run these exact commands on the exact reported HEAD and include their complete, unabridged output:
```sh
go test ./internal/wormhole/ -count=1 -timeout 10s
go test ./internal/comms/ -count=1
go build ./...
```

Do not report completion based on an acknowledgement, W0-111 report, formatting-only commit, plan, partial test set, or unrelated green CI. W0-134 remains open until every acceptance criterion and the exact final verification evidence are independently present.