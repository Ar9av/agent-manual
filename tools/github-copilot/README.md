# GitHub Copilot CLI / VS Code Copilot Chat

> GitHub's AI pair programmer with agent mode, hooks, and MCP integration.

**Vendor:** GitHub / Microsoft | **License:** Proprietary

## Links

- Hooks concepts: https://docs.github.com/en/copilot/concepts/agents/hooks
- Hooks reference: https://docs.github.com/en/copilot/reference/hooks-configuration
- Use hooks (cloud agent): https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/use-hooks
- Pre-tool use hook (SDK): https://docs.github.com/en/copilot/how-tos/copilot-sdk/use-hooks/pre-tool-use
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

Copilot has a fully-documented official hook system ([docs](https://docs.github.com/en/copilot/concepts/agents/hooks)) with 12 events. Both camelCase (`preToolUse`) and PascalCase (`PreToolUse`) naming are supported.

### Hook Events

| Event | When | Can Block? |
|-------|------|-----------|
| `preToolUse` | Before tool call — can allow, deny, or mutate args | ✅ |
| `postToolUse` | After tool completes — can modify result | Partial |
| `postToolUseFailure` | After tool fails | ❌ |
| `permissionRequest` | Before permission service runs — can approve/deny | ✅ |
| `agentStop` | When main agent finishes — can force another turn | ✅ |
| `subagentStop` | When subagent finishes — can force another turn | ✅ |
| `subagentStart` | When a subagent starts | ❌ |
| `sessionStart` | Session start — can inject `additionalContext` | ❌ |
| `sessionEnd` | Session end | ❌ |
| `userPromptSubmitted` | After user input received | ❌ |
| `preCompact` | Before context compaction | ❌ |
| `notification` | On notification delivery — never blocks; errors skipped | ❌ |

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
  "timeoutSec": 30
}
```
Requires HTTPS (except localhost).

**3. Prompt** (LLM instruction, CLI-only):
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
| `2` | Warning logged; for `permissionRequest` treated as deny |
| Other | Failure logged; execution continues (fail-open) |

### stdin Schema (camelCase form)

```json
{
  "sessionId": "abc123",
  "timestamp": 1748300000,
  "cwd": "/workspace",
  "toolName": "run_in_terminal",
  "toolArgs": { "command": "npm test" }
}
```

PascalCase form uses `hook_event_name`, `session_id`, `tool_name`, `tool_input` (ISO 8601 timestamp).

### stdout Schema

`preToolUse`:
```json
{
  "permissionDecision": "allow | deny | ask",
  "permissionDecisionReason": "Required when denying",
  "modifiedArgs": {},
  "additionalContext": "Injected into conversation",
  "suppressOutput": false
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

### Cloud Agent Constraints

- Linux only (`bash` honored, `powershell` ignored)
- Working directory: `/workspace` (cloned repo)
- Filesystem is ephemeral — use HTTP hooks to persist data
- `permissionRequest` does not fire (tools are pre-approved)
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

## Sources (Official)

| Topic | URL |
|-------|-----|
| Hooks concepts | https://docs.github.com/en/copilot/concepts/agents/hooks |
| Hooks reference | https://docs.github.com/en/copilot/reference/hooks-configuration |
| Use hooks (cloud agent) | https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/use-hooks |
| Pre-tool use hook (SDK) | https://docs.github.com/en/copilot/how-tos/copilot-sdk/use-hooks/pre-tool-use |
| VS Code Copilot overview | https://code.visualstudio.com/docs/copilot/overview |
| Agents overview | https://code.visualstudio.com/docs/copilot/agents/overview |
