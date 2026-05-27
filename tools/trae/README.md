# Trae IDE

> ByteDance's AI-first development tool with MCP and rules-based configuration.

**Vendor:** ByteDance | **License:** Proprietary | **Runtime:** Electron

## Links

- Docs: https://docs.trae.ai
- Agent guide: https://docs.trae.ai/ide/agent
- Rules: https://docs.trae.ai/ide/rules
- IDE settings overview: https://docs.trae.ai/ide/ide-settings-overview
- Models: https://docs.trae.ai/ide/models
- Changelog: https://docs.trae.ai/ide/changelog
- GitHub (Trae Agent): https://github.com/bytedance/trae-agent

---

## Installation

Download from https://www.trae.ai — available for Mac and Windows.

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `project_rules.md` | Project | Project-level agent rules |
| `user_rules.md` | Global | User-level agent rules |
| `.trae/mcp.json` | Project | MCP server configuration |
| IDE Settings UI | Global | All other settings |

## Rules (`.rules` / Rules files)

Rules are Markdown text files that define persistent instructions for the agent. Loaded during agent initialization phase, referenced during code completion and generation.

```markdown
# project_rules.md

## Code Style
- Use TypeScript strict mode
- Prefer functional components in React
- All async functions must have error handling

## Testing
- Write Jest tests for all new modules
- Maintain >80% coverage
```

Rules are structured using Markdown and are human-readable. They function as long-term contextual memory for the agent.

## Hooks

> **Note:** Trae does not have a native pre/post tool use hook system as of May 2026. Lifecycle control is achieved through:
> 1. **MCP servers** — expose custom tools and side effects
> 2. **Rules files** — instruct the agent on what to do/avoid

### MCP-Based Hook Patterns

Use MCP server tools to implement hook-like behavior:

```json
// .trae/mcp.json
{
  "mcpServers": {
    "safety-guard": {
      "command": "node",
      "args": ["./mcp/safety-guard.js"]
    }
  }
}
```

The MCP server can implement a `validate_command` tool that the agent calls before running shell commands (by instruction in rules).

## Built-in Agent Tools

| Tool | Description |
|------|-------------|
| Terminal | Execute shell commands |
| File read/write | Read and edit files |
| Web search | Search the web |
| MCP tools | From configured MCP servers |

## MCP Support (v1.3.0+)

Trae IDE v1.3.0 introduced full MCP protocol support.

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "..." }
    }
  }
}
```

MCP connects external data sources and tools: databases, APIs, GitHub, Jira, etc.

## Agent Configuration

Custom agents are configured via the IDE UI with:
- Custom name and system prompt
- Model selection
- Available tools

## Supported Models (2026)

- Claude 3.5 Sonnet / Claude Opus 4
- GPT-4o
- (More via API key configuration)

## Trae CN

See [trae-cn/](../trae-cn/README.md) — the Chinese-market version of Trae with the same architecture but different default models and regional endpoints.

## Notes

- `.rules` is loaded at agent initialization; updates require restarting the agent session.
- Rules use Markdown format, readable by both humans and AI.
- No native hook API; MCP is the primary extensibility path.

## Sources (Official)

| Topic | URL |
|-------|-----|
| Agent guide | https://docs.trae.ai/ide/agent |
| Rules | https://docs.trae.ai/ide/rules |
| IDE settings overview | https://docs.trae.ai/ide/ide-settings-overview |
| Models | https://docs.trae.ai/ide/models |
| Changelog | https://docs.trae.ai/ide/changelog |
| Trae Agent (GitHub) | https://github.com/bytedance/trae-agent |
| Main docs | https://docs.trae.ai |
