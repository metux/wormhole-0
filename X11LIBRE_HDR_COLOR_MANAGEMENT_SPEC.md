# X11Libre HDR and Color Management — Design Specification

**Status:** Working design draft with owner-approved architecture decisions (2026-11-26); not a frozen wire ABI / not implementation authorization  
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

**Owner decision (2026-11-26):** the compositor-facing working space is fixed as linear BT.2020 / D65, output-independent, with neutral RGB 1.0 = 203 cd/m². The negotiated compositor handoff is RGBA IEEE-754 binary16 (FP16), linear premultiplied alpha. Capability negotiation, fallback and wire-level lifecycle details remain open; see sections 24–26.

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

**Scope distinction / owner decision:** the compositor-facing transformed handoff is negotiated RGBA FP16, as approved in section 24. This does **not** mean ordinary core pixmaps or all client source buffers become FP16. Client source formats remain separately capability-negotiated; existing packed 10:10:10:2 / 30-bit paths may be supported where appropriate. Pixel format and colorimetry remain separate.

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

### 16.1 Damage migration and Composite resource identity

A more detailed read of `miext/damage/damage.c` establishes two useful facts:

- `DamageRegister()` records a Damage object both on the window-private list (`pNextWin`) and on the current drawable/pixmap Damage list.
- The `damageSetWindowPixmap()` wrapper removes all Damage objects registered on the window from their previous pixmap list, calls the underlying `SetWindowPixmap()`, then inserts those same Damage objects into the new pixmap's Damage list.

This is specifically designed to keep window-registered Damage objects attached as the window's backing pixmap changes, including Composite redirection and Present/rootless pixmap swaps. Thus a server-owned Damage object registered against the Window is a stronger candidate for source-update tracking than a Damage object registered against a one-time Pixmap pointer.

However, this does **not** solve the compositor-facing pixmap identity problem:

- `SingleCompositeNameWindowPixmap()` increments the current `PixmapRec.refcnt` and stores that exact `PixmapPtr` in the client's pixmap resource.
- A later `SetWindowPixmap()` changes the pixmap associated with the Window; the resource already returned by Composite still references the previously named pixmap object unless an explicit mechanism updates/replaces that resource.
- The server's Damage migration keeps a window-registered Damage object following the Window's current pixmap; it does not automatically retarget a compositor client's already-created named-pixmap resource.

This yields a promising but still unproven prototype direction: keep a server-owned Damage object registered on the Window to observe updates and pixmap swaps, while exposing a separate stable transformed Pixmap object to the compositor and updating its contents. Resize, depth/format changes, compositor re-name behavior and outstanding references still need explicit treatment. The stable-object approach must not be accepted until Composite protocol semantics and real compositor behavior are verified.

Relevant implementation locations:
- `miext/damage/damage.c:getDrawableDamageRef()`
- `miext/damage/damage.c:damageSetWindowPixmap()`
- `miext/damage/damage.c:DamageRegister()`
- `Xext/composite/compext.c:SingleCompositeNameWindowPixmap()`

### 16.2 Composite protocol lifetime and transformed-buffer representation

The Composite protocol documentation adds an important lifecycle constraint: `NameWindowPixmap` returns a reference to the off-screen storage; the window receives a new pixmap when it is mapped or resized, while the old named pixmap remains allocated until the client frees it. A compositor must name the new storage again after those lifecycle events. This matches the Xserver implementation's explicit pixmap reference in `SingleCompositeNameWindowPixmap()`.

Consequences for this design:

1. A transformed compositor-facing buffer must have an explicit generation/lifetime tied to the corresponding redirected-window storage generation.
2. On resize/remap, the server must create/prepare a new transformed generation. The old generation must remain valid while the compositor still holds a reference, then be released safely.
3. The server must not silently mutate an old named pixmap to represent a different window-storage generation or dimensions.
4. The compositor needs an unambiguous notification/update path telling it when to release the old named resource and acquire the new transformed representation. Existing Configure/Map and Damage behavior may help, but the exact ordering must be verified.
5. Present pageflip behavior must be analyzed separately from the Composite map/resize generation rule. A Present pixmap swap is not automatically equivalent to a Composite resize/remap event.

There is also a format/API issue. The legacy CompositeNameWindowPixmap request returns a normal X Pixmap resource representing the window's off-screen storage. A high-precision linear working buffer may require a representation (for example FP16 RGBA) that is not expressible as an ordinary legacy X pixmap with the window's original depth/bpp. Merely returning a different internal PixmapPtr from the existing request risks violating established depth/format expectations and does not provide the compositor with a reliable color-space descriptor.

Therefore, the preferred direction for investigation is a **versioned color-managed Composite handoff** that returns or names a transformed buffer together with:
- an explicit pixel format / DRM fourcc and modifier where applicable;
- dimensions and plane/stride/offset information;
- an immutable working-space identifier/description;
- synchronization primitives or a defined fence protocol;
- a generation identifier and lifetime/release semantics.

This could be implemented as a new X extension or an explicitly negotiated Composite extension revision. The exact transport (X Pixmap resource, DRI3 FD(s), or another server-owned buffer handle) remains **OPEN**. The legacy Composite request and legacy compositor behavior must remain unchanged unless the client explicitly negotiates the color-managed handoff.

Reference: Composite protocol/API documentation states that a new pixmap is created when a window is mapped or resized and that the old pixmap remains allocated until freed. See https://manpages.debian.org/unstable/libxcomposite-dev/XCompositeNameWindowPixmap.3.en.html and the Composite protocol text at https://sources.debian.org/src/xorgproto/2018.4-4/compositeproto.txt.

## 17. References

- X11Libre Xserver source: https://github.com/X11Libre/xserver
- X11Libre HDR/color-management discussion: https://github.com/orgs/X11Libre/discussions/251
- X11Libre misc issue #250: https://github.com/X11Libre/misc/issues/250
- Wayland color-management concepts (reference only): https://wayland.freedesktop.org/docs/book/Color.html
- Linux DRM/KMS documentation: https://docs.kernel.org/gpu/drm-kms.html
- Historical X.Org DeepColor extension discussion: https://groups.google.com/g/xorg-devel/c/JmhOVTpZcs8


### 16.3 EDID length and extension-count safety (2026-11-26 follow-up)

A fresh review of the current EDID entry points found that the Stage 0 integration must first harden the size contract. The existing `xf86ParseEDID()` API is not yet sufficient to make truncated EDID safe:

- `hw/xfree86/ddc/interpret_edid.c:243-268`: `parseEDID()` reads the base EDID fields and extension count before checking that `size >= EDID1_LEN`. It stores the extension count from `(char *)` rather than an unsigned byte, so values above 127 can become negative on platforms with signed `char`.
- `interpret_edid.c:285-291`: `xf86ParseEDID()` rejects NULL/zero inputs but accepts every nonzero size, including buffers shorter than the 128-byte base block.
- `interpret_edid.c:273-274`: the complete-raw-data flag is set for any `size > 128`; it does not verify that the buffer contains all extension blocks claimed by byte 126.
- `interpret_edid.c:465-488`: `cea_for_each_db()` trusts `mon->no_sections` when stepping through `rawData`. The flag proves only that the caller asserted completeness; it does not carry or check the actual allocation length.
- `hw/xfree86/drivers/video/modesetting/drmmode_display.c:3206-3212`: the modesetting path currently calls the legacy `xf86InterpretEDID()` (which receives no size) and sets `MONITOR_EDID_COMPLETE_RAWDATA` whenever the DRM EDID blob is merely longer than 128 bytes. It does not compare the blob length with the extension count in the base block. There is still no production call to `xf86DoInterpretHDRMetadata()`.
- `interpret_edid.c:883-889`: `xf86DoInterpretHDRMetadata(NULL)` is not safe. `cea_for_each_db()` returns early for NULL, but the caller then dereferences `monPtr->hdr` in its return expression.
- `hw/xfree86/ddc/ddc.c:443-457`: the optional `xf86DoEEDID(..., complete=true)` path allocates extension storage from the declared count, does not check `reallocarray()` before using its result, ignores individual extension-read failures, and marks the result complete unconditionally. The normal `xf86DoEDID_DDC2()` wrapper currently passes `complete=false`, so it reads only the base block.

