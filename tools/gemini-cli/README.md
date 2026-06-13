# Gemini CLI

> Google's open-source AI agent for the terminal, powered by Gemini models.

**Vendor:** Google | **License:** Apache 2.0 | **Runtime:** Node.js

## Links

- Docs: https://geminicli.com/docs
- Hooks overview: https://geminicli.com/docs/hooks/
- Hooks reference: https://geminicli.com/docs/hooks/reference/
- Writing hooks: https://geminicli.com/docs/hooks/writing-hooks/
- Configuration reference: https://geminicli.com/docs/reference/configuration/
- GitHub: https://github.com/google-gemini/gemini-cli
- Google blog: https://developers.googleblog.com/tailor-gemini-cli-to-your-workflow-with-hooks/

---

## Installation

```sh
# npm (global)
npm install -g @google/gemini-cli

# npx (zero-install)
npx @google/gemini-cli

# Homebrew (macOS/Linux)
brew install gemini-cli

# MacPorts (macOS)
sudo port install gemini-cli

# Conda (isolated environment)
conda create -y -n gemini_env -c conda-forge nodejs
conda activate gemini_env
npm install -g @google/gemini-cli
```

After installation:
```sh
gemini
```

## Configuration Files

Configuration is merged from multiple layers. Precedence order (highest wins):

| Priority | Layer | Path (Linux) | Path (macOS) | Notes |
|----------|-------|--------------|--------------|-------|
| 7 | CLI flags | — | — | Runtime arguments |
| 6 | Environment variables | `.env` files / shell env | `.env` files / shell env | Includes `.gemini/.env` |
| 5 | System settings | `/etc/gemini-cli/settings.json` | `/Library/Application Support/GeminiCli/settings.json` | Admin overrides |
| 4 | Project settings | `.gemini/settings.json` | `.gemini/settings.json` | Project-specific |
| 3 | User settings | `~/.gemini/settings.json` | `~/.gemini/settings.json` | Per-user global |
| 2 | System defaults | `/etc/gemini-cli/system-defaults.json` | `/Library/Application Support/GeminiCli/system-defaults.json` | Base system-wide |
| 1 | Built-in defaults | — | — | Hardcoded application defaults |

Extensions (installed plugins) also contribute hooks and settings, merged before project settings.

String values in settings files support environment variable substitution: `$VAR`, `${VAR}`, `${VAR:-default}`.

## Hooks

Hooks are scripts executed synchronously at defined lifecycle points in the agentic loop. They can inject context, block actions, modify model requests, and filter tool results.

> **Security:** Hooks execute arbitrary code with your user privileges. Project-level hooks are fingerprinted; if a hook's name or command changes (e.g., via `git pull`), it is treated as a new untrusted hook and triggers a warning before execution.

### Supported Events

| Event | When | Can Block? |
|-------|------|-----------|
| `SessionStart` | Session initialization | ❌ advisory only |
| `SessionEnd` | Session teardown | ❌ advisory only |
| `BeforeAgent` | Pre-planning phase, before each agent loop turn | ✅ |
| `AfterAgent` | After agent loop completion | ✅ (retry or halt) |
| `BeforeModel` | Before LLM request is sent | ✅ |
| `AfterModel` | After LLM response is received | ✅ |
| `BeforeToolSelection` | Before the model selects tools | ❌ advisory only (no decision/continue/systemMessage support) |
| `BeforeTool` | Before a tool call executes | ✅ |
| `AfterTool` | After a tool call completes | ✅ (`decision: deny` hides result) |
| `PreCompress` | Before context compression | ❌ advisory only |
| `Notification` | System alert events | ❌ advisory only |

**AfterAgent details:** `decision: deny` triggers automatic retry using `reason` as feedback; `continue: false` halts the agent loop entirely. AfterAgent also accepts `hookSpecificOutput.clearContext: true` to clear LLM conversation history while preserving UI display. The `stop_hook_active` input field (boolean) indicates whether a retry is already in progress.

### Configuration Schema

Hooks are configured in `settings.json` using a nested structure with a mandatory `type` field:

```json
{
  "hooks": {
    "BeforeTool": [
      {
        "matcher": "run_shell_command",
        "hooks": [
          {
            "name": "validate-shell-cmd",
            "type": "command",
            "command": "~/.gemini/hooks/validate-cmd.sh",
            "timeout": 60000
          }
        ]
      }
    ],
    "AfterTool": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "name": "audit-log",
            "type": "command",
            "command": "~/.gemini/hooks/audit-log.sh"
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "hooks": [
          {
            "name": "inject-context",
            "type": "command",
            "command": "~/.gemini/hooks/inject-context.sh"
          }
        ]
      }
    ]
  }
}
```

**Hook definition fields:**

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `type` | string | **Yes** | Currently only `"command"` is supported |
| `command` | string | Yes | Shell command or script path |
| `name` | string | No | Identifier used in logs |
| `timeout` | number | No | Milliseconds; default `60000` |
| `description` | string | No | Human-readable purpose |
| `matcher` | string | No | Regex (tool events) or exact string (lifecycle events); `"*"` or omit to match all |
| `sequential` | boolean | No | `true` runs hooks sequentially; default is parallel |

