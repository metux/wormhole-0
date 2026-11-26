# Decision: HDR driver compatibility and proprietary NVIDIA fallback

**Date:** 2026-11-26  
**Status:** Owner requirement

## Decision

Future X11Libre HDR/color-management implementation must not break the proprietary NVIDIA Xorg driver. That driver is not required to implement HDR.

When the active DDX/backend does not support the new color-management path, the Xserver must continue to provide the existing functional SDR/X11 path. HDR is optional, runtime-capability-gated and opt-in until complete support is established.

## Consequences

- No new mandatory driver callbacks or mandatory changes to the proprietary NVIDIA DDX.
- Prefer private server state and optional/versioned driver interfaces. Any driver-facing ABI change requires a separate compatibility review; adding a field to a public struct must not be presumed binary-compatible.
- Generic DIX/core code must not acquire mandatory DRM/libdrm/GBM, Vulkan or higher OpenGL dependencies.
- No increase of the global OpenGL baseline for HDR.
- Legacy Composite, Damage, Present, DRI3, RandR gamma, pixmap formats/depths and scanout behavior remain unchanged when the new contract is not negotiated.
- Runtime capability discovery must reflect the actual active DDX/backend and supported buffers/outputs. Unsupported HDR operations fail cleanly; they must not partially transform or silently reinterpret content.
- Proprietary NVIDIA DDX compatibility must be tested on real supported driver versions before claiming it as verified. It is not required to expose HDR.

## Evidence and scope

The X11Libre modesetting DDX has its own DRM/KMS-specific CRTC gamma-LUT/CTM code. That code is not evidence about the separate proprietary NVIDIA DDX. The current Xserver repository does not contain the proprietary NVIDIA DDX implementation, so its behavior remains an external validation target.

The detailed source audit and test gates are recorded in section 28 of X11LIBRE_HDR_COLOR_MANAGEMENT_SPEC.md.

This decision is a compatibility requirement, not authorization to begin Xserver implementation.