**Stage 0 consequence:** do not simply add `xf86DoInterpretHDRMetadata(mon)` after the current modesetting parse. That would make malformed/truncated blobs a potential out-of-bounds read. First define and test a length-aware parse invariant:

1. Reject a NULL input or any size smaller than `EDID1_LEN` before reading base fields.
2. Read byte 126 as `uint8_t`; compute the number of physically available complete extension blocks from the supplied size without overflow.
3. Never expose an extension count larger than the number of complete blocks actually present. Decide explicitly whether an incomplete EDID is rejected or retained as a base/partial monitor record with extension parsing disabled; do not mark it complete.
4. Ensure every extension walker is bounded by the validated available length/count, not merely by a completeness flag. Audit non-HDR walkers too because they also consume `no_sections` and `rawData`.
5. Make `xf86DoInterpretHDRMetadata()` return false for NULL/incomplete inputs without dereferencing them.
6. In modesetting, pass the DRM blob's actual `length` through the size-aware parser and expose HDR sink capabilities only when all claimed extension blocks are present and validated. Preserve the existing legacy EDID behavior when the feature is not used.
7. For DDC reads, check allocation and every extension-read result before setting the complete flag; partial reads must not masquerade as complete EDID.

Required regression cases now include sizes 0, 1, 127, 128, 129, 255, 256, and 383 bytes; extension count 0, 1, 2 and 255; claimed count greater than available complete blocks; exact full-length buffers; extra trailing bytes; NULL monitor; malformed CTA data-block offsets/lengths; and a valid no-HDR CTA EDID. Tests must prove no extension walker reads outside the supplied buffer (ASan where available).

This is a source-review finding and a Stage 0 design constraint only. No Xserver implementation, branch, build or test was changed or run.


### 16.4 EDID parser call-site inventory and bounded-walker implications (2026-11-26)

A repository-wide code search confirms that `xf86ParseEDID()` currently has no production call site: only its declaration, implementation and the generated API feature marker are found. Existing consumers still use the legacy `xf86InterpretEDID()` / `xf86InterpretEEDID()` entry points:

- `hw/xfree86/drivers/video/modesetting/drmmode_display.c:3206-3212` parses the DRM connector EDID blob without passing its length.
- `hw/xfree86/ddc/ddc.c` uses the legacy entry points for DDC1/DDC2 reads; `xf86DoEDID_DDC2()` delegates to `xf86DoEEDID(..., FALSE)`, so that wrapper does not fetch extension blocks.
- `hw/xfree86/int10/vbe.c` parses the VBE-provided EDID through `xf86InterpretEDID()`.
- Tests call the legacy parser directly with captured EDID arrays.

The length issue is not isolated to the new HDR walker. Existing extension consumers in `interpret_edid.c` include `xf86ForEachDetailedBlock()`, `xf86ForEachVideoBlock()`, and `xf86MonitorFindHDMIBlock()`. The first two iterate `mon->no_sections` and compute extension addresses from `mon->rawData`; the CEA data-block helper used by the HDMI lookup also relies on the shared extension walker. Only `cea_for_each_db()` checks `MONITOR_EDID_COMPLETE_RAWDATA`, and that flag currently has no associated byte-length field.

**Design implication:** a fix that only adds a guard inside `xf86DoInterpretHDRMetadata()` is insufficient. The parser needs a single source of truth for the actual raw-data length / number of complete available extension blocks, and every extension walker must be bounded by it. Possible implementation choices (not selected yet):

- add tail-appended raw-data length / available-extension metadata to `xf86Monitor` and make all walkers use it; or
- normalize `no_sections` to the number of complete, validated extension blocks and preserve the advertised count separately only if a real consumer needs it.

The first choice is more explicit but changes the public structure again; the second may alter the historical meaning of `no_sections`. Audit all consumers before choosing. The legacy one-block entry point must not claim or expose unavailable extensions, while size-aware callers should retain complete valid extensions only after validating their actual byte extent. No production call site should infer completeness from `size > 128` alone.

No Xserver source was modified and no build/test was run. This remains a prerequisite design/code-review finding for Stage 0.


## 18. Visual model and client-visible drawable types (proposal; OPEN)

### 18.1 Do not conflate a Visual with a pixel format

An X11 Visual describes the interpretation and colormap model of a window's core pixel values. It is not a general buffer-format descriptor and does not carry transfer functions, primaries, white point, reference white or luminance range.

The current Xserver `VisualRec` contains a Visual ID, core visual class, RGB masks/offsets and colormap properties. Pixmap storage layout is described separately by `PixmapFormatRec` (depth, bits-per-pixel and scanline padding). Render picture formats are another layer. These existing abstractions must not be collapsed into one HDR-specific type.

### 18.2 Core Visual class decision

**Proposal: do not introduce a new core X11 Visual class for HDR/FP16 in the first version.**

The six core Visual classes (StaticGray, GrayScale, StaticColor, PseudoColor, TrueColor and DirectColor) have established protocol and client-library semantics. An implementation-private class value such as 6 is not a portable X11 Visual class. Existing clients may reject or mis-handle an unknown class, and server-side code often branches explicitly on the established classes.

Instead, an HDR-capable window should normally continue to use a core-compatible Visual class, most commonly TrueColor. The color-management extension may expose additional, versioned per-Visual capabilities/attributes, queried by Visual ID. This is referred to in this draft as an **extended Visual profile**, not a new core Visual class.

Candidate extended-Visual attributes:
- whether the Visual may be used with the color-managed window/buffer path;
- supported or preferred client buffer formats;
- whether a core pixmap representation exists for the selected format;
- alpha presence and alpha representation;
- supported rendering/transport paths (core X, Render, GLX, DRI3 or extension buffer import);
- restrictions such as required extension version or synchronization support.

The extension must not change the meaning of the core Visual class or its RGB masks. Unknown extended attributes must be safely ignorable by older clients.

### 18.3 FP16 and the limits of core pixmaps

RGBA FP16 normally requires 64 bits per pixel. The current core pixmap format model and server code have 32-bpp assumptions in important paths; a new Visual ID alone does not make a 64-bpp core pixmap legal or make core X drawing operations understand floating-point channels.

Therefore:
- packed 10:10:10:2 may be represented through a conventional depth-30 / 32-bpp TrueColor-compatible path where the DDX and server support it;
- client-visible FP16 should initially be represented as an extension-managed buffer/surface format, not by pretending it is an ordinary core depth/bpp pixmap;
- the extension must define how such a buffer is associated with a Window and how GLX/DRI3 or another transport imports/exports it;
- core X drawing requests must not silently write integer pixels into an FP16 buffer. Either define explicit conversion semantics or reject unsupported operations on that surface.

The extension may later add a richer drawable configuration model, but it must not require changing the core Visual class enumeration merely to describe a new storage format.

### 18.4 Window creation and Visual selection

A client may select a Visual ID when creating a window using the existing core CreateWindow request. The extended Visual profile is queried before selection and is immutable for that Visual on that screen.

The selected Visual does not itself select the source color description. After window creation, the client explicitly associates an immutable color-description object with the window (or with an extension-managed buffer/frame where required). This keeps window compatibility, pixel storage and colorimetry independent.

