---
id: W0-20261106-011
kind: correction
from: chatgpt
to: starfleetctl
created: 2026-11-06T1220:00Z
in_reply_to: W0-20261106-010
task_ref: active-physical-device-registry-task-2026-11-05
workspace: metux/xav-design@docs/restructure-2026-10-21
---

# Correction: fixed-size wire declaration macros

Follow-up to W0-20261106-010: the first mechanical normalization covered `XAV_*` constants and `xav[A-Z]*` type names but did not catch the old fixed-size macro prefix because it follows an underscore.

This has now been corrected in the normative header:
- Latest header commit: `0bc17f5bda0b8aa4e576a7ef368a0083240cdbbf`
- Current blob: `42dcc8570304e19680724763388dda663f433aa5`
- Fixed-size macros now use `sz_Xaverius*`, e.g. `sz_XaveriusReqHeader`, `sz_XaveriusCreateDeviceReq`, and `sz_XaveriusAsyncErrorEvent`.
- The old `sz_xav*` spelling is obsolete.

The current canonical spelling families are therefore:
- `XAVERIUS_*` constants/macros;
- `Xaverius*` wire types;
- `sz_Xaverius*` fixed-size macros.

No numeric values, fields, packing, offsets, or wire sizes were changed. The three normative `XAVERIUS_DEVICE_STATUS_*` constants remain present. Please use this latest header commit when carrying out W0-20261106-010 and include fixed-size macro references in the stale-name search and independent review.
