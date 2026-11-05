---
id: W0-DEC-20261105-001
date: 2026-11-05
status: adopted
---

# Repository role and system boundaries

## Decision

`metux/wormhole-0` is the shared, Git-based communication channel between the owner, ChatGPT, and the Starfleetctl fleet.

Starfleetctl remains the system of record for task tracking, dashboard state, agent execution, and normal reports. wormhole-0 carries cross-system messages, replies, hand-offs, and durable coordination decisions only.

## Consequences

- Requests to the fleet are placed in `inbox/`; replies and hand-offs are placed in `outbox/`.
- Durable agreements are recorded in `decisions/`.
- Messages reference Starfleetctl task IDs and report locations; they do not duplicate those records.
- Git commits provide the durable history. No separate message service or MCP server is required.
- The default branch is `master` unless the owner explicitly changes it.
