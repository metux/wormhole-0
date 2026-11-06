# ChatGPT Session Bootstrap — Wormhole / Starfleetctl / XAverius

This file is the durable handoff for a new ChatGPT session continuing the user's work. Read it before acting, then refresh all remote state; SHAs below are historical checkpoints, not assumed current state.

## User's standing instruction

When the user says only **"weiter"**, start another concrete work turn on BOTH workstreams:
1. Wormhole / Starfleetctl implementation and coordination.
2. XAverius / Xlibre Xserver registry implementation and spec alignment.

Do not ask what "weiter" means. At each turn refresh the remote repositories and inspect actual changes. Do not promise autonomous polling between turns. Do not claim an agent acknowledged a task merely because an inbox task exists.

## Repositories and authoritative branches

- Coordination/task mailbox: `metux/wormhole-0`, branch `master`.
- XAverius implementation: `X11Libre/xserver`, branch `wip/xaverius`.
- XAverius normative spec: `metux/xav-design`, branch `docs/restructure-2026-10-21`. This is authoritative; do not use `wip/001`.
- Wormhole manager implementation: `mpbt-hq/starfleetctl`, Laforge's isolated branch is `work/wormhole-manager-w0-017`; never assume `master` contains his work.

## Role boundaries

- **Defiant**: sole writer for `X11Libre/xserver:wip/xaverius`.
- **Laforge**: sole writer for the isolated Wormhole implementation branch/worktree in `mpbt-hq/starfleetctl`.
- **Enterprise**: local fleet coordination only. It refreshes the Wormhole repo before each timer tick, delivers tasks, routes messages, nudges stalled agents, and reports factual status/worktree/branch/HEAD/acknowledgement. Enterprise must NOT perform technical audits, code/spec reviews, PASS/BLOCK decisions, or act as an approval gate.
- **ChatGPT / project owner**: independently audits all Defiant and Laforge commits on every visible branch (not only master or PRs), checks spec conformance, and reports technical PASS/BLOCK. User retains merge authority.

No implementer should wait for Enterprise audit or owner review to continue to the next bounded task package. Review is not a gate to keep working.

## Current coordination task IDs (append-only inbox)

- W0-050: Defiant resume immediately; no Enterprise audit gate.
- W0-051: Laforge isolated branch, commit progress, owner audits every branch.
- W0-052: owner audit of Defiant's `eaff9a2`, concrete registry blockers and fix order.
- W0-053: Enterprise local coordination; deliver W0-050/052 to Defiant and W0-051 to Laforge.
- W0-049: role correction — Enterprise coordination only, owner performs audits.

Older architecture and implementation requirements remain relevant:
- Wormhole: W0-017, W0-018, W0-019, W0-024, W0-026, W0-028, W0-031, W0-035, W0-039, W0-042, W0-043.
- XAverius: W0-022, W0-029, W0-034 and the active registry assignment in xav-design.

## Wormhole communication protocol for ChatGPT sessions

1. At the start of every work turn, fetch the current `metux/wormhole-0:master` HEAD and inspect new inbox messages / coordination reports. Also refresh both implementation repos and the authoritative spec.
2. Treat the Wormhole repository as the durable cross-session coordination mailbox. Add a new uniquely named task file under `inbox/` using GitHub contents API; never overwrite an earlier task or assume a task file itself proves delivery/acknowledgement.
3. Verify every created task by fetching the exact path back from `master` and checking the commit SHA.
4. Look for durable responses under the repository's established response/outbox convention; if no convention exists, inspect the repository tree and coordinate with Enterprise to agree on one. Never invent a response path and then claim an acknowledgement.
5. Direct local Comms access is environment-dependent. If an actual Comms connector/tool or a local shell/agent session is available, contact Enterprise through the established Comms API (the Go implementation is under `mpbt-hq/starfleetctl/internal/comms`; dispatch supports `tell`, `inbox`, `ack`, etc.). Use the supported public API, not direct hand-written message-file writes. If this ChatGPT session has no Comms action and no local shell, say so plainly; send a durable coordination request through Wormhole and ask Enterprise to reply through Comms and leave a verifiable durable report. Do NOT pretend a GitHub task file is a live Comms conversation.
6. Distinguish sent/delivered/acknowledged. A remote commit of a task proves publication only, not that Enterprise or an implementer has read it.
7. Enterprise should report: remote refresh time and HEAD, messages actually sent through Comms, acknowledgements, exact local worktree paths, branches, HEAD SHAs, dirty/unpushed status, and blockers.

