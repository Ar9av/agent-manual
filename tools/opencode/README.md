# OpenCode

> Open-source AI coding agent with agents, skills, commands, and plugins.

**Vendor:** Anomaly (anomalyco) | **License:** MIT | **Runtime:** TypeScript / Bun

## Links

- Docs: https://opencode.ai/docs
- Config: https://opencode.ai/docs/config/
- Agents: https://opencode.ai/docs/agents/
- Skills: https://opencode.ai/docs/skills/
- Plugins: https://opencode.ai/docs/plugins/
- GitHub: https://github.com/anomalyco/opencode
- Awesome OpenCode: https://github.com/awesome-opencode/awesome-opencode

---

## Installation

```sh
curl -fsSL https://opencode.ai/install | bash
# or
npm install -g opencode-ai
# or
bun install -g opencode-ai
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.config/opencode/opencode.json` | Global | User settings |
| `opencode.json` | Project | Project settings |
| `.opencode/plugins/` | Project | Local plugins |
| `~/.config/opencode/plugins/` | Global | Global plugins |
| `.opencode/agents/` | Project | Agent markdown files |
| `~/.config/opencode/agents/` | Global | Global agent markdown files |
| `.opencode/skills/` | Project | Reusable skill instruction sets |
| `~/.config/opencode/skills/` | Global | Global skills |

The `.opencode/` and `~/.config/opencode/` directories use **plural names** for subdirectories: `agents/`, `commands/`, `modes/`, `plugins/`, `skills/`, `tools/`, and `themes/`. Config keys in `opencode.json` are **singular**.

## Hooks / Extensibility

OpenCode uses a **plugin system** as its primary extension mechanism. Plugins run on **Bun** and are TypeScript/JavaScript modules. Native hook events are provided via plugins rather than a built-in hook config key.

### Plugin Hook Events

Plugins can register callbacks for 25+ lifecycle events:

| Category | Events |
|----------|--------|
| **Tool** | `tool.execute.before`, `tool.execute.after` |
| **Session** | `session.created`, `session.updated`, `session.deleted`, `session.idle`, `session.error`, `session.compacted`, `session.diff`, `session.status` |
| **Message** | `message.updated`, `message.removed`, `message.part.updated`, `message.part.removed` |
| **File** | `file.edited`, `file.watcher.updated` |
| **Permission** | `permission.asked`, `permission.replied` |
| **LSP** | `lsp.updated`, `lsp.client.diagnostics` |
| **Shell / Command** | `command.executed`, `shell.env` |
| **TUI** | `tui.prompt.append`, `tui.command.execute`, `tui.toast.show` |
| **Server** | `server.connected` |
| **Install** | `installation.updated` |
| **Misc** | `todo.updated` |
| **Experimental** | `experimental.session.compacting` |

Specialized hooks (returned as top-level keys alongside event handlers):

| Hook Key | Purpose |
|----------|---------|
| `tool.execute.before` | Intercept/modify tool args before execution |
| `tool.execute.after` | Process tool results post-execution |
| `chat.message` | Intercept messages before LLM processing |
| `chat.params` | Modify model parameters |
| `permission.ask` | Control permission request handling |
| `config` | Modify configuration at runtime |
| `experimental.chat.system.transform` | Inject context into system prompt |
| `experimental.session.compacting` | Preserve state during compaction |

### Plugin Configuration

```json
{
  "plugin": ["opencode-plugin-logger", "./plugins/my-plugin.js"]
}
```

Note: npm package names are bare — no `npm:` prefix. Local files use relative paths.

Or place plugins directly in `.opencode/plugins/` (auto-loaded, no config entry needed).

### Plugin Structure

Plugins export an **async function** (not a static object) receiving a context argument and returning a hooks object with dot-namespaced keys:

