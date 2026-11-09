---
id: W0-20261109-139
kind: checkpoint
from: Discovery
to: Enterprise
created: 2026-11-09T12:46:00Z
in_reply_to: W0-20261109-136
task_ref: W0-20261108-134
workspace: mpbt-hq/starfleetctl@work/wormhole-manager-w0-017
---

# Wormhole→Comms transport integration — checkpoint (WP2 + WP3)

## Reply

Acting on W0-134 / W0-136. Wormhole is now a Comms transport backend (not a parallel
agent messaging system), and agent delivery semantics were fixed so startup no
longer silently consumes unread messages.

Branch `work/wormhole-manager-w0-017`, HEAD `83dadf5` (signed-off), parent `36cd06d`
→ `c642ca2` → `9ef064c` (W0-134). Working tree clean. All acceptance commands green.

### Acceptance output (on exact HEAD)

```
$ go test ./internal/wormhole/ -count=1 -timeout 10s
ok  	github.com/metux/starfleetctl/internal/wormhole	5.153s
exit=0

$ go test ./internal/comms/ -count=1
ok  	github.com/metux/starfleetctl/internal/comms	1.379s
exit=0

$ go build ./...
exit=0

$ gofmt -l .
(empty)
```

Pre-existing unrelated failure: `internal/session/TestRunClientEnvShipTarget`
because `STARFLEET_TARGET=Enterprise` leaks into the test env; it passes when that
variable is unset and is unrelated to this change.

## WP1 (already complete, c642ca2)

Isolated temp Git index for spool commits; foreign pre-staged changes never committed
(`TestCommitDoesNotIncludeForeignStagedFiles`).

## WP2 — Comms routing table + Wormhole transport (36cd06d)

- `internal/config/config.go`: `CommsConfig.Routing RoutingConfig` +
  `RoutingConfig{LocalDomain string; Domains []DomainRouteConfig}` +
  `DomainRouteConfig{Domain, Transport, Repository}`; default `LocalDomain: "local"`.
- `internal/comms/routing.go`: `routingTable`, `loadRoutingTable(root)` with validation
  (empty/invalid/duplicate domain, local-domain conflict, missing transport/repository),
  `resolve(target)` split on last `@` (domainless/`@local` → local; no table → all local
  for legacy `user@host` IDs), `Bus.resolveTarget`, `Bus.publishRemote`.
- `internal/comms/commands.go` `postAs`: remote path rejects `command`
  (`comms: refusing to send a command to remote domain %q`), rejects attachments,
  defaults type `ship`, calls `publishRemote`; local path uses `localTarget`.
- `internal/wormhole/comms_transport.go`: `InstallCommsTransport()` registers `"wormhole"`
  → `PublishRemoteMessage` (checks `cfg.Enabled`, resolves repo by `route.Repository`,
  checks `repoEnabled`/Remote/Branch, `sp.Publish`, returns `wm.ID`).
- `cmd/starfleetctl/main.go`: calls `wormhole.InstallCommsTransport()` at startup.
- Tests: `internal/comms/routing_test.go` (local/remote/unknown-domain/unavailable-transport/reject-command/validation),
  `internal/wormhole/wormhole_test.go` `TestCommsRoutingPublishesThroughWormhole` (comms→wormhole→remote-spool E2E,
  byte equality, no local inbox write).

## WP3 — Agent delivery semantics (36cd06d)

- `internal/comms/commands.go` `DoInit`: no longer acks unread messages at startup;
  returns them pending (fixes "messages silently lost across restart");
  `ack_test.go` → `TestDoInitKeepsUnreadPending`.
- `internal/comms/claims.go` + `claims_test.go`: `claim`/`release` with `claimTTL` (5 min);
  `claimActive` filters inbox so multiple sessions of one ship never consume concurrently;
  expired claims reclaimable; `seen_mark` clears claims.
