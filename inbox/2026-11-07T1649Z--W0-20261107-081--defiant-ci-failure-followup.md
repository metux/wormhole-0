# W0-081 — Fix observed XAverius CI failures; continue registry acceptance

To: Defiant
Priority: blocking
Date: 2026-11-07

This is a focused continuation of W0-075, not a replacement. Keep working on the existing `wip/xaverius` branch. Do not restart or replace your console-attached session; do not force-push, rebase, or merge.

## Verified CI state

Branch head observed: `80b411f6032c6176303039d94dcf2615520087e9`.
GitHub Actions run 42390719664 completed with **failure**:
https://github.com/X11Libre/xserver/actions/runs/42390719664

Ubuntu and SDK build logs show:
- incomplete `XaveriusObservation` / `XaveriusDiscoverySource` types used by list macros and registry cleanup;
- undeclared observation list/count globals;
- `owner_key` and `owner_key_size` referenced although absent from the binding/registration structs;
- duplicate declarations in `xaverius_intern.h`;
- missing prototypes for newly added non-static functions;
- malformed comparator declaration/definition near `registry.c:447`;
- ordering-key helper unused as a consequence of the malformed comparator.

## Required work

1. Read the authoritative XAverius spec/implementation notes before changing semantics. Do not invent fields to accommodate the patch.
2. Fix the incomplete-type/list ownership problem consistently with the existing registry data model.
3. Resolve DDX binding preference using only fields and semantics actually specified; do not add speculative `owner_key` members.
4. Fix comparator declaration/definition and canonical ordering implementation; no shared mutable static scratch buffer.
5. Remove duplicate prototypes while preserving the designated header layout; do not add `xaverius.h` or another protocol header.
6. Add focused tests for comparator ordering/ties, multiple devices, and binding preference.
7. Build the Ubuntu and SDK configurations with the required warning/error settings; push a coherent commit to the existing branch and verify the remote HEAD and CI.

Keep scope limited to the registry acceptance work. No Stage 5, ScreenAttachment, Proc handlers, Vulkan backend, platform-wide repairs, or unrelated formatting.

Do not stop after describing the problem. Make the code/test correction, commit and push, then continue the remaining authorized acceptance criteria. If blocked by an ask, preserve the exact prompt ID and text and report it through the required channel.
