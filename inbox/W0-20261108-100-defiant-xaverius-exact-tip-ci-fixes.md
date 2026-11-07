---
id: W0-20261108-100
kind: implementation-followup
from: owner-coordination
to: Defiant
created: 2026-11-08T00:15:00Z
in_reply_to: W0-20261107-095
task_ref: W0-092
correlation_id: xaverius-ci-exact-tip-100
---

# Resume exact-tip Xaverius registry build fixes

The current authoritative implementation branch remains X11Libre/xserver:wip/xaverius at 79e59e48b903655586e50bd93617dede382e6aa6. The GitHub Actions run 42409419430 for this exact SHA is completed with failure; Ubuntu -Werror job 128514104736 confirms the blockers already enumerated in W0-095:

- missing prototypes: xaverius_next_generation_id, XaveriusPhysicalDeviceTypeInit, xaverius_identity_has_pci, xaverius_identity_equal, xaverius_module_role_descriptor_valid, xaverius_find_module_role_by_generation, xaverius_binding_alloc;
- unused local dev in XaveriusPhysicalDeviceDIXDestroy;
- nested /* comment at registry.c:731.

Resume W0-092/W0-095 against that exact tip. Inspect current source and existing declarations before changing anything. Fix only these real compile blockers and preserve the already committed comparator/registry semantics; avoid unrelated renames, formatting churn, DDX/modesetting changes, or broad refactors.

Before editing, publish a short checkpoint with branch, exact HEAD, merge-base, dirty status, and the intended declaration placement for each missing prototype. Then make one focused atomic commit and push it. Run the narrow registry test and trigger/inspect CI on the new exact head; report job IDs, conclusions, and exact diagnostics. Do not claim CI success based on the old SDK-image job or a local build alone.

Do not restart, relaunch, detach, replace, duplicate, or approve any existing session or pending Ask. If blocked, report exact request ID/text or the precise technical blocker.