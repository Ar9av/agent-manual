# Crush

> "Glamourous agentic coding for all" — Charmbracelet's open-source, Go-based terminal AI coding agent with multi-provider support, LSP integration, and MCP extensibility.

**Vendor:** Charmbracelet, Inc. (Charm) | **License:** FSL-1.1-MIT (Functional Source License — becomes MIT after 2 years) | **Runtime:** Go (single static binary)

## Links

- GitHub: https://github.com/charmbracelet/crush
- README (primary docs): https://github.com/charmbracelet/crush/blob/main/README.md
- Hooks guide: https://github.com/charmbracelet/crush/blob/main/docs/hooks/README.md
- Contributor/architecture guide: https://github.com/charmbracelet/crush/blob/main/AGENTS.md
- Config schema: https://github.com/charmbracelet/crush/blob/main/schema.json (also available locally via `crush schema`)

---

## Installation

```sh
# Homebrew (macOS/Linux)
brew install charmbracelet/tap/crush

# NPM
npm install -g @charmland/crush

# Arch Linux (AUR)
yay -S crush-bin

# Nix
nix run github:numtide/nix-ai-tools#crush

# FreeBSD
pkg install crush

# Go install
go install github.com/charmbracelet/crush@latest
```

**Windows:**
```powershell
winget install charmbracelet.crush
# or
scoop bucket add charm https://github.com/charmbracelet/scoop-bucket.git
scoop install crush
```

Also available as: direct binary downloads, and Debian/Ubuntu (apt) / Fedora/RHEL (yum) repositories maintained by Charm. ❓ Exact apt/yum repo URLs not independently re-verified beyond the README summary.

## Configuration Files

Crush merges JSON config from multiple locations, walking up the directory tree for project files, in priority order:

| File | Scope | Purpose |
|------|-------|---------|
| `.crush.json` | Project-local | Project config (dotfile variant) |
| `crush.json` | Project-local | Project config |
| `$HOME/.config/crush/crush.json` | Global | User-level settings (providers, MCP, LSP, permissions, hooks) |
| `$HOME/.local/share/crush/crush.json` (Unix) / `%LOCALAPPDATA%\crush\crush.json` (Windows) | Global, ephemeral | Application state (e.g., saved API keys), not meant for manual editing |

Override default paths with env vars `CRUSH_GLOBAL_CONFIG` and `CRUSH_GLOBAL_DATA`. Add a `"$schema"` field pointing at `schema.json` (or run `crush schema`) for editor autocomplete/validation. Crush respects `.gitignore` and additionally supports a `.crushignore` file (same syntax) to exclude paths from context.

## Instruction File

Crush reads two kinds of global instruction/context files, plus a project-level file:

- `~/.config/crush/CRUSH.md` — Crush-specific rules ("that would confuse other agentic coding tools"). If you only use Crush, this is the only global file you need.
- `~/.config/AGENTS.md` — generic, tool-agnostic instructions meant to also be readable by other agentic coding tools.
- Project-level: created by Crush's init flow as **`AGENTS.md`** by default (configurable via `options.initialize_as`, e.g. to `CRUSH.md`).

Global context file paths are themselves configurable via `global_context_paths`.

## Hooks

Crush has a lightweight, still-evolving hook system (per the hooks guide, "preliminary" as of mid-2026). Hooks are user-defined shell commands declared in `crush.json` under a `"hooks"` key, matched against tool names, and run **in parallel** with results composed deterministically in config order (not first-to-finish).

### Supported Events

