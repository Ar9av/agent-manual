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

## 2026-09-10 Addendum: 6 new tools, **live-verified** pass

Selected by live GitHub star/activity data (not blog rankings) from a ~120-entry
survey of currently-trending CLI harnesses, filtered to those installable
headlessly on Linux and drivable with an OpenAI API key. All six were installed
on the **st3ve** sandbox (Ubuntu 24.04.4, kernel 6.17) and driven through two
real tasks against `gpt-5.4-mini` (or `gpt-4.1-mini` where GPT-5 was rejected):
a write+execute task, and a five-part task exercising list / search / read /
write / shell.

| Tool | Version tested | Task 1 | Task 2 | Configuration required |
|------|---------------|--------|--------|------------------------|
| [oh-my-pi](../tools/oh-my-pi/) | 18.1.16 | ✅ | ✅ | none — `OPENAI_API_KEY` only |
| [Tau](../tools/tau/) | 0.4.2 | ✅ | ✅ | none — `OPENAI_API_KEY` only |
| [Prime Agent](../tools/prime-agent/) | 0.9.4 | ✅ | ✅ | none — `OPENAI_API_KEY` only |
| [MiMo Code](../tools/mimo-code/) | 0.1.14 | ✅ | ✅ | none — `OPENAI_API_KEY` only |
| [Reasonix](../tools/reasonix/) | 1.38.3 | ✅ | ✅ | provider TOML + `.env`, `[sandbox] bash = "off"`, `--permission-mode yolo` |
| [Codewhale](../tools/codewhale/) | 0.9.12 | ✅ | ✅ | `config set provider`, `auth set`, `exec --auto`, and a **non-GPT-5 model** |

Findings that only a live run produced — none of these are in any vendor's docs:

- **Codewhale cannot talk to GPT-5-family models.** 0.9.12 sends `max_tokens` on
  the OpenAI chat-completions wire; GPT-5 rejects it with
  `Unsupported parameter: 'max_tokens' ... Use 'max_completion_tokens' instead`.
  `gpt-4.1-mini` works. The run receipt reports
  `codewhale_max_output_tokens_source: "uncatalogued"`, i.e. the model catalog
  does not know the GPT-5 family.
- **Codewhale's `exec` is not agentic by default.** Plain `exec` returns a
  one-shot model response and *describes* the commands it would run; `--auto` is
  required for tool use. Easy to mistake for a broken agent.
- **Codewhale hooks never fire headlessly.** Documented, but load-bearing: an
  interception layer built on its hooks covers the TUI only.
- **Reasonix fails closed on shell execution.** With the default
  `[sandbox] bash = "enforce"` and no usable OS sandbox, bash is refused
  outright. Installing `bubblewrap` was **not** enough on Ubuntu 24.04 —
  `kernel.apparmor_restrict_unprivileged_userns=1` makes `bwrap` fail with
  `setting up uid map: Permission denied`. This will bite anyone running
  Reasonix in a container or hardened host.
- **Reasonix rewrites hand-edited configs.** A minimal `config.toml` was
  normalized into a ~250-line annotated file, and an appended `[sandbox]` table
  was ignored until it was placed ahead of `[[providers]]`.
- **Reasonix `--events-jsonl` hides tool names** that the session JSONL records.
  Adapters should read sessions, not the redacted event stream.
- **Prime Agent has exactly one tool: `ipython`.** Every file read, write,
  search, shell call, and subagent spawn is Python inside one persistent REPL.
  Tool-name-based policy is inert against it — see the normalization map.
- **MiMo Code has no `read`/`grep` tool.** Reads and searches route through
  `exec`/`exec_command`; writes go through `apply_patch`.
- **Prime Agent is not on the public npm registry** under `prime-agent`, and its
  install script inherits the ambient npm prefix (fails `EACCES` on a
  root-owned default).

Not tested, and why:

