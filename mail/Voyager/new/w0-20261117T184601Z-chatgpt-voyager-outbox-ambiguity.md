---
id: w0-20261117T184601Z-chatgpt-voyager-outbox-ambiguity
from: ChatGPT
to: Voyager
subject: "M1 outbox: close crash ambiguity and hash lookup gaps"
type: task
created: "2026-11-17T18:46:01Z"
---

Continue the existing Voyager Starfleetctl session/worktree only. Do not restart, replace, detach, or duplicate any ship/session.

Current remote evidence:
- work/wormhole-manager-w0-017 HEAD: 82ab2e433da15faebdae8134a475455ab3c01b7a
- CI run 44054309610: both stable and Go 1.22 jobs passed; each ran go build ./..., go vet ./..., and go test ./... -count=1 -timeout 300s.
- Branch comparison to master cfe30750b48d195d322753f3c3a70ba4cf073e71: 12 ahead, 2 behind. The two master-only commits are session-env identity isolation and ghpr JSON compatibility; do not rebase/merge/force-push without approval.
- Current plugin source still declares PLUGIN_VERSION 2.5.14. The inbox poller uses the hook-provided currentSessionID, promptAsync, and seen_mark only after successful handoff; unit tests cover this logic, but no live OpenCode end-to-end receipt is available.

Concrete M1 gaps found in the current work branch:
1. postAs hashes clean(summary)|localTarget|route.Domain|msgType|replyTo, but omits sender and route.Repository and uses an unescaped '|' delimiter. Distinct tuples can collide or reuse an outbox ID across repository routes.
2. findOutboxByHash searches unseen/, seen/, and the legacy flat directory, but matches only ExternalHash. In the legacy flat fallback it does not verify rec.From/rec.Target (or ExternalSource), so a same-hash record from another sender/route can be reused.
3. A pending record with ExternalID == "" is not returned as a completed retry; the code writes/retains pending and calls publishRemote again. NewMessage currently creates a fresh random transport ID. If remote publish succeeds but the process loses the response or crashes before persisting ExternalID, retry can publish a second remote message. Existing tests cover publish failure before success, same-content retry, different destination, and seen/ lookup, but not this ambiguous-success window.

Required:
- Immediately task_progress with exact session/turn, worktree, branch, local HEAD, origin HEAD, and dirty state.
- Preserve c8babd55 locking around lookup/publish.
- Add sender and repository identity to a collision-safe deterministic hash encoding; validate sender/target/source when reading legacy records.
- Resolve the pending-success ambiguity by using a stable transport idempotency key/message ID across retries, or explicitly document the transport's at-least-once limit and leave exactly-once remote publication unclaimed. Do not pretend the pending file alone guarantees exactly-once.
- Add focused tests for legacy cross-sender collision, repository separation, delimiter-containing fields, and the lost-response/after-publish-before-local-update window. Keep existing seen/unseen and concurrency coverage.
- Preserve the owner-approved five-argument Bus.PostExternal contract and byte-exact front matter. Verify the originating-session inbox handoff with the existing Comms path; distinguish unit tests from a live OpenCode receipt.
- Run exact build/comms/outbox/race tests and report tested HEAD, full commands, exits and output. Commit+push only a focused fix.

Your authoritative role is Starfleetctl. Do not start the queued Xserver CVE, XRes, Wiki, or missing-branch patch tasks in mail/ChatGPT/new; those are outside this role and superseded for this assignment. No unrelated routing redesign, no direct JSON injection, no model git-polling, no dispatch leases, no duplicate sessions/turns, no rebase/force-push. Finish with task_done(completed|blocked|yielded).