| Event | When | Can Block? | Scope |
|-------|------|-----------|-------|
| `PreToolUse` | Before every top-level-agent tool call | ✅ (exit 2 = deny, exit 49 = halt turn) | Only the top-level agent — sub-agent tool calls are **not** intercepted (see Subagents caveat below — this scoping exists in the code even though the feature isn't user-facing yet) |

Currently `PreToolUse` is the only supported event; the hook engine's input/output parsing is also Claude-Code-compatible (`internal/hooks/input.go` notes "Crush + Claude Code compat").

### Hook Input (stdin JSON)

```jsonc
{
  "event": "PreToolUse",
  "session_id": "313909e",
  "cwd": "/home/user/project",
  "tool_name": "bash",
  "tool_input": { "command": "npm test" }
}
```

### Hook Output (stdout JSON, optional)

```jsonc
{
  "version": 1,
  "decision": "allow",        // "allow" | "deny" | null
  "reason": "LGTM",
  "context": "string or array",
  "updated_input": { "command": "…" },
  "halt": false
}
```

### Exit Code Behavior

| Code | Meaning |
|------|---------|
| `0` | Success; stdout parsed as the JSON envelope above if present |
| `2` | Block/deny the tool call; stderr is used as the deny reason |
| `49` | Halt the entire agent turn; stderr is used as the halt reason |
| Other | Non-blocking error is logged; tool call proceeds |

> **Live-verified 2026-07-23:** Built a real `block-sudo` hook (`crush.json` → `hooks.PreToolUse`, matcher `"bash"`) on Ubuntu 24.04 running crush v0.86.0 with an OpenAI model, and confirmed the exit-2/stderr path exactly as documented: a hook script that `exit 2`s with the deny reason on stderr produced `Tool call blocked by hook. Reason: <exact stderr text>` in the session output, and the underlying `bash` tool call never ran. One thing worth flagging for anyone testing this themselves: printing a `{"decision":"block",...}` stdout envelope (the *Goose*-style schema) does **not** work here — Crush's own schema uses `"decision": "deny"` (not `"block"`), so a hook copy-pasted from another tool's convention will silently fail to surface its custom reason even though the exit-2 block still happens via the stderr fallback.

### Example Config

```jsonc
{
  "hooks": {
    "PreToolUse": [
      {
        "name": "no-rm-rf",
        "matcher": "^bash$",
        "command": "./hooks/no-rm-rf.sh",
        "timeout": 10
      }
    ]
  }
}
```

Default hook timeout is 30 seconds if not specified. A built-in "crush-hooks" skill (`internal/skills/builtin/crush-hooks/SKILL.md`) documents this system to the agent itself.

## Built-in Tools

Derived from the `internal/agent/tools/` source tree:

| Tool | Description |
|------|-------------|
| `bash` | Execute shell commands (with background job support) |
| `view` | Read file contents |
| `write` | Write files |
| `edit` | Apply a single edit to a file |
| `multiedit` | Apply multiple edits to a file in one call |
| `ls` | List directory contents |
| `glob` | Find files by glob pattern |
| `grep` / `rg` | Search file contents (ripgrep-backed) |
| `fetch` | Fetch a URL / web content |
| `download` | Download a file |
| `sourcegraph` | Search public code via Sourcegraph |
| `question` | Ask the user a clarifying question |
| `todos` | Manage an in-session todo/task list |
| `job_kill` / `job_output` | Manage/inspect background shell jobs started by `bash` |
| `diagnostics` | Surface LSP diagnostics |
| `references` | Find symbol references (LSP-backed) |
| `lsp_definition`, `lsp_call_hierarchy`, `lsp_rename`, `lsp_replace_symbol`, `lsp_restart`, `lsp_symbols` | LSP-powered code intelligence tools |
| `list_mcp_resources` / `read_mcp_resource` | Discover and read MCP-exposed resources |
| plus any MCP-server-provided tools | Dynamically registered per configured MCP server |
| `crush_info` / `crush_logs` | Introspect Crush's own runtime info/logs |

Individual tools can be disabled via `options.disabled_tools`, and specific tools can be pre-approved (no permission prompt) via `permissions.allowed_tools`. The `--yolo` CLI flag bypasses all permission prompts.

## MCP Support

Config location: `"mcp"` key in `crush.json` (global or project). Supports three transports (`stdio`, `http`, `sse`), shell-style variable expansion in values (`$VAR`, `${VAR:-default}`, `$(command)`), per-server tool disabling, and OAuth.

```jsonc
{
  "mcp": {
    "filesystem": {
      "type": "stdio",
      "command": "node",
      "args": ["/path/to/mcp-server.js"],
      "timeout": 120,
      "disabled": false,
      "disabled_tools": ["some-tool-name"],
      "env": { "NODE_ENV": "production" }
    },
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/",
      "timeout": 120,
      "headers": { "Authorization": "Bearer $GH_PAT" },
      "oauth": true,
      "oauth_client_id": "Iv1.abc123def456",
      "oauth_client_secret": "$GITHUB_MCP_SECRET",
      "oauth_callback_port": 40704
    },
    "streaming-service": {
      "type": "sse",
      "url": "https://example.com/mcp/sse",
      "headers": { "API-Key": "$(echo $API_KEY)" }
    }
  }
}
```

MCP-exposed resources are reachable through the built-in `list_mcp_resources` / `read_mcp_resource` tools.

## Skills / Commands

Crush implements the open **Agent Skills** standard (the same `SKILL.md` + YAML-frontmatter convention used by Claude Code/Devin) plus a separate custom-commands feature.

**Skills** — discovered from, in order:
- `$CRUSH_SKILLS_DIR`
- `$XDG_CONFIG_HOME/agents/skills` or `~/.config/agents/skills/` (global, tool-agnostic)
- `~/.config/crush/skills/` (global, Crush-specific)
- `.agents/skills` (project, tool-agnostic)
- `.crush/skills` (project, Crush-specific)
- `.claude/skills` (project, Claude Code compat)

Format: Markdown with YAML frontmatter, e.g.:
```yaml
---
name: my-skill
description: A skill invocable as a command
user-invocable: true
disable-model-invocation: true
---
```
Additional paths and disables are configurable via `options.skills_paths` / `options.disabled_skills`. Crush ships built-in skills, e.g. `crush-config`, `crush-hooks`, `jq`, `builtin-skills`, `shell-builtins` (see `internal/skills/builtin/` and `.agents/skills/`).

**Custom commands** — reusable, user-triggerable prompt templates stored as Markdown files (e.g. under `~/.config/crush/commands/` or a project `commands/` dir ❓ exact directory convention not fully confirmed from official docs alone), invoked from the REPL via `/command-name` and discoverable via the `/` command palette (Ctrl+P). Commands can use `$VARIABLES` to render a form collecting free-text input from the user.

## Agent / Subagent Configuration

Crush's internal `Coordinator` manages multiple named agents — at minimum a `"coder"` agent (main interactive agent) and a `"task"` agent (used for sub-tasks/delegation), each with its own Go-template system prompt (`internal/agent/templates/*.md.tpl`). ❓ End-user-facing configuration surface for defining custom named agents (beyond the built-in `coder`/`task`) is not clearly documented in the public README; treat as an internal architecture detail rather than a confirmed user-facing feature.

> **Live-tested 2026-07-23 — subagents are NOT currently user-invokable, despite the scaffolding above.** Ran `crush run "Use a subagent/task tool to create subagent-test.txt..."` on v0.86.0; the main agent just did the work itself via its own `write` tool — no delegation occurred. Checked the built-in tools list directly against the `internal/agent/tools/` source tree on GitHub: there is **no `task`/`agent`/`delegate` tool** among the ~30 built-in tool files (bash, edit, view, grep, glob, etc. — see Built-in Tools above), so there is currently no mechanism for the LLM to invoke the "task" agent even though it's defined in the `Coordinator`. This is confirmed by the `Coordinator` interface's own source comment on `SetMainAgent`: `// INFO: (kujtim) this is not used yet — we will use this when we have multiple agents`. **Conclusion:** Crush's multi-agent architecture is real but not yet wired into a usable feature — don't rely on "ask Crush to delegate to a subagent" working today; the `coder`/`task` split is preparatory scaffolding, not a shipped subagent capability like Claude Code's `Task` tool or Goose's `delegate` tool.

## Multi-Model / Provider Routing

Providers are configured under the `"providers"` key in `crush.json`, supporting several `type`s:

**OpenAI-compatible:**
```jsonc
{
  "providers": {
    "deepseek": {
      "type": "openai-compat",
      "base_url": "https://api.deepseek.com/v1",
      "api_key": "$DEEPSEEK_API_KEY",
      "models": [
        {
          "id": "deepseek-chat",
          "name": "Deepseek V3",
          "cost_per_1m_in": 0.27,
          "cost_per_1m_out": 1.1,
          "context_window": 64000,
          "default_max_tokens": 5000
        }
      ]
    }
  }
}
```

**Anthropic-compatible:**
```jsonc
{
  "providers": {
    "custom-anthropic": {
      "type": "anthropic",
      "base_url": "https://api.anthropic.com/v1",
      "api_key": "$ANTHROPIC_API_KEY",
      "models": [
        { "id": "claude-sonnet-4-20250514", "name": "Claude Sonnet 4", "cost_per_1m_in": 3, "cost_per_1m_out": 15, "context_window": 200000 }
      ]
    }
  }
}
```

**Local models (auto-discovered):**
```jsonc
{
  "providers": {
    "ollama": { "name": "Ollama", "base_url": "http://localhost:11434/v1/", "type": "ollama" },
    "llamacpp": { "name": "llama.cpp", "base_url": "http://localhost:2222", "type": "llamacpp" }
  }
}
```

Out of the box, Crush recognizes standard env vars for built-in providers — including Anthropic, OpenAI, Gemini, Bedrock, GitHub Copilot, Hyper, MiniMax, Vercel AI Gateway, and more (per `AGENTS.md`'s architecture description) — and maps them automatically at load time. API keys entered interactively are persisted to the local ephemeral state file (`~/.local/share/crush/crush.json`). Crush also supports OAuth2 for some providers (e.g., Anthropic "Claude Code" login) with automatic token refresh. Model/provider can be switched at runtime via the in-TUI model picker (Ctrl+L). A `crush login` CLI subcommand and `crush models` / `crush stats` / `crush sessions` subcommands exist per the architecture doc. ❓ Full CLI flag reference (e.g., `--provider`/`--model` flags on `crush run`) not confirmed — a GitHub issue indicates this was requested/discussed but exact current flag support wasn't independently verified.

## Notes

- LSP integration (`"lsp"` key in `crush.json`) lets Crush auto-start language servers (e.g., `gopls`, `typescript-language-server`, `nil`) for diagnostics, symbol lookup, and rename tooling, exposed to the agent via the `lsp_*` and `diagnostics`/`references` tools.
- Built as a spiritual successor / rewrite in the Charm ecosystem, using the same team's `bubbletea`, `lipgloss`, and `glamour` TUI libraries; session/message data is persisted to a local SQLite database.
- Additional `options` flags include `debug`, `debug_lsp`, `disable_notifications`, `disable_provider_auto_update`, `disable_metrics`, and a git commit `attribution` block (trailer style, `generated_with`).
- License is FSL-1.1-MIT: source-available and free for most uses immediately, converts to plain MIT two years after each version's release — not OSI-approved "open source" in the strict sense, though the project markets itself as open source.

## Sources

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| Main README (install, config, LSP, MCP, skills, providers) | https://github.com/charmbracelet/crush/blob/main/README.md | 2026-07-23 | [official] |
| Hooks guide | https://github.com/charmbracelet/crush/blob/main/docs/hooks/README.md | 2026-07-23 | [official] |
| Repo metadata (description, license) | https://github.com/charmbracelet/crush (via `gh api repos/charmbracelet/crush`) | 2026-07-23 | [official] |
| Repository file tree (built-in tools, commands, skills paths) | https://github.com/charmbracelet/crush (via `gh api .../git/trees/main`) | 2026-07-23 | [official] |
| Root `AGENTS.md` (architecture, providers, subagent coordinator) | https://github.com/charmbracelet/crush/blob/main/AGENTS.md | 2026-07-23 | [official] |
| `LICENSE.md` (FSL-1.1-MIT terms) | https://github.com/charmbracelet/crush/blob/main/LICENSE.md | 2026-07-23 | [official] |
| Built-in skills source (`crush-config`, `crush-hooks`, `jq`) | https://github.com/charmbracelet/crush/tree/main/internal/skills/builtin | 2026-07-23 | [official] |
| Custom commands feature discussion | https://github.com/charmbracelet/crush/issues/2219 | 2026-07-23 | [community] |
| Provider/config background (DeepWiki, unofficial secondary source) | https://deepwiki.com/charmbracelet/crush/2.2-configuration | 2026-07-23 | [third-party] |
