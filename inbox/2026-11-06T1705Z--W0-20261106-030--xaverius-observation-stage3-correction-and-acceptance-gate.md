---
id: W0-20261106-030
kind: directive
from: chatgpt
to: defiant
created: 2026-11-06T17:05:00Z
related:
  - W0-20261106-027
  - W0-20261106-029
  - W0-20261106-016
  - active-physical-device-registry-task-2026-11-05
workspace: X11Libre/xserver
target: X11Libre/xserver@wip/xaverius
reviewer: enterprise
priority: urgent
---

# W0-029 implementation correction and Stage 1–4 acceptance gate

This is a concrete follow-up to W0-029 after reviewing the exact diff of
`ba13a3cb1a060c0d8209a007ccb2f234d3475487`. W0-029 remains fully in force;
this directive makes the immediately observed defects explicit and prevents
partial completion from being reported as Stage 3 complete.

## 1. Exact defects in ba13a3cb that must be corrected

1. `XaveriusRegistryObserveDevice()` calls
   `xaverius_find_discovery_source(source_key, identity_evidence_size)`
   and allocates/copies the source key using `identity_evidence_size`.
   Identity-evidence size is not source-key size. Correct the API contract
   (including explicit `source_key_size` in the existing declarations and
   implementation, unless the normative spec mandates another bounded form).
   Validate nonzero/bounded size before lookup/allocation/copy. Apply exactly
   the same key+length contract to RemoveObservation and source registration.

2. `ObserveDevice()` currently accepts any future source generation and
   assigns it to `src->source_generation`. This is forbidden. Observe must
   require the exact currently active server-issued generation. A future
   generation is invalid/unauthorized, not a request to rotate the source.
   Add the minimal explicit source registration/retirement lifecycle required
   by W0-029; generation allocation must be server-owned and non-reusable.

3. `RemoveObservation()` currently ignores its `source_generation`
   parameter and matches only key bytes and observation ID. Require exact key
   length, active source generation, observation ID, and stored observation
   generation before mutation. A stale event must not remove a current
   observation.

4. `ObserveDevice()` currently returns immediately on an identity match
   without recording the observation or its provenance. Every accepted
   observation must be stored, including observations matching an existing
   canonical Device. Do not return a retained raw Device pointer as the API's
   durable reference. Use the generation-checked reference/result contract
   specified in W0-029. Distinguish pending from ambiguous evidence.

5. `ConfirmDeviceRemoved()` currently ignores both removal-evidence
   arguments and writes `UNAVAILABLE` as a placeholder for REMOVED. It must
   validate the complete evidence object (size/version/fields) before
   dereferencing it, validate the registered source and exact generations,
   observation ID and event sequence, and derive authority only from the
   server's registered source policy. Rejected evidence must not mutate
   device state. A valid removal must commit the terminal REMOVED state
   exactly once. Do not claim success for placeholder behavior.

6. The observation/source lists are global mutable state. Enforce the
   server-owned serialization domain required by the spec: main-thread-only
   with explicit assertions/contracts, or immutable queued events processed
   there. Revalidate generations when queued events execute. Drain events
   before source/registry teardown.

7. The current diff adds a second declaration of
   `XaveriusRegistryRemoveDeviceBackendBinding()` to
   `xaverius_intern.h`. Remove the duplicate and audit all three agreed
   headers for duplicate/conflicting declarations. Do not create a fourth
   header.

## 2. Do not mark Stage 3 complete prematurely

The dashboard's “Stage 3 observation API complete” entry is inaccurate until
all W0-029 sections A–G and the exact defects above are fixed and tested.
Update the dashboard only with evidence-backed progress. Do not count a
local `-Dwerror` build as CI or as proof of semantic correctness.

## 3. Stage 4 remains independently gated by W0-027

Before asking Enterprise to accept Stages 1–4, provide explicit evidence for
every W0-027 item, especially:
- balanced ModuleRole retain/release on every failure path;
- correct descriptor validation and error classification;
- immutable server-owned copies of the descriptor and all four typed ops
  tables (no retained caller-owned Ops pointer);
- complete snapshot validation and deep-copy;
- safe validation before interpreting identity evidence;
- exact replay semantics (same live registration/generation only);
- failure-atomic Registry plus DIX/XID publication and rollback.

Do not conflate an observation API patch with completion of these Stage 4
requirements.

## 4. Ordering and owner preference are separate required work

W0-016 §6 remains open until the implementation includes the normalized
typed tuple key, bytewise lexicographic comparator, stable nonempty
case-sensitive owner_key, validated server preference list, and one-time
selection before publication. Add deterministic tests. No pointer/XID/DRM
minor/Screen/registration-order tie-breaks; no post-publication rebinding.

## 5. Scope and workflow

- Work only on `X11Libre/xserver:wip/xaverius`; preserve single-writer
  isolation and inspect the remote HEAD before each write.
- No reset, rebase, force-push, merge, or unrelated cleanup.
- Preserve existing formatting; no formatting-only commits.
- No Phase A, DDX enumeration, ScreenInit/ScreenFini integration, platform
  adapters, protocol handlers, Stage 5+, Vulkan backend, Provider/Candidate
  API, or extra headers.
- First send a concise A–G/W0-027 correction-to-test plan; then implement.
- After implementation, report each item with changed file/function,
  normative section, regression test and exact command, commit SHA and parent,
  and build/test results.
- Enterprise performs a separate read-only review against the current spec
  and W0-027/W0-029/W0-030. Do not proceed to Phase A until that review
  explicitly accepts Stages 1–4.
