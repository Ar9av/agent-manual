# Claude Code

> Anthropic's official AI coding agent for the terminal and IDE.

**Vendor:** Anthropic | **License:** Proprietary | **Runtime:** Node.js

## Links

- Docs: https://code.claude.com/docs
- Hooks reference: https://code.claude.com/docs/en/hooks
- GitHub (community): https://github.com/anthropics/claude-code
- Changelog: https://code.claude.com/docs/en/changelog

---

## Installation

```sh
npm install -g @anthropic-ai/claude-code
claude
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.claude/settings.json` | Global | User-level settings, hooks, permissions |
| `.claude/settings.json` | Project | Project-level settings |
| `.claude/settings.local.json` | Project (git-ignored) | Personal project overrides |
| `CLAUDE.md` | Project | Natural-language instructions loaded into context |

## Hooks

Hook events are defined in `settings.json` under the `hooks` key.

### Supported Events

| Event | When | Can Block? |
|-------|------|-----------|
| `PreToolUse` | Before every tool call | ✅ (exit 2) |
| `PostToolUse` | After every tool call | ❌ |
| `Notification` | When agent sends a notification | ❌ |
| `Stop` | When the agent finishes a turn | ❌ |
| `SubagentStop` | When a subagent finishes | ❌ |
| `PreCompact` | Before context compaction | ❌ |

### Hook Types

| Type | Description |
|------|-------------|
| `command` | Run a shell command |
| `http` | HTTP POST to a URL |
| `prompt` | Single-turn LLM evaluation |
| `agent` | Spawn a subagent with Read/Grep/Glob tools |

### Hook Input (stdin JSON)

```json
{
  "session_id": "abc123",
  "transcript_path": "/tmp/transcript.json",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": { "command": "rm -rf /tmp/test" }
}
```

### Exit Code Behavior

| Code | Meaning |
|------|---------|
| `0` | Success, continue |
| `2` | Block tool (PreToolUse only); stderr sent to Claude |
| Other | Warning shown to user, execution continues |

### Configuration Example

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": "~/.claude/hooks/validate-bash.sh" }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          { "type": "command", "command": "~/.claude/hooks/format.sh", "async": true }
        ]
      }
    ]
  }
}
```

### Async Hooks

Add `"async": true` to run a hook in the background without blocking Claude's execution (released January 2026).

## Built-in Tools

| Tool | Description |
|------|-------------|
| `Bash` | Execute shell commands |
| `Read` | Read file contents |
| `Write` | Write/overwrite files |
| `Edit` | Apply targeted string replacements |
| `MultiEdit` | Multiple edits in one call |
| `Glob` | File pattern matching |
| `Grep` | Search file contents |
| `LS` | List directory contents |
| `TodoRead` / `TodoWrite` | Manage in-session task lists |
| `Agent` | Spawn subagents |
| `WebFetch` | Fetch a URL |
| `WebSearch` | Web search |
| `NotebookRead` / `NotebookEdit` | Jupyter notebook support |

## MCP Support

- Config: `.claude/mcp.json` or `~/.claude/mcp.json`
- Enable servers in settings under `mcpServers`
- Servers expose additional tools available in the agent loop

## Agent / Subagent Configuration

Claude Code supports spawning subagents via the `Agent` tool and via `agent`-type hooks. Subagents have access to Read, Grep, and Glob tools by default.

## Permissions

```json
{
  "permissions": {
    "allow": ["Bash(git *)", "Read(**)"],
    "deny": ["Bash(rm -rf *)"]
  }
}
```

## Skills (Slash Commands)

Custom slash commands live in `.claude/commands/` as Markdown files. They can reference `$ARGUMENTS` and embed tool calls.

## Notes

- `CLAUDE.md` files are loaded recursively from the repo root.
- `disableAllHooks: true` in settings disables all hooks for debugging.
- Hook `agent` type has a 60s default timeout; `prompt` type has 30s.

## Sources (Official)

| Topic | URL |
|-------|-----|
| Hooks reference | https://code.claude.com/docs/en/hooks |
| Main docs | https://code.claude.com/docs |
| Skills / slash commands | https://code.claude.com/docs/en/skills |
| MCP configuration | https://code.claude.com/docs/en/mcp |
| Settings reference | https://code.claude.com/docs/en/settings |
