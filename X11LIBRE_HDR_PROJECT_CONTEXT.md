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


## 2026-11-26 — Internal Xserver-Core / DDX / Glamor boundary

Xserver `master` was refreshed at `3132e4b7997b525d046e727c1db6656d14b5bc20`; workspace `X11Libre/mpbt-workspace:mtx/agent-config` was refreshed at `aba0f93896274daa394cc7e3e343df55d6486ef7`. The current Glamor source was inspected: `glamor_screen_private` is already attached through a Glamor-owned `PRIVATE_SCREEN` key; normal pixmap storage is represented by `glamor_pixmap_private` and FBOs whose formats derive from X pixmap depth. The DRI3 export path is `PixmapPtr`-based and is not itself a color-managed buffer contract. Glamor currently accepts desktop GL 2.1 / GLES 2.0 baselines; FP16 renderability and shader precision are not guaranteed by those baselines.

**Design proposal recorded in specification section 30** (spec commit `e845da71c89f60ddf10526bb3065594ce6753427`):
- Three ownership layers: DIX/core policy and generations; optional per-screen DDX provider; Glamor GPU transform engine. A separately negotiated Composite extension exposes transformed buffers.
- Use a Glamor-private opaque FP16 color-buffer object, not a fake 64-bpp X pixmap or changed core pixmap depth.
- Core owns a copied/versioned optional provider ops table in its existing `PRIVATE_SCREEN` record. The DDX registers after Glamor/context initialization; no registration means legacy SDR.
- First in-tree provider may be modesetting only. It delegates GPU transforms to Glamor and coordinates external buffer export/sync. Proprietary NVIDIA DDX need not register or support HDR.
- No direct Glamor dependency in generic DIX; no GL/EGL/GBM/DRM types in the core provider interface; no mandatory Vulkan/DRI3 dependency and no higher GL baseline.
- Probe actual FP16 FBO renderability, shader precision, source import, compositor export, dimensions and end-to-end synchronization at runtime. Capability is per screen and end-to-end.
- Core owns source/transformed generations; publish immutable synchronized generations, retain old ones until compositor release, and do not update a buffer still in use.
- Legacy Composite semantics remain unchanged absent explicit versioned negotiation. Failure must not leak an untransformed buffer under the working-space contract.
- Exact provider API, transport, Composite wire contract, full source-path coverage, dimension/tiling policy and failure transition remain open.

No Xserver source code was changed. This is a design proposal, not implementation authorization.


### Follow-up: modesetting / Glamor dynamic module lifecycle

A further read of `hw/xfree86/drivers/video/modesetting/driver.c` at Xserver HEAD `3132e4b7997b525d046e727c1db6656d14b5bc20` refined section 30:
- modesetting loads `GLAMOR_EGL_MODULE_NAME` through `xf86LoadSubModule()` and resolves Glamor functions using `LoaderSymbolFromModule()`; function pointers live in the per-screen modesetting record.
- `try_enable_glamor()` only enables Glamor after `egl_init2` succeeds. Existing `glamor_gbm` only proves the current texture-GBM-BO capability, not FP16 renderability/export or compositor synchronization.
- The provider should be owned by modesetting and delegate to new optional Glamor helper symbols resolved separately. Missing symbols or failed feature probes mean HDR unavailable, not DDX initialization failure.
- modesetting `CloseScreen()` frees BOs and may destroy GBM before chaining to the wrapped screen close path. The provider must quiesce and unregister core callbacks before those resources are destroyed; `FreeScreen()` later frees the modesetting record. The loaded Glamor module must remain pinned while callbacks can reach its function pointers.

Specification section 30.9 records these concrete lifecycle constraints (follow-up spec commit `02770890c38c71304c04330e7cf08f79478af6c6`). No source code was changed.


### Owner clarification — paired modesetting / Glamor deployment

The owner confirms that the modesetting DDX and Glamor module are always built and deployed together from the same codebase. This removes the need to treat their internal HDR helper interface as an independently versioned optional module ABI.

