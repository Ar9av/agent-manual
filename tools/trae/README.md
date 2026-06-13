# Trae IDE

> ByteDance's AI-first development tool with MCP and rules-based configuration.

**Vendor:** ByteDance | **License:** Proprietary | **Runtime:** Electron

## Links

- Docs: https://docs.trae.ai
- Agent guide: https://docs.trae.ai/ide/agent
- Rules: https://docs.trae.ai/ide/rules
- IDE settings overview: https://docs.trae.ai/ide/ide-settings-overview
- Models: https://docs.trae.ai/ide/models
- MCP protocol: https://docs.trae.ai/ide/model-context-protocol
- Changelog: https://docs.trae.ai/ide/changelog
- GitHub (Trae IDE, official): https://github.com/Trae-AI/Trae
- GitHub (trae-agent, separate CLI tool): https://github.com/bytedance/trae-agent

---

## Installation

Download from https://www.trae.ai — available for Mac and Windows.

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `.trae/rules/project_rules.md` | Project | Project-level agent rules (see Rules section) |
| `user_rules.md` | Global | User-level agent rules — IDE-managed, auto-created via Settings UI (not manually placed) |
| `.trae/mcp.json` | Project | MCP server configuration |
| IDE Settings UI | Global | All other settings |

> Note: `.trae/rules/` is auto-created by the IDE if it does not exist. The IDE also auto-generates `.trae/rules/git-commit-message.md` for commit message rules.

## Rules (`.rules` feature)

`.rules` is the feature name — not a file extension. Actual rule files use `.md` extension. Rules are Markdown text files that define persistent instructions for the agent, loaded during agent initialization and referenced during code completion and generation.

### Project Rules

Stored at `.trae/rules/project_rules.md` (with the subdirectory `.trae/rules/`, not directly in `.trae/`). Multiple rules files are supported; the IDE creates and manages the `.trae/rules/` folder.

Trae also reads `.trae/rules/` folders in project subdirectories, enabling modular per-component rules.

As of v3.5.18 (January 2026), four application modes are available per rule:

| Mode | Behavior |
|------|----------|
| Always Apply | Applied in every agent session |
| Apply to Specific Files | Applied when matching file globs are in context |
| Apply Intelligently | Agent decides when to apply |
| Apply Manually | Only applied when referenced with `#Rule` in chat |

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

Rules also include AGENTS.md, CLAUDE.md, and CLAUDE.local.md from the project root in the agent's context.

### User Rules

`user_rules.md` is the global personal rules file. It is **IDE-managed and auto-created** — do not place it manually. To create or edit it: open the AI dialog → Settings icon (upper right) → Rules → Personal Rules → click to create. The file applies across all projects.

## Hooks

> **Note:** Trae does not have a native pre/post tool use hook system. Lifecycle control is achieved through:
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

Trae IDE v1.3.0 introduced MCP protocol support. Project-level MCP was added in January 2026.

MCP servers are configured in `.trae/mcp.json` at the project root. Global MCP servers can also be configured in IDE Settings.

**Transport types supported:** stdio, SSE, and Streamable HTTP (three transport types per [official docs](https://docs.trae.ai/ide/model-context-protocol)).

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

❓ The live models page was not fully retrievable. Known state as of mid-2026:

- **GPT-4o** — available built-in
- **Claude models** — built-in access was suspended November 2025 due to Anthropic regional restrictions; Claude can be re-enabled by configuring the Anthropic provider with a personal API key under custom models
- **DeepSeek R1** — available (reported in 2026 reviews)
- Additional models configurable via custom provider API key

See https://docs.trae.ai/ide/models for the current authoritative list.

## Trae CN

See [trae-cn/](../trae-cn/README.md) — the Chinese-market version of Trae with the same architecture but different default models and regional endpoints.

## Notes

- Rules files use `.md` extension; `.rules` is the feature name, not a file extension.
- Rules are loaded at agent initialization; updates may require restarting the agent session.
- `project_rules.md` lives at `.trae/rules/project_rules.md` (note the `rules/` subdirectory).
- `user_rules.md` is IDE-managed (global); create via Settings UI, not by manually placing a file.
- No native hook API; MCP is the primary extensibility path.
- `bytedance/trae-agent` on GitHub is a separate CLI tool, not the Trae IDE. The official Trae IDE repo is `Trae-AI/Trae`.

## Sources

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| Main docs | https://docs.trae.ai | 2026-06-13 | [official] |
| Rules | https://docs.trae.ai/ide/rules | 2026-06-13 | [official] |
| MCP protocol / transport types | https://docs.trae.ai/ide/model-context-protocol | 2026-06-13 | [official] |
| Add MCP servers | https://docs.trae.ai/ide/add-mcp-servers | 2026-06-13 | [official] |
| Models | https://docs.trae.ai/ide/models | 2026-06-13 | [official] |
| Agent guide | https://docs.trae.ai/ide/agent | 2026-06-13 | [official] |
| IDE settings overview | https://docs.trae.ai/ide/ide-settings-overview | 2026-06-13 | [official] |
| Changelog | https://docs.trae.ai/ide/changelog | 2026-06-13 | [official] |
| Trae IDE GitHub (official) | https://github.com/Trae-AI/Trae | 2026-06-13 | [github] |
| trae-agent GitHub (separate CLI) | https://github.com/bytedance/trae-agent | 2026-06-13 | [github] |
| v1.3.0 MCP + rules announcement | https://traeide.com/news/6 | 2026-06-13 | [third-party] |