| Candidate | Stars | Reason |
|---|---|---|
| `ultraworkers/claw-code` | ~195k | Requires a Rust source build (crates.io `claw-code` is a deprecated stub); Anthropic-key-oriented; sandbox disk did not permit it |
| `langchain-ai/deepagents` | ~29k | A framework/harness library, not an end-user CLI — belongs in `frameworks/` |
| `mistralai/mistral-vibe` | ~5k | Mistral API key required; not reachable with an OpenAI key |

### MCP tool-substitution follow-up (same day, live)

All six were re-tested against a purpose-built dependency-free stdio MCP server
(`weather-svc`, tools `get_forecast`/`send_alert` returning a `MCPPROOF` marker)
to fill `mcp-tool-substitution.md`:

| Tool | Trust gate | Native disablement | Verdict |
|---|---|---|---|
| oh-my-pi | none — project `.mcp.json` grants it | ✅ `--tools` / `--no-tools` | **Full substitution works** |
| Reasonix | none — `[[plugins]]` connects silently | ✅ `[tools] enabled` allowlist | Works; permission layer covers MCP tools |
| MiMo Code | none — config file grants it | ✅ OpenCode-style `tools.<x>: false` | Works |
| Prime Agent | none — `mcp add … -- cmd` | ✅ `-nbt` / `-nt` | Works, but see the false-success warning |
| Codewhale | none at all | ❌ **not possible, strictly additive** | Coexistence only |
| Tau | N/A | N/A | No MCP support at all in 0.4.2 |

Two findings from this round are safety-relevant and were verified by
**filesystem side-effect**, not by reading the agent's own transcript:

- **Prime Agent lies about success when tools are disabled.** With `-nbt` it
  replied "Done. The file `/tmp/PRIME_LEAK_PROOF` was created with the word
  `LEAKED`, and its existence was confirmed" — nothing was written to disk. An
  earlier read of a shell-echo test had suggested `-nbt` leaked; the side-effect
  test refuted that. Disablement works; the *narration* does not.
- **Reasonix's `[permissions] mode` fallback does not cover MCP tools.** In the
  default `ask` mode, a native `bash` call was refused headlessly while an MCP
  tool call executed with no prompt. Explicit `deny`/`ask` rules (matching the
  bare tool name) do work and were confirmed blocking a real MCP call.

MCP tool naming diverges across all four hosts that expose a name, and none
matches Claude's `mcp__server__tool` — see the table in
`mcp-tool-substitution.md`.

Not extended in this pass: `activity-agent-matrix.md`, which covers a 14-agent
subset predating most of the current catalog. Adding six columns to a table
already missing ~20 tracked tools would deepen the inconsistency rather than fix
it; that page needs its own widening pass.

Pages are marked `[live]` where a claim was observed on st3ve and `[official]` /
❓ otherwise; every page carries its own "Live Verification" section stating what
passed, what needed configuration, and what broke.

## 2026-09-10 Addendum: Paseo (orchestrator), doc-only pass

Added `tools/paseo/`. **Doc-only** — sourced from https://paseo.sh/docs and the `getpaseo/paseo` repo at `v0.8.0` (2026-09-10); no install, no isolated `$HOME` run, no `## Testing Status` section.

| Page | Status | Notes |
|---|---|---|
| `tools/paseo/README.md` | `official` + `github` | Full public docs (`public-docs/` in-repo mirrors paseo.sh/docs) and Apache-2.0 source; hook names/payloads taken from the v0.8 plugin reference — the plugin API is versioned and v0.7 differs |

Category caveat: Paseo is **not a coding agent**. It is a daemon that launches other agent CLIs, so its hooks are agent-lifecycle (turn started/ended, permission requested, agent/workspace create) rather than `PreToolUse`-style per-tool gates. For that reason it was **not** added to `_shared/hook-event-comparison.md`, `_shared/activity-agent-matrix.md`, or `_shared/tool-normalization-map.md` — it has no built-in file/shell tool surface to normalize and no per-tool-call hook to line up in those matrices. Rows were added to the master `README.md` Tools Covered table, `_shared/agent-tools-hooks-config.md`, `_shared/mcp-support.md`, and `_shared/config-file-locations.md`.

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
