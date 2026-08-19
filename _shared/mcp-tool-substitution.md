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
| Cursor CLI | Verified 2026-08-19 (full agentic runs, real account) | User-local only, restrict-not-grant. Live-confirmed **two independent gates**: `cursor-agent mcp enable <id>` approves the *server* (shows "ready" in `mcp list`), but an individual tool call from that server still gets rejected headlessly unless also covered by `--force`/`--yolo` or an explicit `Mcp(server:tool)` allow rule — server-trust and tool-call-trust are separate | **Confirmed working at runtime, not just config-accepted**: `permissions.deny: ["Shell(*)"]` in `.cursor/cli.json` live-blocked a real shell call in ~4s with `"Permission denied: Command blocked by permissions configuration"` | **Correction — not a hang, but a real ~35-45s stall**: a genuinely fresh, unapproved MCP tool call in headless `-p` mode reliably resolved cleanly on its own in 33-40s across three separate runs (never truly infinite), far slower than peer tools (Kiro/Copilot/Antigravity: 0.1-7s) — plausibly the source of "hangs" reports if someone's timeout is under ~30s. `--force`/`--yolo` bypasses cleanly to a real success in ~6s |
| Auggie CLI | Verified 2026-08-18 | None, any scope grants trust | Yes, `auggie tools remove`, live-confirmed | Fails immediately on auth (OAuth-only, no BYOK) |
| Warp | Verified 2026-08-18 (heavily blocked) | Project can register, not grant (docs) | **Not possible** — only ask/allow friction tiers, no full removal | Mandatory Warp-account login blocks everything, even with BYOK key stored |
| Factory Droid | Verified 2026-08-18 | None, either scope | `--restrict-tools` CLI flag only (not persistent) | `droid mcp list` lies like `claude mcp list`; use stream-json init event |
| OpenHands | Verified 2026-08-19 | None (registration = trust), user-local only | **Not exposed by CLI** (SDK-only `include_default_tools`) — coexistence only | Auto-approves everything once configured |
| Amp | Verified 2026-08-19 | User-local only, restrict-not-grant (workspace servers) | `amp.permissions` reject rules (confirmed); `amp.tools.disable` unconfirmed | Requires real Amp account login, blocks before approval logic |
| jcode | Verified 2026-08-19 | **None at all** — headline finding | `--disable-base-tools` + `--tools`, one command | N/A — no pending-trust state exists to hit |
| Devin CLI | Verified 2026-08-19 (blocked by acct auth) | None for stdio servers (diverges from Claude) | Documented (`permissions.deny`), not runtime-verified | Fails at account/login layer, before MCP/permissions even touched |
| Pi Agent | Verified 2026-08-19 | Gates the whole 3rd-party adapter package, not per-server | Excellent — first-class flags, one-flag full substitution | **Cleanest fail**: `approval_required_headless`, not a hang |
| Grok Build | Verified 2026-08-19 (full agentic runs via a BYOK custom `[model.NAME]` entry — no xAI account needed at all) | User-local `--trust`, restrict-not-grant. Live-confirmed a real MCP server (`grok mcp doctor`) connects and works with zero registration-time prompt | **Nuanced — allowlist works, denylist leaks**: `--disallowed-tools "run_terminal_command,write,..."` live-tested and `run_terminal_command` survived the exclusion anyway (Copilot-style leak); the allowlist form, `--tools "search_tool,use_tool"`, live-confirmed a clean, exact MCP-only tool list with zero built-ins — use allowlist, not denylist, for genuine substitution | Live-confirmed clean: a gated `use_tool` call in headless `-p` mode returned `"stopReason":"cancelled"`, exit 0, in seconds — no hang. `--always-approve` bypasses cleanly to a real success |
| Muse Code | Verified 2026-08-19 (heavily blocked) | No project config exists — user-local only, registration = trust | Coarse flags only (shell/write/web), no per-tool list | No functional BYOK path at all (Meta-specific wire protocol) |
| DeepSeek Harness | Verified 2026-08-19 | None at all | First-class, `disabled: true` per plugin row | Fails hard/exits nonzero immediately, no hang |
| Kimi Code | Verified 2026-08-19 (real BYOK provider, no Moonshot account or decloak fix needed) | Only gates project-*exclusive* servers per docs; MCP server loading itself could not be confirmed working via any of the three documented config paths (`.kimi-code/mcp.json` project/user, Claude-style `.mcp.json`) in this npm package version (`@moonshot-ai/kimi-code` 0.37.2) — an open finding, not a confirmed negative, since a TUI-only `/mcp-config` registration step may be required | `[permission].deny` **confirmed NOT enforced in headless `-p` mode** — live-reproduced twice with different tools: `deny = [{pattern="Bash"}]` still executed a real shell command, and separately `deny = [{pattern="Write"}]` still wrote a real file to disk, both with real, verifiable side effects | **Safety-relevant, live-confirmed**: headless `-p` mode runs with no permission enforcement at all in this version — `--auto`/`--yolo` are explicitly rejected when combined with `-p` (`"Cannot combine --prompt with --auto"`), yet the tool executes fully anyway, ignoring configured deny rules. This is stronger than "forces auto policy" — it's closer to "ignores the permission layer entirely" |
| Gemini CLI | **Deprecated by Google 2026-06-18** | Retired for individual/consumer accounts (Google AI Pro/Ultra, free tier), replaced by the closed-source Antigravity CLI (`agy`). Live-confirmed the shutdown directly: a real Google OAuth login completed but the service rejected it with `IneligibleTierError: This client is no longer supported for Gemini Code Assist for individuals... migrate to the Antigravity suite` | Not independently verified — the tool is end-of-life for this doc's individual-developer audience; `GEMINI_API_KEY` BYOK and paid Standard/Enterprise subscriptions reportedly still work per Google's own migration docs | See the Google Antigravity row instead |
| Google Antigravity | Verified 2026-08-19 (live install via official installer, real account, full agentic `--print` runs) | Confirmed global config is genuinely shared with the old Gemini CLI family (`~/.gemini/config/mcp_config.json`, same schema); a workspace-scoped `.agents/mcp_config.json` (per docs) did **not** get picked up in repeated live testing — left as an open finding, not a confirmed negative, since a GUI-side workspace-registration step may be required. Default is real "Ask" mode: a genuine gated MCP tool call (`exa/company_research_exa`) was denied headlessly by the permission layer | **Confirmed absent**: binary-string search of the shipped `agy` executable found `disabledTools` (matches docs — per-MCP-tool disable) but zero matches for any native/built-in-tool-wide disablement mechanism (`available-tools`, `excluded-tools`, `coreTools`, etc.) — full substitution is not supported | **Correction, live-refuted**: does **not** hang. Reproduced the exact scenario twice — a gated native `read_file` and a gated MCP tool call both failed cleanly in headless `--print` mode in 3-5 seconds with a structured JSON error (`"permission check failed... user denied permission"`, exit 1), not a hang. `--dangerously-skip-permissions` bypasses the same call cleanly to a real success. A `--print-timeout` flag (default 5m) exists as a backstop regardless |
| Kiro (formerly Amazon Q Developer CLI — same product, rebranded; see note) | Verified 2026-08-19 (live install + real login, full agentic runs) | Live-confirmed **no server-load gate**: `kiro-cli mcp add` writes straight to `.kiro/settings/mcp.json` (workspace) with zero prompt or approval step. Tool-*call* trust is a separate layer: `--trust-tools`/`--trust-all-tools`/agent-config `allowedTools`. Live-confirmed the `--no-interactive` + untrusted-tool-call case **fails cleanly** (`"rejected because it matches one or more rules on the denied list: - non-interactive mode (no user to approve)"`, exit 0) — this refutes the old hang bugs (aws/amazon-q-developer-cli#1951, #2195), fixed in the current unified binary | **Confirmed possible and live-tested**: an agent config with `"tools": ["@weather-svc"]` (no built-ins) produced a live chat session where the model explicitly listed only the two MCP tools and stated no shell/bash tool was available at all | `--no-interactive` + `--trust-tools="@server/tool"` (confirmed exact addressing format) successfully executed the gated call in 0.13s — no hang, no prompt, real tool output returned |
| GitHub Copilot | Verified 2026-08-19 (live install, real subscription, full agentic runs) | Live-confirmed folder-trust gate: a project's `.github/mcp.json` server is silently absent from `session.mcp_servers_loaded` in an untrusted dir; setting `GITHUB_COPILOT_PROMPT_MODE_WORKSPACE_MCP=true` force-loads it (confirmed by the server then appearing with `"source":"workspace"`) | **Correction — not actually possible**, despite `--available-tools`/`--excluded-tools`/`--disable-builtin-mcps` flags existing: live-tested an agent restricted to `--available-tools='weather-svc-get_forecast,weather-svc-send_alert'` plus `--disable-builtin-mcps`, and asked it to run `git status` via its shell tool — it executed successfully (real `bash` tool call, real exit code 128 for "not a repo"). The baseline `bash`/shell tool is not excludable by this mechanism, so full native-tool substitution is not achievable | **#3064 reproduced live**: forced a workspace MCP server to fail (`"status":"failed"`, real connection error in the event stream) and the run still finished with the top-level `"exitCode":0` in the structured JSON result — a caller checking only the exit code would see success |
| Trae | Docs-only 2026-08-19 | Project file inert until user flips a global Settings toggle | Inferred only, not confirmed | No headless/CLI mode found (GUI IDE only) |
| Trae CN | Docs-only 2026-08-19 | Mirrors Trae exactly | Inferred only | Same as Trae |
| Junie | Verified 2026-08-19 (CLI install + shipped-binary decompile; blocked on a real JetBrains account for a full agentic run — see note) | Source-confirmed zero gate: decompiled the exact installed build (`2777.8`) — `McpServerBean`'s schema is only `{command, args, env, url, headers, oauth, enabled}`, no `trust`/approval field anywhere, and no approval-gate class exists in the whole `core.mcp` package (only a connection-state `PendingMcpClientException`, unrelated to trust). Config lives at `.junie/mcp/mcp.json` (project) / `~/.junie/mcp.json` (user) | **Not possible** — grepped the full decompiled app (1.2GB) for `excludeTools`/`includeTools`/`disableNativeTools`/`nativeTools`: zero matches anywhere | Not independently live-run (needs a real JetBrains account token — BYOK provider flags like `--anthropic-api-key`/`--litellm-url` do **not** bypass the mandatory `-a/--auth` platform token; tested live and got `"Cannot find authorization"` regardless). Absence of any gate class/field in source is strong indirect evidence for "always trusted, no prompt" |
| Qwen Code | Verified 2026-08-19 (live install, OpenAI-compatible endpoint, no working model needed for the gate itself) | **Scope-based, not just a `trust` bit**: project/workspace-scoped servers are always held `pending` regardless of `trust: true` in config (`trust` only affects post-connection tool-call confirmations); user-scoped servers connect immediately, no gate at all. Approval state lives in `~/.qwen/mcpApprovals.json`, keyed by absolute project path + a content hash of the server config — editing the config invalidates the approval | `--include-tools`/`--exclude-tools` write real config (`excludeTools` in `settings.json`); filtering effect on the exposed tool list not independently confirmed live (no working model call) | **#6131 is fixed** (confirmed in source, v0.21.14, and live): non-yolo mode now logs `[MCP] Skipping MCP server pending approval: <name>` and continues cleanly — no freeze; `--approval-mode yolo` skips the pending gate entirely and actively attempts to connect (confirmed via a real outbound connection attempt to a gated server) |
| QM | Verified 2026-08-19 (local dev-instance harness, mock model) | Admin-REST-only via signed core calls; the shipped Admin **web UI** doesn't even proxy it (no `mcp-servers` route in `plugins/admin`) | **Not possible** — strictly additive, live-confirmed `<serverId>_<tool>` namespacing, no disable-native-tools flag anywhere in source | Live-confirmed: blocked tool call recorded (`blocked: "needs_approval"`), empty assistant reply, run marked "done" but nothing executes; unresolved and unchanged after 15s+, no timeout/TTL field in the approval-store write path |
| Aider | N/A | — no MCP support at all | — | — |

## Cross-cutting patterns

**"Shared config can restrict but not grant trust"** (Claude Code's rule) also holds for: Codex CLI
(project-scoped servers), Cursor CLI, Amp (workspace servers), Grok Build, and Kimi Code
(only for servers that exist *exclusively* at project scope). Kiro's tool-*call* approval (not server
loading) follows the same shape.

**No trust gate at all** — registering a server in project config is immediately sufficient, no
restrict-vs-grant distinction exists because there's only one config layer that matters: OpenCode, jcode,
Goose, Hermes, DeepSeek Harness, Auggie CLI, Factory Droid, Devin CLI (for stdio servers specifically),
OpenHands, Cline, Crush, Kilo Code (weaker still — project file can even *grant* trust). Junie has no
gate on registration either, though it layers a separate, entirely-skipped-in-headless project-trust
prompt on top. Kiro (formerly Amazon Q Developer CLI) also has zero server-load gate, live-confirmed:
`kiro-cli mcp add` writes to config with no prompt at all — trust is enforced only at the tool-*call*
layer.

**Headless failure modes diverge sharply** — this is probably the most safety-relevant finding:
- *Hangs with no signal at all*: Continue CLI (confirmed live, worst case — no log line, no exit, nothing). Gemini CLI's now-historical entry reported the same failure mode before Google deprecated it in favor of Antigravity — but Antigravity itself does **not** reproduce this live (see below); the old claim doesn't carry over to its successor.
- *Returns cleanly but produces nothing, indefinitely*: QM — live-confirmed the call itself doesn't hang (the run completes, HTTP responds normally), but the turn ends with a blocked tool call and an empty assistant reply; nothing resolves it but an explicit `POST /api/approvals/:id {approved:true}` from a human, and no timeout/TTL exists anywhere in the approval-store write path to unstick a headless run automatically.
- *Auto-approves/bypasses the gate silently*: Crush (headless = unconditional yolo, by design, in source), Goose (`GOOSE_MODE` defaults to `auto`), OpenHands (once configured), Kimi Code — live-confirmed `-p` mode bypasses the permission layer entirely, not just the trust prompt: explicit `[permission].deny` rules for native tools were silently ignored twice, with real, verifiable side effects (a real shell command ran, a real file got written).
- *Auto-rejects and continues*: OpenCode is the outlier — a pending permission is auto-denied, fed back to the model as a tool error, and the agent loop continues rather than hanging or silently allowing.
- *Silently drops the gated server and continues*: Qwen Code (non-yolo) — a pending project/workspace-scoped server is skipped at discovery (`[MCP] Skipping MCP server pending approval`) with no error surfaced to the model at all; the turn proceeds as if the server were never configured. `--approval-mode yolo` instead connects it, live-confirmed fixing #6131.
- *Fails clean and stops, no hang*: Claude Code, Codex CLI, Pi Agent (cleanest — a structured `approval_required_headless` result), DeepSeek Harness, jcode (not applicable — nothing to gate), Kiro (formerly Amazon Q Dev CLI) — live-confirmed exit 0 with a denied-list rejection message; the old aws/amazon-q-developer-cli#1951/#2195 hang bugs no longer reproduce on the current unified binary. Google Antigravity — live-confirmed a gated MCP tool call denied headlessly in ~3s with a structured error, refuting the doc's prior (weakly-sourced) hang claim. Grok Build — live-confirmed a gated `use_tool` call in `-p` mode returns a clean `"stopReason":"cancelled"` result in seconds, exit 0.
- *Fails at a layer before MCP/permissions is ever reached*: Devin CLI (account login), Amp (account login), Cursor CLI (when unauthenticated — fails in under a second on `"Authentication required"`) and Auggie CLI (both OAuth-only, no BYOK path exists), Warp (mandatory account login even with a BYOK provider key already stored), Junie (same pattern as Warp — `--anthropic-api-key`/`--litellm-url` are accepted flags but a JetBrains platform `-a/--auth` token is still required; live-confirmed `"Cannot find authorization"` with a real provider key).
- *Resolves on its own, but slowly enough to look like a hang*: Cursor CLI (once authenticated) — a fresh, unapproved MCP tool call in headless `-p` mode isn't a true hang: live-confirmed it reliably self-resolves in 33-40 seconds across three separate runs, reasoning its way to "this was rejected" without any structured denial signal. That's 5-10x slower than every peer tool that fails clean (Kiro/Copilot/Antigravity all resolve in under 10s), which plausibly explains why third-party reports characterized this as "hanging."

**Native-tool disablement is sometimes architecturally absent from the shipped CLI**, even though the
SDK underneath supports it: OpenHands (`include_default_tools` exists in `openhands-sdk`, never wired
into the `openhands` CLI) and Cline (`toolPolicies` exists in `@cline/agents`, no CLI/config surface
exposes it) are both "coexistence only" in practice. Warp, Junie, QM, and Google Antigravity (binary-string
search of the shipped `agy` executable found zero matches for any native-tool-wide disablement flag) have
no such mechanism at any layer — MCP is architecturally additive-only for these four. **GitHub Copilot and
Grok Build are a distinct case**: the mechanism exists and is exposed, but leaks differently. Copilot's
allowlist (`--available-tools`, live-tested set to two MCP tools only, plus `--disable-builtin-mcps`) still
let its baseline `bash` shell tool execute a real command — the leak survives even the allowlist form there.
Grok Build's *denylist* (`--disallowed-tools`) similarly let `run_terminal_command` survive an explicit
exclusion, but its own allowlist form (`--tools`, live-tested naming only the MCP wrapper pair) produced a
genuinely exact MCP-only tool list with zero leaks — so for Grok Build specifically, prefer `--tools` over
`--disallowed-tools`. The lesson doesn't generalize cleanly across tools: verify the actual exposed schema
after applying either flag rather than assuming allowlist-implies-safe.

**MCP tool naming is not standardized** even among the tools that copy Claude Code's `mcp__server__tool`
shape (Codex CLI, jcode, DeepSeek Harness, Kimi Code, Devin CLI per docs). Others use a single underscore
(Gemini CLI, Crush, OpenCode, Kilo Code), replace hyphens with underscores in both segments
(Amp), use a triple underscore (Factory Droid), skip prefixing entirely and pass the raw MCP tool name
through (Continue CLI, Cursor CLI, OpenHands), or route everything through a generic wrapper tool instead
of exposing individually-named tools at all (Pi Agent's default `mcp` proxy tool, Auggie's
`find-tool`/`execute-tool` pair, Antigravity's `mcp(server/tool)` policy-pattern selector, and — live-confirmed,
correcting a prior naming-shape guess — Grok Build's `search_tool`/`use_tool` pair, where the qualified
`server__tool` name only ever appears as an argument value passed to `use_tool`, never as a model-facing
tool name in its own right). A
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
| Gemini CLI (historical, now deprecated) | https://github.com/google-gemini/gemini-cli/blob/main/docs/tools/mcp-server.md | 2026-08-18 | Trust field, naming, includeTools/excludeTools |
| Gemini CLI (historical, now deprecated) | https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/trusted-folders.md | 2026-08-18 | Folder-trust, safe-mode drops project settings |
| Gemini CLI (historical, now deprecated) | https://github.com/google-gemini/gemini-cli/issues/16567, /12362, /19776 | 2026-08-18 | Headless hangs on unapproved tool calls |
| Gemini CLI (deprecation) | Live-reproduced (real OAuth login → `IneligibleTierError`); https://developers.google.com/gemini-code-assist/docs/deprecations/code-assist-individuals; https://developers.googleblog.com/an-important-update-transitioning-gemini-cli-to-antigravity-cli/ | 2026-08-19 | Confirms Gemini CLI is retired for individual accounts as of 2026-06-18, replaced by Antigravity CLI |
| Google Antigravity | `agy@1.1.15` (official installer at antigravity.google/cli/install.sh, live install, real account, full agentic `--print` runs) | 2026-08-19 | Live-confirmed shared global config path/schema, live-reproduced a gated MCP tool call failing cleanly (not hanging) in headless mode, binary-string search confirming no native-tool-wide disablement flag exists |
| Kiro | `brew install --cask amazon-q` (Homebrew cask token `amazon-q` now resolves to `kiro-cli`, confirmed from the cask's own source: download URL is `desktop-release.q.us-east-1.amazonaws.com`, uninstall launchctl service is `com.amazon.codewhisperer.launcher`, and the shipped binary contains an explicit `fig_install::migrate` module migrating `~/.aws/amazonq/{agents,cli-agents,prompts,rules,settings/mcp.json,...}` into the new `.kiro` layout with a `migration.kiro.was_q_user` flag) | 2026-08-19 | Confirms Kiro CLI **is** Amazon Q Developer CLI, rebranded — not a separate product |
| Kiro | `kiro-cli@2.18.1` (live install, real login, full agentic `chat`/`mcp`/`agent` runs) | 2026-08-19 | Live-confirmed config paths (`.kiro/settings/mcp.json`, `~/.kiro/agents/*.json`), zero registration gate, `--no-interactive` fails clean (not hangs), full native-tool substitution via agent `tools` array, `--trust-tools="@server/tool"` addressing |
| GitHub Copilot | `@github/copilot@1.0.80` (npm, live install, real subscription, full agentic runs incl. structured JSON event stream) | 2026-08-19 | Live-confirmed folder-trust gate and its bypass env var, real internal tool naming (`<serverId>-<toolName>`, e.g. `weather-svc-get_forecast`, distinct from the `--allow-tool`/`--deny-tool` targeting syntax), `--available-tools` doesn't actually exclude `bash`, and #3064 reproduced live (`exitCode:0` despite a real MCP connection failure) |
| Amazon Q Dev CLI (now Kiro, see the Kiro rows above) | https://github.com/aws/amazon-q-developer-cli/issues/2984, /2510, /1951, /2195 | 2026-08-19 | "Legacy" config is default; allowedTools bug; --no-interactive hang bugs — no longer reproduce on the current unified Kiro CLI binary |
| Trae / Trae CN | https://docs.trae.cn/ide_add-mcp-servers | 2026-08-19 | Project MCP requires a manual global toggle |
| Junie | `@jetbrains/junie@2777.8.0` (npm install, live CLI + decompiled shipped JAR) | 2026-08-19 | Real config paths, `McpServerBean` schema (no trust field), no approval-gate class anywhere in `core.mcp`, no tool-disablement string anywhere in the app; live `--anthropic-api-key`/`--litellm-url` still blocked by `-a/--auth` |
| Junie | https://junie.jetbrains.com/docs/action-allowlist.html, /junie-cli-configuration.html | 2026-08-19 | All-or-nothing MCP trust; headless always-trusted (docs claim, corroborated by source absence of any gate) |
| Qwen Code | `@qwen-code/qwen-code@0.21.14` (npm, live install + `--debug` session logs) | 2026-08-19 | Scope-based pending-approval gate, `~/.qwen/mcpApprovals.json` hash-bound store, `--exclude-tools` config write, live yolo-vs-default gate behavior |
| Qwen Code | https://github.com/QwenLM/qwen-code/issues/6131 (closed, fix in PR #6177) | 2026-08-19 | `--yolo` doesn't cover new-MCP-server-startup confirmation — confirmed fixed live in 0.21.14 |
| QM | https://github.com/yc-software/qm (main branch, live-run via `scripts/dev-instance.sh` — local Docker, mock harness, no cloud account) | 2026-08-19 | Admin-only registration (REST-only, not even wired into the Admin web UI), additive-only tools, live approval-gate stall |
