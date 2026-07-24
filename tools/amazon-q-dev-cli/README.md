# Amazon Q Developer CLI

> AWS's agentic terminal coding assistant (`q chat`) — open source, but no longer actively maintained; superseded by the closed-source **Kiro CLI**.

**Vendor:** Amazon Web Services (AWS) | **License:** Dual MIT / Apache-2.0 (open source) | **Runtime:** Rust (native binary, `q` / `chat_cli`)

> ⚠️ **Maintenance status:** The `aws/amazon-q-developer-cli` GitHub repo carries a banner stating: *"This open source project is no longer being actively maintained and will only receive critical security fixes. Amazon Q Developer CLI is now available as [Kiro CLI](https://kiro.dev/cli/), a closed-source product."* Everything below describes the last-documented open-source behavior of the `q` CLI. See the separate `kiro` entry in this catalog for its successor. ❓ It is not documented whether Kiro CLI's agent/hook format is identical to what's described here.

## Links

- Docs (AWS User Guide): https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-installing.html
- Technical docs (mdBook, in-repo): https://aws.github.io/amazon-q-developer-cli/
- GitHub: https://github.com/aws/amazon-q-developer-cli
- Successor: https://kiro.dev/cli/

---

## Installation

**macOS (Homebrew):**
```sh
brew install --cask amazon-q
```

**macOS (DMG):** download from `https://desktop-release.q.us-east-1.amazonaws.com/latest/Amazon%20Q.dmg`

**Linux (Ubuntu/Debian, .deb):**
```sh
wget https://desktop-release.q.us-east-1.amazonaws.com/latest/amazon-q.deb
sudo dpkg -i amazon-q.deb
sudo apt-get install -f
```

**Linux (zip, headless/minimal, x86-64, glibc 2.34+):**
```sh
curl --proto '=https' --tlsv1.2 -sSf "https://desktop-release.q.us-east-1.amazonaws.com/latest/q-x86_64-linux.zip" -o "q.zip"
unzip q.zip
# installs to ~/.local/bin by default
```

**Linux (AppImage):**
```sh
curl -LO https://desktop-release.q.us-east-1.amazonaws.com/latest/amazon-q.appimage
chmod +x amazon-q.appimage
./amazon-q.appimage
```

❓ A musl-based build is offered for distros with glibc < 2.34, per AWS docs; exact download URL not independently verified here.

**Build from source (contributors):**
```sh
git clone https://github.com/aws/amazon-q-developer-cli.git
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup default stable && rustup toolchain install nightly
cargo run --bin chat_cli
```

**Authenticate:**
```sh
q login   # AWS Builder ID or IAM Identity Center
```

**Start a chat session:**
```sh
q chat
q chat --agent my-custom-agent
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `.amazonq/cli-agents/*.json` | Project (workspace) | Local custom agent definitions |
| `~/.aws/amazonq/cli-agents/*.json` | Global (user-wide) | Global custom agent definitions |
| `.amazonq/mcp.json` | Project | Legacy/workspace MCP server config |
| `~/.aws/amazonq/mcp.json` | Global | Legacy/global MCP server config |
| `~/.aws/amazonq/profiles/` | Global (legacy) | Pre-agent "profiles" system — auto-migrated to `cli-agents/` on first startup |
| `.amazonq/rules/**/*.md` | Project | Rule/context files auto-loaded as resources by the default agent |
| `.amazonq/cli-todo-lists/` | Project | TODO-list tool storage (experimental) |
| `.amazonq/.subagents/` | Project | Delegate/background-task execution logs (experimental) |

**Agent lookup precedence:** local `.amazonq/cli-agents/` is checked first; if the named agent isn't found there, Q CLI falls back to `~/.aws/amazonq/cli-agents/`. If both a local and global agent share a name, the **local** one wins and a warning is printed (`WARNING: Agent conflict for <name>. Using workspace version.`). The global agents directory is auto-created; the local one must be created manually (`mkdir -p .amazonq/cli-agents`).

**Which agent runs:** `--agent` flag → user-configured default (`q settings chat.defaultAgent <name>`) → built-in default agent (`tools: ["*"]`, `allowedTools: ["fs_read"]`, `resources: ["file://AmazonQ.md", "file://README.md", "file://.amazonq/rules/**/*.md"]`, `useLegacyMcpJson: true`). You can override the built-in default itself by naming an agent file `q_cli_default` in either agents directory.

## Instruction File

There's no single fixed "AGENTS.md"-style file that's always read; instead the **built-in default agent** auto-includes these paths as context via its `resources` field (and any custom agent can declare its own):

- `AmazonQ.md`
- `README.md`
- `.amazonq/rules/**/*.md`

Custom agents replace these defaults entirely — set your own `resources` array (`file://` URIs, globs supported) or point `prompt` at an external file (`"prompt": "file://./prompts/agent.md"`) for a system-prompt-style instruction file.

