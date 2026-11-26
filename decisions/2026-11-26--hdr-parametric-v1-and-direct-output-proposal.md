# Decision and proposal: XLibre HDR Color Description scope and no-compositor output

**Date:** 2026-11-26

## Owner-approved decision: parametric Color Descriptions in v1

**Status:** Approved by owner

Version 1 will support parametric Color Descriptions only. The model will carry explicit parameters for:
- RGB primaries and white point
- Transfer function
- Luminance and reference-white semantics
- Relative-versus-absolute colorimetric interpretation

Arbitrary embedded ICC profile ingestion and general-purpose CMM integration are deferred beyond v1. They require a separate design and owner decision covering rendering intent, PCS semantics, CMM ownership, profile lifetime/cache behavior and exactly-once chromatic adaptation.

This decision scopes v1 but does not freeze the extension wire ABI and does not authorize Xserver implementation.

## Design proposal: direct per-output conversion without a compositor

**Status:** Proposed for investigation; not yet approved as the final architecture.

When no compositor is active, investigate converting the final desktop image independently for each output immediately before scanout, rather than transforming each window into the compositor-facing common working space.

Key constraints:
- The transform must be per output and late in the pipeline. A desktop may span monitors with different output color spaces; one globally preconverted root pixmap cannot be correct for all of them.
- Prefer per-CRTC DRM/KMS color-pipeline blocks (degamma LUT, CTM, gamma LUT, and future 3D LUT support where available).
- If hardware blocks are insufficient, use output-specific GPU/shader conversion into separate scanout buffers.
- If neither path can provide a correct transform, preserve a constrained legacy SDR fallback; do not claim HDR merely because the sink advertises HDR capability.
- A final-output transform cannot recover per-window color semantics lost during rasterization. Rich per-window HDR descriptions may require internal redirection/composition or a future rendering contract.
- Audit Present flips, direct scanout, overlays, cursors and video planes separately; they may bypass or have separate color-pipeline behavior.
- Keep output transfer encoding, tone/gamut mapping and outgoing HDR metadata separate from sink EDID capability data.

This is a distinct path from the compositor-active architecture, where source windows are transformed into common linear BT.2020 / D65 and the compositor performs spatial composition.

## Required investigation

1. Establish exact Xserver/driver ownership for the final desktop and per-CRTC output transform.
2. Inventory KMS color properties and their actual precision/range and ordering on supported drivers.
3. Determine whether the pipeline can express the required transfer, gamut and luminance mapping, or whether output-specific shader buffers are needed.
4. Trace multi-output clone/extend behavior, root/screen pixmap sharing, hotplug and atomic state updates.
5. Audit Present, direct scanout, overlays, cursors and video planes.
6. Define SDR fallback and prevent double application of the legacy SDR reference-white scale.
7. Add tests for differing monitor profiles, windows crossing output boundaries, clone modes, hotplug and unsupported color-pipeline hardware.

No implementation is authorized by this proposal.