**OPEN:** Determine whether GLX FBConfigs should advertise the extended Visual profile and how a client chooses a Visual/buffer combination when the GLX Visual is only a compatibility shell for an extension-managed FP16 surface.

## 19. Pixel and buffer format model (proposal; OPEN)

A buffer format descriptor must describe storage only. At minimum it contains:
- format identifier, with a stable Xlibre-defined mapping to DRM FourCC where applicable;
- channel order and component data type (UNORM, SNORM, FLOAT, etc.);
- component widths and offsets, including alpha;
- plane count, per-plane offsets, strides and modifiers for imported/exported DMA-BUFs;
- dimensions and any required alignment;
- alpha-channel presence and component format (in the pixel format); alpha mode (straight, premultiplied in encoded/electrical values, or premultiplied in linear/optical values) is separate per-surface/buffer representation metadata;
- synchronization/import/export capabilities.

The format identifier is not a color-space identifier. In particular, FP16 does not imply scRGB, and 10:10:10:2 does not imply BT.2020 or PQ.

Initial candidate format tiers:
1. Existing core-compatible 8-bit formats: unchanged legacy path.
2. Packed 10-bpc RGB (commonly 10:10:10:2 in a 32-bit word): optional client-facing format where existing Xserver/DDX/GLX paths support it.
3. RGBA FP16: extension-managed client buffer format, requiring explicit capability negotiation and buffer transport; not a new core pixmap depth in v1.
4. X-server-owned high-precision transformed buffers: internal format selected by the server, independently of the source buffer format and the Visual.

The extension must not promise a format merely because the DRM driver can scan it out. Import, rendering, sampling, transformation, export and compositor consumption are separate capabilities.

## 20. Color-description object and attachment model (proposal; OPEN)

### 20.1 Immutable description object

The protocol should use an immutable, versioned Color Description resource. A client creates it, receives a resource ID, and may bind it to a window or an extension-managed buffer/frame. Immutability avoids races where a client changes a profile while the server is processing Damage or Present work. A changed description is represented by creating a new object and rebinding it.

The object should identify:
- RGB primaries and white point, or a registered named color space with exactly defined chromaticities;
- transfer function / encoding (at least Linear, sRGB and the explicitly selected HDR transfer functions);
- reference-white convention and units;
- minimum, nominal/reference and peak luminance semantics where relevant;
- optional static content/mastering metadata, with field-specific units and validation;
- rendering intent / gamut mapping preference, if supported;
- alpha mode as separate representation metadata, not inferred from the pixel format.

ICC profile support, if added, must be separately versioned and specify maximum profile size, validation, caching, canonical identity and lifetime. It must not be an unbounded arbitrary blob.

### 20.2 Attachment scope

**Proposal for v1:** bind the static source Color Description to a Window. The description characterizes the color values produced for that window, independent of the rendering API.

- A window without an explicit description remains in the legacy/unmanaged color mode until the negotiated color-managed path defines its legacy interpretation.
- Child windows do not silently inherit a parent's explicit description in v1; each window has its own explicit state or the documented legacy default. This avoids accidental propagation across independently rendered child content.
- Reparenting does not change the description.
- Destroying a window releases its binding; the immutable description resource remains alive while referenced by other windows or buffers.
- A description change invalidates the entire transformed content for that window, not merely the next damaged rectangle.

**OPEN:** Decide whether a pixmap/buffer-level override is required in v1 for clients that render differently encoded content into one window. If required, define precedence (frame/buffer override over window default) and make the description immutable for the lifetime of the submitted buffer/frame.

### 20.3 Dynamic metadata

Dynamic per-frame metadata must be attached atomically to the corresponding Present/buffer submission, not updated through a mutable window property. The metadata and buffer generation must travel as one presentation transaction so frame N metadata cannot be applied to frame N+1.

PQ and HLG are distinct transfer functions and require distinct reference/peak luminance definitions. The protocol must not treat one as an alias for the other.

## 21. Color transform and compositor handoff (approved semantics; wire contract open)

### 21.1 Transform semantics

For every window entering the negotiated color-managed compositor path, the X server is responsible for producing the compositor-facing transformed representation. The compositor must not need to inspect or transform each window's original source color description.

The conceptual transform is:

`source encoded samples -> decode transfer function -> source linear RGB -> convert primaries/white point -> common working-space linear RGB -> encode working-buffer representation`

The transform must preserve alpha semantics. Color channels are transformed in linear light. For premultiplied input, the implementation must correctly account for premultiplication (including zero-alpha pixels); it must not apply a nonlinear transfer function directly to premultiplied encoded values.

The compositor-facing contract must specify:
- exact working-space primaries, white point, scale/reference-white convention and numeric range;
- exact buffer format and alpha representation;
- synchronization required before compositor reads;
- generation, replacement, release and resize semantics;
- whether the compositor may sample the buffer directly or must import it through a specified API.

### 21.2 Legacy source content

A color-managed compositor cannot receive a mixture of transformed HDR buffers and untransformed legacy SDR buffers while being told that all buffers share one working-space interpretation.

**Owner-approved legacy interpretation:** windows without explicit color metadata use sRGB/D65 and the session-wide configurable LegacySDRReferenceWhite policy defined in section 25. When the compositor negotiates the color-managed handoff, the X server transforms legacy windows according to that policy as well. If the policy cannot be applied safely, the server must keep the legacy Composite path rather than silently mixing incompatible representations.

A compositor that has not negotiated the new handoff continues to receive the existing legacy Composite representation unchanged.

### 21.3 Responsibility boundary

The X server performs per-window source-to-working-space conversion. The compositor performs spatial composition, clipping, stacking, effects and alpha blending under the declared working-space semantics. The final output transform (tone mapping, gamut mapping, output transfer encoding and output metadata) remains a separate output responsibility and must be coordinated with the final scanout path.

## 22. Proposed extension surface (conceptual; no wire ABI selected)

A single versioned extension should own color-management semantics and capability negotiation. Names below are descriptive placeholders, not approved protocol names or opcode assignments.

Candidate operations:
- QueryVersion / QueryCapabilities;
- QueryVisualProfiles (extended Visual attributes by screen and Visual ID);
- QueryBufferFormats (format/import/export/render/transform capabilities);
- CreateColorDescription / DestroyColorDescription / QueryColorDescription;
- SetWindowColorDescription / GetWindowColorDescription;
- attach a color description and optional dynamic metadata atomically to a buffer/Present transaction where required;
- negotiate a versioned color-managed Composite handoff.

The extension must define error codes, byte order, bounds, resource ownership, authorization, request ordering, event/error behavior and version negotiation. It must not reuse core opcodes or encode private state through malformed core requests.

The extension should be split internally into independent capability groups so a server can expose source metadata support without claiming FP16 import, and expose FP16 buffer transport without claiming a particular output mode. A capability must only be advertised when the full relevant path is usable.

## 23. Owner decisions required before freezing this part of the design

1. **Visual policy:** accept the proposed extended-Visual profile on top of existing core Visual classes, rather than a new core Visual class?
2. **FP16 scope:** accept extension-managed FP16 buffers in v1, while keeping them outside ordinary core pixmap depth/bpp semantics?
3. **10-bpc staging:** support packed 10:10:10:2 as an optional conventional path where available, without making it the universal HDR format?
4. **Color Description binding:** accept immutable window-level descriptions for v1, with frame/buffer-level dynamic metadata only where required?
5. **Legacy interpretation:** choose the legacy SDR profile used when a compositor negotiates the common working-space handoff but a window has no explicit description?
6. **Working space:** select exact primaries, white point, reference-white convention, numeric range and compositor buffer format?
7. **Transfer-function scope:** select which transfer functions are mandatory in v1 (sRGB/linear/PQ/HLG) and define their luminance semantics?