Updated design in HDR specification section 30 (commit `49303af27f6e059caebfce1c36da15769a1f963b`):
- modesetting may resolve required internal HDR helper symbols from its paired Glamor module through the existing module loader; missing symbols indicate a broken/mismatched installation, not an ordinary runtime HDR-unavailable condition;
- no optional-symbol compatibility shim is needed between the paired components;
- HDR itself remains runtime-optional: failed FP16/shader/export/synchronization/hardware probes simply leave HDR unregistered for that screen;
- core remains independent of Glamor, and NVIDIA remains on the legacy SDR path unless it explicitly implements a provider;
- callback/resource teardown ordering remains mandatory.


### 2026-11-26 — Color-buffer ownership / DMA-BUF / synchronization design

Refreshed Xserver master: `3132e4b7997b525d046e727c1db6656d14b5bc20`.

Source facts:
- `glamor` is a static library linked into the separately loaded `libglamoregl.so`; `modesetting_drv.so` loads Glamor through the Xorg module loader and resolves symbols dynamically. The modules are built/deployed together, so new internal HDR helpers are required same-build symbols, not optional compatibility variants.
- Glamor's EGL private state owns the EGL display/context, DRM fd, and GBM device. modesetting obtains that same GBM device through `glamor_egl_get_gbm_device()` and uses it for modesetting BOs when Glamor is enabled.
- Glamor already exports DMA-BUFs from pixmaps and has `glamor_egl_flush_fence_fd()` for exporting an EGL native fence after `glFlush()`.
- Existing DRI3 syncobj support is a distinct mechanism and its capability probe does not prove a compositor-facing acquire/release protocol.

Design proposal recorded in HDR spec section 30.10 (commit `5cb70e943dd77d791e96162d85dc8b1dd1b0a1be`):
- DIX/core owns logical window/buffer generations and references only.
- modesetting owns the optional per-screen provider and orchestration, not a second GBM allocator.
- Glamor owns opaque physical FP16 color-buffer objects (GBM BO/EGLImage/GL texture/FBO), DMA-BUF export, and EGL synchronization.
- The Composite extension owns FD handoff and generation-correlated compositor release.
- Export descriptor includes FourCC, modifier, dimensions and per-plane FD/stride/offset; exact FP16 DRM format remains open.
- Do not require KMS scanout usage for compositor input buffers unless a separate direct-scanout optimization requires it.
- Initial sync proposal is explicit native-fence FD in both directions. Existing fence export helper is only a building block; import/wait and consumer release must be validated end-to-end.
- Section 30.3's illustrative provider table needs separate compositor-release/reuse and final physical destruction semantics before API freeze.

No Xserver source code was changed. No implementation authorization was granted.

FP16 DMA-BUF format research follow-up: `DRM_FORMAT_ABGR16161616F` is a concrete candidate for the RGBA FP16 handoff. Its DRM definition describes little-endian A:B:G:R 16-bit floating components, corresponding to R/G/B/A component order in little-endian memory. It is not yet a fixed requirement; runtime GL renderability, GBM allocation/modifier support, export and compositor import/channel/alpha interpretation must all be tested. Big-endian handling and alternative formats remain open. Specification follow-up commit: `1e7ce056f3277694bd1c2841c048467641e893e9`.


### 2026-11-26 — Buffer-generation state machine and release-fence lifecycle

Detailed lifecycle review used Xserver master HEAD `f6df545643f02d04d06ac08f7c81f4da05f7c4cc`; final cross-check refreshed current master to `ee4791f92516141a45b9014aaab7d0eff6afdd3a`. The relevant Glamor native-fence helper, EGL private state, and modesetting dynamic Glamor-loading/GBM integration are unchanged. A further current-master refresh to `31f845bec4425e1b2048c7763497d752dfe05352` (commit `modesetting: record the probed DRM node before claiming an fb slot`) again confirmed the Glamor fence helper and modesetting integration unchanged. A further current-master refresh to `514357ad2bdb06689f615f87936d60d13cb9a653` (commit `xfree86: pass the active ServerLayout options to configServerFlags()`) again confirmed the Glamor fence helper and modesetting integration unchanged. Rechecked the current Glamor EGL native-fence export helper: `glamor_egl_flush_fence_fd()` creates a native-fence sync, calls `glFlush()`, duplicates the fence FD, destroys the EGL sync and returns the FD. This proves a producer-fence export building block only. The existing `glamor_egl_syncobj_supported()` remains a DRI3-specific timeline-syncobj probe, not proof of compositor release support.

