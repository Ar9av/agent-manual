# Cursor

> AI-native code editor with agent mode, rules, hooks, and MCP integration.

**Vendor:** Anysphere | **License:** Proprietary | **Runtime:** Electron

## Links

- Docs: https://cursor.com/docs
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
globs: src/backend/**/*.py
alwaysApply: false
---

# Python Backend Standards

- Use type hints on all public functions
- Follow PEP 8
- Write pytest tests for all new functions
```

Globs are **bare strings**, not JSON arrays. Multiple patterns use comma separation:

```mdc
---
globs: docs/**/*.md, docs/**/*.mdx
---
```

Place rule files in `.cursor/rules/` as `.mdc` files — plain `.md` files in that directory are ignored. Each file is scoped by its frontmatter.

### Rule Types

| Type | Trigger | Required frontmatter |
|------|---------|---------------------|
| Always Apply | Every chat session | `alwaysApply: true` |
| Apply Intelligently | Agent deems relevant | `description` (no globs) |
| Apply to Specific Files | Matching file opened | `globs` with `alwaysApply: false` |
| Apply Manually | `@`-mention in chat | none of the above |

Additional rule locations: **User Rules** (global Cursor settings), **Team Rules** (dashboard-managed, Team/Enterprise), **AGENTS.md** (plain markdown alternative in project root or subdirectories).

## Hooks

Cursor has **21 hook events** across two config files (`.cursor/hooks.json` project, `~/.cursor/hooks.json` global). All hooks receive JSON via stdin. ❓ A prior audit noted live page may state 23 events — live fetch on 2026-06-13 returned 21 (18 agent + 2 Tab + 1 App Lifecycle).

**Env vars available to all hooks:** `CURSOR_PROJECT_DIR`, `CURSOR_VERSION`, `CURSOR_USER_EMAIL`, `CURSOR_TRANSCRIPT_PATH`, `CURSOR_CODE_REMOTE`, `CLAUDE_PROJECT_DIR` (alias for `CURSOR_PROJECT_DIR`)

**Exit codes:** `0` = allow | `2` = block (stderr → LLM) | other = proceed (or block if `failClosed: true`)

### All Hook Events

| Event | Category | When | Can Block? |
|-------|----------|------|-----------|
| `sessionStart` | Agent | New composer conversation | ❌ |
| `sessionEnd` | Agent | Conversation ends | ❌ |
| `preToolUse` | Agent | Before any tool | ✅ |
| `postToolUse` | Agent | After tool succeeds | ❌ |
| `postToolUseFailure` | Agent | Tool fails/times out/denied | ❌ |
| `subagentStart` | Agent | Before spawning subagent | ✅ |
| `subagentStop` | Agent | Subagent completes/errors | ❌ |
| `beforeShellExecution` | Agent | Before shell command | ✅ |
| `afterShellExecution` | Agent | After shell command | ❌ |
| `beforeMCPExecution` | Agent | Before MCP tool | ✅ |
| `afterMCPExecution` | Agent | After MCP tool | ❌ |
| `beforeReadFile` | Agent | Before agent reads a file | ✅ |
| `afterFileEdit` | Agent | After agent edits a file | ❌ |
| `beforeSubmitPrompt` | Agent | Before user prompt submitted | ✅ |
| `preCompact` | Agent | Before context compaction | ❌ |
| `afterAgentResponse` | Agent | After agent completes message | ❌ |
| `afterAgentThought` | Agent | After thinking block | ❌ |
| `stop` | Agent | Agent loop ends | ❌ |
| `beforeTabFileRead` | Tab | Before Tab feature reads a file | ✅ |
| `afterTabFileEdit` | Tab | After Tab feature edits a file | ❌ |
| `workspaceOpen` | App Lifecycle | Workspace opens/folder changes | ❌ |

`stop` and `subagentStop` support returning `followup_message` in stdout to continue the loop.

### Key stdin fields by event

Base fields present on all events: `conversation_id`, `generation_id`, `model`, `hook_event_name`, `cursor_version`, `workspace_roots`, `user_email`, `transcript_path`

- `preToolUse`: `tool_name`, `tool_input`, `tool_use_id`, `cwd`, `model`, `agent_message`
- `postToolUse`: `tool_name`, `tool_input`, `tool_output`, `tool_use_id`, `cwd`, `duration`
- `postToolUseFailure`: `tool_name`, `tool_input`, `tool_use_id`, `cwd`, `error_message`, `failure_type`, `duration`, `is_interrupt`
- `beforeShellExecution`: `command`, `cwd`, `sandbox`
- `afterShellExecution`: `command`, `output`, `duration`, `sandbox`
- `beforeMCPExecution`: `tool_name`, `tool_input`, `url` or `command`
- `afterMCPExecution`: `tool_name`, `tool_input`, `result_json`, `duration`
- `beforeReadFile`: `file_path`, `content`, `attachments`
- `afterFileEdit`: `file_path`, `edits` (array of `old_string`, `new_string`)
- `beforeSubmitPrompt`: `prompt`, `attachments`
- `sessionStart`: `session_id`, `is_background_agent`, `composer_mode`
- `sessionEnd`: `session_id`, `reason`, `duration_ms`, `is_background_agent`, `final_status`, `error_message`
- `subagentStart`: `subagent_id`, `subagent_type`, `task`, `parent_conversation_id`, `tool_call_id`, `subagent_model`, `is_parallel_worker`, `git_branch`
- `subagentStop`: `subagent_type`, `status`, `task`, `description`, `summary`, `duration_ms`, `message_count`, `tool_call_count`, `loop_count`, `modified_files`, `agent_transcript_path`
- `preCompact`: `trigger`, `context_usage_percent`, `context_tokens`, `context_window_size`, `message_count`, `messages_to_compact`, `is_first_compaction`
- `stop`: `status`, `loop_count`
- `afterAgentResponse`: `text`
- `afterAgentThought`: `text`, `duration_ms`
- `workspaceOpen`: `hook_event_name`, `cursor_version`, `workspace_roots`, `user_email` (no base session fields)

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
| `Run Shell Commands` | Execute terminal commands and monitor output |
| `Read Files` | Read file contents (supports image files: .png, .jpg, .gif, .webp, .svg) |
| `Edit Files` | Suggest and apply edits to files |
| `Search Files and Folders` | Locate files by name, explore directories, search for patterns |
| `Semantic Search` | Search codebase by meaning (not just exact matches) |
| `Web Search` | Generate queries and perform web searches |
| `Browser` | Control a browser: screenshots, navigation, element interaction |
| `Image Generation` | Generate images from text or reference images |
| `Ask Questions` | Request clarification while continuing other operations |
| `Fetch Rules` | Retrieve specific rules based on type and description |
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

- `.cursor/rules/` requires `.mdc` extension; plain `.md` files are ignored by the rules engine.
- Globs in MDC frontmatter are bare strings (not JSON arrays); multiple patterns are comma-separated.
- `failClosed: true` on a hook makes errors block rather than proceed.
- `stop` / `subagentStop` can return `followup_message` in stdout JSON to continue the agent loop.
- Cursor Settings > Tools & MCP provides a GUI for MCP server management.

## Sources

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| Rules (MDC format, globs) | https://cursor.com/docs/rules | 2026-06-13 | [official] |
| Rules (context path) | https://cursor.com/context/rules | 2026-06-13 | [official] |
| Hooks (all events, stdin fields, env vars) | https://cursor.com/docs/hooks | 2026-06-13 | [official] |
| Agent tools list | https://cursor.com/docs/agent/tools | 2026-06-13 | [official] |
| MCP setup | https://cursor.com/docs/mcp | — | [official] |
| Agent best practices | https://cursor.com/blog/agent-best-practices | — | [official] |
| Changelog | https://cursor.com/changelog | — | [official] |
| Main docs | https://cursor.com/docs | — | [official] |
