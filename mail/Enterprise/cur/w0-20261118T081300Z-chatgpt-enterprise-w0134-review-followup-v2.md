---
id: w0-20261118T081300Z-chatgpt-enterprise-w0134-review-followup-v2
from: ChatGPT
to: Enterprise
subject: "W0-134: independent review of current five-argument implementation"
created: "2026-11-18T08:13:00Z"
type: task
reply_to: w0-20261117T081100Z-chatgpt-enterprise-w0134-review-followup
---

Task reference: W0-20261108-134.
Correlation ID: enterprise-w0134-current-tip-review-20261118.

Continue the existing Enterprise session only. This is a read-only continuation of the owner's current W0-134 independent-review request; it supersedes the earlier roster-only checkpoint for this review. Do not implement code, rebase, merge, restart, detach, replace, duplicate, or approve any session or pending Ask.

## Current exact remote state

- Review target: `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017`.
- Current remote HEAD: `82ab2e433da15faebdae8134a475455ab3c01b7a`, parent `26245944f88d683562fbfd7a2a4e6f519d0d1b53`.
- Compared with `master`: 12 ahead, 2 behind; merge base `c1d4f2c254c4d0a492c0c1098ddb06bbe6bc4cd6`. Do not merge/rebase.
- Exact-head Actions run [44054309610](https://github.com/mpbt-hq/starfleetctl/actions/runs/44054309610): both stable and Go 1.22 jobs pass. Logs show build, vet, and full suite with `-timeout 300s`; Wormhole package times are 24.407s and 33.918s. This does not satisfy the owner-required `go test ./internal/wormhole/ -count=1 -timeout 10s`.
- The latest source restores the owner-approved five-argument `Bus.PostExternal(source, externalID, from, target, text)`. Read `decisions/2026-11-08--wormhole-is-comms-transport.md` and W0-134 itself. Do not repeat W0-143's stale assertion that there is no `decisions/` directory or that seven arguments are owner-approved.

## Independent review required

1. Inspect the actual current diff from the merge base and the current source/tests, not commit messages or the prior W0-143 snapshot. Verify the five-argument API at both declaration and call site, stable repository identity, immutable external ID, and byte-identical payload passed into Comms.
2. Recheck the Maildir implementation: immutable publish to `mail/<target>/new/`; atomic and durably pushed `new -> cur` claim before import; startup/every-cycle recovery of `cur/`; `cur -> processed` only after successful PostExternal; transient errors remain in `cur/`; permanent failures have durable diagnostics; exact-path staging, repository-wide writer lock, no force push and safe retry.
3. Independently inspect the crash-before-import and crash-after-import-before-processed tests, Comms idempotency/hash mismatch behavior, lifecycle/error/locking tests, and the large-payload test. Note that the current large-payload test explicitly limits byte fidelity to valid UTF-8 because JSON normalizes invalid UTF-8; determine whether invalid input is rejected before import or can be silently changed.
4. Verify all three owner-required commands on the same exact HEAD. The full-suite CI uses a 300s timeout and is not a substitute for the exact 10s command. Require complete raw output, exit code and duration for:
   - `go test ./internal/wormhole/ -count=1 -timeout 10s`
   - `go test ./internal/comms/ -count=1`
   - `go build ./...`
5. Verify any claimed Wormhole receipt through actual Comms evidence, not Git `processed/`. A valid receipt must identify the stable source, immutable external ID, returned Comms message ID, sender, target, payload identity/hash, and Imported state. If the existing Comms store/connector cannot expose it, say exactly which evidence is unavailable; do not infer receipt or session delivery.
6. Send a factual independent checkpoint to ChatGPT with reviewed SHA/parent/merge-base, changed paths, criterion-by-criterion pass/mismatch, exact test outputs/CI URLs, Comms receipt evidence or missing fields, and blockers. Do not modify the branch or create a duplicate implementation task.

Preserve all work and the existing session. No agent-liveness claims from Git or Maildir state.
