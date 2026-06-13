# Kimi Code

> Moonshot AI's CLI agent powered by the Kimi K2 model.

**Vendor:** Moonshot AI | **License:** MIT | **Runtime:** Node.js (optional; binary install available)

## Links

- Docs: https://moonshotai.github.io/kimi-code/en/
- Hooks (Beta): https://www.kimi.com/code/docs/en/kimi-code-cli/customization/hooks.html
- Config files: https://www.kimi.com/code/docs/en/kimi-code-cli/configuration/config-files.html
- Agents & Subagents: https://www.kimi.com/code/docs/en/kimi-code-cli/customization/agents.html
- Skills: https://www.kimi.com/code/docs/en/kimi-code-cli/customization/skills.html
- GitHub: https://github.com/MoonshotAI/kimi-code
- Kimi Code intro: https://www.kimi.com/resources/kimi-code-introduction

---

## Installation

**Recommended (no Node.js required):**

```sh
# macOS / Linux
curl -fsSL https://code.kimi.com/kimi-code/install.sh | bash

# Homebrew (macOS / Linux)
brew install kimi-code

# Windows (PowerShell)
irm https://code.kimi.com/kimi-code/install.ps1 | iex
```

**npm (requires Node.js ≥ 22.19.0):**

```sh
npm install -g @moonshot-ai/kimi-code
kimi
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.kimi-code/config.toml` | Global | Agent settings, model, hooks (TOML) |
| `~/.kimi-code/tui.toml` | Global | TUI/UI preferences (TOML) |
| `~/.kimi-code/mcp.json` | Global | MCP server declarations (JSON) |
| `.kimi-code/config.toml` | Project | Project-level overrides |
| `.kimi-code/mcp.json` | Project | Project-local MCP servers |

Config format is TOML. Override the default directory with `KIMI_CODE_HOME`. Use `/reload` inside the CLI to apply config changes without restarting; `/reload-tui` for UI preferences only.

## Hooks (Beta)

> Hooks are in **Beta** as of May 2026. Schema may change. Fail-open: hook failures do not interrupt workflows.

### Supported Events (16 total)

| Event | When | Filter field | Can Block? |
|-------|------|-------------|-----------|
| `PreToolUse` | Before tool execution (before permission checks) | Tool name (regex) | ✅ (exit 2 or JSON) |
| `PostToolUse` | After successful tool execution | Tool name | ❌ |
| `PostToolUseFailure` | After tool fails or is blocked | Tool name | ❌ |
| `UserPromptSubmit` | When user sends a message; text appended to context | — | ✅ (blocks model call for turn) |
| `Stop` | Agent turn ends normally | — | ✅ (JSON stdout) |
| `StopFailure` | Turn ends due to error | Error type | ❌ |
| `SessionStart` | Session created or resumed | `startup` or `resume` | ❌ |
| `SessionEnd` | Session closes | `exit` | ❌ |
| `SubagentStart` | Subagent initializes | Agent name | ❌ |
| `SubagentStop` | Subagent completes | Agent name | ❌ |
| `PreCompact` | Before context compaction | `manual` or `auto` | ❌ |
| `PostCompact` | After context compaction | `manual` or `auto` | ❌ |
| `Notification` | On notification delivery | Notification type | ❌ |
| `Interrupt` | User interrupts the current turn (Esc) | — | ❌ |
| `PermissionRequest` | Before user approval prompt is shown | Tool name | ❌ |
| `PermissionResult` | After user approval completes | Tool name | ❌ |

### Configuration Format (TOML)

```toml
[[hooks]]
event = "PreToolUse"
matcher = "Bash"
command = "~/.kimi-code/hooks/validate-bash.sh"
timeout = 30

[[hooks]]
event = "PostToolUse"
command = "~/.kimi-code/hooks/post-tool.sh"
```

### Exit Code Behavior

| Code | Meaning |
|------|---------|
| `0` | Allow; non-empty stdout injected as context to LLM |
| `2` | Block; stderr sent to LLM as correction/feedback |
| Other | Allow; stderr logged only (fail-open) |

For `Stop` blocking, exit 0 and output structured JSON stdout (not exit code 2):
```json
{ "decision": "block", "reason": "Tests failed — please fix before stopping." }
```

### Hook Input (stdin JSON)

```json
{
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": { "command": "npm test" },
  "session_id": "abc123",
  "cwd": "/Users/user/project"
}
```

### Structured Block (alternative to exit 2)

```json
{
  "permissionDecision": "deny",
  "permissionDecisionReason": "Command not allowed by policy"
}
```

## Built-in Tools

| Tool | Description |
|------|-------------|
| `Bash` | Execute shell commands |
| `Read` | Read file contents |
| `Write` | Write files |
| `Edit` | Apply edits |
| `Glob` | File pattern matching |
| `Grep` | Search files |
| `WebFetch` | Fetch URLs |
| `WebSearch` | Web search |
| `SendDMail` | Send email |
| `ReadMediaFile` | Read media/image files |
| `EnterPlanMode` | Enter plan mode |
| `ExitPlanMode` | Exit plan mode |
| `TaskList` / `TaskOutput` / `TaskStop` | Task management |

## MCP Support

MCP server configuration in `.kimi-code/mcp.json` (project) or `~/.kimi-code/mcp.json` (global), manageable conversationally via `/mcp-config`.

## Agents & Subagents

Custom agents defined with specific prompts, model selection, and tool restrictions. Built-in subagents: `coder`, `explore`, `plan`. Subagents run in isolated contexts and are invokable from within an agent session.

## Skills

On-demand expertise modules in `.kimi-code/skills/`.

## Editor Integration (ACP)

Kimi Code CLI speaks the [Agent Client Protocol](https://agentclientprotocol.com/), enabling integration with Zed, JetBrains, and other ACP-compatible editors via `kimi acp`.

## Notes

- Hooks are Beta as of May 2026.
- Config is TOML; there is no documented JSON auto-migration from a prior format.
- Uses same skill format as Claude Code.
- `kimi-cli` repo (MoonshotAI/kimi-cli) is being wound down; canonical repo is MoonshotAI/kimi-code.

## Sources

| Topic | URL | Label |
|-------|-----|-------|
| Hooks event reference | https://www.kimi.com/code/docs/en/kimi-code-cli/customization/hooks.html | [official] |
| Config files | https://www.kimi.com/code/docs/en/kimi-code-cli/configuration/config-files.html | [official] |
| Config files (mirror) | https://moonshotai.github.io/kimi-code/en/configuration/config-files | [official mirror] |
| Getting started / npm install | https://moonshotai.github.io/kimi-code/en/guides/getting-started | [official mirror] |
| Agents & subagents | https://www.kimi.com/code/docs/en/kimi-code-cli/customization/agents.html | [official] |
| Skills | https://www.kimi.com/code/docs/en/kimi-code-cli/customization/skills.html | [official] |
| Kimi Code intro | https://www.kimi.com/resources/kimi-code-introduction | [official] |
| GitHub repo | https://github.com/MoonshotAI/kimi-code | [github] |
| Legacy repo (winding down) | https://github.com/MoonshotAI/kimi-cli | [github] |
