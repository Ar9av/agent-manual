# Devin CLI (Devin for Terminal)

> Cognition's AI software engineer, available as a terminal CLI.

**Vendor:** Cognition AI | **License:** Proprietary

## Links

- Docs: https://cli.devin.ai/docs
- Extensibility overview: https://cli.devin.ai/docs/extensibility
- Hooks overview: https://cli.devin.ai/docs/extensibility/hooks/overview

---

## Installation

```sh
pip install devin-cli
devin
```

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

**Also reads** (compatibility): `AGENTS.md`, `AGENT.md`, `CLAUDE.md`, `.cursor/rules/*.md`, `.cursorrules`, `.windsurf/rules/*.md`, `.claude/`

## Hooks

7 hook events, with two hook types: **command** (shell script) and **prompt** (LLM evaluation).

### Supported Events

| Event | When | Can Block? |
|-------|------|-----------|
| `PreToolUse` | Before tool call | ✅ (exit 2 or stdout JSON) |
| `PostToolUse` | After tool completes | ❌ |
| `PermissionRequest` | Permission decision needed | ✅ |
| `UserPromptSubmit` | User submits a message | ❌ |
| `Stop` | Agent wants to stop | ✅ |
| `SessionStart` | Session begins | ❌ |
| `SessionEnd` | Session ends | ❌ |

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

`DEVIN_PROJECT_DIR` — set to project root during hook execution.

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

Configure under `mcpServers` in `config.json`. Supports all standard MCP server types.

## Skills

Markdown files with YAML frontmatter stored in `.devin/skills/<name>/`:

```yaml
---
name: skill-name
description: What this skill does
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

## Sources (Official)

| Topic | URL |
|-------|-----|
| Extensibility overview | https://cli.devin.ai/docs/extensibility |
| Hooks overview | https://cli.devin.ai/docs/extensibility/hooks/overview |
| Main docs | https://cli.devin.ai/docs |
