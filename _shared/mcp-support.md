# MCP (Model Context Protocol) Support

Which tools support MCP and how to configure it.

## Support Matrix

| Tool | MCP Support | Config Location | Notes |
|------|-------------|----------------|-------|
| Claude Code | ✅ Full | `.claude/mcp.json` | First-class support |
| Codex CLI | ✅ Full | `config.toml` → `[mcp]` | |
| Gemini CLI | ✅ Full | `settings.json` → `mcpServers` | Extensions can contribute servers |
| Kiro | ✅ Full | `.kiro/config.yaml` → `mcpServers` | |
| Kimi Code | ✅ Full | `config.json` → `mcpServers` | |
| Factory Droid | ✅ Full | `settings.json` → `mcpServers` | |
| OpenCode | ✅ Full | `opencode.json` → `mcp.servers` | |
| Devin CLI | ✅ Full | `config.json` → `mcpServers` | |
| Cursor | ✅ Full | `.cursor/mcp.json` | GUI toggle in Settings > Tools & MCP |
| Hermes | ✅ Full | `cli-config.yaml` → `mcp_servers` | |
| GitHub Copilot | ✅ Full | `.github/mcp.json` | Shared with VS Code |
| Pi Agent | ✅ Full | `settings.json` → `mcp` | |
| OpenClaw | ✅ Full | `config.yaml` → `mcpServers` | |
| Trae | ✅ Full (v1.3.0+) | `.trae/mcp.json` | Added in v1.3.0 |
| Trae CN | ✅ Full (v1.3.0+) | `.trae/mcp.json` | Same as Trae |
| Google Antigravity | ✅ Full | `~/.gemini/config/mcp_config.json` | Shared with Gemini CLI, remote requires `serverUrl` |
| Aider | ❌ None | — | Not supported |

## Standard MCP Config Format

Most tools use a compatible JSON format:

```json
{
  "mcpServers": {
    "server-name": {
      "command": "node",
      "args": ["./path/to/server.js"],
      "env": {
        "API_KEY": "..."
      }
    }
  }
}
```

### Variations by Tool

**Codex CLI (TOML):**
```toml
[mcp.servers.my-server]
command = "node"
args = ["./mcp/server.js"]
```

**OpenCode (nested):**
```json
{
  "mcp": {
    "servers": {
      "my-server": {
        "type": "local",
        "command": ["node", "./mcp/server.js"]
      }
    }
  }
}
```

**Google Antigravity (remote HTTP):**
```json
{
  "mcpServers": {
    "my-remote-server": {
      "serverUrl": "https://mcp.example.com",
      "headers": {
        "Authorization": "Bearer ..."
      }
    }
  }
}
```
Remote connections strictly require `serverUrl` (not `url` or `httpUrl`).

## Popular MCP Servers

| Server | Purpose | Package |
|--------|---------|---------|
| GitHub | Repo access, PR/issue management | `@modelcontextprotocol/server-github` |
| Filesystem | Extended file access | `@modelcontextprotocol/server-filesystem` |
| Postgres | Database queries | `@modelcontextprotocol/server-postgres` |
| Slack | Slack messaging | `@modelcontextprotocol/server-slack` |
| Brave Search | Web search | `@modelcontextprotocol/server-brave-search` |
| Puppeteer | Browser automation | `@modelcontextprotocol/server-puppeteer` |

## Tool Discovery

When an MCP server is connected, the agent automatically discovers its tools. Tools appear in the agent's tool list alongside built-in tools.

In Cursor, use **Settings > Tools & MCP** to see all loaded servers and toggle individual tools on/off.

## Sources (Official)

| Tool | MCP docs URL |
|------|-------------|
| Claude Code | https://docs.anthropic.com/en/docs/claude-code/mcp |
| Codex CLI | https://developers.openai.com/codex/mcp |
| Gemini CLI | https://github.com/google-gemini/gemini-cli#mcp |
| Cursor | https://cursor.com/docs/mcp |
| Kiro | https://kiro.dev/docs/mcp |
| Hermes | https://hermes-agent.nousresearch.com/docs/mcp |
| Trae / Trae CN | https://docs.trae.ai/ide/mcp |
| Pi Agent | https://github.com/earendil-works/pi |
| GitHub Copilot | https://code.visualstudio.com/docs/copilot/overview |
| OpenCode | https://opencode.ai/docs/config/ |
