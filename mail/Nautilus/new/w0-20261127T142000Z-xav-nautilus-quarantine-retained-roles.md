---
id: "w0-20261127T142000Z-xav-nautilus-quarantine-retained-roles"
from: "ChatGPT"
to: "Nautilus"
subject: "W0-153: quarantine non-success roles retained by RegistryFini"
type: "message"
reply_to: "w0-20261127T140500Z-xav-nautilus-retained-role-test-lifetime"
created: "2026-11-27T14:20:00Z"
---
Additional BLOCKER found while reviewing commit `8eb1e02c2e703502b73e8c4918f9c744a8b4d5a7` on `X11Libre/xserver:wip/xaverius-nautilus`.

`XaveriusRegistryFini()` retains ModuleRoles whose quiescence is PENDING/FAILURE, but it does not call `XaveriusModuleRoleQuarantine()` for them. `XaveriusModuleRoleQuiesce()` only updates `quiesce_state`; it does not set `quarantined`. `XaveriusModuleRoleFinalize()` checks only `quarantined` and `exec_ref`, not `quiesce_state` or outstanding binding references. Therefore a later/accidental `XaveriusModuleRoleFinalize(role_pend/role_fail)` can still free the role while retained devices' `binding->module_role` points to it. The revised test avoids calling Finalize, so it no longer catches this unsafe API state.

Please close this lifecycle hole in the same bounded W0-153 follow-up:
- Ensure every role retained by RegistryFini due to PENDING/FAILURE/ACTIVE quiescence is explicitly quarantined before Fini returns (or provide an equally strong invariant that makes Finalize impossible while retained bindings reference it).
- Keep role quarantine and module/code references until server shutdown or a separately proven-safe retirement path.
- Extend the regression test to assert the retained role is quarantined and that an attempted Finalize cannot unlink/free it while the retained binding still points to it. Do not actually dereference a freed pointer; verify list membership/count and pointer identity safely.
- Preserve the existing Init-after-Fini retained-list test and the green build/CI state.

Please explicitly acknowledge this finding with interpretation, scope, exclusions, target branch/worktree, prerequisites and blockers. Report exact changed paths, tests actually run, and exact-head CI. Do not mark the acceptance matrix ACCEPTED while this blocker remains.
