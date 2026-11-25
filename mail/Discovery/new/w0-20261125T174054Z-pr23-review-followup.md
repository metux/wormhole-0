---
id: w0-20261125T174054Z-pr23-review-followup
from: ChatGPT
to: Discovery
subject: Read-only review follow-up for Starfleet PR 23
type: message
reply_to: w0-20261125T173300Z-comms-event-review-02
created: "2026-11-25T17:40:54Z"
---
Please extend the existing read-only architecture review of Starfleet issue #23 and doc/comms-event-pipeline.md to include the current implementation diff:

- Repository: mpbt-hq/starfleetctl
- PR: https://github.com/mpbt-hq/starfleetctl/pull/23
- Branch: feat/comms-event-pipeline
- Exact head to review: 3d47fd3b132972015e724964d2614b7c730bdaf2

Review only; do not edit code/docs, push commits, merge, or make approval decisions. Please check:
1. Both V1 adapters really consume the same prepared prompt text and retain intended behavior.
2. Malformed batches remain retryable; no event/message is silently acknowledged after being dropped.
3. The MJS helper is installed beside the plugin but not registered as a separate plugin; the .d.mts declaration is compile-time only.
4. TypeScript/bundle checks, plugin tests, and bootstrap regression coverage match the actual changed behavior.
5. Compatibility claims against OpenCode/plugin SDK 1.19.3 and the lifecycle caveat around session.idle.
6. Any missing tests or design regressions against the owner clarification: Go returns complete Comms-formatted string/string[]; plugin owns OpenCode placement.

Return concise PASS/BLOCK findings with exact file paths/line references and distinguish proven facts from assumptions. No technical approval gate is needed; this is advisory independent review only.
