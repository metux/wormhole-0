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
- alpha representation (straight or premultiplied);
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
- alpha semantics only where not already fixed by the buffer format.

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

## 21. Color transform and compositor handoff (proposal; OPEN)

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

**OPEN — compatibility policy:** define a deterministic legacy source interpretation for windows without explicit color metadata (for example, a documented sRGB assumption or a server-configured legacy profile). When the compositor negotiates the color-managed handoff, the X server must transform legacy windows according to that policy as well. If the policy cannot be applied safely, the server must keep the legacy Composite path rather than silently mixing incompatible representations.

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

All proposals in sections 18–23 remain draft until the owner explicitly approves them. No Xserver implementation is authorized by this document.