## Wormhole implementation audit baseline

A branch now exists: `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017`, HEAD `94af3ab07b5188e6a994dd98c40a93c2365355da` (historical checkpoint; refresh before relying on it).

The commit titled "complete wormhole manager" is NOT accepted. Owner audit found severe architectural and correctness gaps:
- It bypasses the existing `internal/comms.Bus` API and writes message files directly. Requirement is real integration through Comms API / `Bus.PostExternal()`, durable journal, idempotency, tombstones and fanout.
- It scans and rewrites each target's `unseen/` message into the same path; it is not cross-workspace/remote delivery and can overwrite or loop over messages.
- It has no correct W0-026 sequence recovery, sender/recipient identity validation, replay prevention, or recipient authorization.
- Message IDs use Unix seconds (`msg_<seconds>`), causing collisions.
- JSON is manually formatted with `fmt.Sprintf`, so quotes/newlines in content can corrupt records or inject malformed JSON.
- Target and message IDs are joined into filesystem paths without adequate validation; path traversal risk.
- Message queue is in-memory only; no durable journal, tombstones, fanout, or crash-safe transitions.
- `Dequeue` changes state to delivered before durable external delivery.
- Daemon is not a real daemon: `StartDaemon` only sets a package-global variable and writes current process PID; CLI exits immediately, no child worker is launched, no singleton lock, and CLI never calls `RunLoop`.
- PID/state writes are not atomic and startup failures can leave inconsistent files; PID cleanup/ownership and stale PID recovery are absent.
- `RunLoop` has no cancellation-safe cleanup and silently discards processing errors.
- Config parser is a hand-written line parser, silently ignores malformed values, doesn't handle nested `worker:` YAML as documented, and uses mismatched key names; LoadConfig silently falls back on invalid config.
- `listen_addr`, `auto_start`, `max_message_age`, `max_concurrent`, `persist_state` are unused or not meaningfully enforced.
- CLI `sync` is a success-shaped no-op.
- No actual service/autostart integration.
- No template `.starfleet-ai/conf/wormholes.yaml` nor verified `.starfleet-ai/.gitignore` change in the commit.
- Tests only cover basic helpers and happy-path in-process behavior; no crash/fault/race/sequence/authorization/idempotency tests.
- Requirement matrix claims completion without evidence; do not trust self-reported "all pass" without exact test commands/results.

Laforge must correct this in small coherent packages on the isolated branch, not merge it. Do not rewrite master history. Technical review is by owner, but Laforge must continue without waiting.

## XAverius implementation audit baseline

At last checkpoint `X11Libre/xserver:wip/xaverius` was still `eaff9a2fde04d846021c1dddb26b35e4b77bfeac`; refresh before relying on it. The one-line change from parent `b2fea17` removed a zero-size snapshot guard but did not populate the size. Owner audit blockers recorded in W0-052:
- Snapshot callback does not return byte size; snapshot size remains zero.
- Mandatory callback validation missing.
- Nested ops tables are not deep-copied.
- Failure cleanup and ModuleRole reference accounting are incorrect.
- Replay compares binding generation to module-role generation (different domains).
- Binding removal ignores device generation and is not idempotent.
- DIX destructor frees canonical registry records despite tombstone/server lifetime requirement.
- RegistryFini only resets counters and leaks owned objects.
- Identity evidence size validation is incomplete.
- Stage 3 observation/removal authority validation incomplete.
- Duplicate prototypes in `xaverius_intern.h`.
- Exactly the three designated implementation headers: `xaverius_proto.h`, `xaverius_priv.h`, `xaverius_intern.h`. Do not add duplicate public `xaverius.h` in Xserver implementation; preserve user's explicit header decision and avoid duplicate protocol definitions.
- No Stage 5, Screen/DDX integration, Proc handlers, Vulkan backend, unrelated renames or mass formatting.

## Session behavior and reporting

- Work on both project strands each "weiter".
- Audit every newly visible Laforge commit across all branches, even without PR, and every Defiant commit on `wip/xaverius`.
- Inspect exact diffs and source, not commit titles or agent claims.
- Record exact SHAs, tests, status, blockers.
- If an actual tool or local environment is missing, state the limitation and use the durable mailbox fallback; never fabricate direct contact or background polling.

## Update 2026-11-06 evening — Laforge branch discovered and audited

