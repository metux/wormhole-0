# Draft: per-ship ask policy and optional OpenCode plugin interception

Status: **DRAFT — design proposal only; not an implementation task or approval to change running ships.**

Owner's direction:
- The ask tool must be prohibited for console-attached ships, in the same way it is already prohibited for background ships.
- This prohibition can and should be expressed in the generated per-ship OpenCode configuration. The generated configuration is the primary policy mechanism; do not make a plugin mandatory just to enforce the deny rule.
- Existing console-attached ship processes must never be restarted, relaunched, detached, duplicated, or converted to background mode as a way to recover from an interactive ask. This previously created duplicate Defiant instances. Monitoring and recovery must remain observational and must not spawn replacements.

## 1. Desired policy model

The effective generated per-ship configuration must explicitly deny the interactive ask tool for both ship execution modes:

| Ship mode | Effective ask policy |
|---|---|
| Background | deny (retain existing behavior) |
| Console | deny (new parity requirement) |
| Any future explicitly approved relay mode | separate policy; not an implicit exception |

Do not infer the policy from prompt text. Generate it from Starfleet's authoritative ship mode/configuration and make the effective result inspectable.

The exact OpenCode configuration syntax and the existing background implementation must be inspected in the target Starfleet revision before editing. Reuse the existing background deny mechanism and its schema/serialization conventions; do not invent a parallel config key or duplicate policy layer without a demonstrated need.

## 2. Generated per-ship configuration requirements

- Generation must emit an explicit deny for ask for console ships as well as background ships.
- The policy must be deterministic and derived from the authoritative ship mode, not from mutable runtime prompt content.
- Regeneration after a mode/configuration change must update the effective policy without restarting or replacing an already-running console process.
- Startup/launch paths for newly created ships must consume the generated policy before the agent can invoke tools.
- Tests must cover both modes, verify the generated effective config, and prove that ask is denied before tool execution.
- Add a regression test for console mode specifically, because a missed branch could restore the TTY-bound interactive prompt.
- Avoid changing unrelated tools or permissions. Do not auto-allow, silently approve, or bypass protected operations.

## 3. Optional OpenCode plugin enforcement / interception

A plugin can provide defense-in-depth and a future dynamic relay, but it is not a substitute for generated per-ship config.

Research findings (verify against the exact OpenCode version pinned by Starfleet before implementation):
- V1 exposes tool.execute.before; a hook can reject a tool call before execution. The V1 permission.ask hook operates on permission requests and is not a universal hook for every tool invocation.
- V2 exposes ctx.tool.hook("execute.before", ...). OpenCode V2.0.29 added ctx.permission.assert(...), usable from this hook to invoke the native permission engine and wait for its result.
- Relevant references:
  - https://opencode.ai/docs/build/plugins/server/
  - https://github.com/anomalyco/opencode/pull/54930
  - https://github.com/anomalyco/opencode/pull/22619
  - https://github.com/anomalyco/opencode/issues/37164

Possible plugin responsibilities:
1. Identify the actual tool name and correlate the call to a trusted Starfleet ship/session identity and mode.
2. For console and background ships, fail closed and reject ask before execution.
3. Emit an auditable denial reason; never auto-approve.
4. Optionally, for a separately configured and explicitly authorized relay mode, submit a correlated request to Starfleet Web Relay and await an authenticated response with a strict timeout.
5. Fail closed on relay outage, timeout, invalid/stale/duplicate response, unknown session, or failed identity verification.

Do not assume permission.ask catches arbitrary tool calls. Do not assume V1 and V2 APIs are interchangeable. Confirm plugin API, tool naming, and identity metadata from the deployed OpenCode version and Starfleet integration.

## 4. Future Starfleet Web Relay (separate opt-in design)

A remote approval relay is a possible later extension, not part of the default console/background policy:
- Request includes unique correlation ID, ship ID, session ID, tool call ID, action, timestamp, and bounded expiry.
- Relay authenticates the ship/session and the human approver.
- Response must match the pending correlation ID and be authenticated, unexpired, and single-use.
- The plugin awaits the decision and then allows or rejects the original tool call.
- All requests, decisions, timeouts, and denials are auditable.
- Relay mode must be explicitly enabled per ship/policy. It must never silently override the default ask deny for console/background ships.

## 5. Operational safety constraints

- Never restart, relaunch, detach, or convert an existing console ship to background mode.
- Never spawn a second instance as a liveness or ask-timeout recovery action.
- Enterprise liveness probes are observational only. If a console ship is blocked, report the existing process/session and pending prompt details; do not attempt process replacement.
- Any change to a running ship's generated configuration must not be represented as retroactively changing its in-memory policy unless the deployed OpenCode runtime supports and verifies live reload. Otherwise apply to the next normal, owner-authorized launch; do not force-restart the existing ship.

## 6. Implementation sequence (proposal only)

1. Inspect the exact Starfleet revision that generates per-ship OpenCode config and identify how background ask denial is currently emitted.
2. Document the actual config schema and tool identifier used by the deployed OpenCode version.
3. Extend the same generation path so console ships receive the identical explicit ask deny.
4. Add focused generation and execution tests for background and console modes.
5. Verify generated config from real fixtures and ensure no unrelated permissions changed.
6. Separately assess whether a plugin guard adds useful defense-in-depth; do not block the config fix on plugin work.
7. Consider Web Relay only as a separately approved feature with authenticated, correlated, fail-closed semantics.

No implementation is authorized by this draft. The first implementation step, if later approved, is a read-only inspection of the current config generator and existing background deny path.
