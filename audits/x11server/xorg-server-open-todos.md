# X.Org Server security audit — open TODOs

Snapshot: 2026-11-19. Status index: [xorg-server-cve-matrix.md](./xorg-server-cve-matrix.md). Wiki publishing plan: [xorg-server-cve-wiki-publishing.md](./xorg-server-cve-wiki-publishing.md).

## Current branch tips

| Branch | HEAD |
|---|---|
| master | 168e55f3d18a45e93921c6ffc155d954fb98673d |
| release/25.0 | c926955b948f7311e8f21c33fec8f7ffcf704252 |
| release/25.1 | 32c7f7b54bd9723dc3dbb79bb23ba20bc0269222 |
| release/25.2 | a10ba82d33f50729d3a9785801d7cd6e65b12b21 |

## Current exposure / branch work

1. **CVE-2026-109092 — XvShmPutImage, release/25.2.**
   - Master fix is merged as [#4171](https://github.com/X11Libre/xserver/pull/4171); active implementation is in Xext/xv/xvdisp.c, SingleXvShmPutImage().
   - Release/25.1 [#4143](https://github.com/X11Libre/xserver/pull/4143) and release/25.0 [#4118](https://github.com/X11Libre/xserver/pull/4118) are merged.
   - Release/25.2 still uses the unsafe size-plus-offset check. Prepare a focused backport with safe ordering: reject negative size, reject offset beyond segment size, then compare size against the remaining segment length. Avoid addition before bounds validation.
   - Earlier review of #4171 verified the int / unsigned long / CARD32 arithmetic and guard ordering. Build X servers run [44435763792](https://github.com/X11Libre/xserver/actions/runs/44435763792) succeeded.

2. **CVE-2026-109903 — XvQueryImageAttributes, release/25.2.**
   - Master fix [#4184](https://github.com/X11Libre/xserver/pull/4184) is merged.
   - Release/25.1 [#4150](https://github.com/X11Libre/xserver/pull/4150) is merged; follow-up [#4187](https://github.com/X11Libre/xserver/pull/4187) fixes the rpcbuf ownership error in its error path.
   - Release/25.0 [#4127](https://github.com/X11Libre/xserver/pull/4127) is merged.
   - The release/25.2 implementation still needs the negative-size guard before constructing the reply / reserving or writing reply data. Prepare a branch-specific backport; do not assume master or sibling release merges are present.

3. **CVE-2026-109094 — XRes QueryResourceBytes swap path.**
   - Master is **N/A**: the shared request-dispatch / request-field validation path makes the vulnerable swapped-handler path inapplicable on current master. This is an independent code-path change, not a fix PR; leave [#4169](https://github.com/X11Libre/xserver/pull/4169) unmerged and disposition it as obsolete after confirming with its owner.
   - Release/25.2 [#4153](https://github.com/X11Libre/xserver/pull/4153) and release/25.1 [#4141](https://github.com/X11Libre/xserver/pull/4141) remain open. Verify each against its exact branch tip and review whether the original vulnerable swapped path still exists before merging or revising.
   - Release/25.0 [#4113](https://github.com/X11Libre/xserver/pull/4113) is merged.
   - Master test-only [#4168](https://github.com/X11Libre/xserver/pull/4168) is merged. Full master source review is recorded in [cve-2026-109094-master-recheck.md](./cve-2026-109094-master-recheck.md).

4. **CVE-2026-93536 — GestureBuildSprite stale WindowPtr UAF.**
   - The sampled matrix has master and release/25.2 still open; release/25.1 and 25.0 source-fixed.
   - Recheck exact current branch source, especially WindowGone/destruction ordering, before porting [#3893](https://github.com/X11Libre/xserver/pull/3893).
   - Do not substitute the unrelated device-sprite animation cleanup [c5453462](https://github.com/X11Libre/xserver/commit/c5453462eac3d801f8b26e482859da3df4369701).

5. **CVE-2026-50263 — DRI2 GetBuffers attachment-array overflow.**
   - Master fix is [#3266](https://github.com/X11Libre/xserver/pull/3266), merged as [c03bc789](https://github.com/X11Libre/xserver/commit/c03bc789aed3fdf72af0bf0d2ec0d3cf073ecdf8).
   - Release/25.2 still needs a branch-specific backport. The trigger is the maximum 11 attachment entries without FrontLeft but with BackLeft, requiring a synthesized front buffer.
   - Inspect release/25.0 and 25.1 separately; their DRI2 source layout differs. Do not infer their status from 25.2.

6. **CVE-2026-109093 — XTestFakeInput valuator range: status reconciled.**
   - Master [#2964](https://github.com/X11Libre/xserver/pull/2964), release/25.2 [#3115](https://github.com/X11Libre/xserver/pull/3115), and release/25.0 [#2962](https://github.com/X11Libre/xserver/pull/2962) are merged. Release/25.1 already contained the fix before the fork.
   - The November PR [#4116](https://github.com/X11Libre/xserver/pull/4116) is also merged to 25.0 but is redundant against the earlier fix; a comment proposed reverting it because [24ca991](https://github.com/X11Libre/xserver/commit/24ca9910301217e2d5c0b02958669a9f96dc7cf0) already fixed the issue.
   - Master regression-test PR [#4170](https://github.com/X11Libre/xserver/pull/4170) remains open. It is test-only and does not represent an outstanding security fix.
   - Required boundary cases: high firstValuator, equality with numAxes, MAX_VALUATORS overflow, multiple DeviceValuator records, normal and swapped requests, and a valid last-axis boundary.

7. **Resolved November CVEs — no code work currently queued.**
   - CVE-2026-109091: master [#4174](https://github.com/X11Libre/xserver/pull/4174), 25.2 [#4155](https://github.com/X11Libre/xserver/pull/4155), 25.1 [#4144](https://github.com/X11Libre/xserver/pull/4144), 25.0 [#4120](https://github.com/X11Libre/xserver/pull/4120) merged.
   - CVE-2026-109095: master [#4167](https://github.com/X11Libre/xserver/pull/4167), 25.2 [#4152](https://github.com/X11Libre/xserver/pull/4152), 25.1 [#4140](https://github.com/X11Libre/xserver/pull/4140), 25.0 [#4111](https://github.com/X11Libre/xserver/pull/4111) merged.
   - CVE-2026-109096: master [#4185](https://github.com/X11Libre/xserver/pull/4185), 25.2 [#4180](https://github.com/X11Libre/xserver/pull/4180), 25.1 [#4149](https://github.com/X11Libre/xserver/pull/4149), 25.0 [#4126](https://github.com/X11Libre/xserver/pull/4126) merged. The 25.2 merge commit is [a10ba82d](https://github.com/X11Libre/xserver/commit/a10ba82d33f50729d3a9785801d7cd6e65b12b21).
   - CVE-2026-109904: source protection found on all four branches; 25.0 PR [#4114](https://github.com/X11Libre/xserver/pull/4114) merged.

## Other queued engineering work

1. **Backport [#4040](https://github.com/X11Libre/xserver/pull/4040)** — undefined signed left shift in framebuffer line drawing.
   - Check exact source on release/25.0, 25.1 and 25.2.
   - If the vulnerable expression remains, prepare one minimal PR per affected branch.

2. **Repository-wide C undefined-behavior scan.**
   - Check signed shifts, out-of-range shift counts, signed overflow, unsafe signed/unsigned conversions and similar integer UB.
   - Separate reachable bugs from benign patterns; verify each candidate against exact branch source.
   - Avoid broad cleanup.

## Historical audit

- Complete 2025 coverage across all four branches, especially [CVE-2025-26597 fix](https://github.com/X11Libre/xserver/commit/0e4ed94952b255c04fe910f6a1d9c852878dcd64) and [CVE-2025-26599 fix](https://github.com/X11Libre/xserver/commit/c1ff84bef2569b4ba4be59323cf575d1798ba9be).
- Verify sampled CVEs 2025-62229..62231, 2025-49175..49178, 2025-26594..26596, 2025-26598 and 2025-26600..26601 in release/25.0 and release/25.1.
- Finish individual CVE-to-PR mapping for the April, June and July 2026 advisory groups. Keep CVE-2026-50263 separate from the rest of the June group.
- Continue the advisory inventory through 2024 and earlier. Keep client-library advisories out of this xorg-server audit.

## Process / publishing

- Publish the matrix to the [X11Libre GitHub Wiki](./xorg-server-cve-wiki-publishing.md). Update an existing page rather than creating a duplicate; do not force-push.
- Record exact branch HEADs and keep source findings separate from PR state. A merged PR is not by itself proof that every branch contains the fix.
- Regression coverage is expected on master. Release-branch test backports are optional; the security fix itself is not optional on an affected release branch.
- For each active fix, record the exact target branch, source rationale, review status, test/CI evidence and resulting PR/merge commit here, not in the status matrix.

## Acceptance criteria

- Every X.Org server advisory in the selected date range is inventoried with applicability recorded.
- Every applicable CVE has a per-branch source assessment at an exact HEAD.
- Every remaining exposure has a minimal fix PR or an explicit blocker/owner.
- Historical coverage gaps are explicit; do not claim release sign-off from partial samples.
