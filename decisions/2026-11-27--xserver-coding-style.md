# Xserver C coding conventions

**Decision date:** 2026-11-27  
**Scope:** All Xserver code changes and reviews across project sessions.

## Decision

Apply these conventions to newly written or modified Xserver C code:

1. Declare variables as close as practical to their first use. Avoid declaring all function-local variables at the top when they are only needed later.
2. Use braces for control-flow bodies (if, else, for, while, and similar constructs), including single-statement bodies, wherever practical.
3. Keep the return type and function name on the same line in function definitions; do not insert a line break between them.

## Rationale

These conventions improve local readability, reduce accidental scope and control-flow mistakes, and keep code style consistent across Xserver work.

## Consequences

- Apply the rules during implementation and review, not only when touching the exact lines requested.
- When modifying existing code, avoid unrelated style-only rewrites; apply the conventions to new code and the code being materially changed.
- Check all newly added helper and test functions for these rules before publishing a PR.