Unapproved portions of sections 18–23 remain draft. Owner-approved design decisions are recorded in sections 21 and 24–26 and in durable decision files; they do not freeze the wire ABI. No Xserver implementation is authorized by this document.


## 24. Common linear working space (owner-approved design; protocol details open)

### 24.1 Recommended v1 definition

**Owner-approved definition:** one fixed, output-independent, absolute-linear RGB working space for the color-managed Composite handoff:

| Property | Proposed v1 value |
|---|---|
| Working-space primaries | ITU-R BT.2020 / Rec.2020 |
| Red primary | CIE 1931 x=0.708, y=0.292 |
| Green primary | CIE 1931 x=0.170, y=0.797 |
| Blue primary | CIE 1931 x=0.131, y=0.046 |
| White point | CIE D65, x=0.3127, y=0.3290 |
| Transfer function | Linear light (identity between normalized linear values and light) |
| Reference-white scale | R=G=B=1.0 represents D65 white at 203 cd/m² |
| Nominal luminance equation | Y = 0.2627 R + 0.6780 G + 0.0593 B; multiply Y by 203 cd/m² for absolute luminance |
| Compositor handoff format | RGBA IEEE-754 binary16 (FP16), subject to negotiated support |
| Alpha in handoff | Linear, premultiplied |
| Output dependency | None; do not scale or clamp this space according to any connected display |

The space is referred to descriptively as **Xlibre Linear BT.2020 / 203** in this draft. This is a proposed Xlibre working-space profile name, not a standards body registration or wire identifier.

### 24.2 Why this choice

- BT.2020 provides a wide, standardized RGB primary set and D65 white point that align directly with the BT.2100 HDR ecosystem.
- Linear light makes alpha blending and spatial composition well-defined; encoded PQ, HLG or sRGB values must never be blended directly.
- A fixed 203 cd/m² reference scale makes the working values independent of the current output device. It is a canonical scale, not a statement that every display's diffuse/reference white must physically be 203 cd/m².
- 203 cd/m² is used as HDR reference white in common BT.2100 production guidance for a 1 000 cd/m² reference display. ITU-R BT.2408 also documents that operational HDR reference-white levels can vary with nominal peak display luminance. Xlibre deliberately chooses a fixed internal scale instead of inheriting a display-dependent production target; output mapping handles the difference.
- Rec.2020 is preferred over ACEScg for this system compositor contract because BT.2020/D65 is directly aligned with common HDR display signaling. ACEScg remains a valid client/source color space and must be transformable into the working space; it is not the compositor handoff space in v1.

### 24.3 Numeric range and FP16 behavior

The nominal RGB value 1.0 is reference white, **not a maximum**.

- Values above 1.0 are valid and represent highlights above reference white. For example, neutral RGB 4.0 represents 812 cd/m².
- Negative RGB components are valid intermediate values and may be produced by conversion of colors outside the Rec.2020 primary triangle or by color transforms. They must not be clipped merely because they are negative.
- Gamut mapping and clipping are not performed during source-to-working-space conversion. They belong to the later output transform.
- The implementation must not silently clamp working RGB to [0, 1].
- The source-to-working transform must preserve finite values as accurately as the selected buffer precision allows. Non-finite FP inputs (NaN and infinities) must not be allowed to poison compositor blending; the exact sanitization/error policy remains OPEN.
- FP16's finite numeric range is ample for the nominal PQ range when scaled by 203 cd/m² (10 000 cd/m² is approximately 49.26 in working-space units). This does not make FP16 the required client source format; it is the proposed compositor-facing intermediate.

### 24.4 Absolute luminance conversion

All source descriptions that are converted into this working space must resolve their signal values to a common absolute-light scale before the final RGB matrix conversion.

For a source description with an explicit reference-white luminance:
- decode the source transfer function according to its exact definition;
- resolve its linear values to cd/m² using the source description's luminance/reference-white semantics;
- chromatically adapt to D65 when the source white point differs, using one protocol-defined adaptation method (method still OPEN);
- convert source linear tristimulus values to BT.2020 linear RGB;
- divide by 203 cd/m² to obtain Xlibre Linear BT.2020 / 203 values.

For PQ/ST 2084, the transfer function provides an absolute luminance interpretation and must be decoded accordingly. Do not treat PQ code value 1.0 as the working-space reference white.

For HLG, the transfer function alone is insufficient to determine a unique absolute display luminance mapping in all viewing environments. The protocol must define the required HLG system gamma / nominal peak / viewing assumptions before HLG sources can be advertised as fully supported. Until then, a server must not silently treat HLG as PQ or invent a peak-luminance assumption.

For SDR sources, the Color Description's reference-white luminance determines how encoded SDR white maps into the common working space. For unmanaged legacy windows in the HDR color-managed compositor path, the owner-approved implicit source is sRGB/D65 with a session-wide configurable LegacySDRReferenceWhite default of 203 cd/m²; 80 and 100 cd/m² remain configurable alternatives. This is separate from the working-space scale and is never per-output.

### 24.5 Alpha and blending contract

The transformed compositor-facing buffer uses linear-light, premultiplied alpha:
- alpha is linear and remains in [0, 1];
- RGB is multiplied by alpha after color conversion into the working space;
- blending is performed on premultiplied linear RGB and linear alpha;
- the source buffer's alpha mode must be described independently from its pixel format (e.g. straight, premultiplied in encoded/electrical values, or premultiplied in linear/optical values);
- nonlinear transfer decoding must not be applied directly to encoded-premultiplied RGB. The transform must correctly unpremultiply where necessary, decode/convert in the correct domain, then premultiply in linear working space;
- zero-alpha pixels must not leak arbitrary hidden RGB into the transformed result.

The pixel format states whether an alpha channel exists and how it is stored. The alpha mode is independent representation metadata, as demonstrated by the separate color-representation model in Wayland.

### 24.6 Consequences and limits

- Windows spanning multiple outputs receive the same transformed values; no monitor-specific transform is baked into their per-window buffers.
- Moving a window between outputs does not require reinterpreting or regenerating the source-to-working-space transform solely because the output changed. The final output transform may change.
- The compositor can blend transformed windows consistently, but remains responsible for spatial composition, effects and final output composition.
- The output stage maps this common working space to each output's actual color volume, luminance range, transfer function and supported KMS signaling. It may tone-map or gamut-map there.
- A compositor that has not negotiated the color-managed handoff continues to use the unchanged legacy Composite path.

### 24.7 Owner decisions still required

1. Approve or change the BT.2020/D65 primaries and white point.
2. Approve the fixed 203 cd/m² reference-white scale, explicitly independent of output peak.
3. Approve RGBA FP16 as the v1 transformed compositor handoff format, with capability negotiation and a defined fallback when unavailable.
4. Approve linear premultiplied alpha as the compositor-facing contract.
5. Select the chromatic-adaptation method.
6. Define HLG system-gamma / nominal-peak interpretation before advertising HLG support.
7. The legacy SDR default is now owner-approved in section 25: 203 cd/m², session-configurable and not per-output.
8. Define the policy for NaN/infinity values in client-provided floating-point buffers.

Section 24 is a design recommendation, not yet an owner-approved normative ABI. No Xserver implementation is authorized by this section.


## 25. Legacy unmanaged SDR source policy (owner-approved design direction)

### 25.1 Implicit source description

When a compositor has explicitly negotiated the color-managed handoff, every redirected window must be represented in the common working space. A window with no explicit Color Description therefore receives a deterministic implicit legacy profile:

