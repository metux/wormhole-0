# OpenCode plugin helper module packaging

**Date:** 2026-11-25  
**Status:** Verified against starfleetctl bootstrap implementation

## Finding

The bootstrap installer enumerates `fragments/opencode-plugins/` and installs
both `.ts` and `.mjs` assets. It registers only `.ts` files in
`.opencode/opencode.json` as plugin entrypoints; `.mjs` files are runtime
helper modules and are installed beside the entrypoint without registration.

Relevant implementation:
- `internal/bootstrap/checks.go`: `isOpencodePluginAsset()`,
  `verifyOpencodePlugins()`, and `fixOpencodePlugins()`.
- `internal/bootstrap/checks_test.go`: module-asset installation and
  non-registration regression test.

## Decision

Shared helper modules imported by the Starfleet OpenCode plugin should use
`.mjs`, not standalone `.ts` files. A helper `.ts` file would be treated
as another plugin entrypoint by bootstrap and registered for loading as an
independent plugin.

When TypeScript imports such an `.mjs` helper, provide a matching
`.d.mts` declaration for compile-time types. The declaration is not a runtime
asset and is not installed by bootstrap.

The Comms event-pipeline implementation follows this rule with
`prompt-preparation.mjs` and `prompt-preparation.d.mts`, and adds bootstrap
coverage to ensure the helper is installed but not registered.

Related implementation: https://github.com/mpbt-hq/starfleetctl/pull/23
