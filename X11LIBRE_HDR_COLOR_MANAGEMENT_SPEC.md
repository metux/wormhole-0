# X11Libre HDR and Color Management — Design Specification

**Status:** Working draft (2026-11-26); not normative / not implementation authorization  
**Scope:** X11Libre X server HDR and color management  
**Primary implementation target:** `X11Libre/xserver:master`  
**Design authority:** owner decisions plus this document once explicitly approved  
**Related context:** `X11LIBRE_HDR_PROJECT_CONTEXT.md`  
**Related tracker:** `X11Libre/mpbt-workspace:mtx/agent-config/.starfleet-ai/dashboard/topics/xlibre/hdr-display-support.md`

> This document is intentionally a living design draft. Items marked **OPEN** require technical investigation or an explicit owner decision. Proposals must not silently become normative requirements.

## 1. Purpose

Define an API-agnostic X server color-management architecture that supports SDR, wide-gamut SDR and HDR content while preserving existing X11 compatibility.

The central architectural direction is:

**The X server performs per-window source-to-working-space color transformation before exposing window contents to an external compositor. The compositor receives already transformed window buffers and remains responsible for spatial composition.**

The X server does not replace the compositor. It owns source color semantics and source-buffer transformation; the compositor owns window placement, clipping, stacking, effects and composition. Output-specific encoding and signaling remain a distinct final-output responsibility.

This direction is referred to below as **server-side pre-compositor transformation**.

## 2. Goals

1. Give each window/drawable an explicit, validated source color description independent of its rendering API.
2. Transform source pixels in the X server, preferably using the existing Glamor GPU infrastructure.
3. Expose a stable transformed representation to Composite clients, so a compositor does not need to interpret each source's transfer function or primaries.
4. Preserve correct operation with Composite redirection, Damage, Present, DRI3 and buffer replacement.
5. Keep output capabilities, selected output mode and final-frame HDR signaling separate from source metadata.
6. Preserve legacy SDR behavior and existing 8-bit paths when no conversion is required.
7. Make acceleration optional: correctness must not depend on a particular DRM plane, color pipeline, OpenGL version or Vulkan installation.
8. Define explicit behavior for unsupported metadata, formats, transforms and allocation failures.

## 3. Non-goals

- Replacing an external X11 compositor or moving window composition into the X server.
- Requiring all clients to use DRI3, GLX, Vulkan, or a particular graphics API.
- Requiring a compositor to implement per-window HDR source conversion.
- Treating EDID sink capabilities as metadata describing the frame currently being scanned out.
- Reusing a core X11 protocol opcode or introducing an undocumented private wire convention.
- Making Vulkan headers or Vulkan runtime support mandatory for the X server.
- Raising the global minimum OpenGL profile solely to support HDR.
- Requiring a new core visual class as the first HDR delivery mechanism.

## 4. Terminology

- **Source description:** Color semantics of pixels produced by a client (primaries, white point, transfer function, reference white and relevant luminance range).
- **Source buffer:** Original client-rendered pixmap/buffer. It remains the authoritative destination for client rendering and normal X server drawing.
- **Working space:** Common, explicitly defined color space in which transformed window pixels are exposed to the compositor and can be blended correctly.
- **Transformed buffer:** X-server-managed buffer containing source pixels converted to the working space.
- **Compositor-facing pixmap:** The pixmap/resource exposed through Composite to represent a redirected window's contents. Its relationship to the transformed buffer must be specified without changing unrelated internal drawing semantics.
- **Output description:** Per-output sink capabilities and selected output encoding/bit depth.
- **Output frame metadata:** Metadata describing the final composed frame submitted to a display output, not the sink's capabilities and not an individual source window.

## 5. Architecture

### 5.1 Pipeline

```text
X11 client (any supported rendering API)
        |
        v
Original source pixmap / DRI3 buffer
        |
        | source description + Damage / Present update
        v
X server color-management stage
        |
        | GPU transform (Glamor where available)
        v
X-server-owned transformed window buffer
        |
        | Composite-facing pixmap
        v
External compositor
        |
        | stacking, positioning, clipping, alpha, effects,
        | composition in the agreed working-space semantics
        v
Final composed output frame
        |
        v
X server / DDX / DRM-KMS output configuration and signaling
```

