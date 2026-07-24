# Trae IDE

> ByteDance's AI-first development tool with MCP and rules-based configuration.

**Vendor:** ByteDance | **License:** Proprietary | **Runtime:** Electron

## Links

- Docs: https://docs.trae.ai
- Agent guide: https://docs.trae.ai/ide/agent
- Rules: https://docs.trae.ai/ide/rules
- Skills: https://docs.trae.ai/ide/skills
- SOLO mode: https://docs.trae.ai/ide/solo-mode
- IDE settings overview: https://docs.trae.ai/ide/ide-settings-overview
- Models: https://docs.trae.ai/ide/models
- MCP protocol: https://docs.trae.ai/ide/model-context-protocol
- Changelog: https://docs.trae.ai/ide/changelog
- GitHub (Trae IDE, official): https://github.com/Trae-AI/Trae
- GitHub (trae-agent, separate CLI tool): https://github.com/bytedance/trae-agent

---

## Installation

Download from https://www.trae.ai — available for Mac, Windows, and Linux (.deb/.rpm for x64).

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `.trae/rules/<name>.md` | Project | Project-level agent rules (see Rules section) — as of 2026-07, filenames are user-defined at creation time; there is no single canonical `project_rules.md` file |
| `user_rules.md` | Global | User-level agent rules — IDE-managed, auto-created via Settings UI (not manually placed) |
| `.trae/mcp.json` | Project | MCP server configuration |
| `.trae/skills/{skill_name}/SKILL.md` | Project | IDE-managed skill definitions (see Skills section) |
| `.agents/skills/` | Project | Open agent skills standard — alternative skill location |
| `.trae/commands/` | Project | Custom slash commands (v3.5.54+, up to 3 nesting levels) |
| IDE Settings UI | Global | All other settings |

> Note: `.trae/rules/` is auto-created by the IDE if it does not exist. The IDE also auto-generates `.trae/rules/git-commit-message.md` for commit message rules.

## Rules (`.rules` feature)

`.rules` is the feature name — not a file extension. Actual rule files use `.md` extension. Rules are Markdown text files that define persistent instructions for the agent, loaded during agent initialization and referenced during code completion and generation.

### Project Rules

Stored under `.trae/rules/` (subdirectory of the project root, not directly in `.trae/`). Per current docs, when you create a project rule via Settings > Rules, you name the file yourself and the IDE creates it in `.trae/rules/`  — there is no fixed `project_rules.md` filename. Multiple rules files are supported.

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

Trae also reads `AGENTS.md`, `CLAUDE.md`, and `CLAUDE.local.md` from the project root as rule sources (toggleable in Settings > Rules).

### User Rules

`user_rules.md` is the global personal rules file. It is **IDE-managed and auto-created** — do not place it manually. To create or edit it: open the AI dialog → Settings icon (upper right) → Rules → Personal Rules → click to create. The file applies across all projects.

## Skills (v3.5.41+, March 2026)

Skills are on-demand reusable procedures — the key distinction from Rules is loading behavior:

| Feature | Rules | Skills |
|---------|-------|--------|
| Loading | Always-on — loaded into every context | On-demand — only loaded when explicitly invoked |
| Token cost | Every session | Only when used |
| Format | `.md` files in `.trae/rules/` | `SKILL.md` files in skill directories |
| Invocation | Automatic (per mode) | Explicit invocation only |

Skills are stored as `SKILL.md` files and can be placed in:

- `.trae/skills/{skill_name}/SKILL.md` — IDE-managed location
- `.agents/skills/` — open agent skills standard (interoperable with other agents)

Skills allow teams to define reusable procedures (e.g., "run the test suite", "scaffold a new component") that only consume tokens when explicitly called, keeping routine contexts lean.

## SOLO Mode

SOLO is Trae's autonomous multi-agent mode for end-to-end task execution. Unlike the standard agent (which is interactive and step-by-step), SOLO operates autonomously across the full development lifecycle:

- **Requirements analysis** — breaks down tasks and plans implementation
- **Code generation** — writes and edits files across the project
- **Terminal commands** — runs builds, tests, and installs autonomously
- **Browser testing** — interacts with a browser to verify functionality
- **Deployment** — can execute deployment steps end-to-end

SOLO is designed for larger, self-contained tasks where you want minimal interruption. Full documentation: https://docs.trae.ai/ide/solo-mode

## Slash Commands (v3.5.54+)

Custom slash commands can be defined in `.trae/commands/`. Up to 3 levels of nesting are supported, enabling structured command hierarchies (e.g., `/project/setup/database`).

## Hooks

