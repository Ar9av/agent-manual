# GitHub Copilot CLI / VS Code Copilot Chat

> GitHub's AI pair programmer with agent mode, hooks, and MCP integration.

**Vendor:** GitHub / Microsoft | **License:** Proprietary

## Links

- Hooks concepts (cloud agent): https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-hooks
- Hooks reference: https://docs.github.com/en/copilot/reference/hooks-reference
- Use hooks (CLI): https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/use-hooks
- Use hooks (cloud agent): https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/use-hooks
- Error handling hook (SDK): https://docs.github.com/en/copilot/how-tos/copilot-sdk/use-hooks/error-handling
- VS Code Copilot overview: https://code.visualstudio.com/docs/copilot/overview
- Agents overview: https://code.visualstudio.com/docs/copilot/agents/overview

---

## Installation

```sh
# GitHub CLI extension
gh extension install github/gh-copilot

# VS Code: install "GitHub Copilot" and "GitHub Copilot Chat" extensions
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `.github/hooks/*.json` | Project | Repository-level hook config (cloud agent reads only this) |
| `~/.copilot/hooks/` | Global | User-level hook configs |
| `.github/copilot/settings.json` | Project | Inline hooks + settings |
| `~/.copilot/settings.json` | Global | User settings |
| `.github/copilot-instructions.md` | Project | Repo-level natural-language instructions |
| `.github/mcp.json` | Project | MCP server configuration |

## Hooks

Copilot has a fully-documented official hook system ([docs](https://docs.github.com/en/copilot/reference/hooks-reference)) with **14 events**. Both camelCase (`preToolUse`) and PascalCase (`PreToolUse`) naming are supported; they differ in field naming conventions (see stdin schema below).

### Hook Events

| Event | PascalCase alias | When | Can Block? |
|-------|-----------------|------|-----------|
| `preToolUse` | `PreToolUse` | Before tool call — can allow, deny, or mutate args | ✅ |
| `postToolUse` | `PostToolUse` | After tool completes — can modify result or inject context | ❌ |
| `userPromptTransformed` | — | Fires after the runtime transforms a submitted prompt into its model-facing content — mutation only, camelCase-only | ❌ |
| `postToolUseFailure` | `PostToolUseFailure` | After tool fails | ❌ |
| `permissionRequest` | — | Before permission service runs — can approve/deny (CLI only) | ✅ |
| `agentStop` | `Stop` | When main agent finishes — can force another turn | ✅ |
| `subagentStop` | `SubagentStop` | When subagent finishes — can force another turn | ✅ |
| `subagentStart` | — | When a subagent starts | ❌ |
| `sessionStart` | `SessionStart` | Session start | ❌ |
| `sessionEnd` | `SessionEnd` | Session end | ❌ |
| `userPromptSubmitted` | `UserPromptSubmit` | After user input received | ❌ |
| `preCompact` | `PreCompact` | Before context compaction | ❌ |
| `errorOccurred` | `ErrorOccurred` | When an error occurs during agent execution | ❌ |
| `notification` | — | On notification delivery — never blocks; errors skipped | ❌ |

### Hook Types

Three types of hooks:

**1. Command** (shell script):
```json
{
  "type": "command",
  "bash": "path/to/script.sh",
  "powershell": "path/to/script.ps1",
  "cwd": "working/directory",
  "env": { "VAR": "value" },
  "timeoutSec": 30
}
```

**2. HTTP** (webhook):
```json
{
  "type": "http",
  "url": "https://hooks.example.com",
  "headers": { "X-Custom": "value" },
  "allowedEnvVars": [],
  "timeoutSec": 30
}
```
Requires HTTPS (except localhost; set `COPILOT_HOOK_ALLOW_LOCALHOST=1` to enable `http://localhost`).

**3. Prompt** (LLM instruction, CLI-only, fires on `sessionStart` for new interactive sessions only — does not fire on resume or in cloud agent):
```json
{
  "type": "prompt",
  "prompt": "Natural language instruction or /slash-command"
}
```

### Config File Format

```json
{
  "version": 1,
  "disableAllHooks": false,
  "hooks": {
    "preToolUse": [
      {
        "matcher": "run_in_terminal",
        "hooks": [
          { "type": "command", "bash": "./hooks/validate.sh", "timeoutSec": 30 }
        ]
      }
    ],
    "postToolUse": [
      {
        "hooks": [{ "type": "command", "bash": "./hooks/audit.sh" }]
      }
    ]
  }
}
```

`matcher` is an anchored regex on `toolName`. Empty string matches all tools.

### Exit Code Behavior (command hooks)

| Code | Meaning |
|------|---------|
| `0` | Success; stdout parsed as JSON output |
| `2` | Warning logged; for `permissionRequest` treated as deny; for `preToolUse` treated as warning only (tool call proceeds — NOT a deny) |
| Other non-zero | For `preToolUse`: tool call is denied (fail-closed). For all other events: failure logged, execution continues (fail-open) |

### stdin Schema

Two naming conventions are supported; the chosen convention controls both the key used in the `hooks` config object and the field names in the JSON payload:

**camelCase form** — fields use camelCase; `timestamp` is a Unix millisecond integer:
```json
{
  "sessionId": "abc123",
  "timestamp": 1748300000000,
  "cwd": "/workspace",
  "toolName": "run_in_terminal",
  "toolArgs": { "command": "npm test" }
}
```

**PascalCase form** — fields use snake_case; `timestamp` is an ISO 8601 string:
```json
{
  "session_id": "abc123",
  "timestamp": "2025-05-26T20:00:00.000Z",
  "cwd": "/workspace",
  "tool_name": "run_in_terminal",
  "tool_input": { "command": "npm test" }
}
```

Note: `sessionId` / `session_id` is present in all events. Additional fields vary by event (see `errorOccurred`, `agentStop`, etc. below).

### stdout Schema

`preToolUse` (blocking — output fields only):
```json
{
  "permissionDecision": "allow | deny | ask",
  "permissionDecisionReason": "Required when denying",
  "modifiedArgs": {}
}
```

`agentStop` / `subagentStop`:
```json
{ "decision": "block | allow", "reason": "Used as prompt for next turn" }
```

`postToolUse`:
```json
{
  "modifiedResult": { "resultType": "success", "textResultForLlm": "..." },
  "additionalContext": "Capped at 10 KB"
}
```

`permissionRequest`:
```json
{
  "behavior": "allow | deny",
  "message": "string",
  "interrupt": false
}
```

`errorOccurred`: notification-only — no output is processed (Output processed: No). Do not return structured fields; the hook is for side-effects only (logging, alerting, etc.).

`notification`: can return `{ "additionalContext": "string" }` (injected as user message); otherwise never blocks.

`sessionStart`: can inject `additionalContext` via `prompt`-type hooks only.

### Cloud Agent Constraints

- Linux only (`bash` honored, `powershell` ignored)
- Working directory: `/workspace` (cloned repo)
- Filesystem is ephemeral — use HTTP hooks to persist data
- `permissionRequest` does not fire (tools are pre-approved)
- `prompt`-type hooks do not fire (cloud agent is non-interactive)
- Default timeout: 30 seconds per hook
- Outbound network restricted to GitHub/Copilot hosts by default

## Copilot Instructions

```markdown
# .github/copilot-instructions.md

- Always write TypeScript, not JavaScript
- Use conventional commits format
- Run `npm test` before finishing any task
```

## Built-in Agent Tools (VS Code)

| Tool | Description |
|------|-------------|
| `run_in_terminal` | Execute terminal commands |
| `read_file` | Read file contents |
| `create_file` / `insert_edit_into_file` | Write/edit files |
| `list_directory` | List files |
| `get_errors` | Get diagnostics from editor |
| `run_tests` | Run test suite |
| `web_search` | Search the web |
| MCP tools | From configured MCP servers |

## MCP Support

```json
// .github/mcp.json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["./mcp/server.js"]
    }
  }
}
```

## Background Agents (Cloud Agent)

Cloud agent sessions run autonomously. Hooks load only from `.github/hooks/*.json` on the default branch.

## Notes

- `disableAllHooks: true` disables all hooks without deleting config files.
- `ask` decisions are treated as `deny` in non-interactive (cloud) mode.
- `.github/copilot-instructions.md` is the standard location for project-level natural-language instructions.

## Sources

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| Hooks concepts (cloud agent) | https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-hooks | 2026-06-13 | [official] |
| Hooks reference | https://docs.github.com/en/copilot/reference/hooks-reference | 2026-07-23 | [official] |
| Use hooks (CLI) | https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/use-hooks | 2026-07-23 | [official] |
| Use hooks (cloud agent) | https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/use-hooks | 2026-06-13 | [official] |
| Error handling hook (SDK) | https://docs.github.com/en/copilot/how-tos/copilot-sdk/use-hooks/error-handling | 2026-06-13 | [official] |
| VS Code Copilot overview | https://code.visualstudio.com/docs/copilot/overview | — | [official] |
| Agents overview | https://code.visualstudio.com/docs/copilot/agents/overview | — | [official] |
