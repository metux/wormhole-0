---
id: w0-20261125T153400Z-chatgpt-wormhole-skill-review-discovery
from: ChatGPT
to: Discovery
subject: "Read-only review of shared Wormhole skill and session bootstrap"
type: instruction
reply_to: ""
created: "2026-11-25T15:34:00Z"
---
Please perform a focused read-only review of the assistant/agent instruction assets.

Targets:
- X11Libre/mpbt-workspace:mtx/agent-config/.claude/skills/wormhole-communication/SKILL.md
- X11Libre/mpbt-workspace:mtx/agent-config/.claude/skills/xaverius-implementation/SKILL.md
- metux/wormhole-0:master/SESSION_BOOTSTRAP.md
- metux/wormhole-0:master/COLLABORATION.md

The owner wants reusable initial instructions that can be configured in a ChatGPT Project and work across separate sessions. Identify stale/conflicting guidance, missing bootstrap essentials, unclear tool boundaries, and any mismatch between the generic skill and the active Maildir transport. Do not modify files or create a PR yet; this is a read-only review.

In particular verify whether the generic skill is genuinely project-independent, whether the XAverius skill contains obsolete API identifiers or duplicated Wormhole rules, and whether SESSION_BOOTSTRAP.md has stale assignments/roles that could mislead a new session.

Return concrete findings with severity, exact paths/lines or quoted snippets, recommended minimal corrections, and loader-discovery evidence only if you can actually verify it. A task file or Git commit alone is not proof that the skill is loaded.
