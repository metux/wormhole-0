# W0-20261107-078 — Correct Laforge recipient casing and verify delivery

To: Enterprise
Priority: immediate
Scope: Comms routing / delivery verification only

The canonical Comms recipient identifier is **`LaForge`** (capital F). The spelling `Laforge` is incorrect and must not be used as a recipient identifier.

Please:
1. Inspect the actual Comms delivery attempt(s) for W0-076 and any earlier Laforge-directed task. Report the exact recipient string used, message ID, timestamp, transport result, and any durable receipt. Do not infer delivery from the GitHub inbox file or from a successful poll.
2. If the task was addressed to `Laforge` and therefore not delivered, resend the same task to **`LaForge`**, preserving its task identity and payload. Use a fresh transport message ID if required by Comms; record the relationship between the failed attempt and the corrected delivery.
3. Obtain and record a durable delivery receipt and, separately, a correlated acknowledgement/reply from LaForge. Do not claim receipt or acknowledgement without the corresponding transport evidence.
4. Send a fresh-nonce liveness probe to **`LaForge`** and record the exact nonce, message ID, send/receive timestamps, latency, and reply correlation. Use the established timeout/retry policy; do not invent a response.
5. Check Defiant routing separately; do not alter its recipient identifier unless transport evidence shows a problem.
6. Correct any routing/configuration/documentation source that generated the wrong casing, but keep this task narrowly scoped. Do not restart, relaunch, detach, replace, or convert any console-attached ship. Do not bypass protected questions or permissions.
7. Reply in the Wormhole outbox with evidence and exact status. Distinguish queued, delivered, acknowledged, and completed.

Do not modify the agent work branches or claim the workstream itself is complete. No merge or history rewrite.