Specification section 30.11 committed as `cb454408f839cd91db312d97ebcf2214e74f98c5`.

Key proposed rules:
- Separate logical content-generation lifecycle from physical Glamor allocation lifecycle.
- A published generation is immutable and leased to the compositor until one matching release is received.
- Key each lease by client epoch, screen, window and monotonic generation; reject stale, duplicate and mismatched releases.
- Producer fence covers all transform writes; compositor waits before sampling.
- Consumer release fence covers all compositor reads; Glamor imports it and queues a GPU-side wait in the same serialized context/queue before reuse.
- CPU blocking waits are not the normal fallback. If both native-fence directions and server-side wait cannot be proven, do not advertise the HDR handoff.
- No buffer reuse on disconnect, timeout, missing release, or fence-import failure. Quarantine/orphan instead; timeout is diagnostic, never proof of GPU completion.
- Bound the pool, coalesce Damage/Present updates, and never block the Xserver event loop waiting for a buffer.
- v1 proposal uses full-window transforms into fresh destinations; partial updates/copy-forward are deferred until immutable-generation handling is proven.
- Provider API needs separate release-fence ordering, reusable-pool return, terminal destruction, and provider quiesce operations. The old illustrative single `release_buffer()` callback is not sufficient.

No Xserver source code was changed. The new state machine is a proposal pending owner review; implementation remains unauthorized.


## 2026-11-26 — Provider API and compositor FD handoff proposal

Xserver master was refreshed to d96a26a11ca6ddc53050f4a96dd0becee9858077. Source checks of Xext/composite/compext.c, Xext/dri3/dri3_request.c and os/io.c confirmed:
- CompositeNameWindowPixmap registers a reference to the exact current PixmapPtr; NameWindowPixmap does not replace the pixmap resource returned to the compositor.
- DRI3 rejects non-local clients and uses SetReqFds()/ReadFdFromClient() for incoming FDs and WriteFdToClient() for reply FDs. The helpers rely on XTRANS_SEND_FDS and otherwise fail.
- DRI3 imports buffers as ordinary X pixmaps and is not an appropriate color-semantics or FP16 buffer API. The new path must not require DRI3/libdrm in generic DIX.

Spec section 30.12 now proposes a separate versioned color-management extension and a two-phase handoff: BufferReady event without FDs, AcquireBuffer request/reply with DMA-BUF plane FDs plus producer fence, and ReleaseBuffer request with a generation-correlated release fence (or explicit NO_USE when no GPU read was submitted). The protocol is a proposal; exact name, wire layout, compositor authority, Xinerama behavior, XCB FD reception, error handling and security policy remain open.

The internal provider proposal now separates begin/collect transform, descriptor/fence export, consumer release-fence import/queue-wait, pool return, terminal destruction and quiesce. Core retains source snapshots and logical generations; modesetting orchestrates; Glamor owns physical buffers and GPU synchronization. No ScreenRec layout changes or mandatory NVIDIA callbacks.

No Xserver source code was changed. No implementation authorization was granted.

## 2026-11-26 — XCB FD transport verification

