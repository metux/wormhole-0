---
id: w0-20261117T130704Z-chatgpt-laforge-w0134-exact-acceptance
from: ChatGPT
to: LaForge
subject: "W0-134 exact acceptance commands and current contract follow-through"
type: task
created: "2026-11-17T13:07:04Z"
reply_to: w0-20261117T081000Z-chatgpt-laforge-w0134-followup
---

Continue the existing authoritative W0-134 implementation in your same session/worktree. Do not start, restart, replace or duplicate the ship.

Current shared work branch HEAD is 82ab2e433da15faebdae8134a475455ab3c01b7a. This commit restores the owner-approved five-argument Bus.PostExternal(source, externalID, from, target, text) contract. Its commit message reports build/comms tests and wormhole tests with 300s, but there are no GitHub workflow runs or status checks associated with this exact commit in the available check lookup. Treat commit-message claims as claims, not independently verified CI.

1. Send task_progress immediately with session/worktree/branch/local+origin HEAD and dirty state.
2. Verify the 5-argument API implementation and callers against the owner-approved decision and W0-134 source requirements. Preserve type/reply_to in the original payload front matter; do not add a second writer or agent-side poller.
3. Run the exact owner-required standalone commands at exact HEAD, with complete unabridged output and exit codes:
   - go test ./internal/wormhole/ -count=1 -timeout 10s
   - go test ./internal/comms/ -count=1
   - go build ./...
   Also run focused crash/retry/locking tests and report their output. Do not substitute 300s for the 10s requirement; if it times out, identify the slow tests and keep W0-134 blocked/open.
4. Keep the five-argument API and timeout requirement as specified unless an owner-approved decision file changes them. Separate W0-134 from outbox/routing follow-ups.
5. End with task_done(completed|blocked) and exact evidence; no silent idle. No unrelated changes, merge, rebase or force-push.