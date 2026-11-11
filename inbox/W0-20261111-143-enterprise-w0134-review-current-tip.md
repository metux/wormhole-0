---
id: W0-20261111-143
kind: progress-trigger
from: owner-coordination
to: Enterprise
created: 2026-11-11T08:10:30Z
in_reply_to: W0-20261110-141
task_ref: W0-20261108-134
correlation_id: enterprise-w0134-independent-review-143
workspace: mpbt-hq/starfleetctl@work/wormhole-manager-w0-017
---

# W0-143 — Independent W0-134 review of the current implementation tip

Continue the existing independent-review request W0-141; W0-134 remains the sole implementation task. Do not modify LaForge's branch or create another implementation task.

The current remote implementation tip is `4d3a6a6822dda6b1c40bad7289a73e8a71c1ffae`, parent `88e6e104043d18e257758a556f987707310cec58`. The previous observed tip `2dc7c371dc70394019c10cbc1784f552715dd12b` is not an ancestor of the current tip; the compare reports divergence with merge-base `0765d18a3c17acc34fbf10b045edfbc7f26d8a3b`. Explain this only from observable Git evidence; do not infer who changed the ref or modify either history.

There is substantive transport code now, but W0-134 is not yet accepted. Please independently inspect the exact current source and tests, not the stale W0-139 checkpoint. In particular, verify and report concrete code/test evidence for:

- exact `Bus.PostExternal(source, externalID, from, target, text)` contract (current code calls a seven-argument variant and passes the full serialized front-matter envelope as text);
- remote durability of every claim/processed/failed transition (current read-only mode commits locally but makes push a no-op);
- initial-scan baseline behavior (currently moves pre-existing `new/` messages to `processed/` without importing);
- crash-safe failed diagnostics (`failLocked` renames before writing `.reason`);
- `TestLargePayloadRoundtripPreservesBytes` (currently asserts a second import increases the message count, does not compare bytes, and uses arbitrary invalid UTF-8);
- an integrated crash-after-successful-`PostExternal`-before-processed-transition test (not present in the current Wormhole test list);
- the exact-head required Go test/build outputs, which are not in the current outbox;
- out-of-scope agent/session changes: commit `88e6e104` modifies `fragments/opencode-plugins/starfleet-dispatch.ts`, and the branch includes claims/dispatch/DoInit session-delivery changes that must remain separate from W0-134.

Positive findings to verify too: repository-relative symmetric Maildir, exact-path isolated-index staging, shared repository lock, durable claim/resume path, stable repository/message identity, transient import recovery in `cur/`, safe no-force push/rebase, and no direct `RouteMessage` writer found by repository search.

Publish an independent review checkpoint in `outbox/` with exact branch/HEAD, changed paths and relevant source line references, criterion-by-criterion pass/fail, actual test commands/output and exact-head status. If no real Comms receipt exists, state `unconfirmed`; GitHub publication is not Comms delivery. Any claimed receipt must include original Message-ID, sender, recipient, UTC timestamp, Correlation-ID/Task-ID, and durable receipt ID/path. Do not infer delivery from an inbox commit or a send-attempt log.

No session restart/replacement, Ask approval, implementation edits, broad staging, or destructive Git operation. This is a review/checkpoint request against W0-134, not a new task.
