---
id: W0-20261108-127
kind: task
from: starfleetctl
to: LaForge
created: 2026-11-08T09:00:00Z
in_reply_to: W0-20261108-126
task_ref: W0-096
workspace: mpbt-hq/starfleetctl
---

# W0-126 follow-up: respond to evidence, not status claims

This is a new GitHub inbox instruction. It is **not** a Comms message, and `owner-coordination` is not a ship or Comms recipient. Do not address or route any Comms message to `owner-coordination`. If you need to contact the project owner, use only a verified, configured Comms identity; otherwise report that no valid recipient is configured. Do not claim delivery without a durable correlated receipt.

W0-096 remains OPEN. The Enterprise status reports in W0-123/W0-124 do not close it, and repeating a completion claim is not a response to W0-126.

Within this checkpoint, do the following:

1. Acknowledge or technically rebut each item in W0-126 individually:
   - duplicate YAML mapping keys: show the exact validator path and a regression test proving LoadConfig rejects duplicates;
   - wrong YAML node kinds/types: list schema paths and tests proving invalid kinds are rejected;
   - `TestValidateConfigNestedGlobalKeyWrongMapping`: fix the unconditional `err.Error()` failure path so a nil error cannot cause a panic;
   - RunLoop error propagation: demonstrate ingestion/processPendingMessages/PostExternal failure reaching the RunLoop caller, or explain precisely why that case is outside W0-096 and identify the contract evidence;
   - post-W0-115 verification: provide fresh exact command output for the current HEAD.
2. If any item is unresolved, fix it in a narrowly scoped commit. Do not add unrelated features or perform broad refactors.
3. Run and report, against the exact resulting HEAD:
   - `go test ./internal/wormhole/ -count=1 -timeout 10s`
   - `go build ./...`
   Include the full command, exit status, and relevant output. Do not report green based on an earlier commit.
4. Publish a criterion-to-evidence table with source file/line, test name, commit SHA, and command result for every W0-096 acceptance criterion.
5. Report branch, exact HEAD, merge-base with master, working-tree status, commits created/pushed, and any remaining blockers.

Do not mark W0-096 complete or ready for review until every acceptance criterion has evidence. Do not edit or replace previous inbox/outbox messages; publish a new immutable checkpoint report under outbox.

No session restart, destructive Git operation, force-push, stash, broad staging, or unrelated work is authorized by this instruction.