- primaries: sRGB / ITU-R BT.709 primaries;
- white point: D65;
- transfer function: IEC 61966-2-1 sRGB;
- reference white: the session-wide `LegacySDRReferenceWhite` setting;
- no per-output adaptation or monitor-dependent scaling in the per-window transform.

The implicit profile applies only to windows without explicit color metadata. An explicit, valid Color Description takes precedence and is not overridden by this legacy setting.

### 25.2 Default and rationale

**Default: 203 cd/m².** In the negotiated HDR compositor path, legacy SDR graphics white is anchored to the common HDR reference white (203 cd/m²). This follows current HDR compositing guidance for mixing SDR and HDR content. It is not a claim about the physical peak luminance of the connected monitor.

The common working-space scale remains unchanged: neutral working-space RGB 1.0 is 203 cd/m². Consequently, an implicit legacy sRGB white at the default setting maps to neutral working-space RGB 1.0. If the user/session chooses 80 cd/m², it maps to approximately 80/203 = 0.3941; at 100 cd/m² it maps to approximately 0.4926.

This distinction is intentional:
- 80 cd/m² is the conventional sRGB reference-viewing white in an SDR reference-display context.
- 203 cd/m² is the HDR reference / graphics white used by the selected common working-space scale and the recommended anchor for SDR graphics when composited with HDR.
- The user or session policy may choose a lower value (for example, 80 or 100 cd/m²) to suit viewing conditions or personal preference.

The 203 cd/m² default is supported by the W3C CSS Color HDR draft, which identifies 203 cd/m² as HDR reference white and says SDR content should be anchored to that level when composited with HDR. The 80 cd/m² sRGB reference-viewing value remains a useful configurable alternative; it is also reflected in the Wayland color-management protocol's default sRGB luminance semantics. These are references for luminance semantics, not protocol dependencies.

### 25.3 Configuration scope and behavior

The setting is a single session/server-wide luminance value, expressed in cd/m². Its exact configuration interface and validated range remain **OPEN**; candidate sources include an X server configuration option or a session policy exposed through the future color-management extension.

The setting must not be per-output. A window spanning multiple monitors or moving between them must retain the same source-to-working-space values.

If runtime changes are supported, changing the setting invalidates and regenerates the complete transformed content of every affected legacy window. The server must publish a new transformed-buffer generation only when its contents are complete and synchronized. A partial mixture of old and new reference-white mappings is not valid.

Explicit per-window Color Descriptions remain authoritative. Changing the legacy setting must not alter explicitly described windows.

### 25.4 Owner decisions recorded

The owner approves the following design direction:
1. Keep the common working space as output-independent linear BT.2020 / D65 with 1.0 = 203 cd/m².
2. Use negotiated RGBA FP16 for the transformed compositor handoff, with linear premultiplied alpha; capability negotiation and a defined fallback remain required.
3. Interpret unmanaged legacy windows as sRGB/D65 and use a session-wide configurable reference-white value, defaulting to 203 cd/m² to anchor SDR graphics to HDR reference white.
4. Keep that setting independent of output/monitor; explicit Color Descriptions override it. Lower values such as 80 or 100 cd/m² remain configurable.

Still open: exact configuration interface/range, chromatic-adaptation method, HLG absolute-luminance assumptions, non-finite floating-point input policy, and the final wire ABI / compositor handoff protocol. These decisions do not authorize Xserver implementation.


### 25.5 Clarification: SDR reference viewing versus HDR composition

The default above is specifically for the **HDR color-managed composition path**. It must not be confused with the conventional sRGB reference-display viewing condition, where sRGB white is commonly described as 80 cd/m². When legacy SDR content is mixed with HDR content, the proposed default anchors its white/graphics-white level to 203 cd/m² HDR reference white. The setting allows users or session policy to select a lower luminance (including 80 or 100 cd/m²) when desired.

This distinction is important because the implicit source profile's transfer function/primaries and the luminance assigned to its white are separate semantics. The legacy default remains sRGB/D65; only its session-level reference-white mapping is configurable.


## 26. Chromatic adaptation and source-to-working transform (owner-approved design; wire ABI open)

### 26.1 Recommended chromatic-adaptation method

**Owner-approved v1 decision: use the linear Bradford chromatic adaptation transform (CAT) as the single mandatory method for relative-colorimetric source RGB descriptions whose declared white point differs from D65.** Use full adaptation (degree of adaptation = 1); do not expose CAT selection or degree-of-adaptation as a per-window or per-output tuning knob in v1.

Rationale:
- ICC.1:2022 recommends linear Bradford when there is no specific reason to choose another CAT, citing broad implementation and demonstrated visual performance.
- Different CATs produce different results; fixing one method avoids inconsistent output across server backends, compositors and drivers.
- Bradford is widely deployed in profile-based color management and is therefore a practical interoperability baseline.
- CAT16 is a valid later research/extension candidate, but there is no established requirement in this desktop source-to-common-working-space use case that justifies making it the v1 default or adding a negotiation axis now.

The algorithm is the linear Bradford model described in ICC.1:2022 Annex E (not an implementation-specific approximation). Given normalized source and destination white XYZ vectors Ws and Wd (both with Y=1), and the ICC linear Bradford cone-response matrix B, compute:

`M_CAT = inverse(B) * diag((B*Wd).x/(B*Ws).x, (B*Wd).y/(B*Ws).y, (B*Wd).z/(B*Ws).z) * B`

Apply `M_CAT` to XYZ values after transfer decoding and source RGB-to-XYZ conversion. The destination white is D65. If the source white is already D65, skip adaptation (identity transform). Do not apply a second CAT later in the output path merely because the output has another white point; output adaptation, if any, belongs to the output transform.

### 26.2 Relative versus absolute colorimetric meaning

A white-point chromaticity does not by itself tell us whether the source values should undergo appearance adaptation or preserve absolute measured colorimetry. This distinction must be explicit in the source-description model before the wire ABI is frozen.

**Owner-approved v1 semantic rule:**
- Relative-colorimetric RGB source descriptions use the declared source white point and the fixed full Bradford transform to adapt that white to D65.
- Absolute-colorimetric XYZ data, or a profile transform that already produces absolute colorimetric PCS values, must preserve its declared colorimetry and must not receive an additional implicit CAT.
- The source-description model must explicitly distinguish relative RGB semantics from absolute-colorimetric XYZ / already-adapted PCS data. This distinction must be versioned and must not be inferred from transfer function, primaries, or buffer format alone.

ICC profile ingestion and CMM ownership remain a separate open design question; see section 26.6.

### 26.3 Proposed source-to-working pipeline

For a relative-colorimetric RGB source with a valid Color Description:

1. Interpret the source buffer according to its declared pixel format and alpha mode. Correctly unpremultiply encoded-premultiplied input before nonlinear transfer decoding; handle zero-alpha samples without leaking hidden RGB.
2. Decode the declared transfer function into linear-light source RGB. PQ must resolve to absolute luminance; HLG remains unavailable until its system-gamma / nominal-peak semantics are specified.
3. Convert linear source RGB to normalized source XYZ using the matrix derived from its declared primaries and white point.
4. Apply full linear Bradford adaptation from the source white point to D65, unless the source white is already D65.
5. Convert D65 XYZ to linear BT.2020 RGB.
6. Resolve luminance semantics before scaling into the working space: for relative SDR encodings (including sRGB), multiply the normalized linear XYZ values by the Color Description's reference-white luminance in cd/m²; for PQ, the decoded values already represent absolute luminance and must not be multiplied by the reference-white value; for an absolute-colorimetric XYZ/PCS source, preserve its supplied absolute scale. Then divide XYZ by 203 cd/m² to express it on the common working-space scale. A linear scalar commutes with the CAT and RGB/XYZ matrices, but the implementation must use one defined convention and preserve absolute luminance.
7. Premultiply the resulting linear working RGB by linear alpha and write RGBA FP16. Do not clamp gamut excursions or values above 1.0.

