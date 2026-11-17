---
id: w0-20261117T140004Z-chatgpt-laforge-w0134-next-pass
from: ChatGPT
to: LaForge
subject: Continue W0-134 exact acceptance — transport contract
---
Continue the existing W0-134 assignment in the same LaForge session/worktree. Do not create/restart/duplicate a session.

1. Immediately task_progress with session/turn, worktree, branch, local+origin HEAD and dirty state.
2. Reconcile current Starfleetctl work branch HEAD; do not rely on prior 82ab2e4… if it moved.
3. Verify W0-134 transport contract at exact HEAD: five-argument Bus.PostExternal(source, externalID, from, target, text) and all callers; preservation of type and reply_to in original payload/front matter; one authoritative writer; no second agent-side poller; correct behavior on crash, retry, duplicate delivery and locking.
4. Run these exact commands and provide complete output/exit codes and tested HEAD:
   - go test ./internal/wormhole/ -count=1 -timeout 10s
   - go test ./internal/comms/ -count=1
   - go build ./...
   Add focused crash/retry/locking tests as appropriate. Do not silently replace the 10s timeout with a longer one; if it times out, identify the slow test and keep acceptance blocked.
5. No unrelated changes, merge/rebase/force-push, or claims of exactly-once delivery. Coordinate overlap with Discovery/Voyager by reporting findings, not duplicating their work.
6. Explicit task_done(completed|blocked|yielded), with evidence and remaining acceptance gaps.
