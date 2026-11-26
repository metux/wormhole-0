# X11Libre Xserver HDR Project Context

**Purpose:** durable handoff for a dedicated ChatGPT session working only on HDR / color management in X11Libre. This file is the technical context; refresh live GitHub state before making implementation decisions. It is not an authorization to modify code or to revive historical branches.

**Last verified context refresh:** 2026-11-26 (UTC).  
**Primary implementation repository:** `X11Libre/xserver`, target branch `master`.  
**Related workspace notes:** `X11Libre/mpbt-workspace:mtx/agent-config/.starfleet-ai/dashboard/topics/xlibre/hdr-display-support.md`.  
**Normative authority:** owner decisions and current Xserver source/specifications. This handoff records verified facts and proposals separately.

## Session scope and boundaries

- This context is for HDR/color-management work only.
- Starfleetctl and XAverius implementation are handled in separate ChatGPT sessions. Do not silently switch into either workstream.
- The separate HDR session should begin by rereading this file, the current workspace HDR topic, current Xserver `master`, and relevant Xserver skills.
- No Xserver code changes, local builds, or tests were performed as part of this context capture.
- Do not claim access to local worktrees or resurrect local-only branches unless their actual commits/patches are recovered.
- Do not implement the proposed architecture as if it were already approved. Resolve open design decisions with the owner first.

## Verified implementation state (2026-11-26)

