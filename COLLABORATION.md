# Collaboration protocol

## Purpose and boundaries

This repository is the lightweight, Git-based message channel between the owner, ChatGPT, and the Starfleetctl fleet.

- Use it for instructions, questions, acknowledgements, replies, hand-offs, and durable cross-agent decisions.
- Starfleetctl remains authoritative for tasks, dashboard state, agent status, execution logs, and normal reports.
- Do not copy complete Starfleetctl reports or task records here. Link to their identifiers and locations instead.
- Git history is the audit trail. No service, database, bot, or separate MCP infrastructure is required.

## Layout

```
README
COLLABORATION.md
inbox/       Messages addressed to Starfleetctl or a named ship
outbox/      Replies and hand-offs addressed to the owner or ChatGPT
decisions/   Durable agreements that apply beyond one message
```

Create directories implicitly with their first real file; do not add empty placeholder files. Keep temporary status in messages rather than introducing a second dashboard.

## Message files

Use one Markdown file per message:

- `inbox/YYYY-MM-DDTHHMMZ--ID--topic.md`
- `outbox/YYYY-MM-DDTHHMMZ--ID--topic.md`

Use UTC in filenames and metadata. IDs should be unique and stable, for example `W0-20261105-001`. Keep filenames lowercase apart from the ID. Do not edit a sent message to record its reply or completion: create a new message and reference the original with `in_reply_to`. If a message must be corrected, send a replacement that explicitly supersedes it.

### Required message header

Every message starts with this YAML front matter:

```yaml
---
id: W0-YYYYMMDD-NNN
kind: request
from: owner | chatgpt | starfleetctl | <ship-id>
to: starfleetctl | owner | chatgpt | <ship-id>
created: YYYY-MM-DDTHH:MM:SSZ
in_reply_to: null
task_ref: null
workspace: null
---
```

Allowed `kind` values: `request`, `question`, `ack`, `reply`, `report-link`, `handoff`, `decision-proposal`, `notice`.

Use `task_ref` for the Starfleetctl task identifier when one exists. Use `workspace` for the relevant repository and branch, for example `metux/xav-design@docs/restructure-2026-10-21`. Omit neither required header fields nor invent task references. Optional references may be `null`.

### Message body

Use only the sections relevant to the message:

```markdown
# <Short subject>

## Context
<Why this message exists; include relevant links and references.>

## Request
<Concrete action or question.>

## Constraints
<Scope, non-goals, files/branches to avoid, and concurrency rules.>

## Acceptance
<Observable result or evidence required.>

## Reply
<For replies: answer, result, blockers, and next action.>
```

Keep messages concise and actionable. A request must make its recipient, desired outcome, scope, and acceptance criteria clear. A reply should identify what was done, what was not done, blockers, and the next action. Link to commits, PRs, task IDs, and Starfleetctl reports rather than duplicating their contents.

## Decisions

Store durable decisions in `decisions/` as `YYYY-MM-DD--topic.md`. Include the decision, rationale, consequences, and any decision it supersedes. A decision file is not a discussion thread; use messages for discussion and add or amend a decision only when the owner confirms the outcome.

## Operating rules

1. Read the current branch and relevant messages before acting; chat context may be stale.
2. Use `master` unless the owner designates another branch.
3. Make small, scoped commits with descriptive subjects. Do not reformat, rename, or reorganize unrelated content.
4. Never overwrite concurrent changes. Re-read the target and reconcile if its head or content changed.
5. Do not force-push.
6. A task is not complete merely because a message says so: link the resulting commit and, where applicable, the Starfleetctl task/report.
7. Never commit credentials, tokens, private keys, personal data, or other secrets.