The implementation may combine steps 3–6 into a precomputed 3x3 matrix plus scalar for a given immutable Color Description. Such optimization must be numerically equivalent to the specified order and must not change semantics.

### 26.4 Validation and numerical behavior

- Source and destination white chromaticities must be finite and valid; reject invalid descriptions rather than silently substituting D65.
- The CAT matrix must be finite and invertible. Reject a description if the resulting transform is singular or numerically unusable.
- Identical source and destination white points must produce an identity CAT (within the defined canonical numeric representation).
- Negative XYZ/working RGB components created by adaptation or gamut conversion are valid intermediates; do not clamp them.
- No per-output or per-monitor CAT choice is permitted in the source-to-working transform.
- Reference white is a luminance scale, not a chromatic adaptation parameter. Changing `LegacySDRReferenceWhite` must not change the CAT matrix.

### 26.5 Open design questions

1. ~~Approve linear Bradford as the fixed v1 CAT for relative-colorimetric source RGB descriptions?~~ **Approved by owner, 2026-11-26.**
2. ~~Approve full adaptation only (no partial adaptation control in v1)?~~ **Approved by owner, 2026-11-26.**
3. ~~Define the explicit source-description distinction between relative-colorimetric RGB and absolute-colorimetric / already-adapted PCS data?~~ **Approved by owner, 2026-11-26.**
4. ~~Define ICC profile ingestion and CMM ownership so source adaptation occurs exactly once.~~ **Owner decision, 2026-11-26: defer arbitrary ICC profiles and CMM integration beyond v1; use parametric Color Descriptions only.**
5. Define validation tolerances and canonical white-point precision for identity detection and matrix caching.

### 26.6 ICC profile ingestion and CMM ownership (v1 owner decision)

**Owner-approved for v1 (2026-11-26): support parametric Color Descriptions only; defer arbitrary embedded ICC profiles and CMM integration.**

The v1 Color Description model carries explicit, validated parameters: RGB primaries, white point, transfer function, luminance/reference-white semantics and the relative-versus-absolute colorimetric interpretation. It does not carry an arbitrary ICC profile blob or require a general-purpose CMM.

Rationale:
- Keeps the first source-to-working transform deterministic and bounded.
- Avoids ambiguity over rendering intent, profile connection space (PCS), profile-specific chromatic adaptation and which component owns CMM execution.
- Avoids making a CMM dependency a prerequisite for the first color-managed path.

Future ICC support requires a separate owner decision and design covering one clearly identified CMM, rendering-intent behavior, profile lifetime/cache rules, PCS adopted white/scale and exactly-once adaptation. It must not be added implicitly to v1.

### 26.7 Remaining section-26 work

Validation tolerances and canonical white-point precision for identity detection and matrix caching remain open. HLG remains gated pending system-gamma / nominal-peak semantics; non-finite FP16 input/output policy remains open.

The decisions in sections 26.1, 26.2 and 26.6 are owner-approved design direction, not a frozen wire ABI. They do not authorize Xserver implementation. The extension protocol, validation tolerances, canonical numeric representation and remaining open questions must be resolved before implementation.

## 27. Direct output conversion when no compositor is active (design proposal)

### 27.1 Motivation and separation from the compositor path

When no compositor is active, X11 does not normally expose a separately composited, color-managed buffer for every window. The server and clients update the screen/root presentation surface, and the display engine scans that final desktop image out to one or more outputs.

In this mode, it may be more efficient and architecturally correct to convert the **final desktop image independently for each output** into that monitor's selected output color space, rather than first converting every window into the compositor-facing common working space. This is a separate output path, not a change to the owner-approved compositor handoff.

The two paths have different responsibilities:
- **Compositor active:** transform each source window into the common, output-independent linear BT.2020 / D65 working space (203 cd/m² scale); hand the transformed buffers to the negotiated compositor interface. The compositor performs spatial composition; the output stage then converts the composed result for each output.
- **No compositor:** preserve the normal non-redirected desktop rendering path where possible, then apply an output-specific transform to the final desktop image for each CRTC/output immediately before scanout. The target is that output's selected encoding and color volume, not the common compositor handoff format.

### 27.2 Why conversion must be per output and late

A desktop can span outputs with different primaries, white points, transfer functions, bit depths and luminance capabilities. A window may cross an output boundary, and an output may be cloned or reconfigured. Therefore a single globally transformed root pixmap cannot in general be correct for all outputs.

The conversion must happen after the desktop image has been spatially assembled and be independently selectable per output. Candidate mechanisms, in preference order:
1. **Per-CRTC DRM/KMS color pipeline:** use supported degamma LUT, CTM and gamma LUT (and any future 3D LUT or equivalent blocks) to perform the output transform in hardware. This is attractive because the same source framebuffer can feed multiple CRTCs while each CRTC applies its own transform.
2. **Per-output GPU/shader conversion:** render or copy the relevant desktop region into an output-specific scanout buffer, applying the full output transform. This is needed when the hardware color pipeline cannot express the required transform or lacks sufficient precision/range.
3. **Constrained legacy fallback:** retain the existing untransformed SDR scanout when no correct transform is available. Do not silently claim HDR or color-managed output merely because the monitor advertises HDR capability.

A single shared buffer preconverted for one monitor must not be scanned out unchanged to another monitor requiring a different transform. If outputs are cloned, they may share a transformed buffer only when their effective output transforms are equivalent.

### 27.3 Output transform responsibilities

The per-output transform owns:
- Conversion from the desktop/source working representation into the output's native or selected RGB primaries and white point.
- Output transfer-function encoding (for example sRGB/SDR or PQ when explicitly enabled and supported).
- Output luminance mapping, tone mapping and gamut mapping as required by the output's actual capabilities and selected mode.
- Correct output signaling and metadata through the DRM/KMS path.

Sink EDID capabilities are constraints, not outgoing frame metadata. The server must derive output encoding and HDR metadata from the actual composed frame and selected output mode, not copy the monitor's EDID HDR static metadata verbatim.

Hardware gamma LUT alone is not assumed to implement a complete HDR transform: it may be only a 1D per-channel stage and may not provide the required 3D gamut mapping, tone mapping, precision or range. Capability discovery and an explicit fallback are required.

### 27.4 Important limits and unresolved questions

- This path is not a substitute for per-window color management. Without per-window redirection/buffers, source colors have already been rasterized into the desktop surface. Rich per-window HDR Color Descriptions may require internal redirection/composition or a future rendering contract; do not pretend a final-output transform can recover source semantics that were lost during rasterization.
- Legacy unmanaged SDR can use a defined session-level SDR interpretation and then be mapped at the output stage. The compositor-path LegacySDRReferenceWhite policy must be reconciled carefully with this direct-output pipeline; it must not be applied twice.
- If the server's screen pixmap is shared by multiple outputs, output-specific shader conversion needs separate per-output targets or an equivalent hardware per-CRTC pipeline.
- Present flips, direct scanout, overlays, hardware cursors and video planes may bypass the primary-plane color pipeline or have separate color properties. Their color behavior must be audited; a correct primary-plane transform does not prove every plane is color-managed.
- RandR gamma controls and calibration semantics must be kept distinct from the full source-to-output color transform. Existing gamma ramps are not automatically a suitable HDR color-management API.
- Output hotplug, mode changes, clone topology and transform updates require atomic/synchronized state changes so a partially updated frame is never shown.