```ts
// .opencode/plugins/my-plugin.ts
import type { Plugin } from "@opencode-ai/plugin"

export const MyPlugin: Plugin = async ({ project, client, $, directory, worktree }) => {
  return {
    "tool.execute.before": async ({ tool, input }) => {
      if (tool === "bash" && input.command?.includes("rm -rf")) {
        return { block: true, message: "Blocked destructive command" }
      }
    },
    "tool.execute.after": async ({ tool, output }) => {
      console.error(`[audit] ${tool} completed`)
    },
    event: async ({ event }) => {
      // subscribe to any system event by type
    }
  }
}
```

**Context object properties:**
- `project` — Current project info
- `client` — OpenCode SDK client for AI interaction
- `$` — Bun's shell API for command execution
- `directory` — Current working directory
- `worktree` — Git worktree path

Plugin dependencies go in `.opencode/package.json`; OpenCode runs `bun install` automatically.

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

Configure MCP servers in `opencode.json`. Server names are **direct children of `mcp`** — there is no `servers` sub-key:

```json
{
  "mcp": {
    "my-server": {
      "type": "local",
      "command": ["node", "./mcp-server/index.js"]
    },
    "remote-server": {
      "type": "remote",
      "url": "https://example.com/mcp",
      "enabled": true
    }
  }
}
```

## Agents

Define specialized agents with custom prompts, models, and tool access. Config key is `agent` (singular), and the system prompt key is `prompt` (points to a file path):

```json
{
  "agent": {
    "code-reviewer": {
      "model": "anthropic/claude-opus-4",
      "description": "Strict code reviewer",
      "prompt": "{file:./agents/code-reviewer.md}",
      "mode": "subagent",
      "permission": {
        "read": "allow",
        "edit": "ask",
        "bash": "deny"
      }
    }
  }
}
```

Agents can also be defined as markdown files in `.opencode/agents/` (filename becomes the agent name).

**Agent config fields:**
| Field | Description |
|-------|-------------|
| `model` | Override default model (`provider/model-id`) |
| `description` | Brief explanation (required) |
| `prompt` | File path reference `{file:./path}` |
| `mode` | `"primary"`, `"subagent"`, or `"all"` |
| `permission` | Per-tool permission overrides |
| `temperature` | 0.0–1.0 |
| `steps` | Max agentic iterations |
| `color` | Hex or theme name |

### Permissions

Config key is `permission` (singular). Valid values: `"allow"`, `"ask"`, `"deny"`.

```json
{
  "permission": {
    "bash": "ask",
    "edit": "ask",
    "read": "allow"
  }
}
```

Permission keys include: `read`, `edit`, `glob`, `grep`, `list`, `bash`, `task`, `external_directory`, `todowrite`, `webfetch`, `websearch`, `lsp`, `skill`.

## Skills

Skills are reusable instruction sets loaded on-demand by agents. A skill is a directory containing a `SKILL.md` file with a name, description, and instructions.

- Project skills: `.opencode/skills/`
- Global skills: `~/.config/opencode/skills/`

## Commands

Custom slash commands go in `.opencode/commands/`.

## Notes

- OpenCode does not have a native `PreToolUse`/`PostToolUse` hook config key — all lifecycle control goes through plugins.
- Plugins run on **Bun**, not Node.js.
- The `tools` field on agents is deprecated; use `permission` instead for fine-grained tool access control.
- The GitHub repo moved from `sst/opencode` to `anomalyco/opencode` following the SST → Anomaly company rebrand in 2026.

## Sources

| Topic | URL | Label |
|-------|-----|-------|
| Docs home | https://opencode.ai/docs | [official] |
| Config reference | https://opencode.ai/docs/config/ | [official] |
| Agents reference | https://opencode.ai/docs/agents/ | [official] |
| Skills reference | https://opencode.ai/docs/skills/ | [official] |
| Plugins reference | https://opencode.ai/docs/plugins/ | [official] |
| GitHub repo | https://github.com/anomalyco/opencode | [github] |
| Awesome OpenCode | https://github.com/awesome-opencode/awesome-opencode | [github] |
| Plugin guide (gist) | https://gist.github.com/johnlindquist/0adf1032b4e84942f3e1050aba3c5e4a | [github] |
