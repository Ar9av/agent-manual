# MCP Tool Substitution

`mcp-support.md` answers *where the MCP config block lives*. This file answers a different, narrower
question: can you **turn off** an agent's built-in tools and replace them with MCP-provided ones — not
just add MCP alongside the built-ins — and what does this specific host require before that substitution
actually takes effect (server trust, native-tool disablement, MCP tool naming/permissioning, headless
behaviour)? See also `tool-normalization-map.md` for what the built-ins are called in the first place.

## How to read this

Each tool is tagged:
- **Verified `<date>`** — live-tested by installing the CLI and driving real sessions on an Ubuntu box, not just read from docs.
- **Docs-only `<date>`** — official docs / GitHub issues / (for QM) direct source-code reading, no live install — usually because the tool is subscription/OAuth-locked with no BYOK path, admin-gated, or GUI-only with no scriptable headless surface.
- **N/A** — no MCP support at all.

Several "Verified" rows are *partial*: live testing reached real config/permission behavior but a real
model completion was blocked by credential limits (see "Known issue" below) — those rows say exactly
which facts are live-observed vs. inferred from source/docs.

## Summary

| Tool | Tier | Trust gate | Native disablement | Headless failure mode |
|---|---|---|---|---|
| Claude Code | Verified (pre-existing) | User-local only, restrict-not-grant | `permissions.deny` | Reports lacking permission, stops |
| Codex CLI | Verified 2026-08-18 | User-local `trust_level`, restrict-not-grant (project-scoped servers only) | `codex features disable shell_tool` | Fails clean at dir-trust/auth gate, no hang |
| OpenCode | Verified 2026-08-19 | None — project file grants trust | `tools.<x>: false` | Auto-rejects, feeds error back, continues |
| Crush | Verified 2026-08-18 | None — project file grants trust | `options.disabled_tools` | Auto-approves everything (headless = implicit yolo) |
| Continue CLI | Verified 2026-08-18 | None for server reg; permission policy user-global only, no project override at all | `permissions.yaml` exclude (user-only) | **Hangs forever, zero signal** |
| Goose | Verified 2026-08-18 | None | Yes, but 7 separate extensions to disable individually | `GOOSE_MODE` default `auto` = never prompts |
| Cline | Verified 2026-08-18 | None (install = trust) | **Not exposed by CLI** (SDK-only `toolPolicies`) — coexistence only | `--auto-approve` default true; MCP can silently lose race vs. fast auth-fail |
| Kilo Code | Verified 2026-08-19 | **Project file CAN grant trust** (weaker than Claude) | `tools`+`permission` combo, one file | Fails clean (`--format json`) |
| Cursor CLI | Verified 2026-08-18 (partial) | User-local only, restrict-not-grant | `permissions.deny` (config-accepted, not runtime-verified) | Fails immediately on auth (no BYOK — Cursor-issued key only) |
| Auggie CLI | Verified 2026-08-18 | None, any scope grants trust | Yes, `auggie tools remove`, live-confirmed | Fails immediately on auth (OAuth-only, no BYOK) |
| Warp | Verified 2026-08-18 (heavily blocked) | Project can register, not grant (docs) | **Not possible** — only ask/allow friction tiers, no full removal | Mandatory Warp-account login blocks everything, even with BYOK key stored |
| Factory Droid | Verified 2026-08-18 | None, either scope | `--restrict-tools` CLI flag only (not persistent) | `droid mcp list` lies like `claude mcp list`; use stream-json init event |
| OpenHands | Verified 2026-08-19 | None (registration = trust), user-local only | **Not exposed by CLI** (SDK-only `include_default_tools`) — coexistence only | Auto-approves everything once configured |
| Amp | Verified 2026-08-19 | User-local only, restrict-not-grant (workspace servers) | `amp.permissions` reject rules (confirmed); `amp.tools.disable` unconfirmed | Requires real Amp account login, blocks before approval logic |
| jcode | Verified 2026-08-19 | **None at all** — headline finding | `--disable-base-tools` + `--tools`, one command | N/A — no pending-trust state exists to hit |
| Devin CLI | Verified 2026-08-19 (blocked by acct auth) | None for stdio servers (diverges from Claude) | Documented (`permissions.deny`), not runtime-verified | Fails at account/login layer, before MCP/permissions even touched |
| Pi Agent | Verified 2026-08-19 | Gates the whole 3rd-party adapter package, not per-server | Excellent — first-class flags, one-flag full substitution | **Cleanest fail**: `approval_required_headless`, not a hang |
| Grok Build | Verified 2026-08-19 (blocked by auth) | User-local `--trust`, restrict-not-grant | Flags exist (`--disallowed-tools`), not runtime-verified | Binary evidence: rejects outright, no hang |
| Muse Code | Verified 2026-08-19 (heavily blocked) | No project config exists — user-local only, registration = trust | Coarse flags only (shell/write/web), no per-tool list | No functional BYOK path at all (Meta-specific wire protocol) |
| DeepSeek Harness | Verified 2026-08-19 | None at all | First-class, `disabled: true` per plugin row | Fails hard/exits nonzero immediately, no hang |
| Kimi Code | Verified 2026-08-19 (blocked by infra) | Only gates project-*exclusive* servers; shadow-define at user scope to bypass | `[permission].deny` + agent-profile allowlist | Headless forces `auto` policy — **implicitly bypasses** the trust gate |
| Gemini CLI | Docs-only 2026-08-19 | User-local `trustedFolders.json`, restrict-not-grant | Yes, `tools.core`/`tools.exclude`, enterprise-sanctioned | **Hangs** (multiple open GitHub issues) |
| Google Antigravity | Docs-only 2026-08-18 | Distinct Ask/Allow policy layer on top of shared Gemini config family | Not found in docs | Reportedly also hangs (lower-confidence source) |
| Kiro | Docs-only 2026-08-19 | No server-load gate found; tool-call level only | Yes, via custom-agent `allowedTools` | `--no-interactive` requires pre-granted `--trust-tools` |
| GitHub Copilot | Docs-only 2026-08-19 | Explicit folder-trust gate, silently skips in untrusted dirs | **Not first-class** (per-command deny only) | Open bug: CI exits 0 even when all MCP servers fail to start |
| Amazon Q Dev CLI | Docs-only 2026-08-19 | "Legacy" `useLegacyMcpJson` is actually the default-on path (misleading name) | Yes, via custom-agent `tools` array | Open bugs: `--no-interactive` still hangs on approval |
| Trae | Docs-only 2026-08-19 | Project file inert until user flips a global Settings toggle | Inferred only, not confirmed | No headless/CLI mode found (GUI IDE only) |
| Trae CN | Docs-only 2026-08-19 | Mirrors Trae exactly | Inferred only | Same as Trae |
| Junie | Verified 2026-08-19 (CLI install + shipped-binary decompile; blocked on a real JetBrains account for a full agentic run — see note) | Source-confirmed zero gate: decompiled the exact installed build (`2777.8`) — `McpServerBean`'s schema is only `{command, args, env, url, headers, oauth, enabled}`, no `trust`/approval field anywhere, and no approval-gate class exists in the whole `core.mcp` package (only a connection-state `PendingMcpClientException`, unrelated to trust). Config lives at `.junie/mcp/mcp.json` (project) / `~/.junie/mcp.json` (user) | **Not possible** — grepped the full decompiled app (1.2GB) for `excludeTools`/`includeTools`/`disableNativeTools`/`nativeTools`: zero matches anywhere | Not independently live-run (needs a real JetBrains account token — BYOK provider flags like `--anthropic-api-key`/`--litellm-url` do **not** bypass the mandatory `-a/--auth` platform token; tested live and got `"Cannot find authorization"` regardless). Absence of any gate class/field in source is strong indirect evidence for "always trusted, no prompt" |
| Qwen Code | Verified 2026-08-19 (live install, OpenAI-compatible endpoint, no working model needed for the gate itself) | **Scope-based, not just a `trust` bit**: project/workspace-scoped servers are always held `pending` regardless of `trust: true` in config (`trust` only affects post-connection tool-call confirmations); user-scoped servers connect immediately, no gate at all. Approval state lives in `~/.qwen/mcpApprovals.json`, keyed by absolute project path + a content hash of the server config — editing the config invalidates the approval | `--include-tools`/`--exclude-tools` write real config (`excludeTools` in `settings.json`); filtering effect on the exposed tool list not independently confirmed live (no working model call) | **#6131 is fixed** (confirmed in source, v0.21.14, and live): non-yolo mode now logs `[MCP] Skipping MCP server pending approval: <name>` and continues cleanly — no freeze; `--approval-mode yolo` skips the pending gate entirely and actively attempts to connect (confirmed via a real outbound connection attempt to a gated server) |
| QM | Verified 2026-08-19 (local dev-instance harness, mock model) | Admin-REST-only via signed core calls; the shipped Admin **web UI** doesn't even proxy it (no `mcp-servers` route in `plugins/admin`) | **Not possible** — strictly additive, live-confirmed `<serverId>_<tool>` namespacing, no disable-native-tools flag anywhere in source | Live-confirmed: blocked tool call recorded (`blocked: "needs_approval"`), empty assistant reply, run marked "done" but nothing executes; unresolved and unchanged after 15s+, no timeout/TTL field in the approval-store write path |
| Aider | N/A | — no MCP support at all | — | — |

