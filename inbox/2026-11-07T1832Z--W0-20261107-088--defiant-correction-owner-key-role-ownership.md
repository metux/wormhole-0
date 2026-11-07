Message-ID: W0-20261107-088
Date: 2026-11-07T18:32:00Z
From: owner@local
To: Defiant@local
Subject: Correction: owner_key is ModuleRole-owned; return to the immediate syntax/API repair
Content-Type: text/markdown; charset=utf-8
In-Reply-To: W0-20261107-085
References: W0-20261107-075 W0-20261107-085
Correlation-ID: W0-20261107-088
Task-ID: W0-075

This is a new immutable correction supplementing W0-075/W0-085. Do not edit those messages.

The newly integrated commit 962913b5d14cda52fc7fdab63b68b70365822d06 does not satisfy the immediate repair boundary and introduces a direct normative conflict:

1. It adds owner_key/owner_key_size to XaveriusDeviceBackendRegistration in xaverius_priv.h and copies those fields into XaveriusDeviceBackendBinding. This is explicitly wrong. Per the normative model and W0-085, owner_key belongs to XaveriusModuleRoleDescriptor / the server-owned ModuleRole record. Copy and validate it there at ModuleRole creation. Resolve binding preference through the retained ModuleRole. Do NOT add owner_key to the registration descriptor or duplicate it in the binding. Revert these additions in a separate narrow correction commit.
2. The current registry.c still contains the malformed split RegisterDdxDevice declaration and duplicate nested/file-scope xaverius_compare_ordering_key definition. The exact replay path still dereferences binding before allocation. These were the immediate blockers; the new commit did not repair them.
3. xaverius_intern.h still has duplicate declarations for XaveriusFindDeviceByBindingGeneration, XaveriusFirstDevice, XaveriusNextDevice, and XaveriusDeviceCount. The exact-tip Ubuntu/Valgrind and Alpine CI logs fail with -Werror=redundant-decls; Arch fails at registry.c:447 with the malformed declaration.
4. Do not treat adding the observation struct definitions as completion of the immediate syntax/API repair. Keep the immediate commit limited to restoring valid translation-unit/control-flow structure, removing duplicate declarations, and resolving declarations/types without expanding ABI or mixing lifecycle/snapshot/publication redesign.
5. Do not use the new owner_key fields as a shortcut. The spec note and W0-085 already give the ownership rule. Keep changes small, buildable at each boundary, and report exact parent/HEAD, files, test commands and exit codes.

The latest workflow for merged HEAD da839796189a96b75d3bd81210508fa301a98fe8 is still in progress at this message; completed jobs already include failures on Ubuntu/Valgrind, Alpine, and Arch. Do not claim green CI. Recheck the workflow after the syntax repair.

Continue in the existing Defiant session only. No restart, relaunch, detach, duplicate process, or protected-action approval. Please publish a correlated durable receipt for this exact Message-ID when picked up; otherwise delivery remains unconfirmed.
