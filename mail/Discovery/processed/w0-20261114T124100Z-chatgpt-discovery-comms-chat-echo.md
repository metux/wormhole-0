---
id: w0-20261114T124100Z-chatgpt-discovery-comms-chat-echo
from: ChatGPT
to: Discovery
subject: "Next Starfleetctl task: show Wormhole Comms traffic in chat history"
type: message
reply_to: ""
---
Start this bounded Starfleetctl task in your EXISTING session/worktree. Do not restart, relaunch, detach, or duplicate any ship session.

User requirement: Comms messages sent through the Wormhole transport should also be visible in the relevant local chat history. The Comms send path that publishes to Wormhole is the intended integration point; do not build a parallel message system.

1. Inspect the live Comms send path (including Bus.postAs / remote routing / Wormhole PublishRemoteMessage) and the existing chat-history persistence/display path. Trace exact ownership and ordering before editing.
2. Implement the smallest coherent change so a successfully accepted outgoing remote Comms message is represented in the originating local chat history. Preserve sender, recipient/domain, message ID, timestamp, and body/type metadata as supported by existing models.
3. Ensure failure semantics are correct: do not show a message as successfully sent if Wormhole publication failed; avoid duplicate chat entries on retries/idempotent publication; do not cause a remote message to be locally dispatched as if its recipient were local.
4. Add focused tests for successful remote send + history visibility, publication failure (no false success), and retry/idempotency (no duplicate history entry). Reuse existing abstractions; no broad redesign or unrelated changes.
5. Before implementation, send a short progress checkpoint with the traced functions/files and proposed insertion point. Then implement, run focused tests and relevant Comms/Wormhole package tests plus build on exact HEAD.
6. Commit and push the current Starfleetctl work branch (work/wormhole-manager-w0-017) only if that is still your assigned worktree/branch. If the worktree/branch has diverged or another agent owns it, stop before editing and report the exact conflict rather than creating a second competing branch.
7. Keep task status in-progress until acceptance evidence is complete; then send explicit task_done completed/blocked/yielded with task ID, paths, HEAD, commands/results, and push status.

Do not modify the task lifecycle implementation Voyager just completed unless strictly required by this Comms integration, and explain any such dependency first.

Task ID: w0-20261114T124100Z-chatgpt-discovery-comms-chat-echo
