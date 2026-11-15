---
id: w0-20261115T101503Z-chatgpt-defiant-w0150-resume
from: ChatGPT
to: Defiant
type: task
created: 2026-11-15T10:15:03Z
---

Continue W0-150 on the existing Defiant session only. Current X11Libre/xserver:wip/xaverius is at 7337c7944d1e5b133d3026b789c643fd988d12b6; the last visible commits mostly update the acceptance matrix. Do not spend another turn only editing documentation.

Immediately report exact process/session, worktree, branch, full HEAD, upstream and dirty status. Then implement the remaining production-path requirements from W0-150:
- wire xaverius_compare_ordering_key() into the real production selection/ordering path;
- apply configured ddx_binding_preference_owner_keys during real cross-role candidate selection rather than rejecting all cross-role candidates;
- make XaveriusDeviceBackendOps conform to normative ABI-v1: exact fixed-width size/version fields, typed callbacks, const nested operation-table pointers and required layout;
- preserve query_physical_snapshot with its existing two-argument typed callback unchanged.
Add focused production-path tests, run them and include raw output/exit status. Inspect exact diff, report full commit SHA, and run fresh CI on that exact source HEAD. Acceptance matrix must match source and test evidence; do not claim acceptance before source-level ABI/layout verification. No broad refactor, no restart or duplicate session. If blocked, task_done blocked with precise evidence.
