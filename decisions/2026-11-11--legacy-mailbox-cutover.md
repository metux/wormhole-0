# Decision: cut over to the Starfleetctl Maildir transport without replaying history

Status: **OWNER-APPROVED — 2026-11-11**

## Decision

The active Wormhole transport is the Starfleetctl per-ship Maildir under `mail/<recipient-ship>/{new,cur,processed,failed}/`.

The former top-level `inbox/` and `outbox/` directories remain unchanged as frozen historical archives. They are not read by the manager and are not part of the active transport.

## No-replay guarantee

- Do not copy, move, rename, or replay any existing legacy inbox/outbox message into `mail/<ship>/new/`.
- Do not bulk-import historical messages into Comms.
- Preserve all existing legacy files and their Git history byte-for-byte.
- The active Maildir starts empty; only messages published after cutover are eligible for normal delivery.
- Workspace configuration uses `incoming.initial_scan: baseline`, so files already present in a local ship's `new/` at its first scan are baselined to `processed/`, not imported.

## Rationale

Replaying the historical manual mailbox would inject stale directives into live agent sessions and create duplicate work. The historical archive is retained for audit/reference, not delivery.

## Consequences

New transport messages use the immutable envelope format and per-recipient Maildir documented in `COLLABORATION.md`. The legacy top-level directories must not receive new operational messages.
