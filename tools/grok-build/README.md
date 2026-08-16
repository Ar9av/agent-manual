# Grok Build

> xAI's terminal-based coding agent (harness + fullscreen TUI) for Grok models; open-sourced (source-available, non-community-governed) on July 15, 2026.

**Vendor:** xAI (SpaceXAI) | **License:** Apache License 2.0 (first-party code; third-party code retains original licenses, per repo `NOTICE`/docs) | **Runtime:** Rust (native binary; ~1M-line Rust workspace), cross-platform (macOS, Linux, Windows best-effort)

## Links

- Docs: https://docs.x.ai/build/overview
- GitHub: https://github.com/xai-org/grok-build
- Announcement (product launch): https://x.ai/news/grok-build-cli
- Announcement (open source): https://x.ai/news/grok-build-open-source
- Open source landing page: https://x.ai/open-source
- Changelog: https://x.ai/build/changelog (exists — page title confirmed via search as "Grok Build Changelog" — but returned HTTP 403 from Cloudflare bot-protection on every fetch attempt during this pass, both via WebFetch and direct curl; content not independently verified)

---

## Installation

```sh
# macOS / Linux / Git Bash
curl -fsSL https://x.ai/cli/install.sh | bash

# Windows PowerShell
irm https://x.ai/cli/install.ps1 | iex

# verify
grok --version
```

**From source** (per repo README):
- Requirements: Rust (pinned via `rust-toolchain.toml`), the DotSlash CLI, `protoc`
- Run: `cargo run -p xai-grok-pager-bin`
- Release build: `cargo build -p xai-grok-pager-bin --release` → binary at `target/release/xai-grok-pager`

First run requires browser authentication unless the `XAI_API_KEY` environment variable is set. Grok Build is also usable local-first: compile it yourself and point it at your own local inference via `config.toml`.