The diagram is conceptual. The exact point at which the transformed storage is attached to the Composite-facing pixmap is **OPEN** and must be validated against Composite resource semantics, Damage, Present and DRI3 lifetimes.

### 5.2 Responsibility split

**X server**
- Owns and validates source descriptions.
- Tracks source-buffer identity, dimensions, format and content updates.
- Converts source pixels into the agreed working space.
- Publishes only transformed window content to a color-managed Composite consumer.
- Coordinates synchronization and lifetime of source and transformed buffers.
- Exposes output capabilities and controls output configuration through the relevant X server/DDX interfaces.

**External compositor**
- Receives transformed window contents in the declared working-space representation.
- Performs spatial composition and window effects.
- Must use blending semantics compatible with that working space; in particular, alpha compositing must not accidentally operate on transfer-encoded values where linear-light blending is required.
- Produces the final composed frame and cooperates with the X server's output-mode contract.

The compositor may still need a small amount of global information about the working space and output contract. It must not need to reconstruct or transform each window from its original client source description.

### 5.3 Working-space policy

A window can span outputs or move between outputs. Therefore, the per-window transformed buffer must not be encoded directly for one particular monitor.

The default architectural direction is a common, linear-light working space with sufficient range and precision for the supported SDR/HDR sources. The final output transform (gamut mapping, tone mapping, output transfer encoding and output metadata) is separate.

**OPEN — owner decision:** Select the exact working-space primaries, white point, numeric range, reference-white convention and pixel representation. Candidate approaches include a linear wide-gamut RGB space and a scene-referred/scRGB-like representation. No exact choice is approved by this draft.

**OPEN — compositor contract:** Determine how the compositor discovers and consumes the working-space definition, and how legacy compositors behave when they do not understand the new contract.

## 6. Source color description and client API

### 6.1 API independence

Source color semantics belong to the drawable/window content, not to DRI3 or a particular rendering API. DRI3 is a useful initial buffer transport/acceleration path, but must not define the color-management protocol.

The API should be a versioned X extension with explicit version and capability negotiation. Do not repurpose core protocol opcodes.

### 6.2 Candidate description fields

A source description will need, at minimum:

- RGB primaries (or a well-defined named color space);
- white point;
- transfer function / encoding;
- reference-white semantics;
- minimum/nominal/maximum luminance semantics where applicable;
- optional static content metadata, with clearly defined interpretation;
- rendering intent / conversion policy where needed.

Potential transfer functions include SDR transfer functions, PQ (ST 2084), HLG and linear. Their reference and peak luminance semantics must be defined precisely. PQ and HLG are not interchangeable.

An immutable description object referenced by a window/drawable is preferred over a collection of loosely related mutable properties. If ICC profiles are supported, profile transport, size limits, validation, caching and lifetime need explicit design.

### 6.3 Attachment scope and updates

**OPEN — owner decision:** Determine whether descriptions attach to:
- a window;
- a drawable/pixmap;
- both, with explicit inheritance/override rules; or
- a separate immutable description object bound to a window and optionally overridden for a frame.

A window-level description is convenient for conventional applications but may not describe clients that render different color encodings into the same drawable over time.

Dynamic metadata, if required, must be associated with the corresponding frame/Present transaction or immutable buffer state. It must not race a mutable window property such that metadata for frame N is accidentally applied to frame N+1.

The protocol must define:
- default description for clients that do not opt in;
- inheritance for child windows;
- behavior on reparent, resize, redirect/unredirect and destruction;
- update ordering relative to drawing and Present;
- whether a metadata change invalidates all transformed pixels;
- how clients discover supported descriptions and formats.

### 6.4 Validation and security

The server validates all client-provided values. Reject malformed descriptions, unsupported enum values, invalid ranges, NaN/Inf values, overflow and unreasonable profile sizes. A client may describe its own content; it must not be able to set global output mode, connector state or another client's window metadata.

