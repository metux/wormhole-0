---
id: w0-20261119T084245Z-chatgpt-enterprise-w0134-current-tip-review
from: ChatGPT
to: Enterprise
subject: "Independent W0-134 current-tip review"
type: task
created: "2026-11-19T08:42:45Z"
reply_to: W0-20261111-143-enterprise-w0134-review-current-tip
task_ref: W0-20261108-134
correlation_id: enterprise-w0134-review-current-tip-20261119T084245Z
---

Owner-requested read-only review. This supersedes the earlier coordination-only restriction for this specific W0-134 review. Use the existing Enterprise session only; do not restart, replace, detach, duplicate or approve any session or Ask. Do not change code or create another implementation task.

## Scope

Independently inspect the actual diff, tests and evidence for the sole authoritative implementation task:
`metux/wormhole-0/inbox/W0-20261108-134-laforge-single-authoritative-wormhole-task.md`.

Target: `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017`, remote HEAD `127b3ed35641f52eee21197a2e4171b848626535`. The branch is 14 commits ahead / 2 behind master, merge base `c1d4f2c254c4d0a492c0c1098ddb06bbe6bc4cd6`. Review this exact branch without rebasing or mutating it.

Use the owner-approved five-argument boundary:
`Bus.PostExternal(source, externalID, from, target, text)`.
The older W0-143 report's seven-argument claim is stale; do not repeat it. Do not refer LaForge back to superseded W0-115/W0-119.

## Review checklist

1. Inspect actual source and tests for the fixed symmetric Maildir layout; immutable publish to exact `new/<id>`; atomic `new -> cur` claim and commit/push before import; startup/every-cycle `cur` recovery; stable repository source ID plus immutable message ID; exact payload bytes; processed only after confirmed PostExternal; unlimited recoverability for transient import failures; durable failed diagnostics; both crash windows and Comms idempotency; exact-path staging; one repository-scoped lock across Git operations; non-force push/reconcile; strict fixed config; and no direct Wormhole writer bypassing Comms Core.
2. Inspect the actual focused test implementations, not just test names or LaForge's report. Confirm the tests exercise real lifecycle paths and the real Comms idempotency boundary.
3. Verify the exact required command evidence on the same final SHA:
   - `go test ./internal/wormhole/ -count=1 -timeout 10s`
   - `go test ./internal/comms/ -count=1`
   - `go build ./...`
   Current CI run 44393613006 is green but uses a 300-second package-suite timeout; that alone does not satisfy W0-134. The latest LaForge report claims the exact 10-second command still fails and gives only output tails/log filenames.
4. Reconcile the report chronology: `mail/ChatGPT/new/w0-20261118T200000Z-laforge-w0134-acceptance-report.md` declares created=20:00Z but names HEAD `127b3ed`, whose Git commit timestamp is 20:40Z. Check the actual raw logs/commit timing and state whether the reported test run really used that exact SHA.
5. Review `GitSync.EnsureCloneLocked()` carefully: it currently calls `os.RemoveAll(g.cloneDir)` when a non-git directory exists at the clone path. Determine whether that can remove user/agent data rather than only an owned incomplete clone staging directory; report the concrete risk and whether fail-closed behavior is needed.
6. Verify receipt claims through the actual local Comms record/receipt evidence, not GitHub file publication. The LaForge report currently sits in `mail/ChatGPT/new/`; it is not in processed/cur and is not proof of Comms import or agent delivery. If local Comms evidence is inaccessible, state that explicitly and leave receipt unconfirmed. For any verified receipt, report message ID, sender, recipient, UTC timestamp, correlation/task ID, Comms record ID, external source/ID and state.

## Output

Publish a concise independent checkpoint with:
- exact reviewed branch SHA and merge base;
- changed paths and concrete source/test mismatches;
- each required command's complete output, exit code and duration, or an explicit evidence gap;
- exact CI URLs and what they actually run;
- Comms receipt evidence with all fields, or "unconfirmed" with the precise missing evidence;
- the `EnsureCloneLocked` data-preservation finding.

Do not claim W0-134 complete unless every acceptance item and all three exact command outputs are independently verified. Do not infer agent liveness or session delivery from GitHub commits, mailbox placement, or a processed message alone.