---

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.grok/config.toml` (Unix) / `%USERPROFILE%\.grok\config.toml` (Windows) | Global (user) | Model/API config, MCP servers, UI prefs, skills/plugins paths |
| `.grok/config.toml` | Project | Project-shared settings — per docs, **limited to** MCP servers, plugins, and permission rules (not full user config) |
| `~/.grok/managed_config.toml` / `/etc/grok/managed_config.toml` | Enterprise (managed) | Admin-served defaults |
| `~/.grok/requirements.toml` / `/etc/grok/requirements.toml` | Policy | Highest-precedence policy pins |
| `$GROK_HOME` | Env override | Overrides the default `~/.grok` root |
| `~/.grok/mcp_credentials.json` | Global | Stored OAuth tokens for MCP servers |
| `~/.grok/plugins/known_marketplaces.json` | Global | Registered plugin marketplaces |

**Precedence (per docs, high → low):** Requirements → Managed → Project → User → Environment appears to layer on top for `GROK_*` vars overriding others; exact merge order beyond this summary is ❓ (not spelled out in full in the fetched settings page).

Compatibility: Grok Build also reads `.mcp.json`, `.cursor/mcp.json`, and `~/.claude.json` for MCP server definitions, and automatically reads Claude Code– and AGENTS.md–style instruction files without extra configuration.

Run `grok inspect` to see all configuration sources, instructions, skills, plugins, hooks, and MCP servers discovered for the current directory.

---

## Instruction File

The agent reads `AGENTS.md` (project root, discovered automatically). Per the official Project Rules docs (`docs.x.ai/build/features/project-rules`), Grok Build reads any of `AGENTS.md`, `Agents.md`, `AGENT.md`, `CLAUDE.md`, `Claude.md`, and `CLAUDE.local.md` in each directory, plus every `*.md` file under a `.grok/rules/` directory — and for compatibility also reads `.claude/rules/` and `.cursor/rules/`. Files ignored by `.gitignore` are skipped. This confirms the previously-flagged ❓: Claude Code's `CLAUDE.md` (and variants) are read natively, no conversion needed.

---

## Hooks

Hooks run scripts (or webhooks) on tool- and session-lifecycle events. Managed in the TUI via `/hooks`. Discovered from `~/.grok/hooks/*.json`, project `<project>/.grok/hooks/*.json`, and from enabled plugins; for compatibility, Grok Build also reads `.claude/settings.json` and `.cursor/hooks.json` hook definitions.

### Supported Events

| Event | When it fires | Can Block? |
|-------|---------------|-----------|
| `SessionStart` | A session starts | ❌ |
| `SessionEnd` | A session ends | ❌ |
| `UserPromptSubmit` | User submits a prompt | ❌ |
| `PreToolUse` | A tool is about to run | ✅ (exit 2 denies) |
| `PostToolUse` | Tool call completes successfully | ❌ |
| `PostToolUseFailure` | Tool call fails | ❌ |
| `PermissionDenied` | Permission system denies a tool call | ❌ |
| `Stop` | A turn ends | ❌ |
| `StopFailure` | Turn ends due to API error | ❌ |
| `Notification` | Agent sends a notification | ❌ |
| `SubagentStart` | A subagent begins | ❌ |
| `SubagentStop` | A subagent finishes | ❌ |
| `PreCompact` | Context compaction starts | ❌ |
| `PostCompact` | Context compaction completes | ❌ |

This is a shorter, flatter event list than some competitors. Per the official Hooks docs, the `type` field supports exactly two values: `"command"` (run a local script) or `"http"` (POST the event JSON to a `url`). No `mcp_tool`/`prompt`/`agent` hook types are documented.

### Hook Input (stdin JSON)

```json
{
  "hookEventName": "PreToolUse",
  "sessionId": "...",
  "cwd": "...",
  "workspaceRoot": "...",
  "toolName": "Bash",
  "toolInput": { "command": "..." }
}
```
(`toolName` / `toolInput` present only for tool-related events.)

### Hook Output (stdout JSON)

Only meaningful for `PreToolUse`; other ("passive") events have their stdout ignored:

```json
{ "decision": "deny", "reason": "Unsafe command detected" }
```

### Exit Code Behavior

| Code | Meaning |
|------|---------|
| `0` | Allow (`PreToolUse`); success (passive events) |
| `2` | Deny (`PreToolUse` only) |
| Other / timeout / crash | Fail-open — tool call proceeds, failure recorded in session |

### Example Config

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{ "type": "command", "command": "bin/safety-check.sh", "timeout": 10 }]
      }
    ]
  }
}
```

---

## Built-in Tools

Official docs do not publish a single enumerated built-in-tools table (unlike Claude Code's docs). Confirmed categories, per the GitHub repo and docs:

| Tool category | Description |
|------|-------------|
| Terminal / shell execution | Run shell commands |
| File edit | Read/write/patch files |
| Search | Code/file search operations |
| Workspace ops | Filesystem, VCS (git) integration, checkpoints |

Exact tool names (e.g. whether they are literally called `Read`/`Edit`/`Bash` as in Claude Code) are ❓ — not confirmed from an official source during this pass.

---

## MCP Support

Config lives in `~/.grok/config.toml` (global) or `.grok/config.toml` (project), with compatibility fallback to `.mcp.json`, `.cursor/mcp.json`, and `~/.claude.json`. MCP tools are namespaced as `<server>__<tool>` alongside built-ins.

```toml
[mcp_servers.filesystem]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"]
env = { API_KEY = "${MY_API_KEY}" }
startup_timeout_sec = 30
tool_timeout_sec = 6000

[mcp_servers.linear]
url = "https://mcp.linear.app/mcp"
headers = { "x-mcp-session-id" = "{{session_id}}" }
```

- `${VAR}` / `${VAR:-default}` env-var expansion supported in `url`, `command`, `args`, `env`, `headers`.
- OAuth-requiring servers trigger a browser flow on first use; tokens stored at `~/.grok/mcp_credentials.json`.
- CLI: `grok mcp add`, `grok mcp list`, `grok mcp doctor`; `grok inspect` shows loaded servers and their origins.
- Managed from the TUI extensions modal via `/mcps`.

---

## Skills / Commands

- Skills location: `./.grok/skills/` (walked up to repo root), `~/.grok/skills/`, enabled-plugin `skills/` directories, plus custom paths via `[skills] paths` in `~/.grok/config.toml`.
- Format: a `SKILL.md` file with YAML frontmatter — fields: `name`, `description`, `when-to-use`, `paths` (gitignore-style globs for file visibility), `allowed-tools` (declarative allowlist), `argument-hint`, `user-invocable` (default `true`), `disable-model-invocation` (default `false`), `metadata` (free-form, e.g. `author`, `short-description`).
- User-invocable skills surface as slash commands (`/<skill-name>`).
- Managed from the TUI via `/skills`.

Plugins bundle skills, agents, hooks, MCP servers, and language-server integrations. Loaded from `./.grok/plugins/`, `~/.grok/plugins/`, marketplace installs at `~/.grok/plugins/marketplaces/`, custom paths via `[plugins] paths`, or `--plugin-dir <PATH>` on the CLI. Marketplaces are configured via `[[marketplace.sources]]` in config and tracked in `~/.grok/plugins/known_marketplaces.json`. Managed via `/plugins` and `/marketplace`.

---

## Agent / Subagent Configuration

- Custom subagents: `.grok/agents/` (project) or `~/.grok/agents/` (user).
- "Personas": `[subagents.personas]` in config, or `.grok/personas/*.toml` / `~/.grok/personas/*.toml`.
- Built-in subagent types:
  - `general-purpose` — default full-capability child
  - `explore` — read/list/search only, no shell, no edits
  - `plan` — drafts an implementation plan, no shell, no edits
- Subagents are independent child sessions with their own context window; they run in parallel and return a summary to the parent when finished. Enabled by default.
- Managed via `/config-agents` (alias `/agents`) or `/personas`.
- Worktree isolation for subagents is confirmed in xAI's product launch announcement: "Grok Build also supports deep worktree integrations, and you can launch subagents in their own worktrees." The official Subagents docs page (`docs.x.ai/build/features/subagents`) does not add further detail on the isolation mechanism itself — the exact implementation (e.g. git worktree creation/cleanup semantics) remains ❓.

---

## Notes

- The July 15, 2026 open-source release covers four components per xAI's own framing: the **agent loop** (context assembly, response parsing, tool-call dispatch), **tools** (read/edit/search/execute), the **terminal UI** (rendering, input, plan review, inline diffs), and the **extension system** (skills, plugins, hooks, MCP servers, subagents).
- This is **source-available transparency, not a community-governed OSS project**: per the repo's `CONTRIBUTING.md`, external contributions are not accepted, and press coverage independently reported GitHub issues being disabled. Treat it as "read the source to understand behavior," not "send a PR."
- Grok Build ships the same `grok` CLI that powers xAI's Grok 4.5 agentic coding stack (per xAI's open-source announcement); it was previously available in early beta to SuperGrok / X Premium Plus subscribers before this open-source release.
- Headless/scripting mode: `grok -p "<prompt>"`, with `--output-format streaming-json` for structured output. Full Agent Client Protocol (ACP) support exists for editor/bot embedding.
- Because this tool is very new (open-sourced roughly one month before this page was written) and official docs are sparse in places, several fields above are marked ❓ rather than guessed. Community docs (e.g. third-party cookbooks) exist but were **not** used as sourcing for this page per task instructions — only x.ai/xai-org official sources were used.

---

## Sources (Official)

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| Repo README (license, install-from-source, runtime, architecture) | https://github.com/xai-org/grok-build/blob/main/README.md | 2026-08-15 | [github] — re-verified, all claims still supported |
| Repo LICENSE file | https://github.com/xai-org/grok-build/blob/main/LICENSE | 2026-08-15 | [github] — re-verified, Apache-2.0 confirmed |
| Open-source announcement (scope: agent loop/tools/UI/extensions, contribution policy) | https://x.ai/news/grok-build-open-source | 2026-08-15 | [official] — re-verified, all claims still supported |
| Product launch announcement (features, plan mode, subagents, ACP) | https://x.ai/news/grok-build-cli | 2026-08-15 | [official] — re-verified; also surfaced the exact worktree-isolation quote used in the Subagents section |
| Open source landing page | https://x.ai/open-source | 2026-08-15 | [official] — **could not be re-verified this pass**: returned HTTP 403 from Cloudflare bot-protection on every attempt (WebFetch and direct curl with a browser UA both blocked); page is presumed live but content unconfirmed |
| Docs overview (install, config.toml path, `grok inspect`, headless mode) | https://docs.x.ai/build/overview | 2026-08-15 | [official] — re-verified, all claims still supported |
| Docs: Skills, Plugins & Marketplaces | https://docs.x.ai/build/features/skills-plugins-marketplaces | 2026-08-15 | [official] — re-verified, all claims still supported |
| Docs: Subagents | https://docs.x.ai/build/features/subagents | 2026-08-15 | [official] — re-verified; built-in types and paths confirmed, worktree-isolation mechanism still undetailed |
| Docs: Modes and Commands (slash commands, plan mode, auto mode) | https://docs.x.ai/build/modes-and-commands | 2026-08-15 | [official] — re-verified, all claims still supported |
| Docs: MCP Servers | https://docs.x.ai/build/features/mcp-servers | 2026-08-15 | [official] — re-verified, all claims still supported |
| Docs: Hooks | https://docs.x.ai/build/features/hooks | 2026-08-15 | [official] — re-verified; confirmed `http` is a second documented hook `type` alongside `command` |
| Docs: Settings | https://docs.x.ai/build/settings | 2026-08-15 | [official] — re-verified, all claims still supported |
| Docs: Project Rules (instruction file discovery: AGENTS.md/CLAUDE.md variants, `.grok/rules/`) | https://docs.x.ai/build/features/project-rules | 2026-08-15 | [official] |
| Changelog (existence only — not content) | https://x.ai/build/changelog | 2026-08-15 | [official] — HTTP 403 from Cloudflare bot-protection on every fetch attempt this pass; page appears to exist (indexed with title "Grok Build Changelog") but content unconfirmed |