X11Libre/mirror.fdo.libxcb master was inspected at `aa7f3256060451ad65a38feeccf792f44181a4e0`. `xcb_out.c` implements request FD passing with `xcb_send_request_with_fds()`; `xcb_in.c` associates reply FDs only for pending requests marked `XCB_REQUEST_REPLY_FDS`, reads the FD count from reply `pad0`, and appends descriptors after the reply payload. `xcb_get_reply_fds()` locates the array. `c_client.py` generates the request flag and reply accessor automatically when xcb-proto declares `<fd>` fields. Received FDs are caller-owned and must be closed before freeing the reply; outgoing FDs are transferred to XCB ownership.

The current implementation sets `XCB_MAX_PASS_FD` to 16, but this is an implementation limit, not a wire ABI. FD passing is conditional on `HAVE_SENDMSG`. Proposed v1 cap: up to four DMA-BUF object FDs plus one producer fence in AcquireBuffer; one release-fence FD or zero for NO_USE. The extension must ship protocol XML and generated bindings; raw requests without the reply-FD flag are unsafe. Client-side FD support is a build prerequisite that the server cannot infer from its own QueryCapabilities response.

Spec section 30.13 records these details. The concrete XML schema, runtime transport matrix and compositor interoperability remain unverified. No Xserver code changes.

## 2026-11-26 — Composite redirect ownership audit

Xserver master `9f1d28b59b6af78033b7f0d0c170935308de046f`: `CompWindowRec` stores per-window `CompClientWindowRec` registrations. `compRedirectWindow()` permits only one `CompositeRedirectManual` client per window (`BadAccess` on a second), while automatic registrations may coexist. `compRedirectSubwindows()` registers existing children individually and future children through its parent resource; Composite resource cleanup removes the per-client registrations.

Therefore Composite does not provide a global per-screen compositor owner. Spec section 30.14 proposes an exclusive client-owned HDR compositor claim per screen plus a check that the same client owns manual Composite redirection for each subscribed window. Automatic redirection is excluded from v1; `_NET_WM_CM_Sn` remains a discovery convention, not server authorization. PanoramiX/Xinerama scope and helper lifecycle remain open.

No Xserver code changes.

## 2026-11-26 — PanoramiX/Xinerama Composite path

Xserver master `9f1d28b59b6af78033b7f0d0c170935308de046f`: when `compositeUseXinerama` is active, Composite redirect/unredirect requests fan out to each underlying screen. `CompositeNameWindowPixmap` creates a `PanoramiXRes` with one physical pixmap reference per screen. This is distinct from multiple RandR outputs on one ScreenRec, which can still share one output-independent transformed window buffer.

Spec section 30.15 proposes v1 support for ordinary ScreenRec and RandR multi-output, but explicit unsupported capability when the Composite PanoramiX/Xinerama multi-screen path is active. A future protocol could define one logical generation as a set of per-screen buffer descriptors and leases; not designed yet. Existing Composite remains unchanged.

## 2026-11-26 — Provisional v1 protocol matrix

Spec section 30.16 adds a field-level request/reply/event matrix for QueryVersion, QueryCapabilities, AcquireCompositor, ReleaseCompositor, SubscribeWindow, UnsubscribeWindow, AcquireBuffer and ReleaseBuffer. It distinguishes ordinary frame races (`NOT_READY`, `STALE`, `UNSUPPORTED`) from X protocol errors, defines BufferReady/BufferInvalidated/StreamError events and states lease/generation invariants.

The matrix is not a wire ABI. Exact field widths, opcodes, xcb-proto XML, extension error mapping and generated-binding tests remain open. No Xserver implementation.

### 2026-11-26 — XCB protocol-schema mapping

The provisional Composite color-managed handoff was mapped against xcb-proto master commit `c70aba5cc6ae87757111dac737fc814b3605f0d4` and DRI3 XML. xcb-proto supports single FD fields and counted FD lists in requests/replies; reply FD count belongs in the first reply data byte (`nfd`). libxcb generated reply-FD handling and `xcb_send_request_with_fds()` were previously verified at `aa7f3256060451ad65a38feeccf792f44181a4e0`.