### Hook Input (stdin JSON)

All hooks receive a base JSON payload on stdin:

```json
{
  "hook_event_name": "BeforeTool",
  "session_id": "abc123",
  "transcript_path": "/tmp/gemini-session-abc123.json",
  "cwd": "/home/user/project",
  "timestamp": "2026-06-13T10:00:00Z"
}
```

**Event-specific additional fields:**

| Event | Extra stdin fields |
|-------|-------------------|
| `BeforeTool` | `tool_name`, `tool_input`, `mcp_context`, `original_request_name` |
| `AfterTool` | `tool_name`, `tool_input`, `tool_response` (object: `{llmContent, returnDisplay, optional error}`), `mcp_context`, `original_request_name` |
| `BeforeAgent` | `prompt` (string) |
| `AfterAgent` | `prompt` (string), `prompt_response` (string), `stop_hook_active` (boolean) |
| `BeforeModel` | `llm_request` (full request object) |
| `AfterModel` | `llm_request`, `llm_response` |

> **Note:** `tool_name` and `tool_input` are event-specific (BeforeTool/AfterTool), not universal fields. The universal key is `hook_event_name` (not `event`).

### Hook Output (stdout JSON)

Hooks must write **only valid JSON** to stdout. Log messages must go to stderr.

```json
{
  "decision": "allow",
  "reason": "optional explanation",
  "systemMessage": "text shown in terminal",
  "suppressOutput": false,
  "continue": true,
  "stopReason": "shown when stopping",
  "hookSpecificOutput": {
    "hookEventName": "BeforeToolSelection",
    "toolConfig": {
      "mode": "ANY",
      "allowedFunctionNames": ["read_file", "list_directory"]
    }
  }
}
```

### Exit Code Behavior

| Code | Label | Behavior |
|------|-------|----------|
| `0` | Success | stdout parsed as JSON; preferred for all logic including blocking |
| `2` | System Block | Action aborted; stderr content used as rejection reason |
| Other non-zero | Warning | Non-fatal; CLI continues with original parameters |

**Canonical blocking pattern:** Exit `0` and output `{"decision": "deny", "reason": "..."}`. This is the idiomatic method for production hooks.

**Emergency brake:** Exit `2` for simple security gates or when the hook script itself errors. Stderr becomes the displayed reason.

**AfterTool blocking:** `decision: deny` hides the tool output from the model and replaces it with the `reason` string.

### Best Practices

- Write only valid JSON to stdout; use stderr for all debug/log output.
- Keep hooks fast — they run synchronously and block execution.
- Use specific `matcher` patterns to limit execution scope.
- Hooks run with your user privileges; validate all external inputs carefully.
- Use exit `0` + `decision: deny` for production blocking logic; reserve exit `2` for emergency/simple gates.

## Built-in Tools

| Tool | Description |
|------|-------------|
| `run_shell_command` | Execute shell commands |
| `read_file` | Read file contents |
| `write_file` | Write files |
| `edit_file` | Apply edits |
| `list_directory` | List directory |
| `glob` | File pattern matching |
| `grep` | Search file contents |
| `web_fetch` | Fetch URLs |
| `web_search` | Web search (via Google) |

## MCP Support

- Configure MCP servers in `settings.json` under `mcpServers`
- Extensions can also contribute hooks via `settings.json`

## Extensions

Extensions can provide additional tools and hook configurations. Hooks from installed extensions are merged into the configuration stack (between user settings and project settings in precedence).

## Notes

- Tool context injection (git commits, tickets, docs) is a primary hook use case.
- Dynamic tool filtering via `BeforeToolSelection` hooks allows narrowing available tools per-session using `toolConfig.allowedFunctionNames`.
- Environment variables `GEMINI_PROJECT_DIR`, `GEMINI_PLANS_DIR`, `GEMINI_SESSION_ID`, `GEMINI_CWD` are available inside hook processes.

## Sources

| Topic | URL | Label |
|-------|-----|-------|
| Hooks overview | https://geminicli.com/docs/hooks/ | [official] |
| Hooks reference | https://geminicli.com/docs/hooks/reference/ | [official] |
| Writing hooks | https://geminicli.com/docs/hooks/writing-hooks/ | [official] |
| Configuration reference | https://geminicli.com/docs/reference/configuration/ | [official] |
| Hooks index (GitHub) | https://github.com/google-gemini/gemini-cli/blob/main/docs/hooks/index.md | [github] |
| Hooks reference (GitHub) | https://github.com/google-gemini/gemini-cli/blob/main/docs/hooks/reference.md | [github] |
| Writing hooks (GitHub) | https://github.com/google-gemini/gemini-cli/blob/main/docs/hooks/writing-hooks.md | [github] |
| Tools reference | https://geminicli.com/docs/tools/file-system/ | [official] |
| Google blog (hooks intro) | https://developers.googleblog.com/tailor-gemini-cli-to-your-workflow-with-hooks/ | [official] |
| Main docs | https://geminicli.com/docs/ | [official] (Google-hosted; geminicli.com is the canonical Google documentation site, not a third-party mirror) |
| GitHub repo | https://github.com/google-gemini/gemini-cli | [github] |
