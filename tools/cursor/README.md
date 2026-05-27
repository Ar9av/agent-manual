# Cursor

> AI-native code editor with agent mode, rules, hooks, and MCP integration.

**Vendor:** Anysphere | **License:** Proprietary | **Runtime:** Electron

## Links

- Docs: https://cursor.com/en-US/docs
- Agent best practices: https://cursor.com/blog/agent-best-practices
- Changelog: https://cursor.com/changelog
- MCP setup guide: https://www.truefoundry.com/blog/mcp-servers-in-cursor-setup-configuration-and-security-guide

---

## Installation

Download from https://cursor.com — available for Mac, Windows, Linux.

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.cursor/mcp.json` | Global | Global MCP server config |
| `.cursor/mcp.json` | Project | Project MCP servers |
| `.cursor/rules/` | Project | Scoped rule files (MDC format) |
| `.cursorrules` | Project | Legacy single-file rules |
| `~/.cursor/settings.json` | Global | Editor + agent settings |

## Rules

Rules provide persistent context and coding standards to the agent.

### Rule Format (MDC — Markdown-based)

```mdc
---
description: Python backend rules
globs: ["src/backend/**/*.py"]
---

# Python Backend Standards

- Use type hints on all public functions
- Follow PEP 8
- Write pytest tests for all new functions
```

Place rule files in `.cursor/rules/` — each file is scoped to the glob patterns in its frontmatter.

## Hooks

Cursor has **21 hook events** across two config files (`.cursor/hooks.json` project, `~/.cursor/hooks.json` global). All hooks receive JSON via stdin.

**Env vars available to all hooks:** `CURSOR_PROJECT_DIR`, `CURSOR_VERSION`, `CURSOR_USER_EMAIL`, `CURSOR_TRANSCRIPT_PATH`, `CURSOR_CODE_REMOTE`

**Exit codes:** `0` = allow | `2` = block (stderr → LLM) | other = proceed (or block if `failClosed: true`)

### All Hook Events

| Event | When | Can Block? |
|-------|------|-----------|
| `sessionStart` | New composer conversation | ❌ |
| `sessionEnd` | Conversation ends | ❌ |
| `preToolUse` | Before any tool | ✅ |
| `postToolUse` | After tool succeeds | ❌ |
| `postToolUseFailure` | Tool fails/times out/denied | ❌ |
| `subagentStart` | Before spawning subagent | ✅ |
| `subagentStop` | Subagent completes/errors | ❌ |
| `beforeShellExecution` | Before shell command | ✅ |
| `afterShellExecution` | After shell command | ❌ |
| `beforeMCPExecution` | Before MCP tool | ✅ |
| `afterMCPExecution` | After MCP tool | ❌ |
| `beforeReadFile` | Before agent reads a file | ✅ |
| `afterFileEdit` | After agent edits a file | ❌ |
| `beforeTabFileRead` | Before Tab feature reads a file | ✅ |
| `afterTabFileEdit` | After Tab feature edits a file | ❌ |
| `beforeSubmitPrompt` | Before user prompt submitted | ✅ |
| `preCompact` | Before context compaction | ❌ |
| `afterAgentResponse` | After agent completes message | ❌ |
| `afterAgentThought` | After thinking block | ❌ |
| `stop` | Agent loop ends | ❌ |
| `workspaceOpen` | Workspace opens/folder changes | ❌ |

`stop` and `subagentStop` support returning `followup_message` in stdout to continue the loop.

### Key stdin fields by event

- `preToolUse`: `tool_name`, `tool_input`, `tool_use_id`, `cwd`, `model`
- `beforeShellExecution`: `command`, `cwd`, `sandbox`
- `beforeMCPExecution`: `tool_name`, `tool_input`, `url` or `command`
- `beforeReadFile`: `file_path`, `content`, `attachments`
- `beforeSubmitPrompt`: `prompt`, `attachments`
- `sessionStart`: `session_id`, `is_background_agent`, `composer_mode`

### Configuration Example

```json
{
  "event": "preToolUse",
  "command": ".cursor/hooks/validate.sh",
  "type": "command",
  "matcher": "Run shell commands",
  "timeout": 5000,
  "failClosed": true
}
```

`failClosed: true` — if the hook errors, block instead of proceeding.

## Built-in Agent Tools

| Tool | Description |
|------|-------------|
| `Bash` / Terminal | Execute commands |
| `Read file` | Read file contents |
| `Edit file` | Apply edits |
| `List dir` | List directory |
| `Search` | Codebase search |
| `Web search` | Search the web |
| MCP tools | From configured MCP servers |

## MCP Support

Cursor loads MCP servers from two locations:

```json
// .cursor/mcp.json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["./mcp-server/index.js"]
    }
  }
}
```

- Go to **Settings > Tools & MCP** to toggle individual tools on/off.
- Agent scans enabled servers and loads their tools automatically.

## Agent Mode

- Enable via Composer (`Cmd+I`) → switch to Agent mode.
- Background agents can run autonomously via Copilot CLI integration.
- `enabledPlugins` in settings controls which plugins are active.

## Notes

- `.cursor/rules/` supports multiple files; `.cursorrules` is the legacy single-file format (ignored in Agent mode).
- `failClosed: true` on a hook makes errors block rather than proceed.
- `stop` / `subagentStop` can return `followup_message` in stdout JSON to continue the agent loop.
- Cursor Settings > Tools & MCP provides a GUI for MCP server management.

## Sources (Official)

| Topic | URL |
|-------|-----|
| Agent tools list | https://cursor.com/docs/agent/tools |
| Hooks | https://cursor.com/docs/hooks |
| Third-party hooks reference | https://cursor.com/docs/reference/third-party-hooks |
| Rules (MDC format) | https://cursor.com/docs/rules |
| MCP setup | https://cursor.com/docs/mcp |
| Agent best practices | https://cursor.com/blog/agent-best-practices |
| Changelog | https://cursor.com/changelog |
| Main docs | https://cursor.com/en-US/docs |