The new section 30.17 proposes a client-owned per-screen CMCONTEXT resource/epoch, manual-Composite-redirect authorization, AcquireBuffer replies with object FDs followed by producer fence, and ReleaseBuffer counted FD lists (one release fence or zero for NO_USE). Full correlation events need XGE rather than a fixed 32-byte event. FD-less transports must report unsupported. Exact XML, layout/alignment, XGE subscription, Xtrans behavior and generated-binding round-trip tests remain open. No XML or Xserver code changed; implementation remains unauthorized.

### 2026-11-26 — Xtrans FD transport audit

Current Xserver master `d439c01fef4c8dcb3a0d05471e3edb0eb00f6f89` confirms Xtrans FD passing is implemented for `local`/`unix` domain sockets, while TCP/INET/INET6 callbacks explicitly fail. Capability must be per connection, not inferred from build flags or loopback/local classification. `SetReqFds()` only logs mismatches and overwrites `req_fds`; a new handler must strictly compare actual ancillary count to declared `fd_count` before consuming. Critical blocker: `appendFd()` in `os/Xtranssock.c` returns void; `_XSERVTransSocketSendFd()` unconditionally returns success even if queue allocation failed and the FD was closed. `WriteFdToClient()` is therefore not a reliable success signal until Xtrans propagates queue failures and has tests. XGE events are supported by current xcb-proto conventions. No code/tests changed; no implementation authorization.

### 2026-11-26 — Correction: request-scoped ancillary FD accounting

A closer trace corrects one point in spec section 30.18: `ClientRec.req_fds` is the remaining expected-to-consume counter set by `SetReqFds()`, not the actual number of ancillary FDs received for the current request. Xtrans queues received FDs connection-wide without preserving their request-byte boundary; `ReadFdFromClient()` pops from that queue, and cleanup drains only declared-but-unconsumed FDs. A handler cannot compare `client->req_fds` to actual received count. Reliable HDR handoff therefore requires Xtrans request-boundary FD accounting, exact count exposure/rejection of extras, receive queue allocation failure propagation, and send queue allocation/ownership error propagation. No source/tests changed.

### 2026-11-26 — FD marker offsets / output ordering

HDR spec section 30.20 records a new source-backed framing design. Linux `unix(7)` documents SCM_RIGHTS as a SOCK_STREAM barrier: the `recvmsg()` containing ancillary data returns through the byte carrying the FDs. For Linux AF_UNIX, Xtrans can record the marker offset as `read_offset + returned_bytes - 1` if clients attach the FD group to the first byte of the X request. The input parser must correlate that offset with absolute request boundaries; current `ReadRequestFromClient()` reads up to remaining input-buffer capacity and may read ahead across pipelined requests, while Xtrans keeps only a connection FIFO.

New findings: receive path ignores `MSG_CTRUNC` and must reject truncated/malformed cmsgs and fail closed on allocation/association errors. Server output currently attaches queued FDs to the first byte of the next transport write, although `FlushClient()` may start with earlier buffered replies/events. libxcb master `aa7f3256060451ad65a38feeccf792f44181a4e0` has a similar source-level hazard: `send_fds()` does not flush `out.queue`; `send_request()` may buffer the FD-bearing request behind earlier bytes; `_xcb_conn_wait()` attaches FDs to the first byte of the combined vector. Both sides need explicit byte-offset/reply-segment ordering and regression reproducers. Linux AF_UNIX-only capability gate until other platforms are verified. No source/XML/tests changed.

### 2026-11-26 — Proposed transport/API contract

HDR spec section 30.21 proposes the future internal transport contract: per-connection FD capability; receive results carrying ordered FD groups with absolute marker byte offsets; strict request-boundary validation; output FD markers bound to exact reply byte offsets/segments; and libxcb output ordering that drains prior buffered bytes before staging an FD group atomically. Keep new DIX/Xtrans helpers private/additive, preserve legacy DRI3 and NVIDIA ABI, and fail closed to legacy SDR. Implementation order: reproduce Linux AF_UNIX barrier semantics; implement/test Xtrans receive accounting; implement ordered server output; fix/test libxcb request ordering; then add extension round-trip tests and freeze XML. No source/XML/tests changed; no implementation authorization.