## Hooks

5 lifecycle/tool events. Hooks are declared inside an agent's JSON config under the `hooks` key (see `agent-format.md#hooks-field` and the dedicated `hooks.md`), not in a separate hooks file. Each hook is a single shell `command`; there is no built-in "prompt"/LLM-evaluation hook type (contrast with some other agent CLIs).

### Supported Events

| Event | When | Can Block? |
|-------|------|-----------|
| `agentSpawn` | Agent is activated/initialized | ❌ (no tool context; stdout is added to agent context on success) |
| `userPromptSubmit` | User submits a chat prompt | ❌ (stdout added to conversation context; cannot block the prompt) |
| `preToolUse` | Before a tool executes | ✅ (exit code `2` blocks the tool call) |
| `postToolUse` | After a tool executes, with results | ❌ (tool has already run) |
| `stop` | When the assistant finishes responding for a turn | ❌ (post-hoc — useful for compile/test/format/cleanup) |

`stop` hooks don't support a `matcher` (they're not tool-specific). `preToolUse`/`postToolUse` support `matcher` patterns: exact tool name (`fs_write`), wildcard (`fs_*`), all MCP tools from a server (`@git`), a specific MCP tool (`@git/status`), all tools (`*`), built-ins only (`@builtin`), or omitted to match everything.

### Hook Input (stdin JSON)

Base event (all hooks):
```json
{
  "hook_event_name": "agentSpawn",
  "cwd": "/current/working/directory"
}
```

`userPromptSubmit` adds `"prompt"`. `preToolUse`/`postToolUse` add `"tool_name"` and `"tool_input"`; `postToolUse` additionally adds `"tool_response"`:

```json
{
  "hook_event_name": "preToolUse",
  "cwd": "/current/working/directory",
  "tool_name": "fs_read",
  "tool_input": { "operations": [{ "mode": "Line", "path": "/cwd/docs/hooks.md" }] }
}
```

MCP tool names are fully namespaced, e.g. `"tool_name": "@postgres/query"`.

### Hook Output

