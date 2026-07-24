# Qwen Code

> Alibaba's open-source AI coding agent for the terminal — originally forked from Google's Gemini CLI, now supporting OpenAI/Anthropic/Gemini/Qwen model backends.

**Vendor:** Alibaba (QwenLM team) | **License:** Apache 2.0 | **Runtime:** Node.js (>= 22)

> ❓ **Fork lineage:** Qwen Code began as a fork of [Gemini CLI](../gemini-cli/README.md) — it shares Gemini CLI's original TUI architecture and project-context (`@file`) system. The GitHub API no longer reports the repo as a fork (it was detached at some point), and the current README states Qwen Code has since been re-engineered for feature parity with **Claude Code** rather than Gemini CLI (SubAgents, Agent Teams, Hooks, Auto-Memory/Auto-Skills, Plan Mode, Skills, daemon mode). Its hooks system in particular is modeled closely on Claude Code's hooks (event names, JSON schema, and even some copy-pasted doc language — one line in the official hooks doc still literally says "When **Claude** prepares to conclude response") rather than on Gemini CLI's `Before*/After*` hook naming. Treat Qwen Code today as a multi-lineage descendant (Gemini CLI chassis + Claude-Code-shaped extensibility layer), not a pure Gemini CLI clone.

## Links

- Docs: https://qwenlm.github.io/qwen-code-docs/en/users/overview
- Hooks: https://qwenlm.github.io/qwen-code-docs/en/users/features/hooks/
- MCP: https://qwenlm.github.io/qwen-code-docs/en/users/features/mcp/
- Settings reference: https://qwenlm.github.io/qwen-code-docs/en/users/configuration/settings/
- GitHub: https://github.com/QwenLM/qwen-code
- GitHub hooks source: https://github.com/QwenLM/qwen-code/blob/main/docs/users/features/hooks.md
- npm package: https://www.npmjs.com/package/@qwen-code/qwen-code

---

## Installation

```sh
# Linux / macOS (standalone installer)
curl -fsSL https://qwen-code-assets.oss-cn-hangzhou.aliyuncs.com/installation/install-qwen-standalone.sh | bash

# Windows (PowerShell, standalone installer)
irm https://qwen-code-assets.oss-cn-hangzhou.aliyuncs.com/installation/install-qwen-standalone.ps1 | iex

# npm (requires Node.js 22+)
npm install -g @qwen-code/qwen-code@latest

# Homebrew (macOS/Linux)
brew install qwen-code
```

After installation:
```sh
qwen          # launch interactive terminal UI
/auth         # configure your provider and API key (inside the session)
```

Other entry points: `qwen -p "..."` (headless mode for scripts/CI), `qwen serve` (experimental daemon mode exposing a shared agent session over HTTP+SSE via ACP), `qwen channel` (IM bot mode for Telegram/DingTalk/WeChat/Feishu). ❓ IDE plugins (VS Code, Zed, JetBrains) and a desktop app are also distributed but not verified in depth here.

## Configuration Files

Hierarchical settings, precedence lowest → highest:

| Priority | Layer | Path | Notes |
|----------|-------|------|-------|
| 1 | Built-in defaults | — | Hardcoded application defaults |
| 2 | System defaults | ❓ platform-specific | Base system-wide config |
| 3 | User settings | `~/.qwen/settings.json` | Applies globally to current user |
| 4 | Project settings | `.qwen/settings.json` | Project root, overrides user settings |
| 5 | System settings | ❓ platform-specific | Admin overrides |
| 6 | Environment variables | shell env / `.env` files | Includes `<QWEN_HOME>/.env` (defaults to `~/.qwen/.env`) |
| 7 | CLI flags | — | Runtime arguments, highest precedence |

`settings.json` is organized into top-level categories: `general`, `model` (plus `fastModel`/`visionModel`/`voiceModel`), `context`, `tools`, `permissions`, `mcp`/`mcpServers`, `memory`, `telemetry`, `hooks`.

Key environment variables: `QWEN_HOME` (overrides global config directory, default `~/.qwen`), `QWEN_SANDBOX` (`docker`, `podman`, or custom), `QWEN_CODE_MAX_OUTPUT_TOKENS`, `QWEN_PROJECT_DIR` (available inside hook processes).

Ignore files: `.qwenignore` (primary), plus compatibility fallbacks `.agentignore` / `.aiignore`, configurable via `context.fileFiltering.customIgnoreFiles`.

## Instruction File

The agent reads project/user context (memory) from Markdown context files, default name **`QWEN.md`**, configurable via the `context.fileName` setting (string or array, e.g. `["CONTEXT.md", "QWEN.md"]`).

