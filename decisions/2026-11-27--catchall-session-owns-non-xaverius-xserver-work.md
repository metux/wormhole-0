# Decision: catch-all session owns non-XAverius Xserver work

**Date:** 2026-11-27  
**Status:** Active owner clarification  
**Supersedes:** Any broader interpretation of the 2026-11-25 separate-session decision that routes all Xserver work to the XAverius session.

## Decision

The dedicated XAverius ChatGPT session is exclusively for XAverius design, specifications, implementation, and directly related coordination.

The general catch-all ChatGPT session owns all other work, including Xserver issues, pull requests, tests, security fixes, and maintenance that are not explicitly part of XAverius.

In particular, Xserver PR **#4170** (XTESTFakeInput valuator-boundary regression test) belongs to the catch-all session and must not be handed off to the XAverius session merely because its repository is `X11Libre/xserver`.

The Starfleetctl workstream remains in its own dedicated session. HDR remains in its own dedicated session.

## Rationale

Repository identity alone is not the workstream boundary. The owner separates XAverius by subject matter, not by every change in the Xserver repository. Sending unrelated Xserver work to the XAverius session creates unnecessary context switching and delays.

## Consequences

- Before assigning Xserver work, classify whether it is actually XAverius-related.
- If it is not XAverius, keep it in the catch-all session and preserve the existing task/ship ownership.
- The catch-all session may coordinate the fleet or take over non-XAverius Xserver work when the fleet fails to make concrete progress, while avoiding concurrent writes to an agent-owned branch/worktree.
- Do not restart ships without explicit owner permission.
- The 2026-11-25 decision separating Starfleetctl and XAverius sessions remains active; this decision narrows the XAverius boundary and does not merge Starfleetctl or HDR into the catch-all session.
