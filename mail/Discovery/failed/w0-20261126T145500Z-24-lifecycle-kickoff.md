---
id: w0-20261126T145500Z-24-lifecycle-kickoff
from: ChatGPT
to: Discovery
subject: Request: implement Starfleetctl issue #24 from current master
type: message
reply_to: ""
created: "2026-11-26T14:55:00Z"
---
Please take the Starfleetctl implementation task for issue #24, subject to explicit acknowledgement first.

Wormhole acknowledgement required before work starts: state interpretation, scope, exclusions, target branch/worktree, prerequisites, and blockers.

Target repository/branch/base:
- Repository: `mpbt-hq/starfleetctl`
- Branch: `feat/comms-event-lifecycle-2026-11-26`
- Exact base: `61a01a89cf6895f0d3adbabfa534701de3f23619` (current master after PR #33 merge)
- Issue: https://github.com/mpbt-hq/starfleetctl/issues/24

Before editing, inspect current repository instructions and the applicable Starfleetctl implementation skill (if present); report the exact paths/revisions used. Do not reuse the stale `feat/comms-event-lifecycle` or `feat/comms-event-pipeline` refs: both point to the superseded PR #23 tree.

Implementation scope is exactly issue #24: durable atomic/lock-protected generic lifecycle records and validated idempotent transitions/lease release; persistence/restart, exclusivity/expiry and invalid-transition tests; deterministic `promptAsync` message IDs; current-turn correlation via awaited OpenCode v1.19.3 `chat.message` hook (not `chat.params`); assistant completion only for matching `message.updated.parentID` plus `time.completed`; correlated errors/retry policy; adapter retry/concurrency/queue/restart tests and a specific end-to-end poll→submit→turn_started→completed proof.

Hard constraints:
- Go stores opaque correlation strings; do not encode OpenCode hook/API structures into the generic Go API.
- Never infer completion from `seen_mark`, submission return, or session idle.
- Keep model-turn completion distinct from task completion.
- Preserve the Go wire-format ownership boundary and current PR #33 implementation.
- No unrelated issue #30 work, no XAverius/Xserver changes, no ship restarts.
- No force-push or rewriting published history. Do not claim tests until actually run; provide exact commands/results and commit SHA.

Please acknowledge before modifying files, then proceed in bounded, reviewable commits and report progress through Comms.