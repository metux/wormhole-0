# Dashboard-Topic markers are references, not acceptance

Date: 2026-11-26
Status: accepted by the project owner

## Decision

A `Dashboard-Topic` marker with a slug in backticks or single quotes in a
Comms commission message is a navigation/reference link to the dashboard task only. Receiving, importing, or
injecting that message does not mean the receiving ship has accepted or started
the task. The marker must not automatically set the ship's current task or
change its board status.

A ship accepts an assigned task explicitly by running
`starfleetctl task begin <slug>`. This records the task as `in-progress` and
the ship as `working`. Progress and completion are recorded through the
existing task lifecycle commands. A ship that cannot or will not accept the
assignment must communicate that rather than appearing to work on it.

Plugin startup, `comms init`, session recreation, model-activity telemetry,
and heartbeats must not infer or overwrite task association or board status.
They refresh process/model liveness metadata only and preserve an existing
status/task/note record; a new ship with no prior record defaults to `idle`.
Stale or previously offline records may return to `idle`. Task status is
changed only by explicit task lifecycle operations or explicit Comms status
commands.

The Dashboard-Topic marker remains in commission messages as a reference.
Explicit `task <slug>` / `task clear` commands remain supported as manual
status controls; they are not substitutes for the full task lifecycle.

## Rationale

Message delivery and prompt injection prove only that a directive reached a
session. They do not prove that the ship accepted it, started work, or will
complete it. Automatically adopting a topic on delivery therefore makes the
dashboard claim work is underway when the ship may only be reading, querying,
declining, or unable to act on the directive.

The previous plugin behavior coupled marker parsing to task association. It
also mirrored that association in plugin-local state and replayed it from the
periodic heartbeat, which could overwrite task state changed through the CLI.
Separating reference, acceptance, and liveness removes those false transitions.

## Consequences

- The OpenCode plugin does not parse Dashboard-Topic markers in ordinary
  directives and does not auto-adopt their topics.
- The task skill and Comms SOP explain the reference-only marker and explicit
  acceptance step.
- Regression tests cover marker handling and ensure plugin startup, model
  activity, and heartbeats do not send inferred state/task fields.
- This decision supersedes the former automatic marker-based task adoption.
