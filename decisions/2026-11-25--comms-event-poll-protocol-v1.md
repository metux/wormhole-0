# Comms event polling protocol v1

Date: 2026-11-25
Status: implemented on `mpbt-hq/starfleetctl:feat/comms-event-pipeline`

## Contract

The OpenCode plugin polls the generic event interface with `events.poll`.
The response carries `protocol_version: 1` and an explicit `events` array.
Each event has a stable ID (currently the Comms message ID), a type, source
metadata, payload, and optional generic actions.

Initial event types:

- `prompt.inject`: ordinary inbound directive. Go returns the complete prompt
  snippet as `payload.prompt` (string or string array), including sender/context
  prefix and all separators. `actions.start_turn` tells the plugin whether the
  asynchronous adapter should start a new turn.
- `command.execute`: transitional wrapper preserving existing command and
  model-shorthand behavior. The plugin executes the command through its
  OpenCode-aware command handler. Future work may replace individual commands
  with typed events such as `model.switch`, `session.quit`, or `session.reset`.
- `unsupported`: unknown source message type. The plugin must not inject or
  acknowledge it; it remains pending for explicit handling.

Go owns Comms-specific prompt composition. The plugin only validates the
payload, concatenates supplied prompt parts without adding separators, and
places the result using the appropriate OpenCode adapter.

## Compatibility

- `init` returns versioned `events` as well as legacy `messages`, so older
  plugins continue to work.
- The legacy `inbox` command remains during the compatibility window.
- The new plugin uses `events.poll` and falls back to `inbox` only when paired
  with an older CLI that does not implement the event command.
- Polling is non-destructive. Existing `seen_mark` remains the submission
  acknowledgement; it does not prove a model turn started or completed.

## Delivery architecture

Both OpenCode V1 adapters use one single-flight event poller, one coordinator,
one ordering/deduplication queue, one lease mechanism, and one dispatcher.
Failed handoffs release leases; unknown event types remain pending. The
OpenCode-specific distinction is limited to placement: current-turn system
context versus asynchronous new-turn prompt.

## Evidence

Implementation branch: `feat/comms-event-pipeline`
- Code head `542298bf77ee3de68518121c4a0059f4e79f8001`: all 3 CI jobs passed,
  including Go build/vet/full tests on Go 1.22.x and stable, plugin
  bundle/typecheck, and 70 plugin tests.
- Documentation follow-up: `d161b1b81292d11c5f375684db05df1d59c7f9e1`
- PR: https://github.com/mpbt-hq/starfleetctl/pull/23

## Explicitly not solved yet

Durable offered/submitted/turn-started/completed/failed lifecycle, restart
recovery, and correlated end-to-end proof of a specific event's model-turn
start and completion remain the next slice. Neither `promptAsync` returning nor
a session-scoped idle event is completion proof.
