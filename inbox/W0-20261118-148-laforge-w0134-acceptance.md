---
id: W0-20261118-148
kind: acceptance
from: owner-coordination
to: LaForge
created: 2026-11-18
correlation_id: fleet-idle-20261118-laforge-w0134
task_ref: W0-20261108-134
---

# Continue W0-134 acceptance

Continue the single authoritative W0-134 implementation/acceptance task; do not create a parallel task or restart a session.

1. Inspect the current exact branch/HEAD and the latest implementation and review checkpoints.
2. Verify the W0-134 acceptance criteria against current source, tests, and actual exact-tip command output. Do not rely on old reports or commit messages.
3. Focus on any remaining blockers in durable Wormhole transitions, exact payload preservation, stable repository/message identity, crash recovery around PostExternal, remote push/rebase durability, and required test coverage.
4. Run the required acceptance commands on the exact reported HEAD:
   `go test ./internal/wormhole/ -count=1 -timeout 10s`
   `go test ./internal/comms/ -count=1`
   `go build ./...`
5. Publish a criterion-by-criterion acceptance report in outbox with exact HEAD, command outputs, and any remaining fail/block status.

Do not claim Comms delivery based on Git publication. Any delivery claim needs a durable correlated receipt. No destructive Git operations, broad staging, or session replacement.
