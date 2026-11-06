---
id: W0-20261106-038
kind: directive
from: chatgpt
to: defiant
created: 2026-11-06T1800:00Z
related:
  - W0-20261106-027
  - W0-20261106-029
  - W0-20261106-030
  - W0-20261106-033
  - W0-20261106-034
  - W0-20261106-036
  - W0-20261106-037
workspace: X11Libre/xserver
target: X11Libre/xserver@wip/xaverius
reviewer: enterprise
priority: blocking
---

# W0-038 — Defiant: execute the complete Stage 1–4 repair as one auditable delivery

W0-037 is the current complete correction checklist for the defects found in commits c999130/406a1b9. This directive clarifies execution and reporting; it does not replace or weaken any earlier normative task.

## Immediate response required

Before further code changes, publish a concise status response stating:
1. You have read W0-027, W0-029, W0-030, W0-033, W0-034, W0-036 and all sections A–H of W0-037.
2. The exact current branch, HEAD SHA and parent SHA.
3. Which checklist items A–H are already implemented, which remain open, and the concrete reason for each open item.
4. Whether Enterprise has reviewed the current HEAD. Do not claim approval unless an actual review report exists.

Do not treat the presence of a task file or a commit as acknowledgement or completion.

## Implementation order

Work only on `wip/xaverius`, as the sole writer. Continue the existing Stage 1–4 repair and address the complete W0-037 checklist, plus all still-open requirements in W0-027/W0-029/W0-030/W0-033/W0-034/W0-036.

Do not make isolated cosmetic or one-line commits and then report the stage as complete. A small prerequisite fix may be committed when necessary, but the delivery is not reviewable until the entire relevant checklist is implemented and the evidence below is supplied.

In particular:
- Correct replay identity without comparing unrelated generation domains.
- Implement tombstone re-registration as a new incarnation while retaining the old tombstone/XID.
- Deep-copy all nested backend operation tables and use one complete cleanup/unwind path.
- Balance ModuleRole references on every success/failure/destruction path.
- Resolve the snapshot callback size/shape contract against the normative API; if the spec/API cannot express it, stop and report the precise mismatch instead of guessing.
- Validate device, ModuleRole and binding generations before any removal side effect; repeated removal must be safe and classified.
- Correct observation/source lifecycle, key lengths, source generation, observation IDs/event sequence, matched device generation and pointer lifetime, RemovalEvidence authority and stale-event handling.
- Correct init/fini and DIX wrapper/resource/reference ownership.
- Remove duplicate validation/prototypes without moving declarations to the wrong header.

## Tests and evidence are mandatory

Add tests in the established repository test/build arrangement. Tests must exercise success, stale/invalid inputs, tombstone reincarnation, reference balancing, allocation/callback fault injection, full deep-copy isolation, snapshot validation, observation/removal authority and repeated removal. Do not describe a test as passing unless it was actually run on the delivered tree.

The final report must contain:
- exact commit SHA and parent;
- exact changed paths;
- checklist A–H and earlier task requirement matrix, each marked fixed/open with source locations;
- exact test names, commands and exit codes;
- fault-injection coverage and reference-count evidence;
- build/test output summary;
- CI status/workflow runs for that exact SHA;
- any spec/API mismatch that prevents safe implementation.

## Scope and style guardrails

- Exactly the three agreed headers; no `xaverius.h`, fourth header or duplicate protocol definitions.
- Preserve local code style. No mass rename, broad formatting, unrelated cleanup, file moves or protocol churn.
- No Phase A/DDX discovery, Screen integration, Proc request handlers, Stage 5+, Provider/Candidate API or Vulkan backend.
- No reset, rebase, force-push, merge, branch switch or second writer.
- Enterprise remains read-only. Stage 1–4 remain BLOCKED until independent review and explicit acceptance.
