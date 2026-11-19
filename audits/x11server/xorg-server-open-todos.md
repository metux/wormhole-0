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
   - **release/25.2 remains vulnerable at HEAD `a10ba82d`.** `Xext/xv/xvdisp.c:SingleXvShmPutImage()` still checks `(size_needed + stuff->offset) > shmdesc->size`; it neither rejects a negative `size_needed` nor avoids the overflowing addition. A search of current X11Libre PRs found no release/25.2 implementation PR and no merged fix. The 25.2 matrix entry correctly remains OPEN.
   - Required backport ordering: reject negative size, reject offset beyond segment size, then compare size against the remaining segment length. Avoid addition before bounds validation.
   - Earlier review of #4171 verified the int / unsigned long / CARD32 arithmetic and guard ordering. Build X servers run [44435763792](https://github.com/X11Libre/xserver/actions/runs/44435763792) succeeded.

2. **CVE-2026-109903 — XvQueryImageAttributes, release/25.2.**
   - Master fix [#4184](https://github.com/X11Libre/xserver/pull/4184) is merged.
   - Release/25.1 [#4150](https://github.com/X11Libre/xserver/pull/4150) is merged; follow-up [#4187](https://github.com/X11Libre/xserver/pull/4187) fixes the rpcbuf ownership error in its error path.
   - Release/25.0 [#4127](https://github.com/X11Libre/xserver/pull/4127) is merged.
   - **release/25.2 remains vulnerable at HEAD `a10ba82d`.** Its `ProcXvQueryImageAttributes()` still passes the signed result from `ddQueryImageAttributes()` directly into `reply.data_size`; there is no `size < 0` rejection. The current PR search found no release/25.2 implementation PR and no merged fix. The 25.2 matrix entry correctly remains OPEN.
   - Prepare a branch-specific backport with the negative-size guard before constructing the reply / sending reply data. The master fix is [#4184](https://github.com/X11Libre/xserver/pull/4184); release/25.1 also needed [#4187](https://github.com/X11Libre/xserver/pull/4187) to correct rpcbuf cleanup in the error path.

3. **CVE-2026-109094 — XRes QueryResourceBytes swap path: no outstanding baseline exposure.**
   - **`ProcXResQueryResourceBytes()` was checked independently on all four current baselines**, not just dispatch/SProc. The variable-size request check always precedes any spec-array access:
     - **master:** `X_REQUEST_HEAD_AT_LEAST`, `X_REQUEST_FIELD_CARD32`, then `REQUEST_FIXED_SIZE` with `(uint64_t)numSpecs * sizeof(spec)`; only after that does the swapped-client spec loop run. There is no separate SProc handler.
     - **release/25.2:** same 64-bit common-handler ordering; no separate SProc handler.
     - **release/25.1:** `numSpecs` is swapped first when needed, then `REQUEST_FIXED_SIZE` uses a 64-bit product; the spec swap loop follows the size check.
     - **release/25.0:** `ProcXResQueryResourceBytes()` rejects `numSpecs > UINT32_MAX / sizeof(spec)` before `REQUEST_FIXED_SIZE`; `SProcXResQueryResourceBytes()` independently checks the same bound after swapping `numSpecs` and before the length check and spec loop.
   - **master: N/A.** The separate `SProcXResQueryResourceBytes()` path was removed by [d58de12e](https://github.com/X11Libre/xserver/commit/d58de12e73f21d38a74bf93881d56bf8b39b434e) (2025-09-15). Swapping was inlined into `ProcXResQueryResourceBytes()`; the request size is checked using a 64-bit product before the spec loop.
   - **release/25.1: N/A.** The branch has the inlined common handler, explicitly swaps `numSpecs`, validates with `((uint64_t)numSpecs) * sizeof(xXResResourceIdSpec)`, and only then iterates over and swaps the specs. No separate vulnerable SProc loop exists.
   - **release/25.2: N/A.** It has the same safe common-handler pattern, using `X_REQUEST_HEAD_AT_LEAST` / `X_REQUEST_FIELD_CARD32` for header parsing and a 64-bit size product before the spec loop. No separate vulnerable SProc loop exists.
   - **release/25.0: fixed by [#4113](https://github.com/X11Libre/xserver/pull/4113)**, merged 2026-11-16 as [ac8db972](https://github.com/X11Libre/xserver/commit/ac8db972868f2b92c3a60eb71b734d43f5d5901a). This branch retained the separate `SProcXResQueryResourceBytes()` path; #4113 adds the missing `numSpecs > UINT32_MAX / sizeof(specs[0])` rejection immediately after swapping `numSpecs`, before `REQUEST_FIXED_SIZE` and before the spec-swap loop.
   - **release/25.0: fixed by [#4113](https://github.com/X11Libre/xserver/pull/4113)**, merged 2026-11-16 as [ac8db972](https://github.com/X11Libre/xserver/commit/ac8db972868f2b92c3a60eb71b734d43f5d5901a). This branch retained the separate SProc path; #4113 adds the missing overflow rejection in both request paths before validating or touching the spec array.
   - Conclusion: no analogous `numSpecs * sizeof(spec)` overflow remains in `ProcXResQueryResourceBytes()` on any current baseline. Historically, 25.0 was affected until #4113; master, 25.1 and 25.2 already had the protective refactor. The implementation PRs [#4169](https://github.com/X11Libre/xserver/pull/4169), [#4153](https://github.com/X11Libre/xserver/pull/4153) and [#4141](https://github.com/X11Libre/xserver/pull/4141) are obsolete and should be closed with the branch-specific rationale above. I attempted to add disposition comments and close them, but GitHub rejected both write operations with HTTP 403 (`Resource not accessible by integration`); they therefore remain open until a maintainer closes them. [#4168](https://github.com/X11Libre/xserver/pull/4168) is a separate test-only PR and is not an implementation fix.
   - The key historical change is [d58de12e](https://github.com/X11Libre/xserver/commit/d58de12e73f21d38a74bf93881d56bf8b39b434e): it inlined the SProc handlers, routed both byte orders through the same handler, and changed the size calculation to 64-bit before iterating. This predates the current audit branches' relevant baseline code.
4. **CVE-2026-93536 — GestureBuildSprite stale WindowPtr UAF.**
   - The sampled matrix has master and release/25.2 still open; release/25.1 and 25.0 source-fixed.
   - Recheck exact current branch source, especially WindowGone/destruction ordering, before porting [#3893](https://github.com/X11Libre/xserver/pull/3893).
   - Do not substitute the unrelated device-sprite animation cleanup [c5453462](https://github.com/X11Libre/xserver/commit/c5453462eac3d801f8b26e482859da3df4369701).

5. **CVE-2026-50263 — DRI2 GetBuffers attachment-array overflow.**
   - Master fix is [#3266](https://github.com/X11Libre/xserver/pull/3266), merged as [c03bc789](https://github.com/X11Libre/xserver/commit/c03bc789aed3fdf72af0bf0d2ec0d3cf073ecdf8).
   - Release/25.2 still needs a branch-specific backport. The trigger is the maximum 11 attachment entries without FrontLeft but with BackLeft, requiring a synthesized front buffer.
   - Inspect release/25.0 and 25.1 separately; their DRI2 source layout differs. Do not infer their status from 25.2.

6. **CVE-2026-109093 — XTestFakeInput valuator range: status and PR links reconciled.**
   - All four baselines already contain the guard before writing to `valuators[MAX_VALUATORS]`: master [#2964](https://github.com/X11Libre/xserver/pull/2964), release/25.2 [#3115](https://github.com/X11Libre/xserver/pull/3115), release/25.1 [#2982](https://github.com/X11Libre/xserver/pull/2982), and release/25.0 [#2962](https://github.com/X11Libre/xserver/pull/2962) are all merged.
   - The November attempts [#4142](https://github.com/X11Libre/xserver/pull/4142), [#4161](https://github.com/X11Libre/xserver/pull/4161), [#4172](https://github.com/X11Libre/xserver/pull/4172), [#4189](https://github.com/X11Libre/xserver/pull/4189) and [#4190](https://github.com/X11Libre/xserver/pull/4190) were closed unmerged/redundant; they do not indicate current exposure. [#4116](https://github.com/X11Libre/xserver/pull/4116) was also merged to 25.0 but is redundant against the earlier #2962 fix.
   - Master regression-test PR [#4170](https://github.com/X11Libre/xserver/pull/4170) is separate from the implementation fix; it is test-only and does not represent an outstanding security fix.
   - Required boundary cases: high firstValuator, equality with numAxes, MAX_VALUATORS overflow, multiple DeviceValuator records, normal and swapped requests, and a valid last-axis boundary.

7. **Resolved November CVEs — no code work currently queued.**
   - CVE-2026-109091: master [#4174](https://github.com/X11Libre/xserver/pull/4174), 25.2 [#4155](https://github.com/X11Libre/xserver/pull/4155), 25.1 [#4144](https://github.com/X11Libre/xserver/pull/4144), 25.0 [#4120](https://github.com/X11Libre/xserver/pull/4120) merged.
   - CVE-2026-109095: master [#4167](https://github.com/X11Libre/xserver/pull/4167), 25.2 [#4152](https://github.com/X11Libre/xserver/pull/4152), 25.1 [#4140](https://github.com/X11Libre/xserver/pull/4140), 25.0 [#4111](https://github.com/X11Libre/xserver/pull/4111) merged.
   - CVE-2026-109096: master [#4185](https://github.com/X11Libre/xserver/pull/4185), 25.2 [#4180](https://github.com/X11Libre/xserver/pull/4180), 25.1 [#4149](https://github.com/X11Libre/xserver/pull/4149), 25.0 [#4126](https://github.com/X11Libre/xserver/pull/4126) merged. The 25.2 merge commit is [a10ba82d](https://github.com/X11Libre/xserver/commit/a10ba82d33f50729d3a9785801d7cd6e65b12b21).
   - CVE-2026-109904: source protection found on all four branches. `master`, `release/25.1` and `release/25.2` validate the variable-length `QueryClientIds` spec array with a 64-bit size product before accessing/swapping it; `release/25.0` has explicit overflow guards in both normal and swapped handlers, with [#4114](https://github.com/X11Libre/xserver/pull/4114) merged. The common-handler protection is traceable to merged [#1060](https://github.com/X11Libre/xserver/pull/1060) (`Xext: xres: inline SProc*()`), which added the 64-bit `QueryClientIds` length calculation before array access; the only later branch-specific security PR is [#4114](https://github.com/X11Libre/xserver/pull/4114) for 25.0.

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
