# Amp

> Agentic coding tool built by Amp (spun off from Sourcegraph in December 2025), available as a CLI, VS Code/JetBrains/Neovim/Zed extension, and web/Slack surface.

**Vendor:** Amp (formerly Sourcegraph; independent company since Dec 2025) | **License:** Proprietary | **Runtime:** Node.js CLI binary (npm package `@ampcode/cli`, formerly `@sourcegraph/amp`)

## Links

- Manual (docs): https://ampcode.com/manual
- Plugin API reference: https://ampcode.com/manual/plugin-api
- Appendix (stream JSON / tool list): https://ampcode.com/manual/appendix
- News / company spinoff: https://ampcode.com/news/amp-inc
- npm package: https://www.npmjs.com/package/@ampcode/cli

---

## Installation

**macOS / Linux / WSL:**
```sh
curl -fsSL https://ampcode.com/install.sh | bash
```

**Windows:**
```powershell
powershell -c "irm https://ampcode.com/install.ps1 | iex"
```

**Homebrew:**
```sh
brew install ampcode/tap/ampcode
```

**npm** (documented but not the recommended path):
```sh
npm install -g @ampcode/cli
```

**Update:**
```sh
amp update
```

> ❓ The package was published under `@sourcegraph/amp` prior to the December 2025 spinoff and moved to `@ampcode/cli` around May 2026; older tutorials/blog posts may still reference the old package name.

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.config/amp/settings.json` (or `.jsonc`) | Global (macOS/Linux); honors `$XDG_CONFIG_HOME/amp/settings.json` | User-level settings, MCP servers, tool disable list, permissions |
| `%USERPROFILE%\.config\amp\settings.json` | Global (Windows) | Same as above |
| `.amp/settings.json` (or `.jsonc`) | Project/workspace — resolved by searching upward from cwd to repo root | Workspace settings; overrides user settings |
| Custom path via `--settings-file <path>` or `AMP_SETTINGS_FILE` env var | Either | Override the settings file location |
| `/Library/Application Support/ampcode/managed-settings.json` (macOS), `/etc/ampcode/managed-settings.json` (Linux), `%ProgramData%\ampcode\managed-settings.json` (Windows) | Enterprise-managed | Org-enforced settings, takes precedence over user/workspace |

Edit with `amp config edit` (add `--workspace` to edit the workspace file instead of user settings).

## Instruction File

Amp reads **`AGENTS.md`** as its primary guidance file (also accepts `AGENT.md` singular, and `CLAUDE.md`). It is auto-included from:

- The current working directory and all parent directories up to `$HOME`
- Subtrees, as the agent reads files in those directories
- Global locations: `$HOME/.config/amp/AGENTS.md`, `$HOME/.config/AGENTS.md`
- System-wide locations: `/Library/Application Support/ampcode/AGENTS.md` (macOS), `/etc/ampcode/AGENTS.md` (Linux), `%ProgramData%\ampcode\AGENTS.md` (Windows)

Format: Markdown, with optional YAML frontmatter using a `globs` key to scope guidance to matching file patterns. Supports `@`-mentions to pull in other files:

```markdown
See @doc/style.md and @specs/**/*.md.
When making commits, see @doc/git-commit-instructions.md.
```

`@`-mentions support relative paths, absolute paths, `@~/some/path`, and glob patterns. Run `agents-md list` from the command palette to see which files were actually loaded for a session.

## Hooks / Extensibility

**Amp does not have a declarative JSON "hooks" config system** like Claude Code or Devin CLI. Instead it ships a **Plugin API** — plugins are TypeScript files that subscribe to lifecycle events programmatically and can register new tools and commands. This is the mechanism to reach for anything a hooks system would otherwise do (blocking a tool call, injecting context, running validation).

### Plugin locations

| Location | Scope |
|----------|-------|
| `.amp/plugins/*.ts` | Project |
| `~/.config/amp/plugins/*.ts` | User (macOS/Linux) |
| `%USERPROFILE%\.config\amp\plugins\*.ts` | User (Windows) |

Reload after edits with `plugins: reload` from the command palette; `plugins: list` shows loaded plugins, their registered events, commands, and tools.

### Lifecycle events (`amp.on(event, handler)`)

| Event | When | Can block/modify? |
|-------|------|-----------|
| `session.start` | A thread session starts (new message or thread opened) | ❌ fire-and-forget |
| `tool.call` | Before a tool executes | ✅ handler returns `{ action: 'allow' \| 'reject-and-continue' \| 'modify' \| 'synthesize' \| 'error' }` |
| `tool.result` | After a tool executes | ✅ can modify the returned result |
| `agent.start` | User submits a prompt | ✅ can append context messages |
| `agent.end` | Agent finishes handling a prompt | ✅ can return `{ action: 'continue', userMessage }` to trigger a follow-up turn |

### Core plugin API

```typescript
amp.on<E extends keyof PluginEventMap>(
  event: E,
  handler: (event: PluginEventMap[E], ctx: PluginEventContext<E>) => PluginHandlerResult<E>
): Subscription

amp.registerTool({
  name: string,
  description: string,
  inputSchema: { type: 'object', properties, required },
  async execute(input, ctx): Promise<PluginToolResult>
}): Subscription

amp.registerCommand(
  id: string,
  options: { title, category?, description?, availability? },
  handler: (ctx: PluginCommandContext) => void | Promise<void>
): CommandSubscription
```

Handlers get a `ctx` with logger, UI helpers (`ctx.ui.notify()`, `ctx.ui.confirm()`, `ctx.ui.input()`, `ctx.ui.select()`), AI classification (`amp.ai.ask(...)`), and shell/system access.

### Checks (code review, adjacent mechanism)

A separate, non-lifecycle extensibility surface: Markdown files with YAML frontmatter (`name`, `description`, `severity-default`, `tools`) in `.agents/checks/` (project-wide), `<subtree>/.agents/checks/` (subtree-specific), or `$HOME/.config/amp/checks/` / `$HOME/.config/agents/checks/` (global) — used for automated code-review rules rather than lifecycle hooks.

### Legacy: Toolboxes (deprecated)

Amp previously had a "Toolboxes" extensibility mechanism; it is **no longer supported**. The documented migration path is to reimplement toolbox tools as plugins in `.amp/plugins/`, registering them with `amp.registerTool(...)` and preserving the original `tb__*` tool names/behavior.

### Legacy permission settings (still read, superseded by plugins)

`amp.permissions`, `amp.guardedFiles.allowlist`, and `amp.dangerouslyAllowAll` settings keys provide a simpler allow/deny rule system. By default, Amp does **not** ask for approval before running tools; the plugin `tool.call` event is the more powerful/current way to gate tool execution.

## Built-in Tools

Run `amp tools list` in the CLI for the authoritative, live list. Documented tool names (from the manual and `--stream-json` output examples):

| Tool | Description |
|------|-------------|
| `Bash` | Execute shell commands |
| `Read` | Read file contents |
| `create_file` | Create new files |
| `edit_file` | Modify existing files |
| `undo_edit` | Revert a file edit |
| `Grep` | Search file contents by pattern |
| `glob` | Pattern-based file discovery |
| `finder` | Intelligent, multi-step codebase search |
| `read_web_page` | Fetch and parse a web page |
| `web_search` | Internet search |
| `read_mcp_resource` | Read a resource exposed by an MCP server |
| `Task` | Spawn a subagent for isolated/parallel work |
| `oracle` | Specialized tool for code review / design feedback |
| `todo_read` / `todo_write` | Read/write the agent's task list |
| `get_diagnostics` | ❓ retrieve editor/LSP diagnostics (seen in community tool-list reports, not confirmed in the primary manual page fetched) |

Individual tools can be disabled via settings:
```json
{ "amp.tools.disable": ["toolname", "builtin:toolname"] }
```

## MCP Support

**Yes.** Configured under the `amp.mcpServers` key in `settings.json`/`.jsonc` (same schema as the Amp VS Code extension), or via CLI:

```sh
amp mcp add <name> -- <command> [args]   # local/stdio server
amp mcp add <name> <url>                  # remote/HTTP server
amp mcp list
amp mcp approve <server-name>             # required for workspace-declared servers
amp mcp doctor
```

Config path: `~/.config/amp/settings.json` (global) or `.amp/settings.json` (workspace, requires explicit `amp mcp approve` before it runs).

```json
{
  "amp.mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest", "--headless"]
    },
    "linear": {
      "url": "https://mcp.linear.app/sse"
    },
    "sourcegraph": {
      "url": "${SRC_ENDPOINT}/.api/mcp/v1",
      "headers": { "Authorization": "token ${SRC_ACCESS_TOKEN}" }
    }
  }
}
```

- `env` (object) supported on local servers; `headers` (object) on remote servers.
- `includeTools` (string array of names/glob patterns) filters which tools from a server are exposed — recommended to keep context small.
- `${VAR_NAME}` syntax expands environment variables inside config values.
- Load order (highest precedence first): CLI flags (`--mcp-config`) → user/workspace `amp.mcpServers` → skill-declared MCP servers (loaded on demand).
- Enterprise workspaces can enforce an MCP server allowlist.

## Skills

Directory-based skills, each a folder containing `SKILL.md` with YAML frontmatter:

```yaml
---
name: my-skill
description: What this does
---
# Instructions
```

| Location | Scope |
|----------|-------|
| `.agents/skills/` | Project |
| `~/.config/agents/skills/` or `~/.agents/skills/` | User |
| `~/.config/amp/skills/` or `.claude/skills/` | Legacy paths, still read |

A skill directory can also include its own `mcp.json`; those MCP tools stay hidden until the skill is loaded. `skill: list` from the command palette shows what's loaded.

## Subagents

Amp automatically spawns subagents (via the `Task` tool) for suitable work, mostly in its `medium` reasoning/effort mode:

- Subagents work in isolation and cannot communicate with each other.
- Each starts fresh with no conversation context from the main thread.
- The main agent receives only a summary back, not step-by-step monitoring.
- Best suited to multi-step independent work, tasks producing extensive output, or parallelizable operations.
- Users can nudge this behavior by explicitly asking Amp to use subagents / do things in parallel.

There is no separate persistent "subagent profile" config file analogous to Devin's `.devin/agents/` — subagent behavior is driven by the `Task` tool at runtime, plus whatever plugins/skills are loaded.

## Notes

- Amp organizes interactions as **threads** — persistent, save/searchable/shareable conversations (by @-mention or URL) containing messages, context, and tool calls.
- The CLI integrates with VS Code, JetBrains IDEs, Neovim, and Zed; when IDE integration is active, Amp can see the currently open file and selection.
- `--stream-json`, `--stream-json-thinking`, and `--stream-json-input` flags support programmatic/scripted integration (JSON event stream in/out, optional reasoning blocks, multi-turn input over stdin).
- Amp was originally a Sourcegraph product; in December 2025 it spun off into an independent company (Amp Frontier Corporation / "Amp, Inc.") with Sourcegraph continuing as a separate code-search business. This changed the npm package name from `@sourcegraph/amp` to `@ampcode/cli` (~May 2026).
- Pricing shifted post-spinoff to include a free tier plus paid usage-based/subscription options (e.g., ~$10/day "any mode" pricing reported as of January 2026, with monthly subscription options following); ❓ treat exact current pricing as time-sensitive and verify against https://ampcode.com directly rather than this doc.

## Sources

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| Owner's Manual (install, config paths, AGENTS.md, MCP, tools, subagents, threads) | https://ampcode.com/manual | 2026-07-23 | [official] |
| Plugin API reference (lifecycle events, `amp.on`/`registerTool`/`registerCommand`) | https://ampcode.com/manual/plugin-api | 2026-07-23 | [official] |
| Appendix (stream-json examples, built-in tool names) | https://ampcode.com/manual/appendix | 2026-07-23 | [official] |
| Company spinoff announcement | https://ampcode.com/news/amp-inc | 2026-07-23 | [official] |
| Why Sourcegraph and Amp are becoming independent companies | https://sourcegraph.com/blog/why-sourcegraph-and-amp-are-becoming-independent-companies | 2026-07-23 | [official] |
| npm package `@sourcegraph/amp` (legacy name, historical) | https://www.npmjs.com/package/@sourcegraph/amp | 2026-07-23 | [official] |
| Community report on `amp tools list` output (used only to cross-check `get_diagnostics`, marked ❓ above) | https://ampcode.com/threads/T-cc7ac6b0-f29b-4d8a-9483-bf773c66c06c | 2026-07-23 | [community] |
