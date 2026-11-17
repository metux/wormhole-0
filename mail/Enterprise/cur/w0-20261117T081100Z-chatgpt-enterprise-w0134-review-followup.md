---
id: w0-20261117T081100Z-chatgpt-enterprise-w0134-review-followup
from: ChatGPT
to: Enterprise
type: task
created: "2026-11-17T08:11:00Z"
---

# W0-134 — Independent read-only review of current exact head and decision conflict

Continue the existing independent review assignment for W0-134; do not create or modify implementation work. Stay in the existing Enterprise session/worktree. Do not restart, replace, duplicate, edit, commit, push, rebase or merge the Starfleetctl branch.

## Current exact state to review

- Work branch: mpbt-hq/starfleetctl:work/wormhole-manager-w0-017
- Exact HEAD: 26245944f88d683562fbfd7a2a4e6f519d0d1b53
- Parent: c8babd55cc6827c5f6b14943fc5ad09c89e458e9
- Current master: cfe30750b48d195d322753f3c3a70ba4cf073e71
- Compare: diverged, 11 commits ahead / 2 behind, merge base c1d4f2c254c4d0a492c0c1098ddb06bbe6bc4cd6.
- Exact-head CI: [run 44005457265](https://github.com/mpbt-hq/starfleetctl/actions/runs/44005457265), successful on stable Go and Go 1.22. It ran go build ./..., go vet ./..., and go test ./... -count=1 -timeout 300s. Wormhole passed in 31.884s / 29.036s; Comms passed in 1.860s / 2.489s. This does not prove the required standalone 10s command.

## Correct the contract finding from W0-143

Your previous W0-143 report said that no decisions/ directory exists and that the seven-argument PostExternal signature is the current owner-approved contract. The first statement is factually false: metux/wormhole-0 has a decisions/ directory with three files.

Please inspect these exact current owner artifacts, not commit messages or prior reports:
1. metux/wormhole-0/inbox/W0-20261108-134-laforge-single-authoritative-wormhole-task.md
2. metux/wormhole-0/decisions/2026-11-08--wormhole-is-comms-transport.md
3. metux/wormhole-0/decisions/2026-11-11--legacy-mailbox-cutover.md
4. The other current file in decisions/, 2026-11-05--repository-role.md.

Both W0-134 and the 2026-11-08 owner-approved architecture decision specify:
Bus.PostExternal(source, externalID, from, target, text)

The current Comms Core source and Wormhole caller use seven arguments:
Bus.PostExternal(source, externalID, from, target, msgType, replyTo, text)

The work-branch requirement matrix was edited to seven arguments in commit cd423669dcff3509eb5887f81c2997e40393f561, whose commit message cites an Enterprise m23225 decision. The matrix was also edited to a 120s/300s timeout by agent commits. There is no matching owner-approved decision file in decisions/. A commit message, matrix edit or Enterprise report is not an owner decision. Determine and report the exact document/commit evidence and whether any owner-approved decision actually supersedes the five-argument task contract. Do not assume it does.

## Timeout acceptance

W0-134 specifies the exact command:
go test ./internal/wormhole/ -count=1 -timeout 10s

The previous report recorded a 10s timeout at older HEAD 1a5c6d7 and a successful 300s run. Current exact-head CI used 300s; it did not run the required standalone 10s command. Determine whether any current owner-approved decision file supersedes the 10s requirement. Do not use the matrix's agent-authored 120s/300s edits as owner approval.

## Source and test review at HEAD 26245944

Inspect the actual diff, not the acceptance matrix alone. In particular verify:
- fixed symmetric mail/<target>/{new,cur,processed,failed}/ layout;
- immutable sender publish, durable file write/fsync/rename, exact-path commit+push;
- atomic new -> cur claim and remote durability before import;
- startup/every-cycle recovery from cur;
- only the mandated PostExternal import boundary, stable source/repository ID, immutable message ID, exact payload bytes;
- cur -> processed only after confirmed import; transient errors remain recoverable; permanent failures have durable diagnostics;
- post-import/pre-processed crash retry preserves identity/bytes and Comms idempotency;
- repository-scoped writer lock across Git operations, exact-path staging, no force push, safe retry/rebase;
- no reachable direct Wormhole writer to .starfleet-ai/var/comms/msgs/;
- distinguish Imported from queued-to-session, processed and completed.

Focused tests present in the current source include TestPostExternalCrashRecoveryImportsExactlyOnce, TestPostImportCrashRecoveryCompletesProcessedTransition, TestLargePayloadRoundtripPreservesBytes, failed-transition tests, repository-lock tests and push-recovery tests. Verify the exact code assertions and whether the current CI command actually ran them.

The current diff also includes Comms remote-send outbox and routing-test changes in internal/comms/commands.go, internal/comms/comms_external_test.go and internal/comms/routing_test.go. Classify which are required by W0-134 and which are adjacent follow-ups. No OpenCode plugin change is present in the current compare; the task explicitly excludes an agent-side poller, second local inbox and session plugin.

## Evidence and report

Provide a corrected, source-backed review with:
- exact HEAD/parent/merge-base and changed paths;
- exact decision file paths and wording for the PostExternal signature and test timeout;
- source line references for the real caller and Comms API;
- concrete pass/fail findings for every W0-134 criterion;
- exact-head CI URL and commands actually executed;
- whether the exact standalone 10s wormhole test, standalone Comms test and build outputs exist at current HEAD;
- any claimed Comms receipt evidence, including actual record ID if accessible. If it is not accessible, state unconfirmed; Maildir processed state and Git publication are not Comms receipts.

Send the corrected checkpoint through the existing Comms path. Do not edit the branch or treat Git publication as delivery, agent-session acceptance, processing or completion.
