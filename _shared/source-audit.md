# Source Coverage Audit

Audit of source coverage across the repo as of 2026-06-26, with a 2026-07-23 addendum covering 9 newly added tools (Amazon Q Developer CLI, Amp, Goose, OpenHands, Crush, Continue CLI, Auggie, Qwen Code, Warp), followed by a same-day full re-verification pass across all 26 tool pages against their live sources.

**2026-07-23 re-verification pass — real drift found and fixed in:**
- `tools/claude-code` — matcher-pattern charset, canonical-URL redirect chain
- `tools/codex` — docs domain moved to learn.chatgpt.com; fixed a real config error (`[mcp_servers.<id>]`, not `[mcp]`)
- `tools/cursor` — "Semantic Search" built-in tool folded into "Search Files and Folders"
- `tools/devin-cli` — hook stdin schema gained `session_id`/`prompt_id`; `decision` field no longer documents `"deny"`
- `tools/factory-droid` — hooks config fallback to `settings.json`, org-level Enterprise Controls scope, personal Custom Droids scope added
- `tools/gemini-cli`, `tools/github-copilot` — hook event count/list updates (Copilot: 13→14 events, added `userPromptTransformed`)
- `tools/google-antigravity` — corrected hook input JSON schema; formalized `force_ask` decision value
- `tools/hermes` — plugin hook event count 15→17, gateway-exclusive events 8→10, missing CLI subcommands added
- `tools/kimi-code` — corrected project config filename (`local.toml`, not `config.toml`); hooks no longer marked Beta
- `tools/openclaw` — Node.js version requirement updated; `TOOLS.md`/`HEARTBEAT.md` bootstrap files confirmed
- `tools/opencode` — removed a `todowrite` permission key no longer in official docs
- `tools/openhands` — corrected the real settings filename (`agent_settings.json`, not `settings.json`) and documented `--override-with-envs`, both confirmed via live install + smoke test on a real box, not just docs
- `tools/pi-agent` — dependency version bump, repo-move redirect (`badlogic/pi-mono` → `earendil-works/pi`)
- `tools/trae`, `tools/trae-cn` — built-in model list refreshed; project rule filenames no longer fixed to `project_rules.md`; flagged (❓) that Trae now appears to ship a native Hooks feature not yet fully documented in this repo

No changes were needed for `tools/aider`, `tools/kiro`, `tools/amazon-q-dev-cli`, `tools/amp`, `tools/goose`, `tools/crush`, `tools/continue-cli`, `tools/auggie`, `tools/qwen-code`, or `tools/warp` — all re-verified accurate as-is.

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

## 2026-08-15 Addendum: 8 new tools + 1 framework, doc-only pass

Added `tools/cline/`, `tools/kilo-code/`, `tools/junie/`, `tools/grok-build/`, `tools/muse-code/`, `tools/deepseek-harness/`, `tools/jcode/`, `tools/qm/`, plus a `Microsoft Agent Framework` section in `frameworks/README.md`. This was an explicit **doc-only pass** at the user's direction — none of these 9 entries have been sandbox live-verified (no real install, no isolated `$HOME` test run, no `## Testing Status` section), unlike `tools/claude-code/` or `tools/openclaw/`. Master `README.md` (Tools Covered table, Frameworks & SDKs table, Hook Event Cross-Reference) and `_shared/agent-tools-hooks-config.md`, `_shared/hook-event-comparison.md`, `_shared/mcp-support.md`, `_shared/config-file-locations.md` were all updated with rows/columns for the 8 new tools.

| Page | Status | Notes |
|---|---|---|
| `tools/muse-code/README.md` | `mixed` | Beta product (2026-08-05); dedicated hooks/MCP reference sub-pages 404'd on automated fetch — most technical specifics marked ❓, launch/pricing facts cite `[press]` (VentureBeat/TechCrunch/CNBC) |
| `tools/deepseek-harness/README.md` | `mixed` | v0.1 developer preview, only 2 days old at write-up; official docs are thin, many fields ❓ |
| `tools/jcode/README.md` | `github` | No dedicated docs site beyond `jcode.sh/docs`, several subpages are stubs; solo-founder/YC-backed project |
| `tools/qm/README.md` | `github` | Verified as `yc-software/qm` ("Quartermaster") via GitHub repo + `qm.ycombinator.com`; self-hosted service, not a CLI |
| `tools/grok-build/README.md` | `official` + `github` | Source-available (no external PRs), Apache-2.0; exact built-in tool names not enumerated officially |
| `tools/cline/README.md`, `tools/kilo-code/README.md`, `tools/junie/README.md` | `official` | Cleaner official-doc coverage than the others in this batch |
| `frameworks/README.md#microsoft-agent-framework` | `official` | GA date now confirmed via primary source (Harness devblog post, 2026-07-22) — resolved in the 2026-08-15 re-verification pass below |

**Not extended in this pass:** `_shared/activity-agent-matrix.md` (same reason as the 2026-07-23 addendum below — per-activity columns need deeper per-tool verification than a doc-only pass provides; each new tool's own README documents its built-in tools in the meantime).

## 2026-08-15 Re-verification pass: all 8 new tools + Microsoft Agent Framework re-checked

At the user's request, every entry added in the addendum above was re-verified: every Sources URL re-fetched, every high-value claim spot-checked against fresh official sources, and corrections applied in place. Findings:

- **Muse Code** — the hooks/MCP documentation sub-pages that 404'd in the initial pass are reachable via a `.md` URL suffix (e.g. `.../extending.md`). All 12 hook events and full MCP config are now confirmed (`✅ Full` in both, up from `❓ Documented, unconfirmed`). Propagated to `README.md`, `_shared/agent-tools-hooks-config.md`, `_shared/mcp-support.md`, `_shared/config-file-locations.md`, and both hook-event cross-reference matrices.
- **Junie** — hooks confirmed CLI-only (official docs: "ACP and server hosts do not yet invoke any hooks"). MCP config path corrected from `~/.junie/config.json` → `mcpServers` to the actual `.junie/mcp/mcp.json` (project) / `~/.junie/mcp/mcp.json` (user).
- **Kilo Code** — `command.execute.before` hook corrected: it fires on **slash commands**, not shell/bash commands, and (like `chat.message`/`.params`/`.headers`) has no documented block mechanism — both were previously marked ✅ can-block in error.
- **Microsoft Agent Framework** — both previously-flagged ❓s resolved with primary Microsoft sources: Harness GA = 2026-07-22, orchestration patterns (sequential/concurrent/handoff/group/Magentic) reached 1.0 on 2026-07-08.
- **Cline, Grok Build, DeepSeek Harness, jcode, QM** — sources all still valid; corrections were additive/clarifying only (e.g. Cline's `-y` CLI flag renamed to `--auto-approve`, Grok Build's hook `type` field confirmed to support `http` in addition to `command`) — no vendor/license/config-format/hook-count/MCP-path changes, so nothing further propagated to shared tables for these five.
- One dead link found and flagged (not removed, content corroborated elsewhere): `junie.jetbrains.com/docs/junie-plugin-settings.html` (404).

## Audit Outcome

- Every tool page now has a visible sources section.
- Source labels are normalized across the repo: `official`, `github`, `official mirror`, `community`, `third-party`, `press`, `installer-src`.
- Pages that still depend on non-primary sources are listed above instead of being silently treated as fully official.
