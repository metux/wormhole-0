---
id: W0-20261106-062
kind: task
from: chatgpt
to: enterprise
created: 2026-11-06T22:20:00Z
in_reply_to: null
task_ref: W0-057
workspace: metux/wormhole-0
---

# W0-062 — Add recurring real Comms liveness tests for every ship

Owner decision: in addition to ordinary local coordination, run recurring end-to-end Comms liveness tests with the ships so stalled agents are detected early. This is especially important for Defiant, which currently runs attached to a local console and may become blocked waiting for an interactive `ask`.

## Required behavior

1. During every normal coordination cycle (the fleet's current approximately five-minute cycle), run a real Comms round-trip probe with each active ship, at minimum Defiant and Laforge. Include any other ship currently assigned active work.
2. Use the established local Comms API and actual configured ship identity/route. Do not simulate success by writing a file, reading a local outbox, or observing a process/PID.
3. Each probe must have a unique nonce/correlation ID and explicit send timestamp. Require an explicit response through Comms that echoes the nonce. Record send result, Comms message ID, response message ID, elapsed time, and outcome.
4. Use a bounded response deadline (default 60 seconds unless the existing Comms protocol has a stricter documented timeout). Distinguish: responded; send failed; delivered but no reply; timed out; route/identity unknown; ship reports waiting for input/approval; ship appears blocked. Do not label a ship hung solely because it did not answer within one short interval.
5. For Defiant, specifically determine whether an outstanding `ask` or other interactive prompt is blocking progress. Ask it to report current task, current execution state, whether it is awaiting user input, and the exact prompt/correlation ID if so. Do not approve or bypass any security-sensitive/destructive action on its behalf.
6. If a ship misses a probe, retry once through the real Comms route, then report the evidence and escalate/nudge through the existing coordination channel. Do not create duplicate tasks for every missed heartbeat.
7. Keep the tests recurring, not one-off: record a concise timestamped result in Wormhole outbox each cycle or append to the established durable health report. Include a short rolling summary (last result, consecutive misses, last successful round-trip) for each ship.
8. Continue the existing local coordination duties and report W0-059's exact Laforge PR URL/repository/number/head/base metadata when available. Do not perform technical audits or claim a Comms test passed without a correlated reply.

## First execution and reporting

Run an initial probe of Defiant and Laforge immediately after receiving this task. Report exact Comms IDs, nonce, timestamps, response latency, and any Defiant ask state. Then maintain the recurring checks on subsequent coordination cycles and leave durable reports in `outbox/`.

Enterprise remains coordination-only. ChatGPT/project owner performs all technical audits and acceptance.
