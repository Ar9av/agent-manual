# Source Coverage Audit

Audit of source coverage across the repo as of 2026-06-26.

## Status Key

| Status | Meaning |
|---|---|
| `official-only` | All core claims in the page are backed by official vendor docs |
| `official+github` | Core claims are backed by official docs plus vendor GitHub sources |
| `mixed` | Page includes some community, mirror, press, or inferred claims |
| `community-dependent` | Important claims rely on community docs because no better primary source was available |

## Tool Pages

| Page | Status | Notes |
|---|---|---|
| `tools/claude-code` | `official-only` | |
| `tools/codex` | `official+github` | Includes vendor GitHub issue/source links for edge cases |
| `tools/gemini-cli` | `official+github` | GitHub mirrors supplement official docs |
| `tools/kiro` | `official-only` | |
| `tools/kimi-code` | `mixed` | Official docs plus official mirror pages |
| `tools/factory-droid` | `official-only` | |
| `tools/hermes` | `official+github` | Vendor docs plus repo examples |
| `tools/openclaw` | `official+github` | Vendor docs plus repo link |
| `tools/cursor` | `official-only` | |
| `tools/devin-cli` | `mixed` | Some install/package context relies on third-party PyPI page |
| `tools/aider` | `official+github` | GitHub issues used to document hook absence / requests |
| `tools/pi-agent` | `community-dependent` | Hooks and some extension behavior rely on community/vendor GitHub sources |
| `tools/trae` | `official+github` | Separate GitHub repo referenced for `trae-agent` |
| `tools/trae-cn` | `mixed` | Includes community release-note source for version context |
| `tools/google-antigravity` | `mixed` | Some path and hook-pipeline details rely on community sources or press context |
| `tools/github-copilot` | `official-only` | |
| `tools/opencode` | `mixed` | Community references retained for plugin ecosystem examples |

## Shared Pages

| Page | Status | Notes |
|---|---|---|
| `_shared/activity-agent-matrix.md` | `mixed` | Matrix rows aggregate per-tool docs; Pi Agent row is GitHub-backed |
| `_shared/agent-tools-hooks-config.md` | `mixed` | Some sections include installer/community-backed path claims |
| `_shared/config-file-locations.md` | `mixed` | Skills-path section includes installer/inference-backed conventions |
| `_shared/hook-event-comparison.md` | `mixed` | Pi Agent hook names remain community-backed |
| `_shared/mcp-support.md` | `mixed` | Pi Agent and Gemini MCP rows are GitHub-backed rather than docs-site-backed |
| `frameworks/README.md` | `mixed` | LangGraph currently sourced from vendor GitHub repo |

## Unresolved Gaps

These remain intentionally marked rather than guessed:

| Topic | Current source quality | Why not upgraded further |
|---|---|---|
| Pi Agent hooks and advanced extensions | Community/vendor GitHub | No stronger official docs page surfaced in the repo’s existing source set |
| Google Antigravity skill/rules path conventions | Community + installer inference | Official docs confirm the product, but some path specifics are not cleanly documented in one canonical page |
| AutoGen "2.0" branding | Not used | Official docs verify stable AutoGen docs, not the exact `2.0` label requested in the screenshot |
| OpenCode plugin ecosystem examples | Community | Useful examples exist, but they are not normative product docs |

## Audit Outcome

- Every tool page now has a visible sources section.
- Source labels are normalized across the repo: `official`, `github`, `official mirror`, `community`, `third-party`, `press`, `installer-src`.
- Pages that still depend on non-primary sources are listed above instead of being silently treated as fully official.