Hierarchical loading order (later/more-specific entries supplement or override earlier/more-general ones):
1. **Global context file** — `~/.qwen/QWEN.md` (or whatever `context.fileName` resolves to) — default instructions for all projects.
2. **Project root & ancestor directories** — CLI searches the cwd and each parent directory up to the project root (identified by a `.git` folder) or the home directory.

All found files are concatenated with separators indicating origin/path and injected into the system prompt; the CLI footer shows a count of loaded context files. Run `/memory refresh` to reload, and `context.loadFromIncludeDirectories: true` to also load `QWEN.md` from `context.includeDirectories`. ❓ Unlike Devin CLI, no explicit statement was found that Qwen Code also reads `AGENTS.md`/`CLAUDE.md` for cross-tool compatibility — `QWEN.md` (or the configured `context.fileName`) appears to be the sole convention.

## Hooks

Qwen Code has a hooks system that is **not inherited from Gemini CLI's `Before*/After*` naming** — it uses Claude-Code-style event names (`PreToolUse`, `PostToolUse`, `Stop`, etc.) and a richer set of executor types than either Gemini CLI or Claude Code documents publicly. Hooks are **enabled by default**; set `"disableAllHooks": true` at the top level of a settings file (alongside `"hooks"`) to disable all configured hooks without deleting their config.

> **Live-tested 2026-07-23 on qwen-code v0.20.1 with `gpt-4o-mini` — partial confirmation, with a genuinely confusing result worth documenting.** A `PreToolUse` hook with no matcher (matches every tool) **did fire correctly for `read_file` and `todo_write`** tool calls — confirmed via the hook script's own debug-log side effect appearing with the exact documented stdin schema (`hook_event_name`, `tool_name`, `tool_input`, etc.). However, **no `PreToolUse` event was ever logged for the shell-execution tool** (`run_shell_command`), even though the model's final reply claimed the shell command "was successfully run." Inspecting the raw session transcript found **no actual `run_shell_command` tool call anywhere in it** — the model appears to have fabricated a plausible "command succeeded" response without genuinely invoking the shell tool. (Earlier, separately, with a matcher explicitly targeting `run_shell_command`, the model instead fabricated the *opposite* result — a plausible "blocked by your hook" explanation — after using `read_file` to inspect the hook config and script content itself, with **no hook firing occurring at all** despite the confident-sounding refusal.) Net result: hook firing for non-shell tools is confirmed working, but this test could not cleanly confirm or deny whether `PreToolUse` genuinely intercepts shell execution specifically — `gpt-4o-mini` via this integration proved unreliable enough (narrating outcomes instead of taking real actions) that the shell-tool path needs re-testing with a stronger model before trusting either a "works" or "broken" verdict for it.

### Hook Executor Types

| Type | Description |
|------|-------------|
| `command` | Executes a shell command; JSON in via stdin, JSON out via stdout. |
| `http` | POSTs the hook-input JSON to a URL; response body carries the decision. Includes SSRF protection (blocks private IP ranges, allows loopback), DNS-rebinding validation, and `${VAR}`-style env interpolation gated by an `allowedEnvVars` whitelist. |
| `function` | Direct JS/TS function call, session-level only — internal, used by the Skill system, not a public API. |
| `prompt` | Sends hook input to an LLM (default: current model) via a `$ARGUMENTS`-templated prompt; the LLM must return `{"ok": true/false, "reason": "...", "additionalContext": "..."}`. |

### Supported Events

| Event | When | Matcher Target | Can Block? |
|-------|------|-----------------|-----------|
| `PreToolUse` | Before tool execution | Tool id (regex) | ✅ via `hookSpecificOutput.permissionDecision` (`allow`/`deny`/`ask`) |
| `PostToolUse` | After successful tool execution | Tool id (regex) | ✅ (documented as blockable; exact effect is post-hoc result mutation) |
| `PostToolUseFailure` | After tool execution fails | Tool id (regex) | ❌ |
| `UserPromptSubmit` | After user submits a prompt | None | ✅ |
| `SessionStart` | Session starts/resumes | Source: `startup`/`resume`/`clear`/`compact` (regex) | ❌ |
| `SessionEnd` | Session ends | Reason: `clear`/`logout`/`prompt_input_exit`/etc. (regex) | ❌ |
| `MessageDisplay` | Repeatedly, as the reply streams | None | ❌ |
| `Stop` | Before the agent concludes its response | None | ✅ |
| `SubagentStart` | Subagent starts | Agent type (regex) | ❌ |
| `SubagentStop` | Subagent finishes | Agent type (regex) | ✅ |
| `StopFailure` | Turn ends via API error | Error type (regex) | ❌ (fire-and-forget; output/exit code ignored) |
| `PreCompact` | Before conversation compaction | Trigger: `manual`/`auto` (exact match) | ❌ |
| `PostCompact` | After compaction completes | Trigger (exact match) | ❌ (decision fields logged only) |
| `Notification` | System notification sent | Type: `permission_prompt`/`idle_prompt`/`auth_success` (exact match) | ❌ |
| `PermissionRequest` | Permission dialog shown | Tool id (regex) | ❌ |
| `PermissionDenied` | Tool permission denied | Tool id (regex) | ❌ |
| `TodoCreated` | New todo item created | None | ✅ (validation phase can block the write) |
| `TodoCompleted` | Todo item marked complete | None | ✅ |

