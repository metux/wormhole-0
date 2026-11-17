# Temporary patch templates — X.Org server CVE audit

Snapshot: 2026-11-17. Review templates only; not approved patches. Never apply blindly. Recheck exact target HEAD and source before opening a PR.

## CVE-2026-109091 — EDID DS_RANGES (master)
Ticket: https://github.com/X11Libre/xserver/issues/4132
Reference: merged PRs [#4120](https://github.com/X11Libre/xserver/pull/4120) (25.0), [#4144](https://github.com/X11Libre/xserver/pull/4144) (25.1), [#4155](https://github.com/X11Libre/xserver/pull/4155) (25.2).
Target: hw/xfree86/modes/xf86EdidModes.c, handle_detailed_monset(), DS_RANGES.
Template: guard hsync writes/increment with nHsync < MAX_HSYNC and vrefresh writes/increment with nVrefresh < MAX_VREFRESH. Preserve have_hsync/have_vrefresh and maxPixClock behavior. Test excess detailed range descriptors and adjacent MonRec fields.

## CVE-2026-109092 — XvShmPutImage (master, 25.2)
Ticket: https://github.com/X11Libre/xserver/issues/4133
Reference: merged [#4118](https://github.com/X11Libre/xserver/pull/4118) (25.0), [#4143](https://github.com/X11Libre/xserver/pull/4143) (25.1).
Target: Xext/xvdisp.c, SingleXvShmPutImage().
Reference guard:
    if (size_needed < 0 || stuff->offset > shmdesc->size ||
        (unsigned long) size_needed > shmdesc->size - stuff->offset)
        return BadAccess;
This subtraction avoids overflow in size_needed + offset. Keep the negative-size rejection before unsigned conversion. Verify target types/refactors.

## CVE-2026-109093 — XTestFakeInput (master)
Ticket: https://github.com/X11Libre/xserver/issues/4134
No master fix PR. [#4116](https://github.com/X11Libre/xserver/pull/4116) merged to 25.0; [#4142](https://github.com/X11Libre/xserver/pull/4142) and [#4161](https://github.com/X11Libre/xserver/pull/4161) remain open and are not accepted fixes.
Target: Xext/xtest.c, ProcXTestFakeInput(), DeviceValuator handling.
Do not copy rejected boundary logic. Re-derive bounds from base, first_valuator, num_valuators, dev->valuator->numAxes and MAX_VALUATORS. Validate before every stack-array write; reject first_valuator >= numAxes; ensure cumulative end is within both axis count and fixed array capacity. Test normal/swapped multi-record requests, equality-at-end, and valid-last-axis.

## CVE-2026-109096 — XvPutImage (master, 25.2)
Ticket: https://github.com/X11Libre/xserver/issues/4137
Reference: merged [#4126](https://github.com/X11Libre/xserver/pull/4126) (25.0), [#4149](https://github.com/X11Libre/xserver/pull/4149) (25.1).
Target: Xext/xvdisp.c, SingleXvPutImage(). Reference guard immediately after ddQueryImageAttributes: if (size < 0) return BadValue;. Verify it precedes request-size arithmetic and length conversion.

## CVE-2026-109903 — XvQueryImageAttributes (master, 25.2)
Ticket: https://github.com/X11Libre/xserver/issues/4138
Reference: merged [#4127](https://github.com/X11Libre/xserver/pull/4127) (25.0), [#4150](https://github.com/X11Libre/xserver/pull/4150) (25.1).
Target: Xext/xvdisp.c, ProcXvQueryImageAttributes(). Reference guard after ddQueryImageAttributes: if (size < 0) { free(offsets); return BadValue; }. Check allocation ownership and cleanup on every early return.

## CVE-2026-93536 — GestureBuildSprite stale WindowPtr (master, 25.2)
No matching X11Libre CVE issue found by exact search. Reference fix: merged [#3893](https://github.com/X11Libre/xserver/pull/3893) (25.1).
Target: Xi/exevents.c, WindowGone(). Clean gesture sprite traces even if device has no TouchClass; truncate touch and gesture traces at removed window. Verify current WindowGone refactor and destruction ordering; test a gesture-capable device without TouchClass while destroying a window during an active gesture.

## CVE-2026-50263 — DRI2 GetBuffers (25.2)
No matching X11Libre CVE issue found by exact search. Master reference: [commit c03bc789](https://github.com/X11Libre/xserver/commit/c03bc789aed3fdf72af0bf0d2ec0d3cf073ecdf8).
Target: Xext/dri2/dri2.c, do_get_buffers(). Reference allocation is calloc(count + 1, sizeof(buffers[0])); count is already capped at DRI2BufferHiz + 1, and the extra slot is for a synthesized real/fake front buffer. Verify 25.2 source and test maximum attachment count without FrontLeft but with BackLeft.

## Not a fix target — CVE-2026-109094 on master
The old separate swapped handler is gone on current master. ResExtensionInit registers ProcResDispatch for both normal and swapped requests; ProcXResQueryResourceBytes validates using uint64_t before swapping each spec. Do not create a replacement implementation PR. See [master recheck](../cve-2026-109094-master-recheck.md). [#4168](https://github.com/X11Libre/xserver/pull/4168) is test-only; [#4169](https://github.com/X11Libre/xserver/pull/4169) is obsolete as implementation.

## Promotion criteria
1. Re-fetch exact target HEAD and relevant source; record blob SHA.
2. Confirm the vulnerable operation remains after refactors.
3. Keep patch minimal and branch-specific.
4. Add a regression test exercising the actual protocol path.
5. Record commands, exit codes, output, tested HEAD, and CI URL.
6. Update the matrix only after checking GitHub PR state.
