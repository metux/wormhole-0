# Starfleetctl plugin source-of-truth correction

Date: 2026-11-25
Status: accepted working rule

## Canonical source

The canonical source for Starfleetctl skills and OpenCode plugins is the repository's
`fragments/` tree:

- Plugin source: `fragments/opencode-plugins/<name>.ts`
- Skill source: `fragments/starfleet-skills/<name>/SKILL.md`

The deployed copies under `.opencode/plugins/` and `.claude/skills/` are generated
artifacts. Edit the fragment in the Starfleetctl development clone; do not edit the
deployed copy.

This is explicitly documented in
`fragments/starfleet-skills/starfleetctl-dev/SKILL.md` (master). The header in
`fragments/opencode-plugins/starfleet-dispatch.ts` previously contradicted that rule
by saying not to hand-edit the fragment itself. That header was corrected on the
`feat/comms-event-pipeline` branch to identify the fragment as canonical and the
`.opencode/plugins/` copy as generated.

## Verification and deployment

A `PLUGIN_VERSION` bump alone is not proof of deployment. Verify the source and
deployed copies byte-for-byte after bootstrap, and verify behavior in the live
OpenCode runtime. Keep the plugin source and its helper modules/tests together in
`fragments/opencode-plugins/`; update bootstrap helper checks when adding runtime
`.mjs` modules.

## Correction note

This decision supersedes any earlier interpretation of the fragment's misleading
header as evidence that `fragments/opencode-plugins/starfleet-dispatch.ts` is itself
generated. The canonical-source rule in the Starfleetctl development skill is
authoritative.