## 2026-11-26 — SCM_RIGHTS marker-offset correction and Xtrans PRs

Current Xserver master at refresh: `d439c01fef4c8dcb3a0d05471e3edb0eb00f6f89`.

- PR [#4230](https://github.com/X11Libre/xserver/pull/4230), branch `test/xtrans-af-unix-fd-framing`, adds Linux AF_UNIX socketpair regression tests for read-ahead/coalesced reads, separate SCM_RIGHTS deliveries, and `MSG_CTRUNC`. Initial CI exposed a duplicate declaration of `xtrans_fd_test()` in `test/tests.h`; removed the duplicate in commit `6d8ee1bcdf6a261c84690a2a485fdec2be048d84`. CI for this exact head was queued at the last check; do not treat it as green yet.
- PR [#4231](https://github.com/X11Libre/xserver/pull/4231), branch `fix/xtrans-reject-truncated-fd-groups`, rejects truncated ancillary input and closes delivered FDs. Its build matrix was green in the last observed run except that the Solaris job was still running; the sign-off action failed with `Resource not accessible by integration`, so that check is an infrastructure/permission failure, not evidence of a missing sign-off. The exact run must be refreshed before status claims.
- The #4230 read-ahead test proves that a single `recvmsg()` can return ordinary bytes preceding an FD-bearing `sendmsg()` together with the entire FD-bearing payload. Consequently the section 30.20 formula `read_offset + nbytes - 1` is invalid: it points at the last returned byte, not necessarily the first byte of the FD-bearing request. Corrected the design draft in section 30.22; no replacement marker-association algorithm is approved yet.
- Independent follow-up finding: `appendFd()` in `os/Xtranssock.c` returns void and silently drops a descriptor on allocation failure (with a TODO to mark the connection broken). The receive loop ignores this failure and can leave a partial FD queue; the send path also reports success. This is a separate fail-closed/error-propagation gap not addressed by #4231. Needs a scoped design and regression test before implementation.
- No HDR rendering or extension implementation was started. No protocol XML or wire ABI was frozen.

## 2026-11-26 — DRI3 FD-passing compatibility review

Owner correctly noted that DRI has used FD passing successfully for years. Audited current Xserver master `d439c01fef4c8dcb3a0d05471e3edb0eb00f6f89`:

- DRI3 uses the established connection-wide FIFO: Xtrans `recvmsg()` appends received descriptors; DRI3 calls `SetReqFds(N)` and consumes them in order with `ReadFdFromClient()`. Examples: `proc_dri3_pixmap_from_buffer()`, `proc_dri3_pixmap_from_buffers()`, `proc_dri3_fence_from_fd()`.
- Server replies call `WriteFdToClient()` immediately before `X_SEND_REPLY_SIMPLE()`. Xtrans queues FDs and attaches them to the next socket write using `sendmsg()`.
- Therefore the lack of an exact byte offset in a batched `recvmsg()` does not itself invalidate DRI3. Its association is FIFO/protocol-order based. Exact marker offsets should not be made a universal prerequisite for HDR, and a replacement transport API is not justified absent a concrete need.
- Limits remain: `SetReqFds()` only logs mismatches and overwrites the expected-to-consume counter; Xtrans does not retain ancillary group boundaries; `appendFd()` silently closes on allocation failure and its callers still report success. Also test output ordering if an earlier flush leaves bytes buffered after EAGAIN before a later FD-bearing reply.
- Revised HDR spec section 30.23: preserve legacy FIFO API, use explicit per-message FD counts and ordered consumption, harden allocation/truncation/error handling additively, and add tests for read-ahead, multiple FD-bearing requests, interleaved ordinary requests, count mismatches/extras, and output backpressure. Sections 30.20–30.21 are superseded where they require exact offsets universally or propose replacing the FIFO.
- No Xserver implementation or protocol XML was changed.
