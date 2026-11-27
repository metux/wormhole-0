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


## 2026-11-26 — Xtrans FD queue allocation failure PR

Opened draft [X11Libre/xserver PR #4232](https://github.com/X11Libre/xserver/pull/4232), branch `fix/xtrans-fd-queue-allocation-failure`, head `401caa60a35ccb18ce7072528b34b754f7d6d031`, based on master `d439c01fef4c8dcb3a0d05471e3edb0eb00f6f89`.

- `appendFd()` now returns `-1/ENOMEM` without closing the FD on allocation failure.
- Send-side queueing propagates the failure, leaving FD ownership with the caller.
- Receive-side allocation failure closes the failed and remaining descriptors from the current ancillary message plus all descriptors already queued on that connection, then fails the read with `ENOMEM`.
- Added Linux tests through the actual Xtrans local transport API. Linker-wrapped `malloc()` injects failure in send queueing and partway through a multi-FD receive; tests check caller ownership, exact FD cleanup and empty FIFO.
- Initial exact-head workflow run `45881893934` exposed a compile error in the new test: `__wrap_malloc` lacked a prior prototype under `-Wmissing-prototypes`. Added the prototype in commit `f8d278e9711f3a3750d802cb5e5a7855ae514d9f`; a fresh exact-head workflow had not yet appeared at the last check. No local build/test run is claimed. PR remains draft and requires full CI plus code review.
- PR #4232 touches `os/Xtranssock.c`, also modified by #4231. Coordinate merge/rebase order; neither PR is considered merge-ready until exact-head CI and review pass.

This PR hardens the established DRI3-compatible FIFO; it does not introduce byte-offset association or change the public Xtrans API.


## 2026-11-26 — Signed-Off-By workflow permission diagnosis

Inspection of the failed `Check Signed-Off-By` job logs and the action source clarified the failure on Xserver PRs #4230, #4231 and #4232:

- The action successfully fetched and enumerated the PR commits, then raised `Resource not accessible by integration`.
- Its source calls `pulls.get()` and `pulls.listCommits()`, and when unsigned commits are found it calls `issues.createComment()`. The workflow currently grants only workflow-level `contents: write`; the sign-off job lacks permission to post a PR comment.
- Opened draft [PR #4233](https://github.com/X11Libre/xserver/pull/4233), branch `ci/fix-signoff-action-permissions`, head `34444903c2caa2d29e5f576973793109c4b0258b`. It grants `pull-requests: write` only to the sign-off job, enabling the action's required PR API/comment access without widening build-job permissions.
- Exact-head workflow run `45882365691` was queued at the last check. This PR itself is unsigned and should cause the action to report a DCO violation once comment permission works; that is expected and does not waive DCO.
- Important: the existing sign-off failures are not merely harmless infrastructure noise. The action logs indicate it found at least one unsigned commit and then could not post its explanatory comment. After #4233 is validated, rerun checks on #4230–#4232 and add valid `Signed-off-by` trailers where missing. Do not merge until the actual DCO check passes.


## 2026-11-26 — Authorized DCO sign-off and replacement PRs

The owner explicitly authorized ChatGPT to apply the owner's DCO sign-off to these Xserver contributions. The identity was verified against existing signed commits in X11Libre/xserver:

`Signed-off-by: Enrico Weigelt, metux IT consult <info@metux.net>`

To avoid rewriting/force-pushing existing PR branches (the collaboration protocol prohibits force-push), created one-commit replacement branches from the same master base. Each replacement commit has the exact same Git tree as its predecessor PR head, parent `d439c01fef4c8dcb3a0d05471e3edb0eb00f6f89`, and the authorized DCO trailer. GitHub API verification confirmed both commit author and committer as `Enrico Weigelt <info@metux.net>` and the trailer in the commit message.

| Original draft | Signed-off replacement | Replacement head |
|---|---|---|
| #4230 | [#4234](https://github.com/X11Libre/xserver/pull/4234) | `ec8356ff43d5f5674983238bbb7cb7d62ff71a9a` |
| #4231 | [#4235](https://github.com/X11Libre/xserver/pull/4235) | `6499dc22620a0378c76a083a7c0afa028060eead` |
| #4232 | [#4236](https://github.com/X11Libre/xserver/pull/4236) | `e73f28888e3c6070ed6d208af5bd3eb557f29ae9` |
| #4233 | [#4237](https://github.com/X11Libre/xserver/pull/4237) | `d9bdcdd8a505dfc7586ed057993211232dc070c7` |

All four replacement PRs are draft, each contains exactly one commit, and CI was queued on their exact heads. The original PRs remain open but now link to the replacements; do not close them until the replacement CI and DCO checks are verified. No local Xserver build/test run is claimed.


## Xserver SCM_RIGHTS PRs — consolidate signed-off work on original PRs (2026-11-26 20:10Z)

- Owner clarified that separate replacement PRs are not wanted and explicitly authorized amend/force-push for owner-controlled PR branches. Updated `COLLABORATION.md` to allow this narrow exception: use force-with-lease against the observed exact old head; never rewrite another agent/shared branch without the owner's authorization.
- Rewrote the original PR branches as single DCO-signed commits instead of maintaining replacement PRs. All new commits are based directly on current Xserver master `d439c01fef4c8dcb3a0d05471e3edb0eb00f6f89` and preserve their respective PR trees, except #4232 which additionally fixes the Alpine/musl linker issue described below.
  - #4230 branch `test/xtrans-af-unix-fd-framing`, head `8aaa71ad6d424e987ab5880fd97c6fd86b30f2f6`, CI run `45888660431` queued.
  - #4231 branch `fix/xtrans-reject-truncated-fd-groups`, head `dfe2e77d4c832ac00820fd2f231b85c024d0eede`, CI run `45888664565` queued.
  - #4232 branch `fix/xtrans-fd-queue-allocation-failure`, head `c50bd2aa6b4f12f9fd153613f78a5b037c7cbac9`, CI run `45888560671` queued.
  - #4233 branch `ci/fix-signoff-action-permissions`, head `2c65e302c47d6edddb8296cd1321e7354f1f851a`, CI run `45888669984` queued.
- All four commits carry the explicitly authorized trailer `Signed-off-by: Enrico Weigelt, metux IT consult <info@metux.net>`. Branch updates used force-with-lease with the previously observed heads. PRs #4230–#4233 remain open; no merge performed.
- Closed replacement PRs #4234, #4235, #4236 and #4237 as superseded, not merged. The original PRs now hold the active changes.
- CI exposed a genuine Alpine/musl failure on former #4236 head `e73f28888e3c6070ed6d208af5bd3eb557f29ae9`: linker error `hidden symbol __wrap_malloc ... is referenced by DSO`. Updated `test/xtrans-fd-alloc.c` to declare `__wrap_malloc` with `visibility("default")`; new head #4232 is awaiting exact-head CI. This targeted fix is based on the linker log and has not yet been validated by a rerun.
- The signed-off check completed successfully on replacement #4234's exact head (run `45883291726`), confirming the DCO trailer is recognized. New exact-head checks are queued on the original PRs; no current CI success is claimed for the rewritten heads.
- Xserver master remains `d439c01fef4c8dcb3a0d05471e3edb0eb00f6f89`. No local build/tests were run. No PR was merged.


## Xserver SCM_RIGHTS PR deep review and test/merge ordering (2026-11-26 22:30Z)

- Deep review confirmed the completed exact-head CI runs for the first consolidated commits:
  - #4230 run `45888660431`: 18 jobs success, 7 skipped, no failures; DCO job success. Ubuntu `xserver:unit` ran successfully.
  - #4231 previous head run `45888664565`: 18 success, 7 skipped, no failures; DCO job success. This head was subsequently extended with a live MSG_CTRUNC test and is superseded by current head below.
  - #4232 previous standalone head run `45888560671`: 18 success, 7 skipped, no failures; DCO job success. Alpine/musl succeeded after adding default visibility to `__wrap_malloc`. This head was subsequently stacked and is superseded by current head below.
  - #4233 run `45888669984`: 18 success, 7 skipped, no failures; DCO job success. Workflow log confirms the job token had `PullRequests: write`.
- Seven jobs are workflow-skipped (Arch, macOS, Cygwin, image/dependency and release jobs); therefore record this as all executed jobs green, not every possible platform exercised.
- Additional review found that #4231 previously had no live Xtrans test for its `MSG_CTRUNC` cleanup. Added `test/xtrans-fd-trunc.c`: it sends 129 FDs over a real local Xtrans connection (receive buffer is sized for 128), expects `_XSERVTransRead()` to fail with `EMSGSIZE`, and verifies no received descriptor remains open. Current #4231 head `d515dc6959b28fa2f2c25c8259c48e80bfc54a79`, parent master `d439c01fef4c8dcb3a0d05471e3edb0eb00f6f89`; exact-head CI run `45904081384` is in progress; DCO job already passed.
- Review found overlapping test registration insertion points between #4230 and #4232. Moved #4232's `xtrans_fd_alloc_test` registration/prototype after `xkb_test`; #4230 remains after `xfree86_test`, and #4231's truncation test is registered after `ddc_tests`. Their `test/meson.build` insertions also use distinct anchors. This avoids known same-gap conflicts when the three changes are merged.
- Stacked #4232 on #4231 to make the source dependency explicit and avoid competing edits to `os/Xtranssock.c`. Current #4232 base is branch `fix/xtrans-reject-truncated-fd-groups` at `d515dc6...`; current head `f3c106b9960d20cbe31ef4c854e0f5b9d6f47ca7` is a single allocation-failure commit parented on #4231. Its PR diff now contains only the allocation-failure delta (5 files). Exact-head run `45904354611` is pending.
- #4230 remains based on master at `8aaa71ad6d424e987ab5880fd97c6fd86b30f2f6`. #4233 remains based on master at `2c65e302c47d6edddb8296cd1321e7354f1f851a`.
- DCO nuance: GitHub's commit API reports `verification=false/reason=unsigned` because no cryptographic GPG signature was created; this is distinct from DCO. The repository's Signed-Off-By action recognizes the explicit trailer on the checked heads. The old bot comment on #4233 refers to superseded commit `3444490...`; current #4233 head's DCO job passed.
- No PR merged. No local build/test run claimed. Do not mark the new #4231/#4232 heads green until their exact-head runs complete; then inspect unit-test and Alpine logs and recheck all PR bases/mergeability before recommending merges.

## Follow-up: FreeBSD NULL include fix and latest exact heads (2026-11-26 22:30Z)

- The first live-MSG_CTRUNC test run 45904081384 exposed one additional FreeBSD-only compile error in test/xtrans-fd-trunc.c: NULL was used by the unconditional test-suite registration function while stddef.h was included only inside the Linux guard. Fixed by including stddef.h unconditionally.
- Current #4231 head is 60a22b75864952817bd1589deac81571574ce5a7, parent master d439c01fef4c8dcb3a0d05471e3edb0eb00f6f89; new exact-head run 45904923410 is pending.
- Rebasing stacked #4232 onto corrected #4231 head produced current #4232 head b8febd859ebffb83e5a1964d411eaafb92bf008b, parent 60a22b75864952817bd1589deac81571574ce5a7; exact-head run 45904977771 is pending. Its base branch remains #4231.
- Intermediate #4231 heads d515dc6... and a30fba0..., and #4232 heads f3c106b... and a268eff.../eefed71..., are superseded; do not cite their CI as current-head validation.
- Both PR descriptions were updated with the FreeBSD failure/fix and current run IDs. No PR merged. Await exact-head CI before further merge recommendations.

## Follow-up: allocation-test include portability and queued CI (2026-11-26 22:35Z)

- A second portability review found the same unconditional NULL sentinel pattern in test/xtrans-fd-alloc.c: stddef.h had been guarded by LDWRAP_TESTS. Moved stddef.h to the unconditional includes, so non-Linux or no-linker-wrap builds also have NULL defined.
- Latest #4232 head is now e82644d98e58ade396950172ddbd341463ac6f16, parent #4231 current head 60a22b75864952817bd1589deac81571574ce5a7. Exact-head CI run 45905193967 exists but is still pending with no jobs published yet.
- #4231 remains at 60a22b75864952817bd1589deac81571574ce5a7; exact-head CI run 45904923410 is still pending with no jobs published yet.
- PR #4232 body and tracker should cite e82644d..., not superseded b8febd...
- No further code changes until current runs start; no PR merged.

## Final stacking cleanup — #4232 diff normalized to allocation-only (2026-11-26 22:40Z)

- Normalized test/xtrans-fd-trunc.c in #4232 to be byte-identical to the prerequisite #4231 branch. This removes an accidental one-blank-line difference that made GitHub count it as a sixth changed file.
- Current #4232 head is now 2429f7f00860501602a86452f59b037895a0c925, parent #4231 current head 60a22b75864952817bd1589deac81571574ce5a7. The actual PR patch now lists exactly five files: os/Xtranssock.c, test/meson.build, test/tests.c, test/tests.h, test/xtrans-fd-alloc.c. Exact-head CI run 45905471008 is pending.
- #4231 remains at 60a22b75864952817bd1589deac81571574ce5a7 with run 45904923410 pending.
- No PR merged. Current #4232 remains based on #4231; after #4231 merges, rebase/retarget #4232 to updated master and rerun CI.


## 2026-11-27 — Xtrans CI refresh and milestone-0 EDID integration draft

Verified against X11Libre/xserver master `d439c01fef4c8dcb3a0d05471e3edb0eb00f6f89`:

- #4230 head `51b4086038600c50463fd76f530dc48d718a0a88`: exact-head workflows `45915153363` and `45915160161` both completed with no failed jobs. The latter was rerun after an Ubuntu Xephyr/Piglit failure; the retry passed.
- #4231 head `fd0443f1d0e79fcaf2eb3f30b43963e5edbc5a32`: runs `45912011180` and `45912016553` passed; DCO passed.
- #4232 head `44cca19ad3ff98a70afadaf80418318ea44283af`: runs `45915520647` and `45915525711` passed; DCO passed. It remains stacked on #4231 and must be retargeted/rebased after #4231 merges.
- #4233 head `2c65e302c47d6edddb8296cd1321e7354f1f851a`: run `45888669984` passed, including DCO.
- All four PRs remain Draft/open/unmerged. These CI results cover executed jobs; platform/image/release jobs marked skipped were not exercised. No local Xserver build/test has been run.

A new isolated milestone-0 proposal is in Draft PR [#4240](https://github.com/X11Libre/xserver/pull/4240), branch `feature/edid-hdr-bounded-capabilities`, exact head `ff89fbe00aa56b3c80ef3382f83b17f40fff4f50`, based directly on the master SHA above. It records the actual EDID byte length, bounds extension walkers and CTA data-block parsing, only marks raw data complete when all advertised extension blocks are present, uses the size-aware parser in modesetting/full-DDC paths, and adds short/truncated/malformed EDID tests. The change is read-only capability discovery: it does not select HDR output modes, submit KMS HDR metadata, add a client wire ABI, or change rendering. Exact-head workflow `45994542211` is queued (the earlier run `45994123556` covered a superseded head). No local build/test is claimed.

Keep #4240 Draft until exact-head CI completes and the complete diff receives another review. No PR was merged. This milestone does not authorize implementation of the still-unapproved HDR extension ABI, request parser, output signaling or rendering pipeline.


### 2026-11-27 correction — one-byte first-read probe

A further review found that merely capping the first read to needed - gotnow (normally four bytes for xReq) is insufficient: two SCM_RIGHTS groups attached to successive partial header fragments could be coalesced by one four-byte recvmsg(), and _XSERVTransGetLastReadFdCount() reports only the combined descriptor count for that read. The HDR-only candidate therefore requires the first positive read of every request to be exactly **one byte**. This isolates the ancillary group attached to the request's first byte. Every subsequent positive read is bounded to needed - gotnow, and any nonzero FD count on those later reads is rejected. EAGAIN/zero-byte reads do not advance the per-request state. This costs one extra read operation per negotiated HDR request but avoids inferring group boundaries from a coalesced read. The candidate is recorded in spec section 30.27; it remains a design candidate, not an approved wire ABI or implementation authorization.


## 2026-11-27 — One-byte receive-probe regression alignment

A source/test review found that a four-byte first header read could coalesce SCM_RIGHTS groups attached to successive partial-header writes. The HDR-only receive candidate now requires an exact one-byte first positive read at each request boundary; subsequent reads remain bounded to the current request and reject any later nonzero FD count. Specification section 30.27 records the corrected candidate and its one-extra-read cost.

The transport regressions now model this exact rule:

- #4230 head `5bc5fc17afa4dc559f314baa4d3d0b9e11e33b96`: the request-bounded test uses a one-byte first read, and the multiple-groups test attaches groups to the first and second request-header bytes, verifying that the second group arrives on a later read. Exact-head CI run `45995093049` is queued.
- #4232 head `d0cf2465da15f2fdc3ca9fc41dd98da44d90b86a`, still stacked on #4231 head `fd0443f1d0e79fcaf2eb3f30b43963e5edbc5a32`: the per-read count test now uses read sizes 1, 1, 2, 4 and observes 0, 2, 1, 0. A new exact-head workflow run has not appeared yet; previous green runs apply only to the superseded head.

No local Xserver build/test has been run. #4230 and #4232 remain Draft/open/unmerged. The candidate is not an approved wire ABI or authorization to implement the HDR request parser.


### 2026-11-27 superseding correction — no one-byte probe

A direct Linux AF_UNIX/SOCK_STREAM socketpair check with a four-byte recvmsg() buffer and two successive two-byte SCM_RIGHTS sendmsg() payloads returned only the first payload plus its FD group on the first read, then the second payload plus its group on the next read. This means Linux preserves the ancillary sendmsg boundaries even when the receive buffer is larger than each payload. The earlier one-byte-probe correction above was too conservative and is superseded: no one-byte read is required. The HDR candidate still must disable read-ahead across request boundaries and cap reads to the exact remaining bytes of the current parse stage. Ordinary bytes preceding an FD-bearing sendmsg() can coalesce with that payload, but with request-bounded reads those bytes belong to the same current request. The association is request-level, not an inferred byte offset. Spec section 30.27 records the corrected design.

The transport tests now request a four-byte header read while sending two-byte ancillary groups. #4230 current head is `afa92c9ebf26d4faf8be20bee70842767124b82b` (run `45995722324` queued); #4232 current head is `b5387cf06a6614ee58e6608ec15e60680eeb1cdb` (run `45995831968` queued). Earlier one-byte-probe heads and their CI runs are superseded.


### 2026-11-27 — EDID legacy API compatibility correction

The first #4240 CI run (`45994542211`) exposed a real compatibility regression in the new length guard: existing `xf86InterpretEEDID()` callers pass a pointer to a complete multi-block EDID but the legacy API has no length parameter. The regression test `xfree86_edid_ds_ranges_array_bound` correctly caught that extension detailed timings were no longer walked.

Fix: `rawDataLen == 0` now explicitly means “legacy pointer-only API; length unknown”. The legacy walkers retain their historical caller contract in that case, while the size-aware `xf86ParseEDID()` path records the actual length and bounds all extension walkers. The legacy API documentation now states that callers must provide all advertised extension blocks and should use `xf86ParseEDID()` when the real length is known. Current #4240 head: `da9116cab956ea81d6b78c2fc6172741eb0f8377`; exact-head workflow `45996097202` is pending. The failed run is superseded; no merge.


## 2026-11-27 13:30Z — Session continuity checkpoint and live Xserver refresh

The owner requested that every meaningful HDR work checkpoint be recorded in Wormhole so a new ChatGPT session can resume without relying on chat history. Treat this as an ongoing operating requirement: update this handoff after material inspections, decisions, commits, CI changes and before ending a work period. This file remains the durable technical handoff; do not use account Memory or chat summaries as the authoritative project state. The separate workspace topic is also updated for dashboard discoverability.

### Project and communication context refreshed

- Re-read `CHATGPT_PROJECT_INSTRUCTIONS.md`, `COLLABORATION.md`, `SESSION_BOOTSTRAP.md`, and the accepted decision `decisions/2026-11-27--mandatory-wormhole-message-preflight.md` from `metux/wormhole-0:master`.
- Read `X11Libre/mpbt-workspace:mtx/agent-config/.claude/skills/wormhole-communication/SKILL.md`, plus `coding-style-xserver/SKILL.md` and `xserver-testing/SKILL.md`.
- Current session scope remains XLibre HDR only. Do not switch to Starfleetctl or XAverius work.
- No Wormhole message was necessary for this owner-directed checkpoint; no outbound message was published.
- Session continuity note: ChatGPT cannot enable OpenCode-like internal auto-compaction or enlarge/disable the context limit. Repository handoff updates are the durable fallback; do not claim internal compaction or background monitoring.

### Live Xserver state verified at 2026-11-27 13:30Z

- `X11Libre/xserver:master` remains `d439c01fef4c8dcb3a0d05471e3edb0eb00f6f89`.
- Open PRs #4230–#4233 and #4240 remain Draft/open/unmerged. Exact branch heads:
  - #4230 `afa92c9ebf26d4faf8be20bee70842767124b82b`, branch `test/xtrans-af-unix-fd-framing`, base master; run `45995722324` in progress.
  - #4231 `fd0443f1d0e79fcaf2eb3f30b43963e5edbc5a32`, branch `fix/xtrans-reject-truncated-fd-groups`, base master; run `45912016553` completed success.
  - #4232 `b5387cf06a6614ee58e6608ec15e60680eeb1cdb`, branch `fix/xtrans-fd-queue-allocation-failure`, base #4231 branch; run `45995831968` in progress.
  - #4233 `2c65e302c47d6edddb8296cd1321e7354f1f851a`, branch `ci/fix-signoff-action-permissions`, base master; run `45888669984` completed success.
  - #4240 `da9116cab956ea81d6b78c2fc6172741eb0f8377`, branch `feature/edid-hdr-bounded-capabilities`, base master; run `45996097202` queued.
- These are exact-head results. Do not transfer a prior run's result to a changed head. No merge or local Xserver build/test is claimed.

### PR #4240 focused code review

Reviewed all six changed-file patches and relevant surrounding source at the current PR head:

- `hw/xfree86/ddc/ddc.c`
- `hw/xfree86/ddc/interpret_edid.c`
- `hw/xfree86/drivers/video/modesetting/drmmode_display.c`
- `include/edid.h`
- `include/xf86DDC.h`
- `test/ddc-tests.c`

Confirmed from source:
- Size-aware `xf86ParseEDID()` copies the input buffer into the allocated monitor object before the DDC caller frees its temporary buffer; `rawDataLen` records the actual copied length.
- The parser rejects null/short inputs, lengths above 32768, and allocation-size overflow.
- Extension iteration is bounded by the stored length for size-aware callers. The legacy pointer-only API retains its old contract through `rawDataLen == 0`, now documented as requiring all advertised extension blocks.
- CTA data-block lookup validates the declared data-collection offset and each block's length before advancing; video block traversal uses that bounded lookup.
- Full DDC reads now fail closed on extension realloc/read failures; modesetting uses the size-aware parser and calls the HDR metadata parser only for a non-NULL monitor.
- Tests cover short/partial buffers, advertised extension counts through 255, invalid CTA offsets and truncated data blocks, and complete real captured EDIDs.

No confirmed new blocker was established by this patch review. This is not a full runtime/build validation: #4240 exact-head CI is still queued and no local build/tests were run. Revisit the full CI logs once run 45996097202 starts/completes, then continue auditing legacy callers and all extension walkers before recommending any state transition. PR remains Draft.

### Current technical direction unchanged

- Linux AF_UNIX SCM_RIGHTS evidence supports request-bounded reads without a one-byte probe; do not revive the superseded one-byte proposal.
- HDR-only request-parser integration is still only a design candidate; no parser, wire ABI, libxcb changes, output metadata signaling or rendering path has been implemented.
- Output-side FD ordering under buffered output/EAGAIN remains a separate unresolved blocker.
- The read-only EDID milestone is the only HDR runtime integration proposed in #4240; no KMS HDR mode selection/signaling or client-visible behavior is added.


## 2026-11-27 13:45Z — Output-side FD ordering source audit

Reviewed current Xserver master `d439c01fef4c8dcb3a0d05471e3edb0eb00f6f89` in `os/io.c`:
- `WriteFdToClient()` queues descriptors through `_XSERVTransSendFd()` without a reply-byte marker.
- `dixWriteToClient()` appends reply bytes to the existing per-client output buffer.
- `FlushClient()` sends the buffer from its current beginning. When the transport returns EAGAIN it retains the unwritten suffix in `oc->output` and returns 0; after a full flush it sets `oc->output = NULL`.
Therefore a queued FD group can attach to older pending output bytes if a prior flush was blocked and a later FD-bearing reply is appended before the pending bytes drain. This is a confirmed source-level hazard. No test yet demonstrates it dynamically, and no implementation fix has been made.

Added normative-design discussion section 30.28 to `X11LIBRE_HDR_COLOR_MANAGEMENT_SPEC.md` in commit `44cb4ad97f17d65190a5d1814fca15fd5877795a`. It specifies required ordered semantics, a cautious flush-before-stage mitigation candidate, callback/reentrancy and partial-write concerns, and regression cases. It explicitly does not authorize implementation. A stronger long-term design is an output-segment queue that binds reply bytes and FDs together. Keep legacy DRI3 behavior unchanged pending a separate compatibility review.

Next: inspect Xtrans send-queue semantics and actual DRI3 reply call sequences; design a deterministic DIX-level regression harness that can force partial writes/EAGAIN; then return with an evidence-backed minimal implementation proposal. Do not treat FlushClient() == 0 as proof the output buffer drained.


## 2026-11-27 14:00Z — Xtrans send-path confirmation

Follow-up review of Xserver master `os/Xtranssock.c` confirmed the lower-level FD attachment semantics recorded in spec section 30.29 (spec commit `62f1e150a46897d800b518f0d167de77603da45b`): queued descriptors are packed into SCM_RIGHTS on the next `_XSERVTransSocketWrite()`; EAGAIN before any byte leaves them queued; any positive sendmsg result discards the queued group even when only a short prefix was sent. Therefore a pending DIX output prefix can receive the descriptors, and after a positive short send the group is consumed with that prefix. This is stronger evidence for the output-ordering blocker, but remains source inspection only; no dynamic DIX test exists yet.

Exact-head CI recheck at 14:00Z: #4230 run 45995722324 and #4232 run 45995831968 remain in progress; #4240 run 45996097202 remains queued. No head changes, no failures reported by the workflow-run summary, and no local build/test or merge.


## 2026-11-27 14:15Z — Existing queued-FD reply callers inventoried

Current-master search found queued-FD reply call sites in DRI3 (`Xext/dri3/dri3_request.c`), RandR leases (`Xext/randr/rrlease.c`) and MIT-SHM FD passing (`Xext/shm/shm.c`). They all call `WriteFdToClient()` before emitting their corresponding reply. Spec section 30.30 records the inventory (commit `8797047bf040aca527b0e61376a72bd3a0b145d3`). This confirms the next-write contract is used by existing protocols; it does not yet prove an observed backpressure failure. Do not silently change the legacy contract in HDR work. HDR should use an explicitly ordered private helper unless a generic repair is separately justified and compatibility-tested.

The next concrete engineering task is a deterministic DIX-level regression that forces old buffered output/EAGAIN, then sends an FD-bearing reply and inspects which bytes receive the SCM_RIGHTS group. The test must also cover retry, positive short send, callback/reentrancy and teardown. No such test or implementation exists yet.


## 2026-11-27 14:15Z — Owner scope review of Xtrans PRs #4231–#4233

The owner questioned the mismatch between the stated test scope and the Xtrans implementation changes in #4231, and the extra SendFds/DRI3 changes in #4232. Reviewed the exact diffs and narrowed #4232 accordingly.

### #4231 — general fail-closed fix, not merely a test

- Rebased onto current Xserver master `21d70661f216140a6080882b01d2d7bbf20191e4`.
- New head: `4bb84c5e5fa9e9dc60a113d3c60149205a269fa7`, one signed-off commit.
- PR retitled `xtrans: reject truncated SCM_RIGHTS input`; body now explains the actual fix.
- `os/Xtranssock.c` closes the descriptors actually installed by `recvmsg()` when `MSG_CTRUNC` is set and returns `EMSGSIZE`. The 129-FD test exercises the 128-FD control-buffer limit.
- Assessment: not needed for compliant HDR FD groups under the transport limit, but a justified generic Xtrans hardening to prevent incomplete/malformed groups contaminating the connection FIFO. Keep it separate from HDR architectural requirements.
- Exact-head Actions run `46012395834` queued at last check.

### #4232 — narrowed to HDR receive-side prerequisite

- New head: `6ef3a93ef2a0b43d4fc9d77bf9342143d8d12a37`, one signed-off commit, stacked on #4231 head above.
- PR title: `xtrans: expose per-read FD count and handle receive OOM`.
- Kept only: per-read received FD count getter; receive queue allocation failure handling with full cleanup/fail-closed behavior; tests for distinct read-group counts and receive allocation failure.
- Removed from the PR: `SendFds()`, atomic multi-FD send queue, `WriteFdsToClient()`, and DRI3 batch-send conversion. No DRI3 file changes remain.
- The outbound send path is intentionally unchanged. Send-side allocation error propagation is deferred: HDR's current candidate is receive-side only, and the existing DRI3 multi-FD reply path needs a separate atomicity/ownership design before safely surfacing partial enqueue failures.
- Important test correction: the original #4232 test file was not listed in Meson `unit_sources` and was not called by `tests.c`; additionally the `malloc` wrapper had no `-Wl,-wrap,malloc` linker argument. Current head registers the tests in Meson and the runner, and enables the wrapper. Therefore the earlier green CI run `45995831968` for superseded head `b5387cf...` did not exercise the unregistered allocation tests.
- Exact-head run `46013405746` queued at last check.

### #4233

- Owner rebased it onto current master `21d70661f216140a6080882b01d2d7bbf20191e4`.
- Head `d42037bb87537b3592a65eb897e0e08d430a04a2`; one signed-off commit; only grants `pull-requests: write` to the `check-sign-off` job.
- Exact-head Actions run `46011658495` queued at last check. This verifies the rebase target and diff scope, not yet CI success.

### Current Xserver base and next actions

- Master advanced from `d439c01f...` to `21d70661...` with commit `xfree86: list modesetting after all hardware specific drivers` (#4239).
- #4230 and #4240 remain based on `d439c01f...`; rebase them onto current master and refresh exact-head CI before merge consideration.
- #4231–#4233 remain Draft/open/unmerged. No local Xserver build/test has been run.
- Next: wait/check exact-head CI for #4231–#4233; review the new #4232 tests and code against CI results; then rebase #4230/#4240 and continue HDR receive-side parser design. Do not start outbound FD implementation until its DIX ordering/ownership semantics are resolved.


## 2026-11-27 14:20Z — #4232 decoupled from #4231

Correction to the 14:15Z checkpoint: because #4231 is generic MSG_CTRUNC hardening rather than a strict HDR prerequisite, #4232 has now been rebased directly onto current Xserver master and is independent.

- #4231 remains head `4bb84c5e5fa9e9dc60a113d3c60149205a269fa7`, base master `21d70661f216140a6080882b01d2d7bbf20191e4`.
- #4232 new head `466677ef815294c1a43f56a02def3b02e70c3217`, base same master `21d70661f216140a6080882b01d2d7bbf20191e4`; one signed-off commit. The diff is only the per-read FD count, receive queue allocation failure handling and registered tests. It does not contain #4231's MSG_CTRUNC change and has no PR dependency.
- #4232 exact-head Actions run `46013779628` is pending; #4231 run `46012395834` queued; #4233 run `46011658495` queued.
- All remain Draft/open/unmerged. No local build/test claimed.


## 2026-11-27 14:25Z — DIX handling of receive-side transport failures

Checked current `os/io.c` `ReadRequestFromClient()` against master `21d70661f216140a6080882b01d2d7bbf20191e4`: after `_XSERVTransRead()`, only would-block errors are converted into a normal not-ready/yield path. Any other negative read result returns `-1` to the request-processing layer. Therefore the `EMSGSIZE` from #4231 and `ENOMEM` from #4232 are fatal read errors, not conditions under which the server continues parsing later requests with a partial FD queue. This confirms the fail-closed behavior expected by both receive-side changes. Source review only; no runtime test or local build claimed.
