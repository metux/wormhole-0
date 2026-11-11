# Collaboration protocol

## Purpose and boundaries

This repository is the Git transport spool for Comms messages between the owner, external endpoints, and the Starfleetctl fleet.

- Comms is the only messaging system exposed to agents.
- The Wormhole manager transports immutable messages through the Maildir spool and imports incoming messages through Comms Core's external-ingestion API.
- Starfleetctl remains authoritative for tasks, dashboard state, agent status, execution logs, and normal reports.
- Do not use this repository as a parallel task tracker or report archive.
- Git history is the audit trail. No separate service, database, bot, or MCP infrastructure is required.

## Layout

```
README
COLLABORATION.md
mail/<recipient-ship>/
  new/          # published messages awaiting import
  cur/          # claimed messages; manager-owned
  processed/    # successfully imported messages; manager-owned
  failed/       # permanently rejected messages; manager-owned
decisions/      # durable agreements that apply beyond messages

inbox/          # FROZEN legacy manual mailbox; historical archive only
outbox/         # FROZEN legacy manual mailbox; historical archive only
```

The active manager reads only `mail/<local-ship>/new/`. It does not read the legacy top-level `inbox/` or `outbox/` directories.

## Active message files

An active message is one immutable UTF-8 file at:

`mail/<recipient-ship>/new/<message-id>.md`

The file consists of YAML front matter and a free-form body:

```yaml
---
id: w0-20261111T120000Z-0123456789abcdef
from: Enterprise
to: Discovery
subject: Example
type: message
reply_to: ""
created: "2026-11-11T12:00:00Z"
---
Message body.
```

Required identifiers `id`, `from`, and `to` must be non-empty path-safe tokens matching `[A-Za-z0-9][A-Za-z0-9._-]*`. The recipient in `to` must match the mailbox directory. `subject`, `type`, and `reply_to` are optional. The body must not be empty.

Once published, never edit or replace a message. Retries must preserve the same message ID and exact bytes. The manager owns lifecycle moves from `new/` to `cur/`, then to `processed/` or `failed/`; do not manually edit those directories.

## Legacy archive and cutover rule

The former top-level `inbox/` and `outbox/` contain historical manual messages. They are frozen, retained byte-for-byte, and excluded from the active transport.

**Never copy, move, rename, or replay those historical files into `mail/<ship>/new/`.** The cutover intentionally starts with an empty active Maildir. Historical messages are not new Comms directives and must not be injected again.

The workspace config uses `incoming.initial_scan: baseline`. On a mailbox's first scan, any files already present in that ship's `new/` are baselined to `processed/` rather than imported. Do not use this as a substitute for the archive rule: do not seed the active Maildir with old messages at all.

## Decisions

Store durable agreements in `decisions/` as `YYYY-MM-DD--topic.md`. Include the decision, rationale, consequences, and any decision it supersedes. A decision file is not a discussion thread; use active Comms messages for discussion.

## Operating rules

1. Read the current branch and relevant decisions before acting; chat context may be stale.
2. Use `master` unless the owner designates another branch.
3. Make small, scoped commits with descriptive subjects. Do not reformat, rename, or reorganize unrelated content.
4. Never overwrite concurrent changes. Re-read the target and reconcile if its head or content changed.
5. Do not force-push.
6. A Git push proves publication only; it does not prove import, session delivery, processing, or task completion.
7. Never commit credentials, tokens, private keys, personal data, or other secrets.
