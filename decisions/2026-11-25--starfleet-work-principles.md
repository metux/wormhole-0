# Starfleet work principles

**Date:** 2026-11-25  
**Status:** Owner decision

## Decision

For work in the Starfleet / starfleetctl area, ChatGPT should perform as much of the work directly as its available repository, code, and execution tools allow. The fleet is a supporting resource, not the default implementer.

ChatGPT should use the fleet when a step cannot be performed directly in this session, or when parallel implementation, independent review, or runtime verification provides clear value. Delegated work must be narrowly scoped, respect the live task board and existing worktree/branch ownership, and receive explicit acknowledgement. ChatGPT remains responsible for integration, independent technical scrutiny, and evidence-based reporting.

## Rationale

The owner considers the models currently running in the fleet less capable than ChatGPT for implementation and technical reasoning. Direct work therefore minimizes avoidable quality loss and coordination overhead, while fleet assistance remains useful for tasks requiring access to the local runtime or independent parallel effort.

## Consequences

- Prefer direct repository inspection, implementation, tests, and review when the available tools support them.
- Do not delegate merely because a fleet member exists or is idle.
- Delegate local-only operations, runtime checks, or bounded parallel work when direct execution is unavailable or materially less effective.
- Never claim a message was sent, acknowledged, processed, or completed without corresponding evidence.
- Preserve existing task ownership and the no-concurrent-writers rule.
- Keep the actual task backlog in Starfleetctl's authoritative task system; this Wormhole decision records the durable working agreement only.
