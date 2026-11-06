# W0-20261106-050 — Defiant: resume immediately; no Enterprise audit gate or waiting

**Owner:** Defiant. **Priority:** URGENT.  
**Target:** `X11Libre/xserver:wip/xaverius`.  
**Technical reviewer/acceptance:** project owner (assisted by ChatGPT).  
**Enterprise:** local coordination only; not an auditor or approval gate.

## Why this is being sent
The verified remote branch tip remains `eaff9a2fde04d846021c1dddb26b35e4b77bfeac`. No subsequent Xserver commit is visible. W0-047 is still unacknowledged by any durable response. The one-line eaff9a2 change did not fix the snapshot-size contract; it merely removed the zero-size rejection while the registration path still uses an unset size.

## Immediate instruction — do not wait
1. Resume work now. Do not wait for Enterprise, an Enterprise audit, another user message, or further approval. Enterprise has no technical review role.
2. Acknowledge W0-047 in Wormhole with the exact branch/HEAD and a concise list of the defects you are fixing.
3. Start with a small, buildable snapshot-ABI correction:
   - inspect the normative registration and backend ABI;
   - make the snapshot callback return an explicit data pointer and byte size (or an equivalently well-defined descriptor);
   - initialize outputs to safe sentinels;
   - validate non-NULL data, nonzero/bounded size, count/pointer pairs, overflow and all mandatory snapshot fields;
   - deep-copy exactly the validated data into server-owned immutable storage;
   - add focused tests for success, NULL output, zero size, oversized size and callback failure.
4. Build/test, commit, push, and verify the exact remote SHA. Then continue with the next bounded W0-047 correction package without waiting for review.
5. Follow with mandatory callback validation, deep-copy of all nested operation tables, centralized failure cleanup and correct ModuleRole references; then replay/generation-safe removal; then tombstone/DIX lifetime and ordered RegistryFini; then Stage 3 authority validation and regression coverage.
6. Record exact test commands and results. No commit-message-only claims.
7. Do not expand scope: no Stage 5, no DDX/Screen integration, no protocol handlers, no Vulkan backend, no mass formatting, no unrelated renames, and no extra `xaverius.h`.

## Coordination
Enterprise may deliver this task locally and nudge if you stop, but must not audit or approve code. Technical findings and acceptance come from the owner. Keep Defiant as sole writer on the branch.

**Next expected deliverable:** an acknowledgement plus the first real snapshot ABI correction commit and its test evidence. Continue autonomously through the remaining W0-047 items in small committed packages.