- Newly visible branch: `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017`, HEAD `94af3ab07b5188e6a994dd98c40a93c2365355da`.
- Commit title claims completion of W0-017/W0-018/W0-024/W0-026, but owner rejects it as a non-accepted first draft. See W0-054 for exact findings and required correction sequence.
- Major defects: bypasses `internal/comms.Bus` and real `Bus.PostExternal()`; local same-inbox rewrite instead of remote transport; no durable journal/idempotency/tombstones/fanout; second-based colliding IDs; unsafe manual JSON/path handling; fake in-process daemon with no worker process; non-atomic PID/state; weak YAML parser and inert config; no-op sync; no service/autostart; inadequate tests; requirement matrix overclaims.
- W0-054: Laforge owner audit / real Comms integration correction.
- W0-055: Enterprise must establish an actual Comms feedback loop to the project owner, send a targeted status update using configured recipient routing, report message ID, and leave a durable report. Do not guess the target. Enterprise remains coordination-only.
- This ChatGPT tool session currently exposes GitHub repository actions but no direct local Starfleet Comms action or shell into the flagship workspace. Do not claim a direct Comms message was sent from this session. Enterprise has been explicitly asked to initiate the real Comms contact locally; until a message ID/acknowledgement is reported, the feedback loop is requested but not yet verified.

## Latest known Wormhole mailbox state

- W0-050 commit `6bf5bcfb8070ce26829bd8d6865ae414e891136a`
- W0-051 commit `bfce7ad6ef6a63183c44b56f478d2559001b7d04`
- W0-052 commit `eaecd53a16d78da6334d5d75036663b6966ff900`
- W0-053 commit `aa6228802335d8b323890aebb2599a4dfa903e70`
- W0-054 commit `1e39a5e20751ed61bfe8448c4a6865bd4e3f86db`
- W0-055 commit `ea437911b5ec1721b148b08c186f379c47e559fe`
- Bootstrap file first published at `e52db09ca64179df4f79e72ba3c2312d26e62d2b`; later updates supersede this SHA.


## Update 2026-11-06 22:20Z — recurring Comms health probes and Defiant ask-stall follow-up

New standing coordination requirement from the owner:
- Enterprise must run real end-to-end Comms round-trip liveness probes during every normal coordination cycle (approximately every five minutes) with all active ships, at minimum Defiant and Laforge.
- Each probe uses a unique nonce/correlation ID and records send/response Comms IDs, timestamps, latency, and outcome. Require a correlated reply; local file inspection or process/PID presence is not a successful probe.
- Default response deadline is 60 seconds, with one retry. Distinguish slow/no response from a confirmed hang; track consecutive misses and last successful round-trip.
- Specifically query Defiant for pending interactive `ask`/approval prompts and exact prompt correlation ID. Never auto-approve protected actions. Enterprise reports every cycle durably in Wormhole outbox.
- W0-062 is the recurring Enterprise liveness task. Publication is not delivery; await actual Comms delivery/ack and the initial probe report.
- W0-063 assigns Laforge to diagnose and, if confirmed, eliminate Defiant's console/TTY-bound ask stall mode. Desired direction: supervised background worker, durable correlated async question/response, restart-safe pending asks, watchdog states, and no unsafe auto-approval.
- Laforge's latest known branch tip at this checkpoint is `8f0d7b4385abedf2a93e5c187a710c2bad9fc43e`; W0-060 is its active correction task.
- Defiant's latest known Xserver tip is `eaff9a2fde04d846021c1dddb26b35e4b77bfeac`; W0-061 is its active registry correction task.
- Latest authoritative spec tip checked: `dff0d69ef45ff55cceebb744a7458109ff3b1dc2`.


## Update 2026-11-06 — console-ship safety rule and OpenCode ask interception research

Owner clarification — this is a hard operational invariant:
- **Never attempt to restart, relaunch, detach, or convert an already-existing console-attached ship into a background ship.** This recovery path has repeatedly failed and previously created two simultaneous Defiant instances. Do not use it as a watchdog, timeout recovery, liveness repair, or workaround for an interactive prompt. Preserve and inspect the existing process/session; escalate to the owner when it is blocked.
- The intended fix for a console ship blocked on interactive input is to prevent the interactive `ask` tool from being invoked in that mode, not to restart the ship. Treat the `ask` tool as unavailable/denied for console-attached ships, just as it is for background ships, unless the owner explicitly approves a different policy. Do not auto-approve or silently bypass protected actions.
- Enterprise's liveness checks must be observational: probe the existing Comms session/process, report correlated response or exact pending ask state, and never spawn a replacement ship.