Open design work includes the exact server/driver ownership boundary, KMS property discovery and capability model, transform implementation fallback, multi-output scanout lifecycle, per-plane behavior, direct-scanout policy, and tests. This is a design proposal only; it does not authorize implementation or freeze a protocol/API.


## 28. Driver compatibility and proprietary NVIDIA (owner requirement)

### 28.1 Compatibility is a correctness requirement

**Owner requirement (2026-11-26): future HDR implementation must not break the proprietary NVIDIA Xorg driver.** The proprietary driver is not required to implement HDR. A configuration using that driver must retain a working, predictable legacy X11/SDR path when HDR capabilities are absent.

This is a compatibility requirement, not a promise that every NVIDIA driver release or GPU combination has been tested. The proprietary NVIDIA DDX is external to this repository; its source and binary compatibility must be validated separately on supported target versions before claiming tested compatibility.

### 28.2 Optional capability model

- HDR and color-managed buffer handoff are optional capabilities, never prerequisites for starting the server, initializing a screen, creating a window, using Composite, or presenting ordinary SDR content.
- Discover support at runtime from the actual active DDX, rendering backend, buffer formats and output path. Do not infer it from GPU vendor, PCI ID, EDID claims, presence of a DRM node, or a compile-time feature define alone.
- A DDX/backend that cannot provide a requested capability must report it as unavailable. The extension must expose an explicit capability result and reject unsupported opt-in operations cleanly; it must not partially apply a transform or silently reinterpret pixels.
- Clients/compositors that do not negotiate the new versioned contract retain existing protocol behavior and existing Composite pixmap semantics.
- If the proprietary NVIDIA DDX does not implement the optional server/driver integration, it remains on the existing SDR path. HDR is not advertised as active on that path.

### 28.3 Keep vendor DDXs outside the new mandatory dependency chain

- Do not require the proprietary NVIDIA DDX to implement new callbacks merely to load or run.
- Prefer implementation in private Xserver structures and optional, versioned capability interfaces. Avoid changing existing public driver SDK structures, callback signatures, module ABI contracts or established hook semantics.
- If a new driver-facing hook is unavoidable, design its ABI/versioning and compatibility behavior explicitly before implementation. Do not assume that appending a field is automatically binary-compatible with already-built out-of-tree drivers; inspect the actual SDK/module ABI and reserved-field policy first.
- Keep DRM/libdrm/GBM-specific output work inside the modesetting DDX or another backend that already owns those dependencies. Do not pull these dependencies into generic DIX/core code.
- Do not introduce mandatory Vulkan headers/runtime, a higher global OpenGL baseline, or a new mandatory GPU API for HDR.
- Do not change legacy Composite, DRI3, Present, RandR gamma, pixmap format/depth, or scanout behavior when the feature is not negotiated.

### 28.4 Modesetting findings from current source

The following observations were verified against `X11Libre/xserver:master` at `bb06f6ddca766c41c81494ae228c11e53df59590` (2026-11-26):

- The inspected DRM/KMS implementation is the Xorg `modesetting` DDX under `hw/xfree86/drivers/video/modesetting/`; it is not evidence about the separate proprietary NVIDIA DDX.
- Its CRTC property table currently includes `ACTIVE`, `MODE_ID`, `GAMMA_LUT`, `GAMMA_LUT_SIZE` and `CTM`. No `DEGAMMA_LUT` property handling was found in that CRTC property model.
- CRTC properties are discovered from the actual DRM CRTC with `drmModeObjectGetProperties()`. `use_gamma_lut` is enabled only when a nonzero `GAMMA_LUT_SIZE` is reported; the default policy accepts sizes up to 4096, with the existing `UseGammaLUT` option as an override. Otherwise the legacy `drmModeCrtcSetGamma()` path is used.
- The gamma LUT is a 1D per-channel `drm_color_lut`; CTM is a separate 3x3 DRM color transform. This is not a complete HDR pipeline and does not establish support for PQ encoding, tone mapping, 3D gamut mapping, or final-frame HDR metadata.
- The existing CTM path is gated by gamma-LUT use and property presence. The atomic modeset request inspected here configures CRTC active/mode and primary-plane framebuffer geometry; it does not currently include a full HDR color-pipeline state bundle. Do not treat the current gamma/CTM support as an HDR implementation.
- Present page-flip eligibility checks that the buffer format/modifier is supported by every active CRTC. The current path can flip a full-screen window/framebuffer; a transformed format that is not accepted by all active CRTCs can disable that flip path. This reinforces that format negotiation and output transforms must be optional and preserve the ordinary scanout format.
- Hardware cursors are programmed through separate cursor operations, not through the primary framebuffer's pixel transform. The modesetting Present path also has separate flip eligibility logic and refuses some flips when sprites/overlays are visible. Cursor, overlay and video-plane color behavior therefore needs independent treatment.
- The modesetting DDX has a shared screen pixmap and per-CRTC scanout state; its reverse-PRIME path tracks per-CRTC scanout pixmaps. This is not yet proof that arbitrary output-specific shader buffers can be introduced safely. Multi-CRTC, clone, PRIME and hotplug behavior require dedicated tests.

### 28.5 NVIDIA-specific validation plan

Before enabling HDR by default or changing driver-facing interfaces, validate at least:

1. Proprietary NVIDIA DDX starts the server and supports ordinary SDR clients with the HDR feature compiled in but unnegotiated.
2. Composite redirection/name-pixmap, Damage, Present copy/flip, DRI3 import/export and RandR mode changes retain their legacy behavior.
3. No new driver callback is required; capability discovery reports no HDR support where the DDX/backend cannot provide it.
4. Missing DRM/KMS color properties, unsupported formats/modifiers, unavailable Glamor/GBM and unavailable optional shader features all lead to a clean SDR fallback, not server failure or corrupted output.
5. The same server build still works with the modesetting DDX, with and without optional HDR/KMS properties.
6. Build/package dependency checks confirm that generic Xserver and proprietary-driver loading do not acquire mandatory Vulkan, GBM, or higher OpenGL requirements from the HDR work.

Test on real proprietary NVIDIA driver versions and supported GPU generations. Until that is done, report NVIDIA compatibility as a design constraint and unverified test target, not as a tested fact.

### 28.6 Implementation gate

No HDR path may become mandatory for server startup or legacy rendering. The first implementation must be opt-in, capability-gated, and independently disableable, with a demonstrably unchanged legacy path. Any proposed public driver ABI change requires a separate owner-reviewed compatibility plan. This section records a compatibility requirement; it does not authorize implementation.


### 28.7 Multi-CRTC and plane implications from the source audit

Further inspection of `drmmode_crtc_get_fb_id()`, `drmmode_glamor_handle_new_screen_pixmap()`, `drmmode_crtc_create_planes()` and the modesetting Present implementation establishes:

- The normal modesetting screen pixmap is backed by the shared `front_bo`. A CRTC normally obtains that framebuffer ID and uses its own source origin (`crtc->x`, `crtc->y`); per-CRTC PRIME scanout can instead select a per-CRTC pixmap/target.
- DRM CRTC color properties are attached to a **CRTC**, not independently to each connector. Two cloned connectors driven by the same CRTC therefore share that CRTC's gamma-LUT/CTM state. A distinct transform for each cloned connector cannot be assumed from CRTC properties alone. Connector-level colorspace signaling is a separate concern and is not equivalent to a full per-connector pixel transform.
- Separate CRTCs can in principle apply independent CRTC color state while scanning different regions of a shared framebuffer. The current Xserver code demonstrates per-CRTC CRTC objects and scanout origins, but does not yet demonstrate a complete independently managed per-output HDR pipeline.
- The modesetting driver discovers and selects a primary plane per CRTC. Cursor planes are identified separately and not selected as the primary scanout plane. Hardware cursor content is uploaded and positioned through cursor-specific DRM calls; it is not transformed by the primary framebuffer's color data path.
- Present's format/modifier check iterates all enabled CRTCs and requires the candidate framebuffer format to be supported by each. A future FP16 or other HDR scanout format must not become the mandatory screen-pixmap format, because doing so could disable existing multi-CRTC flips or prevent a screen from initializing on less capable hardware.
- The current gamma-LUT update uses a CRTC property write, while CTM is set separately; the inspected atomic modeset request does not yet bundle a complete color-pipeline state with framebuffer/mode changes. Any future color transform update must define atomicity with mode, connector, plane and framebuffer state rather than relying on these legacy gamma controls.

