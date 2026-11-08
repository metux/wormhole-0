---
id: W0-20261108-131
from: starfleetctl
to: LaForge
task_ref: wormhole-manager
type: review-findings
priority: high
---

# W0-131 — Close concrete W0-129 integration gaps

This is a focused correction checklist for the existing W0-129 work, not a replacement task and not authorization to abandon or restart it. Preserve all current work. Keep W0-129 as the active implementation series; do not create a parallel implementation branch.

The current pushed tip inspected by the owner is:
- branch: work/wormhole-manager-w0-017
- HEAD: a9812bf21168ec020a6e7b2dd89fd3d78b926454
- commit: "W0-129: simplify WormholesConfig per architecture decision"
- parent: 9b62d697a076457eeefbd1aa490b26a95af77e13

The commit changes only internal/config/config.go. It is a useful first step, but it does not yet implement the fixed Maildir spool lifecycle.

## Findings that must be addressed

### 1. Update the strict YAML schema allowlist

In internal/wormhole/wormhole.go, allKnownKeys still allows the removed configuration:
- repositories[].outgoing
- incoming.messages
- incoming.decisions
- incoming.messages.paths / target_aliases
- incoming.decisions.paths / target
- outgoing.paths / commit_message

The Go config structs no longer contain these fields, but the custom YAML-node validation still explicitly accepts them. Node.Decode does not by itself make this custom allowlist strict for removed fields.

Update allKnownKeys to exactly match the simplified schema. Add regression tests proving that each removed section/key is rejected, including legacy nested configuration. Also retain positive tests for the new minimal schema.

### 2. Replace the old inbox scanner with the fixed Maildir importer

At the inspected HEAD, processPendingMessages() still scans:
.star­fleet-ai/var/wormholes/<repoID>/inbox/*.md

It calls Bus.PostExternal(), which is the correct Comms boundary, but it does not implement the approved fixed remote spool:
mail/<target-ship>/new/
mail/<target-ship>/cur/
mail/<target-ship>/processed/
mail/<target-ship>/failed/

Implement the W0-129 receiver lifecycle against that fixed layout. Claim new messages into cur before processing, import using PostExternal, and move to processed only after confirmed Comms publication. Permanent malformed/invalid messages go to failed with useful diagnostics. Transient errors remain retryable in cur. Recover safely after a crash at every transition.

Use stable source identity and immutable message ID for PostExternal. Replays must use the exact same source, externalID and payload bytes. A PostExternal success is Imported only, not agent/session delivery or task completion.

### 3. Audit and eliminate direct Comms-record writers

internal/wormhole/messages.go still exports RouteMessage(), which writes directly to:
.star­fleet-ai/var/comms/msgs/<ship>/unseen/

That bypasses Bus.PostExternal and violates the approved ownership boundary if it is reachable. Search all call sites and tests. Remove the bypass or refactor every relevant path to use the Comms API. Do not leave an alternate writer that can be accidentally reused. If a legacy function is intentionally retained for compatibility, demonstrate that it is not used by Wormhole transport and document why it is safe; prefer removal if there are no required callers.

### 4. Add tests for the actual new lifecycle

The existing tests named TestRouteMessage and TestProcessPendingMessagesPartialFailure do not prove the fixed Maildir lifecycle.

Add focused tests for:
- new -> cur claim;
- successful PostExternal -> processed;
- transient PostExternal failure leaves cur retryable;
- malformed/permanent failure -> failed with diagnostic;
- crash/restart after claim;
- crash/restart after PostExternal but before processed move (retry must deduplicate);
- exact payload preservation and stable external identity;
- two workers cannot claim/process the same message concurrently;
- no direct Comms file writes;
- old removed YAML keys fail closed.

Run and report exact outputs for:
go test ./internal/wormhole/ -count=1 -timeout 10s
go test ./internal/comms/ -count=1
go build ./...

Do not claim success without actual command output from the exact commit being reported.

## Required checkpoint

Reply through the established Comms route with:
- confirmation that W0-131 was read;
- current branch and HEAD;
- which findings are already addressed or in progress;
- any uncommitted changes that must be preserved;
- the next concrete code change.

Then continue implementation. Do not stop at acknowledgement or a plan. Keep commits atomic and push each completed commit. W0-130's Comms transport decision remains binding; the separate OpenCode plugin follow-up remains out of the W0-129 commit series.