OpenCode plugin research (checked 2026-11-06):
- OpenCode's V1 plugin API supports `tool.execute.before`; throwing from the hook blocks the tool call. It also has a `permission.ask` hook that can change an existing permission request to allow/deny/ask. The latter is for permission requests, not a universal interception point for arbitrary tool invocation.
- V2 plugin API documents `ctx.tool.hook("execute.before", ...)` and a permission evaluation hook. OpenCode V2.0.29 added `ctx.permission.assert(...)`, explicitly usable from `tool.execute.before` to gate the original call and wait for the native permission flow. PR #54930 says this closes #37164 and supports Promise and Effect plugins.
- Therefore a plugin-based gate is feasible. The implementation must first confirm the exact OpenCode version/API deployed by Starfleet. In V2, inspect the effective tool name and ship/session/agent metadata in `execute.before`; deny the `ask` tool for console-attached ship sessions before execution (fail closed). A permission assertion alone only gates according to configured rules; configure an explicit deny for the dedicated ask action if using `ctx.permission.assert`. Do not assume that a generic `permission.ask` event catches tool calls that do not themselves request permission.
- A future dynamic Starfleet Web relay is technically plausible: an awaited plugin hook can submit a correlated approval request to the relay, wait with a bounded timeout, and allow/deny the original tool call based on the authenticated response. This should be a separate opt-in policy for cases where remote approval is intended; it must not be used to re-enable interactive `ask` for console ships by default. Timeout, relay failure, stale/duplicate response, unknown ship/session, and unauthorized response must fail closed. Preserve request/session/tool correlation and audit records.
- Relevant sources: OpenCode V2 server plugin docs https://opencode.ai/docs/build/plugins/server/ (tool hooks and permission evaluation; `ctx.permission.assert` documented in plugin permissions); V2.0.29 permission assertion PR https://github.com/anomalyco/opencode/pull/54930; earlier V1 permission.ask hook fix https://github.com/anomalyco/opencode/pull/22619; `tool.execute.before` dynamic approval discussion closed by #54930 https://github.com/anomalyco/opencode/issues/37164.
- This is research and a policy decision, not evidence that a plugin has already been implemented or deployed. Before coding, inspect Starfleet's pinned OpenCode version and actual tool registration/configuration. Keep existing console ships running; no replacement process.


## Update 2026-11-06 — ask policy draft

- Owner clarified that the ask prohibition should be emitted through the generated per-ship OpenCode configuration, in parity with the existing background-ship deny rule. The plugin approach is optional defense-in-depth / future relay, not a prerequisite for the config change.
- Draft only; no implementation authorized: `design-drafts/ASK_TOOL_POLICY_AND_RELAY.md` (published in commit `58a6017cf0b7cdda8a9c0b10e87a0195a0c35c89`).
- The draft requires inspecting and reusing the existing background config-generation path and actual OpenCode schema; console and background must both explicitly deny interactive ask. It also records the plugin interception research and a separately opt-in Starfleet Web Relay concept.
- Existing console processes must never be restarted/relaunched/duplicated to apply this policy. Config changes must not be claimed to affect an already-running process unless live reload is verified; otherwise apply only on a normal owner-authorized future launch.

## Update 2026-11-06 — concurrent ChatGPT sessions

Owner requirement: Wormhole should eventually support multiple independent ChatGPT sessions accessing the same installation concurrently, so separate planning/control conversations can proceed in parallel.

- Treat ChatGPT sessions as first-class principals, distinct from Starfleet ships.
- Give each session a trusted stable identity; isolate session-local context and pending requests by default.
- Make shared project/task resources explicitly addressable and versioned. Use optimistic concurrency/transactions; reject or surface stale writes instead of last-writer-wins.
- Correlate every request/response and make retries idempotent. Route replies to the originating session unless explicitly delegated.
- Model task ownership, shared access, claims/leases with fencing, and explicit handoff records.
- Preserve append-only audit history with actor/session attribution. Enforce authorization per session/resource; one session must not implicitly inherit another's private context or approvals.
- Avoid a global current-session variable or global lock. Test simultaneous writes, duplicate delivery, crashes, stale leases, handoffs, and independent parallel tasks.
- Architecture draft (not implementation authorization): `design-drafts/MULTI_SESSION_ACCESS.md`.
