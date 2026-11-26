# Decision: use DIX screen privates for optional HDR operations

**Date:** 2026-11-26  
**Status:** Owner-approved design direction; not implementation authorization

## Decision

If HDR needs additional screen/backend operations, do not extend or reorder ScreenRec. Prefer a core-owned, versioned optional operations table attached through the existing DIX devPrivates mechanism using the existing PRIVATE_SCREEN type.

The proprietary NVIDIA DDX must remain able to load and operate without registering this table. It is not required to implement HDR.

## Rationale and constraints

- ScreenRec has a legacy single devPrivate pointer and the DIX-managed devPrivates store. HDR must use the latter, not overwrite the former.
- The screenSpecificPrivates[PRIVATE_LAST] array is part of ScreenRec; adding a new DevPrivateType / changing PRIVATE_LAST changes structure size/offsets. Do not do this.
- Keep the private key and record lifetime in Xserver core. A DDX must not own a key or callback-table storage in unloadable module static memory.
- Copy and validate versioned operations tables into core-owned storage; validate size, mandatory members and capability consistency.
- No registration means no HDR operations. Missing capabilities and allocation failure disable HDR for that screen without affecting server startup or legacy SDR.
- No changes to the existing CONFIG_LEGACY_NVIDIA_PADDING, reserved NVIDIA field, loader ABI selection, or legacy_nvidia_340x compatibility path.
- Keep the first implementation's registration API internal to X11Libre/modesetting until a stable third-party driver contract is explicitly designed. An exported helper used by a new external DDX requires a separate ABI minor-version decision.

## Source evidence

Verified against X11Libre/xserver:master at 3132e4b7997b525d046e727c1db6656d14b5bc20:

- include/scrnintstr.h: ScreenRec contains devPrivate, devPrivates, screenSpecificPrivates[PRIVATE_LAST], and the conditional reserved_for_nvidia_470_and_390 field.
- include/privates.h: explicitly warns that changing private type values/count changes ScreenRec size and field offsets; it provides dixRegisterPrivateKey() and dixLookupPrivate() APIs.
- dix/privates.c: dixRegisterPrivateKey(..., PRIVATE_SCREEN, ...) can resize existing regular and GPU screen private allocations through fixupScreens().
- include/xf86Module.h, meson_options.txt, include/meson.build, hw/xfree86/loader/loader.c and dix/pixmap.c: the legacy NVIDIA padding option and special loader ABI path are deliberate compatibility shims and must remain unchanged.
- DRI3 provides a precedent for core-owned screen-private registration and close-hook cleanup.

Detailed rules and the regression matrix are in section 29 of X11LIBRE_HDR_COLOR_MANAGEMENT_SPEC.md.

## Required validation

Build both legacy_nvidia_padding configurations; compare ScreenRec size/offsets and ABI selection to pre-HDR baselines; test real proprietary NVIDIA driver combinations (including 470 and modern dual-ABI where applicable); verify absent HDR operations preserve the full legacy SDR path; test private registration on existing regular/GPU screens, allocation failure, screen teardown and server generations.

This decision records the preferred design only. No Xserver implementation is authorized by it.