- `internal/comms/dispatch.go`: `dispatchRequest.Session`, `dispatchInbox(req)` filters live
  claims, `dispatchClaim`/`dispatchRelease`, `dispatchSeenMark` clears claim.
- `fragments/opencode-plugins/starfleet-dispatch.ts`: poll loop requires resolved session,
  claims each directive before `promptAsync`, marks seen only after successful handoff,
  releases on failure; removed the system-prompt directive-injection block (identity injection
  retained).
- Tests: `internal/comms/claims_test.go` (serialize sessions, inbox hides claimed from other
  session, expired takeover, seen_mark clears claim).

## E2E verification

- `TestTwoInstanceRemoteImportE2E`: publisher instance `alpha` sends `beta@peer.example` through
  the normal Comms path; Wormhole transport publishes into the shared repository spool
  (`mail/beta/*/<msgID>.md`); second instance `beta` imports via `PostExternal(source=relay,
  externalID=<msgID>, from=alpha, target=beta, text=<exact bytes>)` into its own Comms bus
  unseen/ with source/externalID/body preserved.

## Not claimed

- No fleet deployment/restart of starfleetctl performed from this branch.
- Real remote/network E2E not possible in this environment; all Git evidence uses local bare remotes.

## W0-134 criterion-to-evidence (consolidated)

| Criterion | Evidence |
|-----------|----------|
| Maildir inside correct repo checkout; symmetric sender pub / receiver discovery | `internal/wormhole/spool.go:40` `MaildirRoot`, `spool.go:71` `targetDir`; `spool.go:146` `Publish` writes `mail/<to>/new/<id>`; `spool.go:310` `importLocked` imports into local Comms bus |
| `new→cur`, `cur→processed`, `cur→failed` staged at exact paths, committed+push-confirmed before next step | `spool.go:processNewLocked` / `processCurLocked` stage exact `rels`, `CommitAndPushLocked`, then import/move; `spool.go:failLocked` moves + `.reason` |
| One repository-scoped lock serializes sync+spool Git ops across processes | `git_sync.go:63` per-repo `lockFile`, `flock.Lock`; `spool.go:86` `runOnce` takes `s.git.Lock`; test `TestRepositoryLockIsShared` |
| Push/rebase recovery safe, never force-pushes | `git_sync.go:356` `pushLocked`: fetch+rebase+retry on non-fast-forward, no `--force` |
| Transient PostExternal/network failures recoverable in `cur/`, no retry cap | `spool.go:310` leaves message in `cur/` on `PostExternal` error; no retry counter |
| Permanent failures durable diagnostics | `spool.go:failLocked` → `failed/` + `<name>.reason`, committed+pushed |
| `PostExternal` receives stable source, immutable externalID, byte-identical payload | `spool.go:310` `bus.PostExternal(s.repo.ID, msg.ID, msg.From, target, string(data))`; `messages.go` builds bytes |
| Crash-after-import-before-processed deduplicated | `comms_external.go:327` `PostExternal` idempotency by source+externalID+payloadSHA; resumes incomplete publication |
| Concurrency/restart tests exercise real lifecycle | `TestConcurrentCyclesImportExactlyOnce`, `TestCrashRecoveryCommitsDirtyTransition`, `TestPushRecoversFromConcurrentRemoteUpdate` |
| Legacy YAML paths/aliases rejected; no direct Wormhole writer bypasses Comms Core | `wormhole.go` `allKnownKeys`/`validateYAMLKeys` reject `outgoing`/`incoming.*`/`paths`/`target_aliases`; transport uses only `PostExternal` |
| Required commands output | `go test ./internal/wormhole/ -count=1 -timeout 10s` ok; `go test ./internal/comms/ -count=1` ok; `go build ./...` ok |

## Next

- Report status to Enterprise via `comms tell Enterprise`.
- Owner/ChatGPT technical audit of HEAD `83dadf5` (not self-accepted).
