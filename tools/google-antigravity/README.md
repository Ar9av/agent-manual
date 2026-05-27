# Google Antigravity

> Google's agent-first software development platform and AI-powered IDE. Docs at antigravity.google/docs.

**Vendor:** Google | **License:** Proprietary (Pre-GA / Preview Terms) | **Runtime:** Go / Native CLI (`agy`)

## Links

- Docs: https://antigravity.google/docs/home
- Community forum: https://discuss.ai.google.dev/t/does-antigravity-support-hooks-similar-to-the-hook-functionality-in-windsurf/121062

---

## Installation

### macOS / Linux
```sh
curl -fsSL https://antigravity.google/cli/install.sh | bash
```

### Windows (PowerShell)
```powershell
irm https://antigravity.google/cli/install.ps1 | iex
```

### Windows (CMD)
```cmd
curl -fsSL https://antigravity.google/cli/install.cmd -o install.cmd && install.cmd && del install.cmd
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.config/antigravity/config.toml` | Global | CLI configurations (model selection, endpoints) |
| `~/.gemini/antigravity-cli/settings.json` | Global | CLI interface settings & preferences |
| `~/.gemini/config/hooks.json` | Global | Global lifecycle hook configurations |
| `.agents/hooks.json` | Project | Project-level lifecycle hook overrides |

## Instruction File

The agent reads declarative, behavioral rules and multi-step automated workflows from the project root:
- `.agent/rules/` (Project-level behavioral rules/triggers)
- `.agent/workflows/` (Multi-step automated workflows)

## Hooks

Antigravity supports a standard lifecycle interceptor system. Interceptors are classified into three strict categories based on their function:
- **Inspect** (Read-Only, Non-Blocking): Used for observability, logging, audit trails, and metrics.
- **Decide** (Read-Only, Blocking): Used for policy enforcement and security guardrails. Returns a decision (e.g., `allow` or `deny`).
- **Transform** (Modifying, Blocking): Used for data sanitization, prompt optimization, or error recovery.

The execution order enforces a strict pipeline: **Decide → Transform → Inspect** to prevent Time-of-Check to Time-of-Use (TOCTOU) vulnerabilities.

### Supported Events

| Event | When | Can Block? |
|-------|------|-----------|
| `PreToolUse` | Before a tool is executed | ✅ (exit 2 or JSON block) |
| `PostToolUse` | After a tool completes | ❌ |
| `PreInvocation` | Before the agent calls the model | ✅ |
| `PostInvocation` | After model calls finish | ❌ |
| `Stop` | When the execution loop terminates | ✅ |

### Hook Input (stdin JSON)

```json
{
  "hook_event_name": "PreToolUse",
  "tool_name": "run_command",
  "tool_input": { "command": "npm test" },
  "session_id": "session-12345"
}
```

### Hook Output (stdout JSON, optional)

```json
{
  "decision": "block",
  "reason": "Security policy violation"
}
```

### Exit Code Behavior

| Code | Meaning |
|------|---------|
| `0` | Success, continue |
| `2` | Block action (blocking events only); stderr/reason sent to LLM |
| Other | Warning; execution continues |

### Example Config (`hooks.json`)

```json
{
  "safety-gate": {
    "enabled": true,
    "PreToolUse": [
      {
        "matcher": "run_command",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/safety-check.sh"
          }
        ]
      }
    ]
  }
}
```

## Built-in Tools

| Tool | Description |
|------|-------------|
| `view_file` | Read and retrieve file contents |
| `replace_file_content` | Targeted contiguous edits within a file |
| `multi_replace_file_content` | Non-contiguous edits across a single file |
| `write_to_file` | Create new files and write code contents |
| `list_dir` | List contents of a directory |
| `grep_search` | Search exact patterns using ripgrep |
| `run_command` | Execute bash/shell commands on the host |
| `search_web` | Perform Google search queries for external information |
| `read_url_content` | Fetch content from URLs and convert HTML to markdown |

## MCP Support

Antigravity supports the Model Context Protocol (MCP) using a shared configuration file.

Config locations:
- `~/.gemini/config/mcp_config.json` (Shared configurations)
- `~/.gemini/antigravity-cli/mcp_config.json` (CLI specific)

Supports both local stdio and remote HTTP/SSE transport modes.

### Local (stdio) and Remote HTTP Configurations

```json
{
  "mcpServers": {
    "my-local-server": {
      "command": "node",
      "args": ["./mcp/server.js"],
      "env": {
        "API_KEY": "..."
      }
    },
    "my-remote-server": {
      "serverUrl": "https://mcp.example.com",
      "headers": {
        "Authorization": "Bearer ..."
      }
    }
  }
}
```

> [!IMPORTANT]
> For remote servers, the field `serverUrl` is strictly required (`url` or `httpUrl` are not supported).

## Skills / Commands

- Skills location: Global `~/.agents/skills/` or Project `.agents/skills/`
- Format: Markdown file `SKILL.md` (YAML frontmatter: `name`, `description`, `trigger`, etc.). Windows variant uses PowerShell.
- Convention shared with Codex CLI.

## Agent / Subagent Configuration

Antigravity CLI and IDE allow spawning and managing background subagents (e.g. `research` and `self`). Subagents can inherit, branch, or share the parent workspace to run concurrent processes.

## Notes

- Debugging: Run `agy inspect` via the CLI to check active hooks, settings, and rule configurations.
- Run `agy plugin import` to migrate existing extensions or configurations from the older Gemini CLI ecosystem.

## Sources (Official)

| Topic | URL |
|-------|-----|
| Docs home | https://antigravity.google/docs/home |
| Lifecycle Hooks | https://antigravity.google/docs/hooks |
| Workflows & Rules | https://antigravity.google/docs/rules-workflows |
| Community forum | https://discuss.ai.google.dev/t/does-antigravity-support-hooks-similar-to-the-hook-functionality-in-windsurf/121062 |
