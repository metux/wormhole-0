# X.Org Server security audit — open TODOs

Snapshot: 2026-11-17. Companion matrix: [xorg-server-cve-matrix.md](./xorg-server-cve-matrix.md). Wiki publishing plan: [xorg-server-cve-wiki-publishing.md](./xorg-server-cve-wiki-publishing.md).

## P0 — close currently confirmed exposure

1. **CVE-2026-109093 / XTestFakeInput — all four branches.**
   - Rework the fix as a minimal bounds-check-only patch; validate before every array write.
   - Preserve the correct `firstValuator >= dev->valuator->numAxes` rejection and the `MAX_VALUATORS` bound.
   - [#4116](https://github.com/X11Libre/xserver/pull/4116) is the least invasive starting point, but verify its exact head/CI and normal + swapped request paths.
   - Do not merge [#4142](https://github.com/X11Libre/xserver/pull/4142) as-is: its diff is over-broad and its boundary check is wrong.
   - Do not merge [#4161](https://github.com/X11Libre/xserver/pull/4161) as-is: it repeats the off-by-one boundary concern.
   - Create a minimal master fix as well; the old commit [335fffed](https://github.com/X11Libre/xserver/commit/335fffed92d1e3ac5e12ff71ad57f1c7906c7a07) does not prove the current code safe.

2. **CVE-2026-109092 / XvShmPutImage — master and 25.2.**
   - Port the proven shared-memory bounds arithmetic from the merged [#4118](https://github.com/X11Libre/xserver/pull/4118) (25.0) / [#4143](https://github.com/X11Libre/xserver/pull/4143) (25.1) changes.
   - Add/verify regression tests for offset, size overflow and segment boundary cases.

3. **CVE-2026-109096 / XvPutImage — master and 25.2.**
   - Port the negative-size rejection from [#4126](https://github.com/X11Libre/xserver/pull/4126) (25.0) and [#4149](https://github.com/X11Libre/xserver/pull/4149) (25.1).

4. **CVE-2026-109903 / XvQueryImageAttributes — master and 25.2.**
   - Port the negative-size/reply-length protection from [#4127](https://github.com/X11Libre/xserver/pull/4127) (25.0) and [#4150](https://github.com/X11Libre/xserver/pull/4150) (25.1).

5. **CVE-2026-93536 / GestureBuildSprite UAF — master and 25.2.**
   - Port the Gesture sprite-trace cleanup from [#3893](https://github.com/X11Libre/xserver/pull/3893).
   - Check WindowGone/destruction ordering and add a regression test if feasible. Do not substitute the unrelated device-sprite animation fix [c5453462](https://github.com/X11Libre/xserver/commit/c5453462eac3d801f8b26e482859da3df4369701).

6. **CVE-2026-109091 / EDID DS_RANGES — master.**
   - Port the bounds guard already merged to 25.0/25.1/25.2 via [#4120](https://github.com/X11Libre/xserver/pull/4120), [#4144](https://github.com/X11Libre/xserver/pull/4144), [#4155](https://github.com/X11Libre/xserver/pull/4155).
   - Complete and review the open regression tests [#4119](https://github.com/X11Libre/xserver/pull/4119) and [#4154](https://github.com/X11Libre/xserver/pull/4154); add master coverage.

7. **CVE-2026-50263 / DRI2 GetBuffers — 25.2.**
   - Backport the safe allocation sizing from master commit [c03bc789](https://github.com/X11Libre/xserver/commit/c03bc789aed3fdf72af0bf0d2ec0d3cf073ecdf8).
   - Inspect the older DRI2 implementations in 25.0/25.1 separately; their source layout differs, so do not infer their status from 25.2.

## P1 — resolve pipeline PRs and validate merged fixes

8. **XRes PR disposition:** [#4141](https://github.com/X11Libre/xserver/pull/4141) (25.1) and [#4153](https://github.com/X11Libre/xserver/pull/4153) (25.2) need exact-head rechecks before disposition. [#4169](https://github.com/X11Libre/xserver/pull/4169) (master, CVE-2026-109094) is rejected as submitted: its diff rewrites large parts of `Xext/xres/xres.c` (dispatch, reply serialization, client/resource handling, fragment management and includes) for a narrowly scoped request-size issue. Current master source blob `d891685e` already uses `uint64_t` in the `REQUEST_FIXED_SIZE` multiplication. Do not merge or salvage the old patch. Voyager is assigned to re-check the exact swapped path and advisory, then create a completely fresh, minimal fix/test PR only if a concrete defect remains. The old PR should be closed as obsolete once the replacement or no-fix conclusion is documented.

9. **CVE-2026-109093 tests:** test high `firstValuator`, `numAxes` equality boundary, `MAX_VALUATORS` overflow, multiple DeviceValuator records, and both swapped/normal requests. Verify valid last-axis boundary still works.

10. **EDID test PRs:** inspect CI and exact current diff for [#4119](https://github.com/X11Libre/xserver/pull/4119) and [#4154](https://github.com/X11Libre/xserver/pull/4154); verify the test catches both hsync and vrefresh array overflow and does not only exercise a helper in isolation.

11. **All recently merged CVE backports:** verify the current branch tips still contain the merged commits and run the relevant focused tests. The merged PRs include #4111/#4120/#4113/#4114/#4118/#4126/#4127 on 25.0; #4140/#4143/#4144/#4149/#4150 on 25.1; #4152/#4155 on 25.2. This is a validation task, not a reason to reopen known-good changes without a concrete regression.

## P2 — finish the historical audit

12. Finish 2025 CVE coverage for **all four** branches, especially:
    - [CVE-2025-26597 fix commit](https://github.com/X11Libre/xserver/commit/0e4ed94952b255c04fe910f6a1d9c852878dcd64)
    - [CVE-2025-26599 fix commit](https://github.com/X11Libre/xserver/commit/c1ff84bef2569b4ba4be59323cf575d1798ba9be)
    - Verify the already sampled CVEs 2025-62229..62231, 2025-49175..49178, 2025-26594..26596, 2025-26598, 2025-26600..26601 in release/25.0 and release/25.1, and confirm exact current source in 25.2/master.
13. Complete individual CVE-to-PR mapping and regression-test coverage for the April/June/July 2026 advisory groups. June's DRI2 variant must remain a separate row.
14. Continue the advisory inventory back through 2024 and earlier. Include only X.Org server advisories in this repository scope; keep libX11/libXi/libXpm/libXrender/libXtst/libXfont2 advisories in a separate client-library audit.

## P3 — publishing and process reliability

15. **Publish the matrix to the X11Libre GitHub Wiki.** The dedicated plan is [xorg-server-cve-wiki-publishing.md](./xorg-server-cve-wiki-publishing.md). This is queued as a follow-on task for Voyager's existing session, after its current M1 assignment reaches explicit `task_done`. The task must clone `X11Libre/xserver.wiki.git` as a separate project, inspect for an existing matrix page, update rather than duplicate, push normally (no force-push), and verify the remote commit and rendered page. The current Wormhole matrix is incomplete historically; the Wiki must state coverage gaps rather than overclaim completeness.

16. Repair the failed Voyager CVE audit task envelope before retrying. The previous message failed because the YAML `subject` contained an unquoted colon:
    `subject: X11Libre: CVE-first audit...`
    Quote the value (or avoid the colon), validate front matter before publishing, then verify receipt/processing in `mail/Voyager/{new,cur,processed,failed}`. The failed message is archived at [failed task](https://github.com/metux/wormhole-0/blob/master/mail/Voyager/failed/w0-20261116T193000Z-voyager-x11libre-cve-backport-audit-01.md) and [failure reason](https://github.com/metux/wormhole-0/blob/master/mail/Voyager/failed/w0-20261116T193000Z-voyager-x11libre-cve-backport-audit-01.md.reason).
17. Require explicit `task_progress` and terminal `task_done(completed|blocked|yielded)`; do not infer completion from a pushed commit, message delivery, or a processed marker alone.
18. Record tested HEAD, exact commands, full exit status/output, and CI run links for every security fix. Distinguish source inspection, unit/regression tests, sanitizer/reproducer tests, and CI.
19. Keep the role boundaries: Defiant = Xaverius; Discovery = Starfleetctl; Voyager = assigned X11Libre audit/backport work only when explicitly dispatched; Enterprise = coordination/Comms, not technical audit. Do not create duplicate ship sessions or restart existing console ships.

## Acceptance criteria for calling this audit complete

- Every X.Org server advisory in the chosen date range is inventoried, with component applicability recorded.
- Every applicable CVE has a per-branch source assessment at an exact HEAD.
- Every still-open exposure has a minimal fix PR or an explicit owner/blocker.
- Every proposed backport has a focused regression test and recorded CI/test evidence.
- Redundant/obsolete PRs are identified and dispositioned.
- Wormhole delivery/processing is verified independently from task completion.