Matcher syntax: `""` or `"*"` matches everything; standard regex for tool/subagent/session events (`^run_shell_command$`, `write_.*`, `(Bash|Explorer)`); exact-string match for `Notification`/`PreCompact`; no matcher support for `UserPromptSubmit`, `Stop`, `MessageDisplay`, `TodoCreated`, `TodoCompleted`. Tool hooks match against runtime tool ids (e.g. `write_file`); display names like `WriteFile` are accepted as compatibility aliases.

### Hook Input (stdin JSON, command/http hooks)

```json
{
  "session_id": "string",
  "transcript_path": "string",
  "cwd": "string",
  "hook_event_name": "PreToolUse",
  "timestamp": "string",
  "permission_mode": "default | plan | auto_edit | yolo",
  "tool_name": "write_file",
  "tool_input": { "...": "..." },
  "tool_use_id": "toolu_xxx",
  "tool_call_id": "call_xxx"
}
```

Event-specific fields are appended per event; when running inside a subagent, `agent_id` and `agent_type` are also included.

### Hook Output (stdout JSON / HTTP response body)

```json
{
  "continue": true,
  "decision": "allow",
  "reason": "Operation approved",
  "stopReason": "string",
  "suppressOutput": false,
  "systemMessage": "text shown in terminal",
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "Security policy blocks database writes",
    "additionalContext": "Current environment: production."
  }
}
```

For `PreToolUse`, `hookSpecificOutput.permissionDecision` (`allow`/`deny`/`ask`) and `permissionDecisionReason` are the documented control fields (required); `updatedInput` can rewrite the tool's parameters before execution. `"ask"` falls back to `"deny"` in headless/subagent contexts that cannot prompt the user.

### Exit Code Behavior (command hooks)

| Code | Meaning |
|------|---------|
| `0` | Success — stdout parsed as JSON to control behavior |
| `2` | Blocking error — stdout ignored, stderr passed to the model as feedback |
| Other | Non-blocking error — stderr shown only in debug mode; execution continues |

