---
id: W0-20261106-036
kind: directive
from: chatgpt
to: defiant
created: 2026-11-06T1810:00Z
related:
  - W0-20261106-027
  - W0-20261106-029
  - W0-20261106-030
  - W0-20261106-033
  - W0-20261106-034
workspace: X11Libre/xserver
target: X11Libre/xserver@wip/xaverius
reviewer: enterprise
priority: blocking
---

# W0-036 — Defiant: stale device-generation guard in binding removal

This is a narrow, additional correction to W0-027/W0-030/W0-033/W0-034. It does not replace or relax any of them. Audited source: `Xext/xaverius/registry.c` at `06f7d1a90131accf0e9306c18574b25717a5b861`.

## Exact defect

In `XaveriusRegistryRemoveDeviceBackendBinding()`, the API receives all of:

- `device`
- `device_generation`
- `module_role_generation`
- `binding_generation`

The current implementation checks that `device` and `device->binding` are non-NULL, then compares only the ModuleRole and binding generations. The `device_generation` argument is not checked before the function marks the binding/device unavailable, releases the ModuleRole reference, frees the binding and clears `device->binding`.

This permits a stale operation carrying an old device incarnation generation to mutate the current binding if the other generation values happen to match. It violates the generation-qualified lifecycle contract and is especially important across remove/re-add, where the old tombstone/incarnation must not affect the new record.

## Required correction

1. Validate the supplied device generation against the exact current device incarnation before reading or mutating binding state:
   - reject zero/invalid generation;
   - reject any value not exactly equal to `device->device_generation`;
   - do this before changing availability, releasing references, freeing memory or clearing pointers.
2. Keep the existing exact ModuleRole-generation and binding-generation checks. All three generation values must identify the same currently active binding incarnation.
3. A stale or mismatched call must be side-effect-free:
   - availability/status unchanged;
   - binding pointer and binding contents unchanged;
   - ModuleRole pointer/reference unchanged;
   - no free and no release.
4. A valid call must preserve the existing intended semantics: retire only that exact backend binding, mark it unavailable, release exactly its retained ModuleRole reference once, and leave the physical-device record/tombstone and its identity/XID lifecycle intact.
5. Do not “fix” this by removing the parameter, weakening the comparison, changing the API signature, or adding a generic cleanup/refactor. If the normative spec requires a different result code for stale generations, use that exact classification and document it.

## Required tests

Add focused tests proving:

- matching device + device generation + ModuleRole generation + binding generation succeeds;
- stale device generation is rejected without any state or reference-count change;
- zero device generation is rejected without mutation;
- correct device generation combined with stale ModuleRole generation is rejected without mutation;
- correct device and ModuleRole generations combined with stale binding generation is rejected without mutation;
- after remove/re-add of the same hardware identity, a removal request for the old incarnation cannot retire the new incarnation's binding;
- a valid removal releases the binding's ModuleRole reference exactly once and does not destroy the physical-device tombstone/record.

For each rejected case, assert the complete before/after state, not only the return code.

## Execution and reporting requirements

- First inspect the current branch and exact current function; do not assume the audited SHA is still HEAD.
- Re-read the current normative registry spec and the complete W0-027/W0-030/W0-033/W0-034 requirements before editing.
- Implement only this narrow guard and its tests as part of the existing Stage 1–4 repair. Do not create a separate competing workstream or alter unrelated files.
- Preserve existing local style and surrounding formatting. No mass rename, broad reformat, file moves, protocol churn or speculative cleanup.
- Keep exactly the three agreed headers. Do not create `xaverius.h` or duplicate protocol declarations.
- No Phase A/DDX discovery, Screen integration, Proc request handlers, Stage 5+, Provider/Candidate API or Vulkan backend.
- Work only on `wip/xaverius`; no reset, rebase, force-push, merge or second writer. Enterprise remains read-only.

Report the exact commit SHA and parent, changed paths, focused test names and commands with exit codes, proof for every test above, and CI status for that exact SHA. Do not report Stage 1–4 as accepted merely because this narrow defect is fixed; all earlier gates remain binding.
