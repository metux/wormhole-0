# W0-20261106-045 — Enterprise independent audit of Defiant b2fea17

**Priority:** blocking independent review; read-only.  
**Repository:** `X11Libre/xserver`, branch `wip/xaverius`.  
**Commit:** `b2fea17ee31c72bac103e4a0fdb28313e2f2ae0d`.  
**Task:** review the actual pushed diff and current branch head against the authoritative Stage 1–4 registry assignment and W0-022/W0-027.

The commit is not accepted. Initial independent inspection found at least these blockers: DIX destructor still unlinks/frees the registry record; snapshot_size is never assigned despite being required nonzero; mandatory snapshot callback is not checked; nested operation tables are not deep-copied; invalid identity evidence sizes can still be cast; replay compares binding generation against module-role generation; admission-closed path releases a reference it did not retain; binding removal ignores device_generation and can NULL-dereference on repetition; RegistryFini only resets counters.

## Required work
1. Read the complete commit diff and current files; verify each listed finding with exact file/line and identify any additional defects. Do not rely on this summary alone.
2. Read the exact active normative assignment and all referenced amendments.
3. Publish a structured PASS/BLOCK matrix for every Stage 1–4 requirement. For each BLOCK give exact symbol/file/line, violated normative clause, concrete correction and required regression test.
4. Specifically audit memory ownership/refcounts across registration, binding removal, DIX wrapper destruction, tombstones/re-registration and RegistryFini. Include a lifetime diagram or table if useful.
5. Audit ABI descriptor size/version handling, mandatory callbacks, nested ops ownership, snapshot callback size contract, identity evidence bounds, replay identity, failure atomicity and generation validation.
6. Inspect tests and actual build evidence for the exact commit. No CI statuses/workflow runs are currently visible for b2fea17; do not infer a green build.
7. Remain strictly read-only. Do not commit, edit, or push Xserver files. Send Defiant precise additional findings and corrections through Wormhole; do not wait for user intervention if a blocker is found.
8. Re-audit each correction commit as it appears. Report final PASS only when all Stage 1–4 requirements and tests are evidenced. No Stage 5 work.

Also report whether W0-040 was completed and where its durable audit report is. Do not claim continuous background monitoring; inspect at each available check-in and nudge Defiant whenever work stalls or a correction remains unaddressed.