Consequences for the no-compositor proposal: per-CRTC transforms are a plausible optimization for extended-desktop outputs with distinct CRTCs, but clone mode, connector-specific requirements, direct scanout, cursors and auxiliary planes remain separate cases. The safe fallback remains the existing SDR path.


## 29. ScreenRec ABI boundary and optional private operation tables

### 29.1 Owner direction

**Owner preference (2026-11-26):** if HDR requires new screen/backend operations, do not add fields to `ScreenRec`. Prefer an additional private attachment through the existing DIX `devPrivates` mechanism, with a versioned operations table. This is the preferred design direction, subject to the constraints below.

### 29.2 The three ScreenRec mechanisms are not interchangeable

Current `include/scrnintstr.h` has:
- `void *devPrivate`: the legacy single opaque DDX-private pointer. Do not repurpose it for HDR or overwrite driver-owned contents.
- `PrivateRec *devPrivates`: the DIX-managed private-key store. This is the appropriate existing attachment point for new server-owned screen-private state.
- `DevPrivateSetRec screenSpecificPrivates[PRIVATE_LAST]`: per-screen private allocation bookkeeping for other object types. Do not add a new `DevPrivateType` merely to host HDR screen operations.

Use the existing `PRIVATE_SCREEN` type and `dixRegisterPrivateKey()`, then access the private with `dixLookupPrivate(&pScreen->devPrivates, key)` / `dixSetPrivate()` according to whether the key reserves inline storage or a pointer slot. The DIX layer owns `devPrivates`; never manipulate its layout or allocation directly.

For a new HDR implementation, the preferred shape is a core-owned screen-private record containing:
- an operations-table ABI version and structure size;
- a capability bitset (advertise only operations fully implemented by this screen/backend);
- a copied, immutable operations table for the registered version;
- an opaque backend context pointer, if required;
- explicit registration/teardown state.

The operations table is optional. No registration means no HDR backend operations for that screen. A missing operation or capability must be handled as unsupported and leave the legacy SDR path operational.

### 29.3 ABI and lifetime constraints

- Do not add or reorder any `ScreenRec` fields, including tail fields, for this feature.
- Do not add a value to `DevPrivateType` / `PRIVATE_LAST`: `PRIVATE_LAST` sizes the in-struct `screenSpecificPrivates[]` array, so changing it changes `ScreenRec` size and offsets of following members.
- Do not alter the position, presence, or conditionality of `reserved_for_nvidia_470_and_390`.
- Do not write to the legacy `devPrivate` field or directly change `devPrivates`.
- Keep the private key and private-record lifetime owned by the Xserver core, not by an unloadable DDX module. A DDX must not leave a key object or callback-table storage in module static memory that can become invalid if the module is unloaded/reloaded.
- Copy the registered table into core-owned private storage; validate version, size, mandatory members, and capability/member consistency before publishing it. Do not retain a pointer to a caller's temporary table.
- Define callback-context ownership and teardown. Unregister/disable callbacks before backend teardown; no callback may run after the backend context or DDX code is unloaded.
- Prefer private/internal headers and exports for the first in-tree modesetting implementation. Do not expose a public third-party driver API until its semantics and compatibility contract are stable.
- Adding an exported helper symbol is additive for old modules that do not reference it, but a new external module that calls it requires a server providing that symbol. Decide ABI minor-version policy before publishing such an API.
- Registration/allocation failure disables the optional HDR capability for that screen; it must not fail server startup or damage normal screen operation.

### 29.4 Existing NVIDIA ABI accommodation is especially sensitive

The current Xserver explicitly supports legacy proprietary NVIDIA DDX layout expectations:

- `include/scrnintstr.h` conditionally contains `reserved_for_nvidia_470_and_390` between `NameWindowPixmap` and `totalPixmapSize` under `CONFIG_LEGACY_NVIDIA_PADDING`.
- `meson_options.txt` exposes `legacy_nvidia_padding` (default false). Its description says it adds padding to `ScreenRec` to match an older ABI.
- `include/xf86Module.h` selects Video Driver ABI 28.1 with that option and 28.0 without it.
- `hw/xfree86/loader/loader.c::LoaderGetABIVersion()` has a specific compatibility path: when the padding option is enabled and the module is identified as proprietary NVIDIA, it reports `ABI_NVIDIA_VERSION` 25.2. The comments explain that 470 knows the older `ScreenRec` layout and modern 570 drivers can select the corresponding old/new code path.
- `dix/pixmap.c` initializes the reserved field to NULL. Preserve this behavior.
- The separate `legacy_nvidia_340x` option controls extra old 340x entry points; do not conflate it with the ScreenRec padding option.

These are deliberate compatibility shims, not spare extension slots. HDR must not change their field offset, initialization, compile-time selection, or loader ABI decision logic. Do not "simplify" or remove them as part of HDR work.

### 29.5 Registration timing and private allocation

The current `dixRegisterPrivateKey()` implementation explicitly supports `PRIVATE_SCREEN` keys after screens already exist: its `allocated_early[PRIVATE_SCREEN]` handler walks both regular and GPU screens and fixes up their private allocations. This makes the existing mechanism suitable for optional late registration without changing `ScreenRec` layout. It can still fail on allocation; handle that as a per-screen feature-unavailable result.

Use a core-owned static key and a core-owned initialization/registration helper. The driver-facing call, if any, should only submit a versioned operations table and backend context to that helper. This avoids module-owned key lifetime and keeps DDX-specific state out of the public `ScreenRec`.

### 29.6 Compatibility and regression test matrix

At minimum, before merging the first implementation:

1. Build with `-Dlegacy_nvidia_padding=false` and `true`; verify the pre-existing Video Driver ABI selection and ScreenRec reserved-field layout remain unchanged in both configurations.
2. With the HDR feature compiled in but not negotiated, run the normal SDR path with modesetting and with the proprietary NVIDIA DDX on real supported driver/GPU combinations. NVIDIA HDR support is not required.
3. Exercise proprietary NVIDIA 470 and a modern dual-ABI driver (e.g. 570) with the padding compatibility configuration where applicable; verify the loader selects the expected ABI path. Do not claim this from a compile-only test.
4. Verify no HDR registration is required for a DDX to initialize, and that absent/unsupported operations are reported as unavailable without changing Composite, Damage, Present, DRI3, RandR, cursor, or scanout behavior.
5. Exercise private-key registration with already-created regular and GPU screens, allocation failure, screen close/re-init across server generations, and backend teardown. Confirm no stale callbacks or private pointers survive.
6. Verify `sizeof(ScreenRec)`, relevant `offsetof(ScreenRec, ...)` values, `PRIVATE_LAST`, and the conditional NVIDIA padding layout are identical to the pre-HDR baseline for each build configuration.
7. Test both no-compositor and compositor paths with the HDR feature absent, unsupported, and negotiated. The unsupported case must be visually and behaviorally equivalent to the legacy SDR path.

This is a design/ABI constraint, not implementation authorization.