> **Update (2026-07-23):** Trae now ships a native **Hooks** feature — "Hooks" appears as its own top-level entry (alongside Skills, Rules, Memories, Commands) in the official docs sidebar under "AI coding essentials." The dedicated docs page could not be reached directly during this audit (JS-rendered SPA, slug not discoverable), so exact event names/config format are ❓ unverified — do not assume the MCP-only workaround below is still the only option. Treat the rest of this section as historical/fallback guidance until the native Hooks docs are confirmed.

### MCP-Based Hook Patterns (fallback / pre-Hooks-feature approach)

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

Built-in models as of 2026-07-23 (per live docs — this list churns frequently, treat as a snapshot):

| Model | Status |
|-------|--------|
| GPT-5.4 | Available built-in (not available to US users) |
| GPT-5.2 | Available built-in (not available to US users) |
| Dola-Seed-2.0-Code | Available built-in |
| MiniMax-M3 | Available built-in (not available to US users) |
| MiniMax-M2.7 | Available built-in (not available to US users) |
| Kimi-K2.5 | Available built-in |
| Gemini-3.1-Pro-Preview | Available built-in |
| Gemini-3-Flash-Preview | Available built-in |
| Claude models | Not in the built-in list; add via custom provider (Anthropic Messages API format) with a personal API key |

GPT-4o, Gemini 2.5 Pro, DeepSeek R1/V3, and Kimi-K2 (non-.5) are no longer on the built-in list — superseded by the models above. See https://docs.trae.ai/ide/models for the current authoritative list.

## Trae CN

See [trae-cn/](../trae-cn/README.md) — the Chinese-market version of Trae with the same architecture but different default models and regional endpoints.

## Notes

- Rules files use `.md` extension; `.rules` is the feature name, not a file extension.
- Rules are loaded at agent initialization; updates may require restarting the agent session.
- `project_rules.md` lives at `.trae/rules/project_rules.md` (note the `rules/` subdirectory).
- `user_rules.md` is IDE-managed (global); create via Settings UI, not by manually placing a file.
- `AGENTS.md`, `CLAUDE.md`, and `CLAUDE.local.md` are read as rule sources (toggleable in Settings > Rules).
- Skills (v3.5.41+) load on demand only — unlike Rules which are always-on — saving tokens on large contexts.
- SOLO mode is the autonomous multi-agent mode for end-to-end task execution without step-by-step confirmation.
- Custom slash commands live in `.trae/commands/` (v3.5.54+, up to 3 nesting levels).
- As of 2026-07-23, docs.trae.ai lists a native "Hooks" feature in its sidebar (previously Trae had no native hook API and relied on MCP as the primary extensibility path — see the Hooks section above for details/caveats).
- `bytedance/trae-agent` on GitHub is a separate CLI tool, not the Trae IDE. The official Trae IDE repo is `Trae-AI/Trae`.

## Sources

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| Main docs | https://docs.trae.ai | 2026-06-13 | [official] |
| Rules | https://docs.trae.ai/ide/rules | 2026-07-23 | [official] — rule filenames confirmed user-defined, not `project_rules.md` |
| Skills | https://docs.trae.ai/ide/skills | 2026-07-23 | [official] |
| SOLO mode | https://docs.trae.ai/ide/solo-mode | 2026-06-13 | [official] |
| MCP protocol / transport types | https://docs.trae.ai/ide/model-context-protocol | 2026-07-23 | [official] — stdio/SSE/Streamable HTTP unchanged |
| Add MCP servers | https://docs.trae.ai/ide/add-mcp-servers | 2026-07-23 | [official] — `mcpServers` JSON format unchanged |
| Models | https://docs.trae.ai/ide/models | 2026-07-23 | [official] — built-in model list changed significantly, see table above |
| Agent guide | https://docs.trae.ai/ide/agent | 2026-06-13 | [official] |
| IDE settings overview | https://docs.trae.ai/ide/ide-settings-overview | 2026-06-13 | [official] |
| Changelog | https://docs.trae.ai/ide/changelog | 2026-07-23 | [official] — no entries currently listed ("No update record yet") |
| Trae IDE GitHub (official) | https://github.com/Trae-AI/Trae | 2026-06-13 | [github] |
| trae-agent GitHub (separate CLI) | https://github.com/bytedance/trae-agent | 2026-06-13 | [github] |
| v1.3.0 MCP + rules announcement | https://traeide.com/news/6 | 2026-06-13 | [third-party] |
| Hooks (new feature, unverified) | https://docs.trae.ai | 2026-07-23 | [official] ❓ — confirmed present in docs nav, page content not yet verified |
