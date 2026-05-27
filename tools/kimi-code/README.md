# Kimi Code

> Moonshot AI's CLI agent powered by the Kimi K2 model.

**Vendor:** Moonshot AI | **License:** Open Source | **Runtime:** Node.js

## Links

- Docs: https://www.kimi.com/code/docs/en/kimi-code-cli/
- Hooks (Beta): https://www.kimi.com/code/docs/en/kimi-code-cli/customization/hooks.html
- Config files: https://www.kimi.com/code/docs/en/kimi-code-cli/configuration/config-files.html
- Agents & Subagents: https://www.kimi.com/code/docs/en/kimi-code-cli/customization/agents.html
- Skills: https://www.kimi.com/code/docs/en/kimi-code-cli/customization/skills.html
- GitHub: https://github.com/MoonshotAI/kimi-cli
- Kimi Code intro: https://www.kimi.com/resources/kimi-code-introduction

---

## Installation

```sh
npm install -g @moonshotai/kimi-cli
kimi
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.kimi/config.toml` | Global | User settings (TOML format) |
| `.kimi/config.toml` | Project | Project settings |

Config format is TOML. Old JSON configs are auto-migrated (backed up as `config.json.bak`).

## Hooks (Beta)

> Hooks are in **Beta** as of May 2026. Schema may change. Fail-open: hook failures do not interrupt workflows.

### Supported Events (13 total)

| Event | When | Filter field | Can Block? |
|-------|------|-------------|-----------|
| `PreToolUse` | Before tool execution | Tool name (regex) | ✅ (exit 2 or JSON) |
| `PostToolUse` | After successful tool execution | Tool name | ❌ |
| `PostToolUseFailure` | After tool fails | Tool name | ❌ |
| `UserPromptSubmit` | Before user input is processed | — | ❌ |
| `Stop` | Agent turn ends normally | — | ✅ (JSON stdout) |
| `StopFailure` | Turn ends due to error | Error type | ❌ |
| `SessionStart` | Session created or resumed | Source | ❌ |
| `SessionEnd` | Session closes | Reason | ❌ |
| `SubagentStart` | Subagent initializes | Agent name | ❌ |
| `SubagentStop` | Subagent completes | Agent name | ❌ |
| `PreCompact` | Before context compaction | Trigger reason | ❌ |
| `PostCompact` | After context compaction | Trigger reason | ❌ |
| `Notification` | On notification delivery | Sink name | ❌ |

### Configuration Format (TOML)

```toml
[[hooks]]
event = "PreToolUse"
matcher = "Bash"
command = "~/.kimi/hooks/validate-bash.sh"
timeout = 30

[[hooks]]
event = "PostToolUse"
command = "~/.kimi/hooks/post-tool.sh"
```

### Exit Code Behavior

| Code | Meaning |
|------|---------|
| `0` | Allow; non-empty stdout injected as context to LLM |
| `2` | Block; stderr sent to LLM as correction/feedback |
| Other | Allow; stderr logged only |

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

MCP server configuration in `.kimi/config.toml` under `[mcp]`.

## Agents & Subagents

Custom agents defined with specific prompts, model selection, and tool restrictions. Subagents invokable from within an agent session.

## Skills

On-demand expertise modules in `.kimi/skills/`.

## Notes

- Hooks are Beta as of May 2026.
- Config is TOML; old JSON configs auto-migrated.
- Uses same skill format as Claude Code.

## Sources (Official)

| Topic | URL |
|-------|-----|
| Hooks (Beta) | https://www.kimi.com/code/docs/en/kimi-code-cli/customization/hooks.html |
| Agents & subagents | https://www.kimi.com/code/docs/en/kimi-code-cli/customization/agents.html |
| Skills | https://www.kimi.com/code/docs/en/kimi-code-cli/customization/skills.html |
| Config files | https://www.kimi.com/code/docs/en/kimi-code-cli/configuration/config-files.html |
| Kimi Code intro | https://www.kimi.com/resources/kimi-code-introduction |
| GitHub repo | https://github.com/MoonshotAI/kimi-cli |
