# OpenCode

> Open-source AI coding agent with agents, skills, commands, and plugins.

**Vendor:** OpenCode.ai | **License:** MIT | **Runtime:** Go / Node.js

## Links

- Docs: https://opencode.ai/docs
- Config: https://opencode.ai/docs/config/
- Agents: https://opencode.ai/docs/agents/
- GitHub: https://github.com/sst/opencode
- Awesome OpenCode: https://github.com/awesome-opencode/awesome-opencode

---

## Installation

```sh
curl -fsSL https://opencode.ai/install | bash
# or
npm install -g opencode-ai
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.config/opencode/opencode.json` | Global | User settings |
| `opencode.json` | Project | Project settings |
| `.opencode/plugins/` | Project | Local plugins |
| `~/.config/opencode/plugins/` | Global | Global plugins |

## Hooks / Extensibility

OpenCode uses a **plugin system** as its primary extension mechanism. Native hook events are provided via plugins rather than a built-in hook config key.

### Plugin Hook Events

Plugins can register callbacks for:

| Event | When |
|-------|------|
| `before_tool` | Before tool execution |
| `after_tool` | After tool execution |
| `on_message` | On every LLM message |
| `on_session_start` | Session start |

### Plugin Configuration

```json
{
  "plugin": ["npm:opencode-plugin-logger", "./plugins/my-plugin.js"]
}
```

Or place plugins directly in `.opencode/plugins/`.

### Plugin Structure

```js
// .opencode/plugins/my-plugin.js
export default {
  hooks: {
    before_tool: async ({ tool_name, tool_input }) => {
      if (tool_name === 'bash' && tool_input.command.includes('rm -rf')) {
        return { block: true, message: 'Blocked destructive command' }
      }
    },
    after_tool: async ({ tool_name, result }) => {
      console.error(`[audit] ${tool_name} completed`)
    }
  }
}
```

## Built-in Tools

| Tool | Description |
|------|-------------|
| `bash` | Execute shell commands |
| `read` | Read file contents |
| `write` | Write files |
| `edit` | Apply edits |
| `glob` | File pattern matching |
| `grep` | Search files |
| `ls` | List directory |
| `fetch` | Fetch URLs |

## MCP Support

Configure MCP servers in `opencode.json`:

```json
{
  "mcp": {
    "servers": {
      "my-server": {
        "type": "local",
        "command": ["node", "./mcp-server/index.js"]
      }
    }
  }
}
```

## Agents

Define specialized agents with custom prompts, models, and tool access:

```json
{
  "agents": {
    "code-reviewer": {
      "model": "claude-opus-4",
      "system": "You are a strict code reviewer...",
      "tools": ["read", "grep", "glob"]
    }
  }
}
```

### Permissions

```json
{
  "permissions": {
    "bash": "ask",
    "write": "ask",
    "read": "allow"
  }
}
```

## Skills & Commands

- Skills: Reusable expertise modules in `.opencode/skills/`
- Commands: Custom slash commands in `.opencode/commands/`

## Notes

- OpenCode does not have a native `PreToolUse`/`PostToolUse` hook config key — all lifecycle control goes through plugins.
- The plugin format supports both local JS files and npm-published packages.

## Sources (Official)

| Topic | URL |
|-------|-----|
| Config | https://opencode.ai/docs/config/ |
| Agents | https://opencode.ai/docs/agents/ |
| GitHub (opencode) | https://opencode.ai/docs/github/ |
| GitHub repo | https://github.com/sst/opencode |
| Awesome OpenCode | https://github.com/awesome-opencode/awesome-opencode |