## Cross-cutting patterns

**"Shared config can restrict but not grant trust"** (Claude Code's rule) also holds for: Codex CLI
(project-scoped servers), Cursor CLI, Amp (workspace servers), Grok Build, Gemini CLI, and Kimi Code
(only for servers that exist *exclusively* at project scope). Kiro's tool-*call* approval (not server
loading) follows the same shape.

**No trust gate at all** — registering a server in project config is immediately sufficient, no
restrict-vs-grant distinction exists because there's only one config layer that matters: OpenCode, jcode,
Goose, Hermes, DeepSeek Harness, Auggie CLI, Factory Droid, Devin CLI (for stdio servers specifically),
OpenHands, Cline, Crush, Kilo Code (weaker still — project file can even *grant* trust). Junie has no
gate on registration either, though it layers a separate, entirely-skipped-in-headless project-trust
prompt on top.

**Headless failure modes diverge sharply** — this is probably the most safety-relevant finding:
- *Hangs with no signal at all*: Continue CLI (confirmed live, worst case — no log line, no exit, nothing), Gemini CLI and Google Antigravity (multiple GitHub issues).
- *Returns cleanly but produces nothing, indefinitely*: QM — live-confirmed the call itself doesn't hang (the run completes, HTTP responds normally), but the turn ends with a blocked tool call and an empty assistant reply; nothing resolves it but an explicit `POST /api/approvals/:id {approved:true}` from a human, and no timeout/TTL exists anywhere in the approval-store write path to unstick a headless run automatically.
- *Auto-approves/bypasses the gate silently*: Crush (headless = unconditional yolo, by design, in source), Goose (`GOOSE_MODE` defaults to `auto`), OpenHands (once configured), Kimi Code (`-p` forces an `auto` policy that implicitly skips the trust prompt that gates the same action interactively).
- *Auto-rejects and continues*: OpenCode is the outlier — a pending permission is auto-denied, fed back to the model as a tool error, and the agent loop continues rather than hanging or silently allowing.
- *Silently drops the gated server and continues*: Qwen Code (non-yolo) — a pending project/workspace-scoped server is skipped at discovery (`[MCP] Skipping MCP server pending approval`) with no error surfaced to the model at all; the turn proceeds as if the server were never configured. `--approval-mode yolo` instead connects it, live-confirmed fixing #6131.
- *Fails clean and stops, no hang*: Claude Code, Codex CLI, Pi Agent (cleanest — a structured `approval_required_headless` result), DeepSeek Harness, Grok Build (binary-string evidence), jcode (not applicable — nothing to gate).
- *Fails at a layer before MCP/permissions is ever reached*: Devin CLI (account login), Amp (account login), Cursor CLI and Auggie CLI (both OAuth-only, no BYOK path exists), Warp (mandatory account login even with a BYOK provider key already stored), Junie (same pattern as Warp — `--anthropic-api-key`/`--litellm-url` are accepted flags but a JetBrains platform `-a/--auth` token is still required; live-confirmed `"Cannot find authorization"` with a real provider key).