## 7. Pixel formats and precision

Pixel storage format and colorimetry are separate concepts. A 10:10:10:2 buffer does not, by itself, mean BT.2020 or PQ. Likewise, FP16 does not define a color space.

**OPEN — owner decision:** First supported client-facing HDR format:
- packed 10-bpc RGB in the existing 32-bit pixel model; or
- FP16 RGBA as a required first-class client format; or
- a staged approach where packed 10-bpc is the first client format and FP16 is used internally / added later.

A practical staged proposal is to support existing packed 10:10:10:2 / 30-bit TrueColor-compatible output where supported, while using a higher-precision floating-point intermediate where available. Client-visible FP16 need not block an initial HDR10 path.

The implementation must define alpha representation (straight vs premultiplied), channel order, component range, dithering and quantization. It must preserve alpha semantics while transforming color channels. For premultiplied inputs, conversion must correctly handle un-premultiplication/re-premultiplication or an equivalent mathematically correct operation, including zero-alpha pixels.

## 8. Server-side transform and buffer lifecycle

### 8.1 Source and transformed storage are distinct

The server must not overwrite a client-owned source buffer with transformed pixels. The source buffer remains the target for client rendering, DRI3/Present ownership and normal X server drawing.

Each color-managed redirected window needs server-managed transformed storage. Its dimensions, format and lifetime must track the relevant source content and window lifecycle.

### 8.2 Damage-driven updates

Damage is a candidate trigger for updating only changed regions of the transformed buffer. The implementation must:
- transform every changed source region with correct coordinate mapping;
- account for filter footprints if a transform ever involves resampling;
- preserve unchanged transformed regions;
- handle full invalidation after source-description changes, format changes, resize, buffer replacement or transform-state changes;
- avoid reporting server-generated writes as new source damage and causing feedback loops.

The existing Damage extension and Composite damage tracking must be studied for safe integration. Do not assume that a single Damage callback is sufficient for all source paths.

### 8.3 Present and DRI3

Present may replace the pixmap currently associated with a window. A transformed-buffer implementation must observe every relevant buffer replacement, not merely the first CompositeNameWindowPixmap request.

For every Present update, the server must identify:
- the exact source pixmap/buffer and its generation;
- acquire synchronization (fence or syncobj point);
- update region / valid region semantics;
- when the source becomes safe to read;
- when the transformed result is complete;
- when the compositor may read the transformed buffer;
- when the source and transformed buffers may be reused or released.

The implementation must not modify the source buffer while the client/driver owns it. Explicit synchronization should be used where supported; a correct fallback must exist for supported implicit-sync paths.

### 8.4 Composite-facing pixmap

The current CompositeNameWindowPixmap implementation obtains the current window pixmap through GetWindowPixmap(), references it as a pixmap resource, and optionally invokes the DDX Screen->NameWindowPixmap hook. That DDX hook is not, by itself, an established replacement mechanism for the pixmap returned by the Composite request.

Therefore, the design must not assume that adding a transform to the existing DDX NameWindowPixmap hook is sufficient.

**OPEN — primary integration investigation:** Find a robust way for a Composite client to keep its named pixmap resource while the server updates the transformed contents. Candidate mechanisms to investigate:
1. a stable transformed pixmap object whose storage/content is updated as source buffers change;
2. a Composite-internal per-window transformed-pixmap object exposed by CompositeNameWindowPixmap;
3. a versioned Composite/color-management contract that explicitly advertises transformed-pixmap semantics.

The solution must preserve normal GetWindowPixmap/SetWindowPixmap behavior for internal X server drawing and must not globally redirect drawing into the transformed buffer.

The contract must define behavior for:
- repeated CompositeNameWindowPixmap requests;
- compositor-held pixmap XIDs and references;
- resize / depth / format changes;
- redirect and unredirect;
- window destruction;
- Present flips and copies;
- compositor restart or replacement;
- multiple Composite clients (if supported);
- Damage notification ordering and region semantics.

### 8.5 Direct scanout

