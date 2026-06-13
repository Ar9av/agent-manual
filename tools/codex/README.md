# Codex CLI

> OpenAI's lightweight coding agent that runs in your terminal.

**Vendor:** OpenAI | **License:** Apache 2.0 | **Runtime:** Rust

## Links

- Docs: https://developers.openai.com/codex
- CLI reference: https://developers.openai.com/codex/cli/reference
- Hooks: https://developers.openai.com/codex/hooks
- Skills: https://developers.openai.com/codex/skills
- AGENTS.md guide: https://developers.openai.com/codex/guides/agents-md
- GitHub: https://github.com/openai/codex
- Config schema: https://github.com/openai/codex/blob/main/docs/config.md

---

## Installation

```sh
# Shell script (Mac/Linux)
curl -fsSL https://chatgpt.com/codex/install.sh | sh

# PowerShell (Windows)
powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 | iex"

# npm
npm install -g @openai/codex

# Homebrew
brew install --cask codex
```

Manual binary downloads for macOS (Apple Silicon, x86_64) and Linux (x86_64, arm64) are available on GitHub Releases.

> Note: `cargo install codex` is **not supported**.

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.codex/config.toml` | Global | User settings, model, hooks |
| `~/.codex/hooks.json` | Global | Hooks (separate from config.toml) |
| `.codex/config.toml` | Project | Project overrides |
| `.codex/hooks.json` | Project | Project hooks |
| `requirements.toml` | Managed | Admin-enforced policies |
| `AGENTS.md` | Project | Natural-language instructions |

If a single config layer has both `hooks.json` and inline `[hooks]` in `config.toml`, Codex loads both and warns. Prefer one representation per layer.

## Hooks

Hooks are an extensibility framework for injecting scripts into the agentic loop.

### Enabling/Disabling

```toml
[features]
hooks = true   # default; set false to disable all hooks
```

To force disable at admin level:
```toml
# requirements.toml
allow_managed_hooks_only = true
```

> Note: `features.codex_hooks` is a deprecated alias for `features.hooks`.

### Supported Events

| Event | When | Scope |
|-------|------|-------|
| `SessionStart` | Session begins (startup, resume, clear, or compact) | Session |
| `SubagentStart` | A subagent initializes | Turn |
| `UserPromptSubmit` | User submits a prompt | Turn |
| `PreToolUse` | Before a tool executes (Bash, apply_patch, MCP) | Turn |
| `PermissionRequest` | When approval is needed (shell escalation, network access) | Turn |
| `PostToolUse` | After tool output is generated | Turn |
| `PreCompact` | Before conversation compaction | Turn |
| `PostCompact` | After conversation compaction | Turn |
| `SubagentStop` | A subagent finishes | Turn |
| `Stop` | A conversation turn completes | Turn |

> Note: As of mid-2026, hooks reliably fire for Bash tool calls but not reliably for `apply_patch` or most MCP tool calls ([issue #16732](https://github.com/openai/codex/issues/16732)).

### Hook Input

Hooks receive data via **stdin as JSON** (not environment variables). Common fields:

```json
{
  "session_id": "string",
  "transcript_path": "string | null",
  "cwd": "string",
  "hook_event_name": "string",
  "model": "string",
  "turn_id": "string",
  "permission_mode": "default|acceptEdits|plan|dontAsk|bypassPermissions"
}
```

Event-specific fields (e.g. `tool_name`, `tool_input`, `tool_response`) are appended for relevant events.

### Hook Output

Hooks return JSON to stdout. Universal fields (most events):

```json
{
  "continue": true,
  "stopReason": "optional string",
  "systemMessage": "optional string",
  "suppressOutput": false
}
```

Event-specific shapes:
- **PreToolUse** — deny with `"permissionDecision": "deny"`, rewrite with `"updatedInput"`, or inject `"additionalContext"`
- **PermissionRequest** — return `"decision": {"behavior": "allow"|"deny", "message": "..."}`
- **PostToolUse** — block with `"decision": "block"` (does not undo side effects)
- **Stop / SubagentStop** — `"decision": "block"` prompts continuation

Exit code `2` with stderr text also signals blocking/denial for some events.

### Hook Security Model

- Non-managed command hooks require explicit user review before first execution.
- Trust is persisted keyed by hook hash; changed hooks re-prompt for approval.
- Managed hooks (from MDM / `requirements.toml`) bypass user review.
- Plugin hooks must be trusted individually.
- Use `/hooks` in the CLI to inspect, trust, or disable hooks.
- Use `--dangerously-bypass-hook-trust` for one-off automation.

### Configuration Example (TOML)

```toml
[[hooks.PreToolUse]]
matcher = "^Bash$"

