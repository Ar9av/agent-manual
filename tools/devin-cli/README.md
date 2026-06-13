# Devin CLI (Devin for Terminal)

> Cognition's AI software engineer, available as a terminal CLI.

**Vendor:** Cognition AI | **License:** Proprietary

## Links

- Docs: https://docs.devin.ai/cli
- Extensibility overview: https://docs.devin.ai/cli/extensibility
- Hooks overview: https://docs.devin.ai/cli/extensibility/hooks/overview

---

## Installation

**macOS / Linux / WSL:**
```sh
curl -fsSL https://cli.devin.ai/install.sh | bash
```

**macOS (Homebrew):**
```sh
brew install --cask devin-cli
brew upgrade devin  # for upgrades
```

**Windows (install via PowerShell — the `irm`/`iex` install command requires PowerShell; after install, the CLI itself runs in Git Bash too):**
```powershell
irm https://static.devin.ai/cli/setup.ps1 | iex
```

**Devin Desktop (Enterprise):** Command Palette (Cmd+Shift+P / Ctrl+Shift+P) → Install Devin CLI

> ⚠️ `pip install devin-cli` installs an **unofficial third-party PyPI package** (author: Revanth Pobala) that is not affiliated with Cognition AI. Do not use it as the official Devin CLI.

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.config/devin/config.json` | Global (Unix) | User settings |
| `%APPDATA%\devin\config.json` | Global (Windows) | User settings |
| `.devin/config.json` | Project | Project config, MCP, permissions |
| `.devin/config.local.json` | Project (git-ignored) | Personal overrides |
| `.devin/hooks.v1.json` | Project | Lifecycle hooks (recommended standalone file) |
| `.devin/skills/` | Project | Custom skill definitions |
| `.devin/agents/` | Project | Custom subagent profiles |

Files containing `.local.` are auto-excluded from git.

> Legacy paths (`~/.config/cognition/`, `.cognition/`) are still read with deprecation warnings; migrate to `.devin/`.

**Also reads** (compatibility): `AGENTS.md`, `AGENT.md`, `CLAUDE.md`, `.cursor/rules/*.md`, `.cursorrules`, `.windsurf/rules/*.md`, `.claude/`

## Hooks

8 hook events, with two hook types: **command** (shell script) and **prompt** (LLM evaluation).

### Supported Events

| Event | When | Can Block? |
|-------|------|-----------|
| `PreToolUse` | Before a tool executes | ✅ (exit 2 or stdout JSON) |
| `PostToolUse` | After a tool finishes | ❌ (cannot block — post-execution) |
| `PermissionRequest` | When a permission decision is needed | ✅ |
| `UserPromptSubmit` | When the user submits a message | ✅ (exit 2 blocks the prompt) |
| `Stop` | When the agent wants to stop | ✅ |
| `SessionStart` | When a session begins | ❌ (cannot block — lifecycle event) |
| `SessionEnd` | When a session ends | ❌ (cannot block — lifecycle event) |
| `PostCompaction` | After context compaction (summary on stdin) | ❌ (cannot block — post-execution) |

### Configuration Format

Standalone file (`.devin/hooks.v1.json`) — top-level keys are event names:

```json
{
  "PreToolUse": [
    {
      "matcher": "exec",
      "hooks": [
        {
          "type": "command",
          "command": "./scripts/validate.sh",
          "timeout": 10
        }
      ]
    }
  ],
  "PostToolUse": [
    {
      "hooks": [
        { "type": "command", "command": "./scripts/audit.sh" }
      ]
    }
  ]
}
```

When embedded in `config.json` or `.claude/settings.json`, the hooks object nests under a `"hooks"` key.

### Hook Types

**Command** — runs a shell script; receives JSON on stdin, can output JSON to stdout:
```json
{ "type": "command", "command": "./scripts/validate.sh", "timeout": 10 }
```

**Prompt** — evaluates an LLM prompt instead of running a shell command:
```json
{ "type": "prompt", "prompt": "Check that the command is safe before running it." }
```

### stdin Schema

```json
{
  "hook_event_name": "PreToolUse",
  "tool_name": "exec",
  "tool_input": { "command": "rm -rf /" }
}
```

### stdout Schema (optional)

```json
{ "decision": "block", "reason": "Destructive command blocked by policy" }
```

Valid `decision` values: `"approve"`, `"block"`, `"deny"`.

### Exit Code Behavior

| Code | Meaning |
|------|---------|
| `0` | Continue — stdout parsed as JSON if present |
| `2` | Block action |
| Other | Error logged; execution continues |

### Environment Variables

`DEVIN_PROJECT_DIR` — automatically set to the project root directory during hook execution.

### Verification

Run `/hooks` slash command inside Devin CLI to list all active hooks and their source config files.

## Configuration Example (`config.json`)

```json
{
  "model": "claude-opus-4",
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["./mcp/server.js"]
    }
  },
  "permissions": {
    "allow": ["Bash(git *)", "Read(**)"],
    "deny": ["Bash(rm -rf *)"]
  }
}
```

## Built-in Tools

| Tool | Description |
|------|-------------|
| `Bash` | Execute shell commands |
| `Read` | Read files |
| `Write` | Write files |
| `Edit` | Apply edits |
| `Glob` | File search by pattern |
| `Grep` | Text search in files |
| `WebFetch` | Fetch URLs |
| `WebSearch` | Web search |

## MCP Support

Configure under `mcpServers` in `config.json`. Supports all standard MCP server types, including OAuth and legacy SSE protocol.

## Skills

Markdown files with YAML frontmatter stored in `.devin/skills/<name>/SKILL.md` (subagent profiles use `AGENT.md` at `.devin/agents/<name>/AGENT.md`):

```yaml
---
name: skill-name
description: What this skill does
subagent: true
model: claude-opus-4
allowed-tools:
  - Bash
  - Read
triggers:
  - user
  - model
---
Skill instructions here.
```

| Location | Scope | Git-tracked |
|----------|-------|-------------|
| `.devin/skills/<name>/` | Project | Yes |
| `~/.config/devin/skills/<name>/` | Global (Unix) | No |
| `%APPDATA%\devin\skills\<name>\` | Global (Windows) | No |

## Subagent Profiles

Custom subagent profiles in `.devin/agents/` with distinct system prompts, tool access, and model selection.

## Notes

- `hooks.v1.json` format mirrors Claude Code hooks — hooks from Claude Code projects often work directly.
- `config.local.json` pattern allows personal overrides without committing credentials.
- User-level config at `~/.config/devin/` applies across all projects.
- Background auto-updates on macOS/Linux: new releases download while Devin runs; next invocation picks up the latest version. Homebrew users must upgrade manually via `brew upgrade devin`.

## Sources

| Topic | URL | Label |
|-------|-----|-------|
| CLI docs (main) | https://docs.devin.ai/cli | [official] |
| Hooks overview | https://docs.devin.ai/cli/extensibility/hooks/overview | [official] |
| Stable changelog | https://docs.devin.ai/cli/changelog/stable | [official] |
| Unofficial PyPI package (NOT official) | https://pypi.org/project/devin-cli/ | [third-party] |
| Unofficial PyPI source (NOT official) | https://github.com/revanthpobala/devin-cli | [github] |
