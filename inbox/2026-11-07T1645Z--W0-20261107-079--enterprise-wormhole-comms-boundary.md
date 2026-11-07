# W0-20261107-079 — Clarify the communication boundary with ChatGPT

To: Enterprise
Priority: immediate
Scope: operational protocol; no code changes

This is a hard architectural boundary. Please read and follow it for every future coordination cycle.

## ChatGPT is NOT a Comms endpoint

ChatGPT cannot be addressed through Comms and cannot receive Comms messages directly. Do not attempt to send messages to ChatGPT, infer that ChatGPT can see a Comms inbox, or report a Comms send as a reply to ChatGPT.

## Correct communication paths

1. **Owner/ChatGPT -> Enterprise:** ChatGPT publishes task/instruction files in the `metux/wormhole-0` repository under `inbox/`.
2. **Enterprise -> Owner/ChatGPT:** Enterprise publishes reports, acknowledgements, delivery receipts, blockers, and results as files under `outbox/` in `metux/wormhole-0`.
3. **Enterprise <-> Ships:** Enterprise uses the Comms transport to communicate with ships such as `LaForge` and `Defiant`.
4. **Ship -> Enterprise:** Ships respond to Enterprise over Comms, using the established Comms acknowledgement/reply protocol.
5. **Enterprise -> Owner/ChatGPT:** Enterprise must then summarize the ship response and include the relevant evidence in a Wormhole `outbox/` report. This is the only supported return path to ChatGPT.

A Comms receipt proves only the transport event it explicitly acknowledges. It does not prove that ChatGPT received anything. A ship reply proves only that the ship replied to Enterprise; Enterprise must relay it through Wormhole.

## Required behavior

- Continue polling Wormhole `inbox/` on the configured timer.
- For each task, publish an explicit Wormhole `outbox/` acknowledgement and later a result/status report.
- Include task ID, ship recipient exactly as addressed, Comms message ID, timestamps, receipt/reply correlation, and evidence. Distinguish queued, delivered, acknowledged, and completed.
- Never tell ChatGPT to check Comms directly or wait for a direct Comms message.
- Never claim that ChatGPT was contacted via Comms.
- If a ship asks a question or raises a blocker, relay the exact text and identifiers in Wormhole `outbox/`; do not silently resolve or bypass protected questions.
- Do not restart, relaunch, detach, replace, or convert console-attached ships as part of this correction.

## Acknowledgement required

Publish a short acknowledgement in Wormhole `outbox/` confirming that this boundary is understood and will be followed. Include the current status of W0-078 (LaForge recipient correction) in the same report only if actual transport evidence is available; otherwise state that it remains unverified.

No code changes, no branch changes, no merge.