[[hooks.PreToolUse.hooks]]
type = "command"
command = '/usr/bin/python3 "$(git rev-parse --show-toplevel)/.codex/hooks/pre_tool_use.py"'
timeout = 30
statusMessage = "Checking Bash command"

[[hooks.PostToolUse]]
matcher = "^Bash$"

[[hooks.PostToolUse.hooks]]
type = "command"
command = "~/.codex/hooks/log-tool.sh"
```

### Configuration Example (JSON)

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "^Bash$",
        "hooks": [
          {
            "type": "command",
            "command": "~/.codex/hooks/pre_tool_use.py",
            "timeout": 30,
            "statusMessage": "Checking Bash command"
          }
        ]
      }
    ]
  }
}
```

### Matcher Patterns

| Event | Filters By | Example values |
|-------|-----------|----------------|
| `PreToolUse`, `PostToolUse`, `PermissionRequest` | Tool name | `Bash`, `apply_patch`, `mcp__filesystem__read_file` |
| `SessionStart` | Start source | `startup`, `resume`, `clear`, `compact` |
| `PreCompact`, `PostCompact` | Trigger type | `manual`, `auto` |
| `UserPromptSubmit`, `Stop` | N/A | Matcher ignored |

Use `"*"`, `""`, or omit `matcher` to match all.

## Built-in Tools

| Tool | Description |
|------|-------------|
| `shell` | Execute shell commands |
| `read_file` | Read file contents |
| `write_file` | Write files |
| `patch_file` | Apply diffs |
| `list_files` | List directory |
| `search` | Search files |

## MCP Support

- Configure MCP servers in `config.toml` under `[mcp]`
- Servers provide additional tools to the agent

## Agent Skills

Skills are reusable, on-demand expertise modules. Configure in `config.toml` or place in `.codex/skills/`.

## AGENTS.md

Place `AGENTS.md` at the repo root to give the agent persistent natural-language instructions. Supports sections for tools, style, and constraints.

## GitHub Action

Codex ships a first-party GitHub Action for running automated coding tasks in CI:
https://developers.openai.com/codex/github-action

## Sources

| Topic | URL | Label |
|-------|-----|-------|
| Hooks reference | https://developers.openai.com/codex/hooks | [official] |
| CLI reference | https://developers.openai.com/codex/cli/reference | [official] |
| CLI features | https://developers.openai.com/codex/cli/features | [official] |
| Config reference | https://developers.openai.com/codex/config-reference | [official] |
| Advanced config | https://developers.openai.com/codex/config-advanced | [official] |
| Config file schema | https://github.com/openai/codex/blob/main/docs/config.md | [github] |
| AGENTS.md guide | https://developers.openai.com/codex/guides/agents-md | [official] |
| Skills | https://developers.openai.com/codex/skills | [official] |
| GitHub Action | https://developers.openai.com/codex/github-action | [official] |
| Main docs | https://developers.openai.com/codex | [official] |
| GitHub repo | https://github.com/openai/codex | [github] |
| Issue: hooks not firing for apply_patch/MCP | https://github.com/openai/codex/issues/16732 | [github] |
