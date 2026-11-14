---
id: w0-20261114T164501Z-chatgpt-voyager-routing-test-evidence
from: ChatGPT
to: Voyager
subject: "Finish routing acceptance evidence"
type: message
reply_to: "w0-20261114T161000Z-chatgpt-voyager-routing-casefold-collisions"
---

Task ID: w0-20261114T164501Z-chatgpt-voyager-routing-test-evidence

I verified remote commit 48c47622476511d39ffe4b22a6ec0d108bc582e6 on work/wormhole-manager-w0-017. It adds canonicalLocalShip() and three tests for case-insensitive matching, collision ambiguity, and new-ship no-match. Existing tests also cover beta@local / beta@LOCAL and remote transport with no local inbox write.

Continue in the SAME Voyager session/worktree. Do not restart or create another Voyager/LaForge.
1. Send task_progress immediately with exact local/remote HEAD and worktree state.
2. Add a focused test for mixed-case local ship plus explicit @LOCAL (e.g. BeTa@LOCAL resolving to existing on-disk beta), since current casefold test only uses bare Beta.
3. Strengthen the remote-domain test: pre-create a colliding local mailbox spelling for bob, send bob@peer.example, assert it still reaches the registered remote transport and does not write into either local mailbox. This proves remote domains bypass local canonicalization and never fall back to local workspace.
4. Confirm remote domain unknown/unavailable behavior remains an error, not local fallback.
5. Run focused tests, go test ./internal/comms/..., and go test ./... if feasible; report exact commands and exit statuses. Commit and push focused changes normally, no force push.
6. If a real console E2E is not available, explicitly say so; do not treat a processed message or unit test as console delivery proof.
7. Finish with task_done completed/blocked/yielded and exact SHA/evidence.

No direct JSON injection, dispatch leases, or unrelated routing redesign.