A direct-scanout path must not bypass the required source transform. If the X server or DDX submits a window buffer directly to scanout, it must submit a buffer that already satisfies the selected output contract, or the path must be rejected/fall back to a correct composition path.

Output-specific transforms may not be cached as a single per-window transform when a window spans multiple outputs. The distinction between common-space source transform and final per-output transform must remain explicit.

## 9. Output, RandR and DRM/KMS contract

Source descriptions and output descriptions are separate.

### 9.1 Sink capabilities

Read sink capabilities from EDID, including relevant CTA-861 colorimetry and HDR static metadata blocks. Existing parsing in `xf86DoInterpretHDRMetadata()` is present on master but, at the verified checkpoint, had no production call site. Integrate it safely only after validating that a complete non-NULL EDID monitor object exists.

Malformed/short EDID and missing CTA blocks must be ordinary negative test cases, not crash conditions.

### 9.2 Output mode selection

Output colorspace, bit depth and HDR enablement are selected per output only when both sink and DRM driver support the requested mode. RandR must expose the appropriate capability/configuration model and handle hotplug, mode changes and rollback.

**OPEN — owner decision:** Define the RandR API/property model and policy for selecting output colorspace, bit depth, EOTF and HDR mode.

### 9.3 Final-frame metadata

DRM/KMS `HDR_OUTPUT_METADATA` describes the frame being sent to the output. It is not a copy of the sink's EDID capability metadata and is not the metadata of an arbitrary source window.

The output path must derive final-frame signaling from the actual composed output (including any tone mapping / gamut mapping and applicable static metadata policy). The exact policy for mixed SDR/HDR windows and multiple source metadata values is **OPEN**.

DRM color-pipeline, plane and hardware LUT features may be used as optimizations. They are not correctness prerequisites.

## 10. Compositor compatibility and fallback

A compositor that supports the new contract receives transformed buffers and consumes the declared working space.

A legacy compositor that does not understand the contract must not silently receive values whose interpretation differs from its existing assumptions. The server needs explicit capability negotiation and a defined fallback.

**OPEN — owner decision:** Choose the compatibility strategy:
- keep legacy Composite behavior unless the compositor opts in to a versioned color-managed mode;
- expose a new Composite/color-management capability and separate request;
- or another explicit negotiation mechanism.

The default must preserve existing SDR behavior. A partial transform path must never silently transform only some rendering operations while leaving others in an incompatible encoding.

If a transform cannot be completed, the server must not publish stale or partially updated pixels as if they were valid. Define whether the affected window falls back to a legacy SDR interpretation, retains its last valid transformed frame, or reports an error / disables color management. The policy must avoid undefined mixed semantics.

## 11. Rendering coverage

A complete implementation must account for all ways pixels can reach a window's source buffer, including:
- core X drawing operations;
- Render/compositing operations;
- XRender glyph/text paths;
- GLX and DRI3 client rendering;
- Present copy and flip paths;
- SHM and other image upload paths;
- driver-specific pixmap import/export paths.

A Damage-based transform stage may centralize update detection, but it does not remove the need to verify that all rendering paths generate correct Damage and that the source buffer is readable with the required synchronization.

No implementation may claim full color management while only selected Glamor operations are transformed.

## 12. Failure handling and compatibility invariants

1. No HDR capability or no color-management opt-in: existing SDR behavior remains unchanged.
2. Unsupported source description: reject or explicitly fall back according to the negotiated API; never guess silently.
3. Unsupported output mode: retain a valid SDR/output mode and do not send unsupported KMS properties.
4. Allocation or shader failure: do not expose an incomplete transformed buffer.
5. Source buffer replacement: invalidate/rebuild transformed content before notifying the compositor that the new content is ready.
6. Server reset, VT switch, hotplug and output reconfiguration: rebuild or invalidate derived buffers safely.
7. Optional acceleration unavailable: use a correct supported fallback or disable the feature; never require Vulkan or a higher global GL baseline.
8. Legacy 8-bit clients retain their current fast path unless an explicitly negotiated policy requires conversion.

## 13. Testing and acceptance criteria

Tests are required at protocol, DIX/Composite, Damage/Present, Glamor and modesetting layers.

