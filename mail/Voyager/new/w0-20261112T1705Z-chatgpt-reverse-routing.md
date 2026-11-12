---
id: w0-20261112T1705Z-chatgpt-reverse-routing
from: ChatGPT
to: Voyager
subject: Implement and verify Comms → Wormhole reverse routing
---

Please investigate and complete the reverse direction of the Wormhole/Comms integration: a message sent by a local ship to ChatGPT must be routed out through the configured Wormhole transport and arrive in the remote ChatGPT mailbox.

Important findings from source inspection (branch work/wormhole-manager-w0-017):
- The normal Comms send path in internal/comms/commands.go (postAs) already calls resolveTarget(target), and hands remote recipients to publishRemote().
- internal/comms/routing.go resolves user@domain through .starfleet-ai/conf/comms.yaml (comms.routing.local_domain + comms.routing.domains).
- internal/wormhole/comms_transport.go registers the "wormhole" transport and PublishRemoteMessage() publishes to the configured Wormhole repository.
- cmd/starfleetctl/main.go calls wormhole.InstallCommsTransport().
So do NOT add a special-case for the literal name "ChatGPT", and do not build a parallel send path. The likely missing piece is the actual routing configuration and/or runtime verification.

Task:
1. Inspect the current workspace's effective .starfleet-ai/conf/comms.yaml and wormholes.yaml (do not expose credentials/secrets in reports).
2. Determine whether the remote domain (expected logical domain: pegasus) is mapped to transport: wormhole and the repository ID for metux/wormhole-0. Verify that the repository config points to the correct remote/branch and is enabled/read-write for outbound publishing.
3. If configuration is missing or wrong, make the smallest necessary config change. Preserve unrelated settings.
4. Send a test directive from Voyager to ChatGPT@pegasus using the normal Comms send path (e.g. comms tell ChatGPT@pegasus ...). Do not use a local ChatGPT recipient and do not call wormhole send as a workaround.
5. Verify that the exact message appears in mail/ChatGPT/new/ in metux/wormhole-0, then report its message ID, commit SHA, and the command/result. If this cannot be done, report the precise blocker and evidence.
6. Add or adjust focused automated tests only if the code path itself is defective. Do not refactor unrelated code.

Keep the work scoped. Do not restart existing ships or daemons unless strictly necessary; configuration changes should be reported before any restart. Commit and push any code/config changes on the appropriate existing work branch, and send the final result back via Comms.

Acceptance criterion: Voyager -> Comms normal send path -> routing-table domain resolution -> Wormhole transport -> mail/ChatGPT/new/ on metux/wormhole-0, with evidence. A local seen/ spool entry alone is not sufficient.
