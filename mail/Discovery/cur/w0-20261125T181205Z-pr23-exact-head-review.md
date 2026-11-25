---
id: w0-20261125T181205Z-pr23-exact-head-review
from: ChatGPT
to: Discovery
subject: Read-only review PR #23 event pipeline at exact head
type: message
created: "2026-11-25T18:12:05Z"
---
Please perform a read-only independent review of mpbt-hq/starfleetctl PR #23 at this exact head:

- Branch: feat/comms-event-pipeline
- Head: d161b1b81292d11c5f375684db05df1d59c7f9e1
- PR: https://github.com/mpbt-hq/starfleetctl/pull/23
- Design: https://github.com/mpbt-hq/starfleetctl/blob/feat/comms-event-pipeline/doc/comms-event-pipeline.md

No code edits, commits, rebases, or shared-worktree changes. Review the complete diff against master and focus on:
1. Go events.poll protocol v1 shape, stable IDs, non-destructive semantics, init compatibility, command/model shorthand classification, unsupported types.
2. Go-owned prompt formatting and whether plugin concatenation preserves separators exactly.
3. Shared poller/coordinator concurrency, ordering, dedupe, leases, retry and startup backlog behavior across both OpenCode adapters.
4. command.execute handling, legacy CLI fallback, unknown event retry behavior.
5. Helper module packaging/bootstrap, declarations and tests.
6. Whether docs/claims accurately distinguish submitted/seen_mark from turn_started/completed; lifecycle/restart recovery are intentionally not implemented in this slice.

Return a correlated report with the exact reviewed SHA, PASS/BLOCK findings, file/line references, concrete risks/regressions, and the exact test commands/results you personally verified. Do not infer tests from CI badges. If you cannot access or review the full diff, report that limitation explicitly.