### Example Config

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "write_file",
        "hooks": [
          {
            "type": "command",
            "command": "$QWEN_PROJECT_DIR/.qwen/hooks/security-check.sh",
            "name": "security-check",
            "timeout": 10000
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Evaluate whether all tasks are complete: $ARGUMENTS. Respond with {\"ok\": true} or {\"ok\": false, \"reason\": \"...\"}.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

## Built-in Tools

| Tool | Description |
|------|-------------|
| File System tools | Read/write/list/search/modify files under a `rootDirectory`, generally confirmed for sensitive ops |
| `read_many_files` | Multi-file read across paths/directories |
| `run_shell_command` | Execute shell commands |
| `web_fetch` | Fetch a URL and process content with an AI model |
| `web_search` | Web search |
| `save_memory` | Save/recall information across sessions (Memory tool) |
| `todo_write` | Create and manage structured task lists |
| MCP-provided tools | Tools dynamically contributed by configured MCP servers |

❓ Exact tool ids beyond those named in docs (e.g. individual `read_file`/`write_file`/`edit_file` names referenced in hook matcher examples) are inferred from the hooks documentation's matcher examples, not from a single canonical tools table.

## MCP Support

**Yes.** Configured under `mcpServers` in `settings.json` (project scope: `.qwen/settings.json`; user scope: `~/.qwen/settings.json`). Supports three transports: stdio (local process), HTTP (streamable, recommended for remote/cloud servers), and SSE (legacy). Discovery is progressive — the UI is interactive immediately while MCP servers connect in the background (`QWEN_CODE_LEGACY_MCP_BLOCKING=1` restores the old synchronous-wait behavior). Supports `includeTools`/`excludeTools` filtering, per-server `discoveryTimeoutMs`, OAuth, and `@server:uri` resource references in messages.

```json
{
  "mcpServers": {
    "pythonTools": {
      "command": "python",
      "args": ["-m", "my_mcp_server", "--port", "8080"],
      "cwd": "./mcp-servers/python",
      "env": {
        "DATABASE_URL": "$DB_CONNECTION_STRING",
        "API_KEY": "${EXTERNAL_API_KEY}"
      },
      "timeout": 15000
    },
    "httpServerWithAuth": {
      "httpUrl": "http://localhost:3000/mcp",
      "headers": { "Authorization": "Bearer your-api-token" },
      "timeout": 5000
    },
    "sseServer": {
      "url": "http://localhost:8080/sse",
      "timeout": 30000
    },
    "oauthServer": {
      "url": "https://api.example.com/sse/",
      "oauth": {
        "enabled": true,
        "clientId": "your-client-id",
        "authorizationUrl": "https://provider.example.com/authorize",
        "tokenUrl": "https://provider.example.com/token",
        "redirectUri": "https://your-server.com/oauth/callback",
        "scopes": ["read", "write"]
      }
    }
  }
}
```

Connection method precedence when multiple are present: `httpUrl` → `url` → `command`.

## Skills / Commands

Built-in skills/commands are ready out of the box, including `/review`, `/batch`, `/loop`, `/bugfix`. ❓ File-based custom-skill layout (analogous to Devin's `.devin/skills/<name>/SKILL.md`) is documented in the "Skills" feature page but was not independently verified here — the repo does reference `.qwen/skills/<name>/SKILL.md` in its own README (e.g. `.qwen/skills/qwen-code-claw/SKILL.md`), suggesting that path convention.

## Agent / Subagent Configuration

Qwen Code advertises SubAgents and "Agent Teams" with dynamic workflows, plus an "Agent Arena" mode for running multiple models head-to-head on the same task — capabilities not present in Gemini CLI. ❓ Exact subagent config file format/location was not independently confirmed beyond the feature-list mention.

## Notes

- **Multi-protocol:** unlike Gemini CLI (Gemini-only) or Devin CLI (Anthropic-only), Qwen Code's `/auth` flow supports OpenAI, Anthropic, Gemini, and Qwen (DashScope) APIs plus local models (Ollama/vLLM) — model choice is switchable at runtime.
- **Beyond the terminal:** IDE plugins (VS Code, Zed, JetBrains), a desktop app, `qwen serve` daemon mode (experimental, shared agent session over HTTP+SSE via ACP), SDKs (TypeScript, Python alpha, Java alpha), and IM channel bots (Telegram, DingTalk, WeChat, Feishu, QQ, WeCom).
- **Sandboxing:** ships a sandbox image (`ghcr.io/qwenlm/qwen-code`) and `QWEN_SANDBOX` env var, echoing Gemini CLI's sandbox model.
- The GitHub repository is `QwenLM/qwen-code`; the current npm package is `@qwen-code/qwen-code` (older tutorials/gists may reference a prior/renamed package — not verified here).
- Node.js 22+ is required to run from npm/source.

## Sources

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| GitHub repo (README, install, capability table) | https://github.com/QwenLM/qwen-code | 2026-07-23 | [official] |
| GitHub repo metadata (license, fork status) | https://api.github.com/repos/QwenLM/qwen-code | 2026-07-23 | [official] |
| package.json (npm name, version, Node engine) | https://raw.githubusercontent.com/QwenLM/qwen-code/main/package.json | 2026-07-23 | [official] |
| LICENSE | https://raw.githubusercontent.com/QwenLM/qwen-code/main/LICENSE | 2026-07-23 | [official] |
| Hooks feature docs | https://qwenlm.github.io/qwen-code-docs/en/users/features/hooks/ | 2026-07-23 | [official] |
| Hooks doc (GitHub source, full event/type/exit-code detail) | https://github.com/QwenLM/qwen-code/blob/main/docs/users/features/hooks.md | 2026-07-23 | [official] |
| Settings/configuration reference | https://qwenlm.github.io/qwen-code-docs/en/users/configuration/settings/ | 2026-07-23 | [official] |
| Settings doc (GitHub source, context.fileName / precedence detail) | https://raw.githubusercontent.com/QwenLM/qwen-code/main/docs/users/configuration/settings.md | 2026-07-23 | [official] |
| MCP feature docs (transports, config examples) | https://qwenlm.github.io/qwen-code-docs/en/users/features/mcp/ | 2026-07-23 | [official] |
| Docs site overview / nav structure | https://qwenlm.github.io/qwen-code-docs/en/ | 2026-07-23 | [official] |
| Docs site overview page | https://qwenlm.github.io/qwen-code-docs/en/users/overview | 2026-07-23 | [official] |
| Tools docs search (file-system, shell, web-fetch, web-search, memory) | https://qwenlm.github.io/qwen-code-docs/en/developers/tools/ | 2026-07-23 | [official] |
| Background on fork lineage / community comparison | https://elite-ai-assisted-coding.dev/p/qwen-code-tool-review | 2026-07-23 | [community] |
| Background on fork lineage / feature comparison | https://www.infoworld.com/article/4054914/qwen-code-is-good-but-not-great.html | 2026-07-23 | [community] |
