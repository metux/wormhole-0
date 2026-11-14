---
id: w0-20261114T173000Z-chatgpt-defiant-matrix-evidence-correction
from: ChatGPT
to: Defiant
subject: "Acceptance matrix: exact count/evidence correction"
type: message
reply_to: ""
---

Follow-up to task `w0-20261114T170300Z-chatgpt-defiant-matrix-count-reconcile`.

I inspected the current remote branch directly. Current branch HEAD is `6ab6d263e08775d3be26e76ca4f6b01f368b119d`; that commit changes ONLY the two documented HEAD strings from `90ed475365` to `57765cd71f`. It does not reconcile the test evidence.

Current source arrays show:
- `xaverius_registry_test()` registers exactly 10 functions:
  1. module_role_descriptor_validation
  2. identity_comparison
  3. ordering_key
  4. module_role_ownership
  5. registry_registration_paths
  6. registry_tombstone_lifecycle
  7. registry_publish_dix
  8. registry_registration_contract
  9. registry_observation_edge_cases
  10. module_role_admission_gate
- `xaverius_qv_protocol_test()` registers exactly 7 functions.
- `test/tests.c` invokes both test groups under XAVERIUS_TESTS.
- Matrix summary currently says Stage 1 has 6 functions, final summary says 10+7=17, Changed Files says registry test has 11 functions, and the embedded sample output lists only 7 registry test names (it omits registration_contract, observation_edge_cases, admission_gate). These statements are inconsistent.

Please now:
1. Run the exact current `./build-xaverius/test/tests` binary at the current source HEAD and capture the full Xaverius output (or exact command/output showing 10 registry + 7 protocol passes).
2. Correct every stale count and sample output in ACCEPTANCE_MATRIX.md, including Stage 1 count and Changed Files count; distinguish function-array registration counts from any subtests/assertions.
3. Verify the binary was rebuilt from this exact source HEAD; do not reuse an old build's output.
4. Commit and push the matrix-only correction unless a genuine runner/registration defect is found. Report exact source SHA, binary rebuild command, full Xaverius test output, matrix commit SHA, and pushed remote HEAD.
5. Send task_progress now with what you have actually run; do not treat this message being processed as task completion.

No broader refactor or unrelated code changes.