### Protocol and metadata
- version negotiation and capability discovery;
- valid and invalid descriptions;
- unknown transfer functions / primaries;
- NaN/Inf, overflow, invalid luminance ranges and oversized profiles;
- object lifetime, inheritance, destruction and update ordering;
- unauthorized attempts to modify another client's metadata or output state.

### Buffer lifecycle and Composite
- first redirect and first CompositeNameWindowPixmap;
- repeated naming of the same window;
- Damage partial and full updates;
- source-description change invalidation;
- resize, depth/format changes and reparent;
- redirect/unredirect and compositor restart;
- multiple outstanding compositor references;
- source pixmap replacement through Present copy and flip;
- synchronization proving compositor never reads an incomplete transformed buffer;
- no feedback loop from transformed writes into source Damage.

### Pixel correctness
- SDR pass-through and SDR-to-working-space conversion;
- wide-gamut SDR;
- PQ and HLG test vectors once semantics are approved;
- alpha edges, premultiplied alpha, zero-alpha pixels;
- clipping, quantization, dithering and precision;
- mixed SDR/HDR windows and correct linear-light blending;
- no dark-window, halo or color-fringe regressions.

### Output
- no-HDR EDID;
- captured HDR-capable EDIDs;
- malformed and short EDID;
- unsupported/missing DRM properties;
- hotplug, output reconfiguration and rollback;
- multiple outputs and windows spanning outputs;
- correct distinction between EDID sink capabilities and final-frame KMS metadata.

### Compatibility / build matrix
- feature disabled and enabled;
- legacy compositor / no color-managed opt-in;
- SDR-only clients;
- DRI3 and non-DRI3 source paths;
- existing supported GL baseline;
- builds without Vulkan headers or runtime;
- existing 8-bit pass-through regression tests.

Acceptance requires no silent partial coverage: every supported source path must either produce a correctly transformed compositor-facing buffer or be explicitly outside the negotiated feature capability.

## 14. Staged implementation plan

**Stage 0 — read-only EDID capability integration**
- Connect the existing CTA HDR/colorimetry parser to the normal EDID path with NULL/length safety.
- Expose/log read-only sink capabilities.
- No HDR output signaling or rendering behavior changes.
- Add malformed/short/no-HDR EDID integration tests.

**Stage 1 — output capability and mode plumbing**
- Add explicit output capability discovery and RandR configuration model.
- Add guarded DRM Colorspace/max_bpc/HDR metadata plumbing.
- Feature-gate experimental mode selection and provide rollback/SDR fallback.
- Do not yet claim per-window client color management.

**Stage 2 — client source-description API**
- Add a versioned extension and immutable source-description model.
- Define default SDR semantics, validation, object lifetime and capability discovery.
- Define the compositor working-space contract and explicit opt-in/compatibility behavior.

**Stage 3 — transformed-buffer prototype**
- Implement one well-defined source path, initially DRI3/Glamor where practical.
- Prove stable Composite-facing pixmap semantics with Damage and Present.
- Keep source and transformed storage separate.
- Add synchronization and lifecycle regression tests before enabling broadly.

**Stage 4 — complete source-path coverage**
- Cover the supported X drawing, Render, GLX/DRI3, SHM and Present paths.
- Add mixed SDR/HDR and alpha-correct conversion.
- Add non-DRI3 paths or explicitly document capability limitations.

**Stage 5 — output composition and final signaling**
- Define final output tone/gamut mapping and metadata policy.
- Ensure composited and direct-scanout paths obey the same output contract.
- Optimize with hardware color pipeline / planes where available, without making them mandatory.

Stages may be split into independently reviewable patches. Do not revive the stale `wip/hdr-prep` or historical monolithic PoC as the implementation base.

## 15. Open decisions register