**Native-tool disablement is sometimes architecturally absent from the shipped CLI**, even though the
SDK underneath supports it: OpenHands (`include_default_tools` exists in `openhands-sdk`, never wired
into the `openhands` CLI) and Cline (`toolPolicies` exists in `@cline/agents`, no CLI/config surface
exposes it) are both "coexistence only" in practice. Warp, Junie, GitHub Copilot, and QM have no such
mechanism at any layer — MCP is architecturally additive-only for these four.

**MCP tool naming is not standardized** even among the tools that copy Claude Code's `mcp__server__tool`
shape (Codex CLI, jcode, DeepSeek Harness, Kimi Code, Devin CLI per docs). Others use a single underscore
(Gemini CLI, Crush, OpenCode, Kilo Code, Grok Build), replace hyphens with underscores in both segments
(Amp), use a triple underscore (Factory Droid), skip prefixing entirely and pass the raw MCP tool name
through (Continue CLI, Cursor CLI, OpenHands), or route everything through a generic wrapper tool instead
of exposing individually-named tools at all (Pi Agent's default `mcp` proxy tool, Auggie's
`find-tool`/`execute-tool` pair, Antigravity's `mcp(server/tool)` policy-pattern selector). A
catalog entry or allow-list template should never assume `mcp__` — check the specific tool.

## Known issue: the OpenAI test credential

The `OPENAI_API_KEY` secret registered for this investigation decloaks to an 11-byte value — far short of
a real OpenAI key (`sk-...`, normally 50+ characters). Every live agent correctly got a `401` from OpenAI
using it, which was sufficient to observe config/permission/trust behavior (most of what this file
documents happens before or independent of a successful completion) but **not** sufficient to observe an
actual model-driven tool call in most tools — those gaps are called out per-tool above as "not
live-verified past the auth boundary."

Separately, in at least four cases (Grok Build, Kimi Code, Goose, and Continue CLI/Pi Agent/OpenCode to
a lesser extent) the Prismor Warden decloak substitution for `@@SECRET:OPENAI_API_KEY@@` did not appear
to resolve reliably when the placeholder was embedded inside a *nested* `ssh ubuntu@... "..."` command or
written into a file via a remote heredoc — the literal placeholder string reached the target API instead
of a real value, and one agent (Grok Build) confirmed this happens even outside SSH in that environment.
This looks like a Warden hook scoping/session issue worth its own investigation, separate from anything
about the tools themselves.

## Sources (Docs-only rows)

| Tool | URL | Fetched | Confirms |
|---|---|---|---|
| Gemini CLI | https://github.com/google-gemini/gemini-cli/blob/main/docs/tools/mcp-server.md | 2026-08-18 | Trust field, naming, includeTools/excludeTools |
| Gemini CLI | https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/trusted-folders.md | 2026-08-18 | Folder-trust, safe-mode drops project settings |
| Gemini CLI | https://github.com/google-gemini/gemini-cli/issues/16567, /12362, /19776 | 2026-08-18 | Headless hangs on unapproved tool calls |
| Google Antigravity | https://antigravity.google/docs/cli/mcp, /docs/mcp | 2026-08-18 | Config paths, Ask-mode default, policy patterns |
| Kiro | https://kiro.dev/docs/mcp/configuration/, /docs/mcp/usage/, /docs/mcp/security/ | 2026-08-19 | Config, `mcp_<server>_<tool>` naming, autoApprove |
| Kiro | https://kiro.dev/docs/cli/v3/permissions/, /docs/cli/headless/ | 2026-08-19 | permissions.yaml, `--trust-tools`/`--trust-all-tools` |
| GitHub Copilot | https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-mcp-servers | 2026-08-19 | Config precedence, folder-trust gate |
| GitHub Copilot | https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/allowing-tools | 2026-08-19 | `--allow-tool`/`--deny-tool`, `<Server>(<tool>)` naming |
| GitHub Copilot | https://github.com/github/copilot-cli/issues/3064 | 2026-08-19 | CI exits 0 despite MCP servers failing to start |
| Amazon Q Dev CLI | https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-mcp-security.html | 2026-08-19 | Security model overview |
| Amazon Q Dev CLI | https://github.com/aws/amazon-q-developer-cli/issues/2984, /2510, /1951, /2195 | 2026-08-19 | "Legacy" config is default; allowedTools bug; --no-interactive hangs |
| Trae / Trae CN | https://docs.trae.cn/ide_add-mcp-servers | 2026-08-19 | Project MCP requires a manual global toggle |
| Junie | `@jetbrains/junie@2777.8.0` (npm install, live CLI + decompiled shipped JAR) | 2026-08-19 | Real config paths, `McpServerBean` schema (no trust field), no approval-gate class anywhere in `core.mcp`, no tool-disablement string anywhere in the app; live `--anthropic-api-key`/`--litellm-url` still blocked by `-a/--auth` |
| Junie | https://junie.jetbrains.com/docs/action-allowlist.html, /junie-cli-configuration.html | 2026-08-19 | All-or-nothing MCP trust; headless always-trusted (docs claim, corroborated by source absence of any gate) |
| Qwen Code | `@qwen-code/qwen-code@0.21.14` (npm, live install + `--debug` session logs) | 2026-08-19 | Scope-based pending-approval gate, `~/.qwen/mcpApprovals.json` hash-bound store, `--exclude-tools` config write, live yolo-vs-default gate behavior |
| Qwen Code | https://github.com/QwenLM/qwen-code/issues/6131 (closed, fix in PR #6177) | 2026-08-19 | `--yolo` doesn't cover new-MCP-server-startup confirmation — confirmed fixed live in 0.21.14 |
| QM | https://github.com/yc-software/qm (main branch, live-run via `scripts/dev-instance.sh` — local Docker, mock harness, no cloud account) | 2026-08-19 | Admin-only registration (REST-only, not even wired into the Admin web UI), additive-only tools, live approval-gate stall |
