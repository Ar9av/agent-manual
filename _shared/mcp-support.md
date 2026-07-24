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
| Amazon Q Developer CLI | ✅ Full | Per-agent `mcpServers`, or legacy `~/.aws/amazonq/mcp.json` | Two config layers; opt-in legacy path via `useLegacyMcpJson` |
| Amp | ✅ Full | `settings.json` → `amp.mcpServers` | Also `amp mcp add/list/approve` CLI; local + remote |
| Goose | ✅ Full | `config.yaml` → `extensions:` | Called "extensions"; 70+ available |
| OpenHands | ✅ Full | `~/.openhands/mcp.json` | FastMCP-standard schema; also `mcp_config` param in SDK |
| Crush | ✅ Full | `crush.json` → `mcp` | stdio/http/sse; OAuth supported |
| Continue CLI | ✅ Full | `.continue/mcpServers/*.yaml` or `*.json` | stdio/sse/streamable-http |
| Auggie CLI | ✅ Full | `settings.json` → `mcpServers`, or `--mcp-config` | Also `--mcp` (acts as MCP server) and `--acp` mode |
| Qwen Code | ✅ Full | `settings.json` → `mcpServers` | stdio/HTTP/SSE, OAuth support |
| Warp | ✅ Full | `.warp/.mcp.json` (project) / `~/.warp/.mcp.json` (global) | Also reads Claude Code/Codex-style MCP config for compat |

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

## Sources

| Tool | MCP docs URL | Fetched | Label |
|------|-------------|---------|-------|
| Claude Code | https://docs.anthropic.com/en/docs/claude-code/mcp | 2026-06-26 | [official] |
| Codex CLI | https://developers.openai.com/codex/mcp | 2026-06-26 | [official] |
| Gemini CLI | https://github.com/google-gemini/gemini-cli#mcp | 2026-06-26 | [github] |
| Cursor | https://cursor.com/docs/mcp | 2026-06-26 | [official] |
| Kiro | https://kiro.dev/docs/mcp | 2026-06-26 | [official] |
| Hermes | https://hermes-agent.nousresearch.com/docs/mcp | 2026-06-26 | [official] |
| Trae / Trae CN | https://docs.trae.ai/ide/mcp | 2026-06-26 | [official] |
| Pi Agent | https://github.com/earendil-works/pi | 2026-06-26 | [github] |
| GitHub Copilot | https://code.visualstudio.com/docs/copilot/overview | 2026-06-26 | [official] |
| OpenCode | https://opencode.ai/docs/config/ | 2026-06-26 | [official] |
| Amazon Q Developer CLI | https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-installing.html | 2026-07-23 | [official] |
| Amp | https://ampcode.com/manual | 2026-07-23 | [official] |
| Goose | https://goose-docs.ai/ | 2026-07-23 | [official] |
| OpenHands | https://docs.openhands.dev/ | 2026-07-23 | [official] |
| Crush | https://github.com/charmbracelet/crush | 2026-07-23 | [github] |
| Continue CLI | https://docs.continue.dev/cli/quickstart | 2026-07-23 | [official] |
| Auggie CLI | https://docs.augmentcode.com/cli/overview | 2026-07-23 | [official] |
| Qwen Code | https://qwenlm.github.io/qwen-code-docs/en/users/overview | 2026-07-23 | [official] |
| Warp | https://docs.warp.dev/agent-platform/ | 2026-07-23 | [official] |