| ID | Topic | Current proposal / issue | Required resolution |
|---|---|---|---|
| D-01 | Working space | Common linear-light space; exact primaries/range not selected | Owner decision after technical comparison |
| D-02 | Client pixel formats | Packed 10-bpc first; FP16 later is a candidate | Owner decision |
| D-03 | Source metadata scope | Immutable description object bound to drawable/window; frame metadata for dynamic values | Define attachment, inheritance and Present association |
| D-04 | Transfer functions | SDR, linear, PQ and HLG need explicit semantics | Define reference white, luminance and conversion rules |
| D-05 | Composite contract | Compositor sees transformed buffers | Choose opt-in/versioned negotiation and stable pixmap mechanism |
| D-06 | Damage/Present integration | Damage updates plus explicit buffer-generation/sync tracking | Prove complete lifecycle and ordering in current code |
| D-07 | Failure fallback | Never publish stale/partial output as valid | Choose observable API behavior and legacy fallback |
| D-08 | Output API | Per-output RandR configuration separate from sink metadata | Define properties, mode selection and rollback |
| D-09 | Mixed-content output metadata | Must describe final composed frame | Define metadata aggregation / output policy |
| D-10 | ICC support | Possible immutable profile object | Decide whether required for first version or deferred |
| D-11 | Dynamic HDR metadata | Associate with frame/Present if supported | Decide supported metadata types and protocol representation |
| D-12 | Direct scanout | Must not bypass source transform | Define transformed-buffer scanout eligibility and fallback |

## 16. Current code investigation notes (2026-11-26)

Verified against `X11Libre/xserver:master` at `8f47e0e2dd7a9f2dec43acffb2cf0f80b68e5b8e`:

- `Xext/composite/compext.c`, `SingleCompositeNameWindowPixmap()`: obtains the current window pixmap using `GetWindowPixmap()`, registers a pixmap resource and optionally calls `Screen->NameWindowPixmap()`. The existing DDX hook is not demonstrated to replace the pixmap resource returned by the Composite request.
- `Xext/present/present.c`: `present_set_tree_pixmap()` changes window pixmap associations via `SetWindowPixmap()` when Present changes the displayed pixmap.
- `Xext/present/present_scmd.c`: after a successful flip, the server updates window/root pixmap associations and reports Damage for the updated window region.
- `Xext/composite/compwindow.c` and `Xext/composite/compalloc.c`: Composite redirection allocates offscreen window pixmaps and changes window pixmap associations. This must remain distinct from any compositor-facing transformed storage.
- `miext/damage/damage.c`: Damage wraps `SetWindowPixmap` and drawing operations. It is a candidate observation point, but the complete callback and lifetime behavior still needs review.
- `Xext/dri3/` and `include/dri3.h`: DRI3 provides pixmap FD import/export, formats/modifiers and syncobj-related hooks; it does not itself define source color semantics.
- `glamor/glamor_egl.c`: Glamor/GBM import/export paths provide a possible GPU-buffer basis. Format/modifier/scanout constraints need to be respected.
- `hw/xfree86/drivers/video/modesetting/present.c`: Present pageflip eligibility depends on driver/format/modifier/scanout capabilities. A transformed buffer may not automatically be eligible for direct scanout.

Relevant source paths:
- `Xext/composite/compext.c`
- `Xext/composite/compwindow.c`
- `Xext/composite/compalloc.c`
- `miext/damage/damage.c`
- `Xext/present/present.c`
- `Xext/present/present_scmd.c`
- `Xext/dri3/dri3_request.c`
- `Xext/dri3/dri3_screen.c`
- `include/dri3.h`
- `glamor/glamor_egl.c`
- `hw/xfree86/drivers/video/modesetting/present.c`

This investigation is a source review only. No Xserver code, build, runtime or test changes were made as part of this draft.

## 17. References

- X11Libre Xserver source: https://github.com/X11Libre/xserver
- X11Libre HDR/color-management discussion: https://github.com/orgs/X11Libre/discussions/251
- X11Libre misc issue #250: https://github.com/X11Libre/misc/issues/250
- Wayland color-management concepts (reference only): https://wayland.freedesktop.org/docs/book/Color.html
- Linux DRM/KMS documentation: https://docs.kernel.org/gpu/drm-kms.html
- Historical X.Org DeepColor extension discussion: https://groups.google.com/g/xorg-devel/c/JmhOVTpZcs8
