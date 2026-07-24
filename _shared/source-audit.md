# Source Coverage Audit

Audit of source coverage across the repo as of 2026-06-26, with a 2026-07-23 addendum covering 9 newly added tools (Amazon Q Developer CLI, Amp, Goose, OpenHands, Crush, Continue CLI, Auggie, Qwen Code, Warp).

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
| `tools/crush` | `mixed` | Core config/hooks/tools/MCP from official README + repo source tree; custom-commands directory convention and DeepWiki provider background remain community/third-party sourced |
| `tools/amazon-q-dev-cli` | `official+github` | AWS User Guide docs + `aws/amazon-q-developer-cli` GitHub repo docs (`docs/*.md`, mdBook site); project is marked unmaintained by AWS in favor of Kiro CLI |
| `tools/amp` | `official-only` | All core claims from ampcode.com/manual pages plus the official Sourcegraph spinoff blog post; `get_diagnostics` tool flagged ❓ from a community thread |
| `tools/goose` | `official-only` | All claims from goose-docs.ai and the AAIF/Linux Foundation press pages; skills-vs-recipes distinction flagged ❓ as unverified |
| `tools/openhands` | `official+github` | Core claims from docs.openhands.dev; supplemented by GitHub repo README/LICENSE and one community-labeled GitHub issue for internals; several details flagged ❓ |
| `tools/continue-cli` | `official+github` | Config/MCP/tools from official docs; hooks section and acquisition/maintenance notes rely on GitHub source and one press citation (Cursor/Anysphere acquisition, June 2026) |
| `tools/auggie` | `official-only` | All claims traced to docs.augmentcode.com and official npm/GitHub pages; license string flagged ❓ as unpublished |
| `tools/qwen-code` | `mixed` | Core claims from official docs/GitHub; fork-lineage narrative and some tool-id specifics rely on community reviews |
| `tools/warp` | `official-only` | All claims from docs.warp.dev; `rules_enabled` flag flagged ❓ as unverified; classified as a borderline terminal-app entry, not a pure CLI |

## Shared Pages

| Page | Status | Notes |
|---|---|---|
| `_shared/activity-agent-matrix.md` | `mixed` | Matrix rows aggregate per-tool docs; Pi Agent row is GitHub-backed |
| `_shared/agent-tools-hooks-config.md` | `mixed` | Some sections include installer/community-backed path claims |
| `_shared/config-file-locations.md` | `mixed` | Skills-path section includes installer/inference-backed conventions |
| `_shared/hook-event-comparison.md` | `mixed` | Pi Agent hook names remain community-backed; Continue CLI and Crush hook rows are GitHub-source-backed (`[github]`) rather than docs-site-backed, with several event names marked ❓ |
| `_shared/mcp-support.md` | `mixed` | Pi Agent and Gemini MCP rows are GitHub-backed rather than docs-site-backed |
| `_shared/config-file-locations.md` (2026-07-23 addendum) | `mixed` | Rows added for the 9 newly catalogued tools (Amazon Q Dev CLI, Amp, Goose, OpenHands, Crush, Continue CLI, Auggie, Qwen Code, Warp); a few paths (Goose project config, Warp global settings, several skills paths) are marked ❓ where no canonical doc page confirmed them |
| `frameworks/README.md` | `mixed` | LangGraph currently sourced from vendor GitHub repo |

## Unresolved Gaps

These remain intentionally marked rather than guessed:

| Topic | Current source quality | Why not upgraded further |
|---|---|---|
| Pi Agent hooks and advanced extensions | Community/vendor GitHub | No stronger official docs page surfaced in the repo’s existing source set |
| Google Antigravity skill/rules path conventions | Community + installer inference | Official docs confirm the product, but some path specifics are not cleanly documented in one canonical page |
| AutoGen "2.0" branding | Not used | Official docs verify stable AutoGen docs, not the exact `2.0` label requested in the screenshot |
| OpenCode plugin ecosystem examples | Community | Useful examples exist, but they are not normative product docs |
| `_shared/activity-agent-matrix.md` not yet extended to the 9 tools added 2026-07-23 | N/A | Adding accurate per-activity columns (file ops, shell, browser, etc.) for Amazon Q Dev CLI, Amp, Goose, OpenHands, Crush, Continue CLI, Auggie, Qwen Code, Warp requires deeper per-tool verification than the initial pass did; each new tool's own README already documents its built-in tools |
| `_shared/agent-tools-hooks-config.md` not yet extended to the 9 tools added 2026-07-23 | N/A | Same reason — the unified per-tool spec table is large (1150+ lines) and wasn't backfilled in this pass; each tool's own `tools/<name>/README.md` is the authoritative source in the meantime |

## Audit Outcome

- Every tool page now has a visible sources section.
- Source labels are normalized across the repo: `official`, `github`, `official mirror`, `community`, `third-party`, `press`, `installer-src`.
- Pages that still depend on non-primary sources are listed above instead of being silently treated as fully official.