Hooks do not emit a structured JSON decision object on stdout (unlike Claude Code / Devin's `{"decision": "block", ...}` pattern) — the contract is purely **exit code + stdout/stderr text**:

- On success (exit `0`), `agentSpawn` and `userPromptSubmit` hooks have their **stdout appended directly to agent/conversation context**; `preToolUse`/`postToolUse`/`stop` stdout is captured but not surfaced to the user.
- On block (`preToolUse` only, exit `2`), **stderr is returned to the LLM** as the block reason.
- On other non-zero exit codes, **stderr is shown to the user as a warning**; execution continues (tool still runs for `preToolUse`, since exit 2 specifically is required to block).

### Exit Code Behavior

| Code | Meaning |
|------|---------|
| `0` | Success — stdout added to context (`agentSpawn`/`userPromptSubmit`) or captured silently (other events); tool allowed to run (`preToolUse`) |
| `2` | **`preToolUse` only** — blocks the tool call; stderr sent to the LLM |
| Other | Hook considered failed; stderr shown to user as a warning; execution continues (tool still runs for `preToolUse`) |

### Timeout & Caching

- Default timeout: 30 seconds (30,000 ms); override per-hook with `timeout_ms`.
- `cache_ttl_seconds`: `0` (default) = no caching; `> 0` caches successful hook output for that many seconds. `agentSpawn` hooks are **never** cached.

### Example Config (embedded in an agent JSON file)

```json
{
  "hooks": {
    "agentSpawn": [
      { "command": "git status", "cache_ttl_seconds": 300 }
    ],
    "userPromptSubmit": [
      { "command": "ls -la" }
    ],
    "preToolUse": [
      {
        "matcher": "execute_bash",
        "command": "{ echo \"$(date) - Bash command:\"; cat; echo; } >> /tmp/bash_audit_log"
      }
    ],
    "postToolUse": [
      { "matcher": "fs_write", "command": "cargo fmt --all" }
    ]
  }
}
```

## Built-in Tools

| Tool | Description |
|------|-------------|
| `execute_bash` | Execute a shell command |
| `fs_read` | Read files, directories, and images (trusted by default) |
| `fs_write` | Create and edit files |
| `use_aws` | Make AWS CLI API calls (service/operation/parameters) |
| `introspect` | Answer questions about Q CLI's own capabilities/docs (trusted by default) |
| `report_issue` | Open a pre-filled GitHub issue in the browser (no config options) |
| `knowledge` | Store/semantically search a persistent cross-session knowledge base (experimental) |
| `thinking` | Internal step-by-step reasoning mechanism (experimental) |
| `todo_list` | Create/manage multi-step TODO lists, stored in `.amazonq/cli-todo-lists/` (experimental) |
| `delegate` | Launch/monitor background sub-agent tasks in parallel, stored in `.amazonq/.subagents/` (experimental) |

`execute_bash`, `fs_read`, `fs_write`, and `use_aws` accept `toolsSettings` config (allow/deny lists for commands, paths, or AWS services, plus `autoAllowReadonly`/`denyByDefault`). `fs_read` and `report_issue` are trusted (auto-approved) by default; `execute_bash`, `fs_write`, and `use_aws` prompt for permission unless allow-listed via `allowedTools` or `toolsSettings`.

## MCP Support

Two layers:

1. **Per-agent `mcpServers`** — declared directly in an agent's JSON config:
```json
{
  "mcpServers": {
    "fetch": { "command": "fetch3.1", "args": [] },
    "git": {
      "command": "git-mcp",
      "args": [],
      "env": { "GIT_CONFIG_GLOBAL": "/dev/null" },
      "timeout": 120000
    }
  },
  "tools": ["fs_read", "@git", "@fetch/fetch_url"]
}
```
Tools from a server are referenced as `@server_name` (all tools) or `@server_name/tool_name` (specific tool). `toolAliases` can rename tools to resolve cross-server name collisions.

2. **Legacy/global `mcp.json`** — `~/.aws/amazonq/mcp.json` (global) and `.amazonq/mcp.json` (workspace), same `mcpServers` object shape. An agent opts into these via `"useLegacyMcpJson": true` (this is the default for the built-in default agent). Managed with `q mcp add|remove|list|import|status`. ❓ Exact `q mcp` subcommand flags not independently verified beyond the list above.

MCP server timeout defaults to 120,000 ms if unspecified.

## Skills / Commands

There is no separate Markdown-with-frontmatter "skills" directory format (unlike Claude Code/Devin-style `SKILL.md`). Reusable behavior is instead composed from:
- **Custom agents** (see below) acting as the unit of "skill" — a named, purpose-built configuration of tools/prompt/resources/hooks/MCP servers.
- Slash commands in-session (`/agent generate` to have Q draft an agent config from a description, `/agent create`, `/knowledge`, `/todos`, `/experiment`, `/tangent`, `/checkpoint`).

## Agent / Subagent Configuration

Agents are the core extensibility unit: one JSON file per agent (filename minus `.json` = agent name), created manually or via `q agent create --name <name> -d <description>` / the in-session `/agent generate` command. Full field reference:

| Field | Purpose |
|-------|---------|
| `name` | Agent identifier (optional; derived from filename) |
| `description` | Human-readable purpose |
| `prompt` | System-prompt-like context; inline string or `file://` URI |
| `mcpServers` | MCP servers this agent can access |
| `tools` | Tools available (`"fs_read"`, `"@server"`, `"@server/tool"`, `"*"`, `"@builtin"`) |
| `toolAliases` | Rename tools to resolve collisions |
| `allowedTools` | Tools usable without prompting (glob patterns `*`/`?`; no `"*"` wildcard allowed here) |
| `toolsSettings` | Per-tool config (allow/deny lists, paths, services) |
| `resources` | Local file context via `file://` (globs supported) |
| `hooks` | Lifecycle/tool commands (see Hooks above) |
| `useLegacyMcpJson` | Whether to also load `~/.aws/amazonq/mcp.json` / `.amazonq/mcp.json` |
| `model` | Model ID override (e.g. `claude-sonnet-4`); falls back to default with a warning if invalid |

**Subagents (Delegate, experimental):** the `delegate` tool launches background `q chat` sessions running a named agent in parallel with the main conversation; results/notifications surface back into the main session. Task output persists in `.amazonq/.subagents/` until the same agent is re-run. Requires `chat.enableDelegate` setting; only one task runs per agent at a time.

Complete example agent config:
```json
{
  "name": "aws-rust-agent",
  "description": "A specialized agent for AWS and Rust development tasks",
  "mcpServers": {
    "fetch": { "command": "fetch3.1", "args": [] },
    "git": { "command": "git-mcp", "args": [] }
  },
  "tools": ["fs_read", "fs_write", "execute_bash", "use_aws", "@git", "@fetch/fetch_url"],
  "toolAliases": { "@git/git_status": "status", "@fetch/fetch_url": "get" },
  "allowedTools": ["fs_read", "@git/git_status"],
  "toolsSettings": {
    "fs_write": { "allowedPaths": ["src/**", "tests/**", "Cargo.toml"] },
    "use_aws": { "allowedServices": ["s3", "lambda"] }
  },
  "resources": ["file://README.md", "file://docs/**/*.md"],
  "hooks": {
    "agentSpawn": [{ "command": "git status" }],
    "userPromptSubmit": [{ "command": "ls -la" }]
  },
  "useLegacyMcpJson": true,
  "model": "claude-sonnet-4"
}
```

## Notes

- **Project is unmaintained.** The GitHub repo is explicitly marked as no longer actively developed (critical security fixes only); AWS is directing users to the closed-source **Kiro CLI** for ongoing development. Treat this catalog entry as documenting the last stable open-source behavior.
- The legacy "profiles" system (`~/.aws/amazonq/profiles/`, `global_context.json`) auto-migrates to agents on first startup after upgrade; `global_context.json` support was dropped entirely (must be manually re-added as agent `resources`). Legacy hook trigger names `conversation_start`/`per_prompt` map to the new `agentSpawn`/`userPromptSubmit`.
- Experimental features (`knowledge`, `thinking`, `todo_list`, `delegate`, checkpointing, tangent mode, context-usage-percentage) are toggled via the in-session `/experiment` menu or `chat.enable*` settings, and "may change or be removed at any time."
- Checkpointing (experimental) snapshots file changes into a shadow bare git repo per session, enabling `/checkpoint list|expand|diff|restore|clean`.
- Dual-licensed **MIT** and **Apache-2.0**; core CLI (`chat_cli`) is written in Rust.

## Sources

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| GitHub repo README (install, license, maintenance status) | https://github.com/aws/amazon-q-developer-cli | 2026-07-23 | [official] |
| Agent Format (mdBook) | https://aws.github.io/amazon-q-developer-cli/agent-format.html | 2026-07-23 | [official] |
| Agent Format source | https://raw.githubusercontent.com/aws/amazon-q-developer-cli/main/docs/agent-format.md | 2026-07-23 | [official] |
| Hooks reference | https://raw.githubusercontent.com/aws/amazon-q-developer-cli/main/docs/hooks.md | 2026-07-23 | [official] |
| Built-in Tools (mdBook) | https://aws.github.io/amazon-q-developer-cli/built-in-tools.html | 2026-07-23 | [official] |
| Agent File Locations | https://raw.githubusercontent.com/aws/amazon-q-developer-cli/main/docs/agent-file-locations.md | 2026-07-23 | [official] |
| Default Agent Behavior | https://raw.githubusercontent.com/aws/amazon-q-developer-cli/main/docs/default-agent-behavior.md | 2026-07-23 | [official] |
| Legacy Profile → Agent Migration | https://raw.githubusercontent.com/aws/amazon-q-developer-cli/main/docs/legacy-profile-to-agent-migration.md | 2026-07-23 | [official] |
| Experimental Features | https://raw.githubusercontent.com/aws/amazon-q-developer-cli/main/docs/experiments.md | 2026-07-23 | [official] |
| Knowledge Management | https://raw.githubusercontent.com/aws/amazon-q-developer-cli/main/docs/knowledge-management.md | 2026-07-23 | [official] |
| AWS User Guide: Installing Amazon Q for command line | https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-installing.html | 2026-07-23 | [official] |
| AWS User Guide: MCP configuration (legacy paths, `q mcp` subcommands) | https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-mcp-configuration.html | 2026-07-23 | [official] |
| Kiro CLI (successor product) | https://kiro.dev/cli/ | 2026-07-23 | [official] |