- Xserver `master` HEAD observed during the review: `8f47e0e2dd7a9f2dec43acffb2cf0f80b68e5b8e`. Refresh before use.
- PR [#3216](https://github.com/X11Libre/xserver/pull/3216) merged on 2026-08-24, merge commit `2d72f9e1921a6a0f8d975845c07d7040f6fa8379`.
- The merged change adds CTA-861-G EDID parsing for Colorimetry (extended tag 5) and HDR Static Metadata (extended tag 6), stores parsed data in `xf86Monitor::hdr`, and adds tests in `test/ddc-tests.c`.
- `xf86DoInterpretHDRMetadata()` is declared and implemented. The source search at the checkpoint found only its declaration, definition, and test calls from `test/ddc-tests.c`; there was no production call in the normal DDC/modesetting EDID path. The parser is therefore merged but not integrated into normal server operation.
- No `HDR_OUTPUT_METADATA` property/API implementation was found in the current Xserver source search.
- Remote `wip/hdr-prep` was at `34ad4284c98c8eb3bf049bc762382effb194a14d`, two commits ahead and 587 behind then-current master. It is stale/diverged; do not use as a current base without reconciliation.
- Historical `wip/hdr-full` was recorded as local-only on 2026-07-02. No corresponding remote branch was found. Do not assume it can be resumed unless the actual commit/patch is recovered.
- External PoC [cepelinas9000/xserver:wip/x11hdrV2](https://github.com/cepelinas9000/xserver/tree/wip/x11hdrV2) was at `159a6f484a2a92769787995198f38d80208ec59b` (2026-03-14): one 271-file / +46.5k-line commit against an old base. Its own commit description reports incomplete glamor operation coverage, dark windows and artifacts. Treat as design/source reference only, not a merge candidate.

## Historical PoC: useful material and rejection gates

The old PoC contains relevant design/source material in `glamor/hdrext/`, shader sources, draft wire protocol, modesetting DRM property plumbing, glamor shader refactoring and EDID parsing. It also brings major unresolved issues:

1. It repurposes core protocol opcode 125 for a private visual/depth latch. **Reject this approach.** Use a versioned X extension with explicit capability discovery and request/reply semantics.
2. It increases `MAXFORMATS` from 8 to 10, explicitly noting an ABI break. Do not accept this as-is.
3. It raises the minimum OpenGL core profile to 4.6 without a verified hardware requirement. Do not impose this global compatibility regression.
4. It introduces a mandatory Vulkan-header build dependency. Vulkan must not become a mandatory server build/runtime dependency merely to enable HDR.
5. Its glamor coverage is incomplete and has reported visual artifacts/dark windows. A partial rendering path must not silently misrender SDR/HDR content.
6. RandR output switching/support was noted as missing/incomplete.
7. The historical local-only reconstruction had a missing-braces path in `drmmode_output_get_modes()` that could call `xf86DoInterpretHDRMetadata(mon)` when `mon == NULL`; parser dereferences must be guarded. Treat malformed/short EDID as a first-class negative test.
8. The PoC's EDID sink-capability data must not be copied directly to the outgoing KMS `HDR_OUTPUT_METADATA` blob; sink capability and final composed-frame metadata are different concepts.

## Design references

- [X11Libre org Discussion #251 — Color Management & HDR](https://github.com/orgs/X11Libre/discussions/251)
- [X11Libre/misc issue #250](https://github.com/X11Libre/misc/issues/250)
- [Historical X.Org DeepColor extension RFC](https://groups.google.com/g/xorg-devel/c/JmhOVTpZcs8)
- [Wayland color-management concepts](https://wayland.freedesktop.org/docs/book/Color.html) — conceptual reference only; do not copy protocol/code.
- [Linux DRM/KMS documentation](https://docs.kernel.org/gpu/drm-kms.html) — output colorspace, max_bpc and HDR metadata properties.

Keep XLibre's protocol and implementation independent. The historical DeepColor discussion is useful for drawable metadata and compositor capability discovery, but compatibility concerns around a new core visual class remain relevant.

## Unresolved design decisions — owner review required

Do not silently choose defaults for these:

- **Pixel representation:** first target as packed 10-bpc RGB in the existing 32-bit pixel model, or FP16 RGBA as a required first-class client format? Bit depth alone does not define HDR; colorimetry and transfer function are separate.
- **Metadata scope:** static image description on a window/drawable/pixmap versus dynamic metadata attached to a frame/presentation transaction. Define inheritance, validation, lifetime, update ordering and redraw behavior.
- **Transfer functions:** define PQ and HLG support, reference/peak luminance semantics, and SDR/scRGB conversion. Prior discussions contain competing preferences; no default is approved.
- **Mixed-client composition:** define conversion of SDR, wide-gamut SDR and HDR sources into a common blending space, tone mapping and final output metadata selection.
- **Output policy:** define RandR output colorspace and bit-depth negotiation separately from the KMS `HDR_OUTPUT_METADATA` blob. KMS metadata describes the final composed output, not the monitor's EDID capabilities.
- **Legacy compatibility:** preserve existing 8-bit direct/low-overhead presentation when no color conversion is requested. HDR must not silently force every client through an external compositor or Vulkan layer.
- **Acceleration:** DRM color-pipeline/overlay features may be opportunistic optimizations, never a correctness prerequisite; retain a correct shader/software fallback.

## Proposed architecture baseline (discussion draft, not normative)

Keep source-image description and output signaling separate.

### Source image description
An immutable, versioned description of primaries, white point, transfer function, reference white and luminance range. Associate static content semantics with a drawable/window. If dynamic metadata is needed, attach it to a frame/presentation transaction rather than racing mutable window properties.

### Output description
Per-output sink capabilities and selected output mode/colorspace/bit depth. Read capabilities from EDID; select an output encoding only when both sink and DRM driver support it.

### Composition
Convert sources into a common linear-light working space before blending. Deliberately tone-map mixed SDR/HDR or mismatched peak luminance, then encode the final composed frame for the output. Never blend PQ/HLG-encoded samples directly.

### KMS handoff
Modesetting submits metadata describing the final composed frame through the DRM connector `HDR_OUTPUT_METADATA` blob and configures supported `Colorspace`/`max_bpc` properties. Do not forward the monitor's EDID static metadata blob as outgoing frame metadata.

### Compatibility / format strategy
Preserve the existing 8-bit direct path when no transform is requested. Avoid mandatory Vulkan, forced external compositor, a new core visual class, or a global GL 4.6 minimum.

A possible first format strategy, requiring owner approval, is packed 10:10:10:2 / 30-bit TrueColor-compatible output where supported, with floating-point intermediate buffers internally when available. Client-visible FP16 RGBA can be a later capability, not a prerequisite for first HDR10 output. PQ may be the first practical transfer target because of display support, but HLG semantics must be explicitly defined rather than treated as PQ.

## Suggested staged implementation

0. **Read-only EDID capability integration:** connect the existing parser to the live output path safely and expose capabilities only. No HDR signaling or rendering behavior changes.
1. **Experimental output mode:** behind an explicit runtime option, negotiate DRM Colorspace/max_bpc and `HDR_OUTPUT_METADATA`; provide safe disable/rollback and SDR fallback.
2. **Versioned client color-description API:** define source metadata and mixed SDR/HDR conversion; do not reuse core opcodes.
3. **Complete rendering path:** cover all relevant glamor operations, dynamic metadata where required, advanced formats and optional hardware color-pipeline/plane offload.

Every milestone should test: no-HDR EDID, malformed/short EDID, unsupported KMS properties, hotplug/reconfiguration, SDR-only clients, mixed SDR/HDR, and unchanged legacy 8-bit pass-through. Build/test both with the feature disabled and enabled as applicable.

## Immediate next work for the dedicated HDR session

1. Refresh Xserver `master`, PR #3216, current EDID parsing call sites, RandR/modesetting DRM property handling, and workspace HDR note. Verify all SHAs and call-site claims against current source.
2. Write a compact normative design/RFC for owner review before implementation. It must define the source metadata object, output/RandR/KMS contract, color pipeline, pixel formats, PQ/HLG semantics, fallback/compatibility, lifetime/security, and test plan. Mark each unresolved item explicitly rather than deciding it implicitly.
3. Once the design is agreed, implement only milestone 0 as a small isolated change with malformed-EDID safety and integration tests.
4. Do not revive the stale `wip/hdr-prep` or old PoC as a bulk patch. Create a fresh branch from current master after ownership/worktree checks.
5. Keep all work within HDR/color management. Do not take over Voyager's PR #4170 or other security/backport work in this separate session.

## Relevant workspace note and skills

- HDR tracker: https://github.com/X11Libre/mpbt-workspace/blob/mtx/agent-config/.starfleet-ai/dashboard/topics/xlibre/hdr-display-support.md
- Xserver coding style: `.claude/skills/coding-style-xserver/SKILL.md`
- Xserver test procedure: `.claude/skills/xserver-testing/SKILL.md`
- Wormhole communication protocol: `.claude/skills/wormhole-communication/SKILL.md`

For cross-agent communication, acknowledgements, status updates and reports, follow the project-independent `wormhole-communication` skill and `COLLABORATION.md`.


## 2026-11-26 — server-side pre-compositor transformation design

Owner clarified the intended architecture in the dedicated HDR session:

- The X server performs the color transformation of each window's content.
- When a compositor is active, it must see only the already transformed window buffers.
- The compositor remains responsible for spatial composition (stacking, placement, clipping, alpha, effects and blending); it does not perform per-window source color conversion.
- This is not a request to move full window composition into the X server.

A detailed working draft was added at `X11LIBRE_HDR_COLOR_MANAGEMENT_SPEC.md` (commit `44266969db36fe487cf8108f717101e3560cbca0`). It is explicitly a discussion draft, not normative and not implementation authorization. It records the above architecture, source/output separation, candidate working-space model, buffer lifecycle, Composite/Present/DRI3 concerns, failure/compatibility rules, tests, milestones and unresolved owner decisions.

Important source-review findings against `X11Libre/xserver:master` HEAD `8f47e0e2dd7a9f2dec43acffb2cf0f80b68e5b8e`:

- `CompositeNameWindowPixmap` currently gets the current window pixmap via `GetWindowPixmap()`, registers a resource reference and optionally calls `Screen->NameWindowPixmap()`. The existing DDX hook has not been shown to replace the pixmap resource returned to the Composite client.
- Present can change the pixmap associated with a window through `present_set_tree_pixmap()`; successful flips update window/root pixmap associations and report Damage. Therefore, a one-time transform during the first `CompositeNameWindowPixmap` request cannot keep compositor content current.
- Composite redirection itself uses offscreen window pixmaps and `SetWindowPixmap`; this internal drawing/storage path must remain distinct from the transformed compositor-facing buffer.
- The design must not globally change `GetWindowPixmap` semantics or redirect ordinary X server drawing into the transformed buffer.
- A stable transformed pixmap object with updated contents is one candidate, but its resource/refcount, resize, Present replacement, Damage ordering and compositor-held-XID semantics need further investigation before choosing it.

Next HDR design work:
1. Reconstruct the exact CompositeNameWindowPixmap resource and pixmap lifetime semantics, including what happens when the window pixmap changes after the compositor has named it.
2. Trace Composite redirection, Damage registration/callback ordering and Present copy/flip paths end-to-end.
3. Compare a stable transformed pixmap object against a new versioned Composite/color-management contract; do not assume the DDX NameWindowPixmap hook is sufficient.
4. Refine the spec with evidence and mark unresolved decisions for owner review.
5. Do not modify Xserver implementation until the design gates in the spec and owner authorization are satisfied.


Further lifecycle finding from the 2026-11-26 source review:

- `DamageRegister()` registers a window Damage object both in the Window's private Damage list and in the current backing Pixmap's list.
- The `damageSetWindowPixmap()` wrapper migrates all window-registered Damage objects off the old Pixmap list, invokes the underlying `SetWindowPixmap()`, and attaches them to the new Pixmap list. This makes a server-owned Damage object registered on the Window a viable candidate for following Present/Composite pixmap swaps.
- This does **not** retarget a compositor's existing CompositeNameWindowPixmap resource: `SingleCompositeNameWindowPixmap()` stores a reference to the exact PixmapPtr current at request time. Later window pixmap swaps do not, in that code path, update that resource.
- Therefore, the likely prototype shape is a window-attached Damage observer plus a separate stable transformed Pixmap object for the compositor, but this remains a hypothesis until Composite protocol semantics, resize handling and actual compositor re-name behavior are verified.

The detailed analysis is recorded in section 16.1 of `X11LIBRE_HDR_COLOR_MANAGEMENT_SPEC.md` (latest follow-up commit `2cd6b5781e61594cea9a99750eea87341e16fc3f`). No Xserver source was changed.


Composite protocol lifetime and format implications were then added to section 16.2 of `X11LIBRE_HDR_COLOR_MANAGEMENT_SPEC.md` (latest spec commit `2fb2742dada488450adf79d78a31a4f1d42a9c94`):

- CompositeNameWindowPixmap names a reference to the current off-screen storage. A new pixmap is allocated when a window is mapped or resized; the old named pixmap remains allocated until freed. Compositors must reacquire the current pixmap after those lifecycle changes.
- A transformed buffer therefore needs a generation/lifetime corresponding to redirected-window storage. Old transformed generations must remain valid while held by compositor references.
- A linear/high-precision working buffer (e.g. FP16 RGBA) may not be representable as an ordinary legacy X pixmap with the window's original depth/bpp. Returning a different PixmapPtr through the legacy request is not sufficient unless format and color-space semantics are preserved.
- The preferred investigation direction is a versioned color-managed Composite handoff that exposes the transformed buffer plus format/modifier, dimensions/planes, working-space description, synchronization and generation/lifetime semantics. Transport (X pixmap, DRI3 FD(s), or another handle) remains open.
- Legacy Composite semantics should remain unchanged unless the compositor explicitly negotiates the color-managed handoff.

The external Composite API documentation and protocol text were consulted; this is design evidence, not yet a full audit of all compositor implementations. No implementation changes were made.


## 2026-11-26 — Visual, buffer-format and Color Description model

The design draft was extended in sections 18–23 of `X11LIBRE_HDR_COLOR_MANAGEMENT_SPEC.md` (commit `7157c7360b4200db468304bea0fb07853c5f7ab5`).

Current proposal, still awaiting explicit owner approval:

- Do **not** add a private seventh core Visual class for HDR. Keep standard core Visual classes (normally TrueColor) and expose versioned extended-Visual capability/profile attributes through the new color-management extension.
- A Visual selects a compatible window/drawable configuration; it does not define the buffer's storage format or its colorimetry.
- Packed 10:10:10:2 may use a conventional depth-30 / 32-bpp path where supported. Client-visible RGBA FP16 is a 64-bpp format and should initially use an extension-managed buffer/surface path, not pretend to be an ordinary core pixmap depth.
- Buffer format descriptors define channel order, component type/width, planes, strides/modifiers, alpha representation and synchronization. They do not define primaries or transfer functions.
- Colorimetry is represented by an immutable, versioned Color Description resource (primaries, white point, transfer function, reference white and luminance semantics), initially bound to a Window. Dynamic metadata belongs atomically to the corresponding buffer/Present frame.
- For a negotiated color-managed Composite path, the X server transforms every window—including legacy SDR windows under an explicitly chosen legacy profile—into one common linear-light working space before the compositor sees it. A legacy compositor keeps the old Composite behavior.
- The extension surface is conceptual only; no wire ABI, opcode, extension name, exact working space, transfer-function policy or legacy SDR profile has been approved.
- No Xserver implementation changes were made. Sections 18–23 are proposals and do not authorize implementation.

Owner decisions are enumerated in section 23 of the specification.


## 2026-11-26 — Common linear HDR working-space proposal

HDR specification section 24 was added in commit `0b5d37730544a05752e696d3616b0f7993f4b6e3`. This is a technical recommendation awaiting explicit owner approval, not a frozen normative ABI.

Recommended compositor-facing space:
- BT.2020 / Rec.2020 primaries (R .708/.292, G .170/.797, B .131/.046), D65 white (.3127/.3290).
- Linear-light RGB, absolute luminance scale with neutral RGB 1.0 = 203 cd/m².
- Output-independent; do not normalize to current monitor peak.
- RGBA IEEE-754 binary16 (FP16) transformed handoff, subject to negotiated support.
- Values above 1.0 and negative RGB intermediates are valid; no source-stage clamp to [0,1] and no gamut mapping until output conversion.
- Transformed handoff uses linear premultiplied alpha; alpha mode on source buffers is separate metadata from pixel format.
- PQ must be decoded to absolute luminance. HLG cannot be advertised as fully supported until system gamma / nominal peak assumptions are specified.
- Legacy SDR reference white is now owner-decided (203 cd/m² default, configurable); chromatic adaptation is owner-decided in section 26. Remaining open items include non-finite FP input policy and HLG luminance semantics.

Rationale: BT.2020/D65 aligns with BT.2100 HDR signaling; a fixed 203 cd/m² scale keeps per-window transforms independent of monitor/output. ITU-R BT.2408 notes reference-white practice can vary with display peak; Xlibre's proposal intentionally uses 203 as an internal canonical scale, not a universal output-white setting.

No Xserver source was changed. No implementation authorization follows from this proposal.


## 2026-11-26 — Owner decisions: working space, FP16 handoff and legacy SDR

The owner approved the following design direction in the dedicated HDR session; the detailed proposal is in sections 24–25 of `X11LIBRE_HDR_COLOR_MANAGEMENT_SPEC.md`:

- Common compositor working space: linear BT.2020 / D65, output-independent, neutral RGB 1.0 = 203 cd/m².
- Negotiated compositor handoff: RGBA IEEE-754 binary16 (FP16), linear premultiplied alpha. This remains capability-negotiated; the fallback and complete wire contract must still be designed.
- Unmanaged legacy windows: implicit sRGB primaries, D65 and IEC 61966-2-1 transfer function.
- Legacy SDR reference white: session/server-wide configurable setting, default 203 cd/m² to anchor SDR graphics white to HDR reference white in the mixed HDR composition path. The setting is not per-output; explicit Color Descriptions override it.
- Lower values such as 80 cd/m² (conventional sRGB reference viewing) or 100 cd/m² remain user/session-configurable alternatives. At the 203 cd/m² default, legacy sRGB white maps to working RGB 1.0; at 80 cd/m² it maps to approximately 0.3941.

Configuration interface and allowed range remain open. If runtime changes are supported, every affected legacy window must be fully retransformed into a new synchronized buffer generation before publication. No Xserver implementation was authorized or changed.


### 2026-11-26 correction — legacy SDR default in HDR composition

A standards cross-check refined the default recorded above: the HDR color-managed compositor path should default legacy SDR white to **203 cd/m²**, not 80 cd/m². The W3C CSS Color HDR draft states that SDR content should be anchored to 203 cd/m² HDR reference white when composited with HDR. The conventional 80 cd/m² sRGB reference-viewing value remains a valid configurable lower setting, as does 100 cd/m².

Thus the session-wide `LegacySDRReferenceWhite` default is 203 cd/m²; it is configurable (e.g. 80 or 100 cd/m²), never per-output. The implicit profile remains sRGB/D65. Explicit Color Descriptions override the legacy default. See spec section 25.5 for the distinction between SDR reference-display viewing and HDR composition.


## 2026-11-26 — Chromatic-adaptation decisions and remaining work

**Owner-approved for v1 (2026-11-26):**
- ICC linear Bradford is the single chromatic-adaptation method for relative-colorimetric RGB sources whose white point differs from D65.
- Full adaptation only; no partial-adaptation control and no per-window/per-output CAT selector.
- The source-description model must explicitly distinguish relative-colorimetric RGB from absolute-colorimetric XYZ / already-adapted PCS. Do not infer this from transfer function, primaries or buffer format; this prevents double adaptation.

The approved pipeline is: alpha interpretation → transfer decode → source RGB-to-XYZ → full Bradford to D65 if needed → XYZ-to-linear-BT.2020 → resolve absolute luminance semantics and divide by 203 cd/m² → linear premultiply → RGBA FP16. Negative intermediate components remain valid and must not be clamped. Reference-white scaling is independent of CAT.

**Remaining design questions:**
- ICC profile ingestion/CMM: owner-approved decision is parametric Color Descriptions only in v1; arbitrary ICC profiles and CMM integration are deferred to a separately approved future version.
- Define validation tolerances and canonical white-point precision for identity detection and matrix caching.
- HLG remains gated pending system-gamma / nominal-peak semantics; non-finite FP16 input/output policy remains open.

Evidence: ICC.1:2022 clauses 6.2 and Annex E recommend linear Bradford where there is no reason to choose another method, citing broad implementation and excellent visual performance. W3C CSS Color 4 (2026-11-12 CRD) uses linear Bradford for D65/D50 conversions and specifies the transform pipeline from linear RGB through XYZ with adaptation where needed. Sources: ICC.1:2022, https://www.color.org/specifications/ICC.1-2022-05.pdf; W3C CSS Color 4, https://www.w3.org/TR/2026/CRD-css-color-4-20261112/.

Durable owner decision: `decisions/2026-11-26--hdr-linear-bradford-and-source-semantics.md`. These are design decisions only; they do not freeze the extension wire ABI or authorize Xserver implementation.

Clarification to the section 26 pipeline: relative SDR (e.g. sRGB) linear XYZ is scaled by its declared reference-white cd/m²; PQ decoding already yields absolute luminance and must not be multiplied by that reference-white value; absolute-colorimetric XYZ/PCS keeps its supplied absolute scale. All are then expressed on the common 203 cd/m² working scale. See latest spec commit `3444503d`.


## 2026-11-26 — Owner decision: parametric Color Descriptions only in v1

The owner approved deferring arbitrary embedded ICC profiles and general-purpose CMM integration beyond v1. The first Color Description model will use explicit parameters: RGB primaries, white point, transfer function, luminance/reference-white semantics and relative-versus-absolute colorimetric interpretation. See specification section 26.6. This is a scope decision, not a wire-ABI freeze or implementation authorization.

## 2026-11-26 — Direct output path without a compositor (proposal)

The owner raised the possibility of converting directly into each monitor's output color space when no compositor is active. The design proposal is documented in specification section 27:

- Convert the final desktop image late and independently per output, not once globally, because outputs may have different color spaces and a window may span them.
- Prefer per-CRTC DRM/KMS color-pipeline blocks (degamma LUT, CTM, gamma LUT) where sufficient; otherwise use output-specific GPU/shader scanout buffers; preserve legacy SDR fallback when no correct transform is available.
- Treat this as a separate path from per-window conversion into the common linear BT.2020 compositor handoff.
- A final-output transform cannot recover per-window color semantics already lost during rasterization. Rich per-window HDR descriptions may require internal redirection/composition or a future rendering contract.
- Audit Present/direct scanout, overlays, cursors and video planes separately; primary-plane color management alone is insufficient.

This is a proposal for investigation, not yet an owner-approved architecture or implementation authorization.

## 2026-11-26 — modesetting output-pipeline audit and NVIDIA compatibility requirement

Xserver `master` was refreshed at `bb06f6ddca766c41c81494ae228c11e53df59590`. The current modesetting DDX source was inspected in `hw/xfree86/drivers/video/modesetting/drmmode_display.c`, `drmmode_display.h` and `present.c`.

Verified observations:
- The modesetting CRTC property model currently includes ACTIVE, MODE_ID, GAMMA_LUT, GAMMA_LUT_SIZE and CTM; no DEGAMMA_LUT support was found in that model.
- GAMMA_LUT is enabled only when the CRTC reports a nonzero GAMMA_LUT_SIZE. The current default enables it for sizes up to 4096, with the existing UseGammaLUT option as an override. Otherwise the legacy drmModeCrtcSetGamma path is used.
- The LUT is 1D RGB; CTM is a separate 3x3 transform. This does not constitute a complete HDR pipeline.
- Present flip eligibility checks format/modifier support across every active CRTC. Cursor programming is separate; sprite/overlay visibility can affect flip eligibility.
- The modesetting DDX uses shared screen-pixmap and per-CRTC scanout state; more investigation is required before claiming support for arbitrary per-output shader targets, clone topologies or all PRIME paths.

**Owner requirement:** future HDR work must not break the proprietary NVIDIA Xorg DDX; NVIDIA is not required to support HDR. Keep HDR opt-in and runtime capability-gated. A DDX without support must preserve legacy SDR/X11 behavior. Avoid mandatory new driver callbacks, public driver-ABI changes, generic-core DRM/GBM/Vulkan dependencies, or a raised global OpenGL baseline. Preserve legacy Composite, Damage, Present, DRI3, RandR gamma, pixmap and scanout behavior when the new contract is not negotiated.

The proprietary NVIDIA DDX is external to this repository and was not inspected or tested. Its compatibility is a mandatory future validation target, not a verified fact. Detailed findings and test gates: spec section 28. Durable owner requirement: `decisions/2026-11-26--hdr-driver-compatibility-and-nvidia.md`.

No Xserver implementation changes were made. No implementation authorization follows from this audit.


Further source audit detail (same Xserver master HEAD `bb06f6ddca766c41c81494ae228c11e53df59590`): the normal modesetting screen pixmap is backed by a shared `front_bo`; CRTC scanout normally uses its own source origin, while PRIME paths may select per-CRTC scanout pixmaps. KMS gamma-LUT/CTM properties are CRTC-scoped, so cloned connectors on one CRTC share that state; per-connector distinct pixel transforms cannot be assumed. Cursor planes and cursor DRM operations are separate from primary-plane pixels. Present format/modifier checks require support across all enabled CRTCs, so a new FP16 screen-pixmap requirement could regress existing flips or initialization. See spec section 28.7.


## 2026-11-26 — ScreenRec ABI / private operation-table audit

Xserver master was refreshed at `3132e4b7997b525d046e727c1db6656d14b5bc20`. Owner prefers that any new HDR screen/backend operations be attached through additional DIX private state rather than by extending ScreenRec.

Verified distinction: ScreenRec's legacy `void *devPrivate` is a single opaque DDX pointer and must not be repurposed; `PrivateRec *devPrivates` is the DIX-managed private-key store and is the appropriate attachment. Do not change `DevPrivateType` / `PRIVATE_LAST`: `screenSpecificPrivates[PRIVATE_LAST]` is embedded in ScreenRec, so its size affects ScreenRec layout. Use the existing PRIVATE_SCREEN key type with a core-owned key and private record. `dixRegisterPrivateKey()` supports late registration and fixes up both regular and GPU screen private allocations. Core must own key/record lifetime; copy and validate an optional versioned operations table and clear callbacks before backend teardown. No registration means no HDR operations and legacy SDR continues.

The NVIDIA compatibility shim is unusually explicit: `CONFIG_LEGACY_NVIDIA_PADDING` conditionally inserts `reserved_for_nvidia_470_and_390` between `NameWindowPixmap` and `totalPixmapSize`; Meson option `legacy_nvidia_padding` controls it; `xf86Module.h` selects ABI 28.1 vs 28.0; `LoaderGetABIVersion()` reports special NVIDIA ABI 25.2 when padding is enabled and the module is identified as proprietary NVIDIA. `dix/pixmap.c` initializes the reserved field to NULL. The separate `legacy_nvidia_340x` option concerns old entry points. HDR must not change any of these layouts, options, or loader decisions.

Detailed owner direction, lifetime constraints and test matrix: spec section 29. Durable decision: `decisions/2026-11-26--hdr-screen-private-ops-and-nvidia-abi.md`. No Xserver source was modified and no implementation is authorized.
