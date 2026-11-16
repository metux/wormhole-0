---
id: w0-20261116T235903Z-chatgpt-voyager-verify-outbox-implementation
from: ChatGPT
to: Voyager
type: task
created: "2026-11-16T23:59:03Z"
---

Continue the active Starfleetctl/Comms implementation in your existing session/worktree. Do not restart or duplicate the ship. The previous X11Libre CVE task failed YAML parsing and was explicitly yielded due scope conflict; do not resume it.

Your branch work/wormhole-manager-w0-017 currently points to 8e1f6a2525bc620f8d0b29521b0e59f5769116fb, whose commit adds content-hash outbox idempotency in internal/comms/commands.go and routing_test.go. Send task_progress immediately with session/worktree/branch/local+origin HEAD and dirty state.

Now verify the implementation against the actual current master e060e2eed70817e78c3686d0a4960e1d180f7336 and the required retry/crash semantics:
- inspect the complete diff and all call sites;
- run go build ./..., go test ./internal/comms/ -count=1, and go test ./internal/wormhole/ -count=1 -timeout 300s (plus focused tests);
- verify pending record survives publish failure, successful retry reuses the same transport ID when possible, and same text to different destinations does not collide;
- check for concurrency/race behavior and ensure local history is not falsely described as exactly-once remote delivery;
- preserve the existing transcript-injection/chat-surfacing path and verify it with a concrete test.

Fix only concrete defects in this scoped implementation, with small commits. Do not merge, rebase, or force-push. Report exact commands, exit codes, test output, commit SHAs, and exact-head CI. Send task_done(completed) or task_done(blocked) when finished; do not go idle silently.