# Decision: XLibre HDR v1 chromatic adaptation

**Date:** 2026-11-26  
**Status:** Approved by owner

## Decision

For the XLibre HDR source-to-working-space transform, v1 shall:

1. Use ICC linear Bradford as the single chromatic-adaptation transform when a relative-colorimetric RGB source has a white point different from D65.
2. Use full adaptation (degree of adaptation = 1). Do not expose partial adaptation or per-window/per-output CAT selection in v1.
3. Encode source colorimetric semantics explicitly and versionedly. Relative-colorimetric RGB sources must be distinguishable from absolute-colorimetric XYZ data and already-adapted PCS data. Never infer this distinction from transfer function, primaries or pixel format.

Apply Bradford to XYZ after transfer decoding and source RGB-to-XYZ conversion. Skip it when the source white is already D65. Preserve negative intermediate values; do not clamp gamut excursions in the source-to-working transform. Reference-white luminance is a separate scale and must not affect the CAT matrix.

## Rationale

ICC.1:2022 recommends linear Bradford when there is no specific reason to choose another CAT. Fixing one deterministic transform avoids backend-dependent results. The explicit relative/absolute distinction prevents accidental or duplicate adaptation.

## Consequences

- The common compositor working space remains linear BT.2020 / D65, with neutral RGB 1.0 corresponding to 203 cd/m².
- The source pipeline adapts relative RGB to D65 exactly once before conversion to the common working space.
- Absolute-colorimetric XYZ and already-adapted PCS values preserve their declared colorimetry and must not receive an implicit source adaptation.
- CAT16 may be reconsidered only if a concrete requirement emerges; it is not a v1 negotiation option.
- ICC profile ingestion/CMM ownership, validation tolerances, canonical white-point precision, HLG luminance semantics and non-finite FP16 handling remain open design work.
- This decision does not freeze the extension wire ABI and does not authorize Xserver implementation.

## References

- `X11LIBRE_HDR_COLOR_MANAGEMENT_SPEC.md`, section 26.
- ICC.1:2022, https://www.color.org/specifications/ICC.1-2022-05.pdf
- W3C CSS Color 4 CRD (2026-11-12), https://www.w3.org/TR/2026/CRD-css-color-4-20261112/
