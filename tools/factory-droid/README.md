# Factory Droid

> AI coding agent CLI from Factory.ai with deterministic hook controls.

**Vendor:** Factory.ai | **License:** Proprietary

## Links

- Docs: https://docs.factory.ai
- Hooks guide: https://docs.factory.ai/cli/configuration/hooks-guide
- Hooks reference: https://docs.factory.ai/reference/hooks-reference
- CLI reference: https://docs.factory.ai/reference/cli-reference
- Custom Droids (subagents): https://docs.factory.ai/cli/configuration/custom-droids
- Plugins: https://docs.factory.ai/cli/configuration/plugins
- Building plugins: https://docs.factory.ai/guides/building/building-plugins
- Quickstart: https://docs.factory.ai/cli/getting-started/quickstart

---

## Installation

```sh
npm install -g @factory-ai/droid
droid
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.factory/settings.json` | Global | User settings |
| `.factory/settings.json` | Project | Project settings |
| `$FACTORY_PROJECT_DIR` | Env var | Project root reference |

## Hooks

Hooks provide deterministic control over Droid's behavior — actions that always happen rather than relying on AI choices.

### Supported Events

| Event | When | Can Block? |
|-------|------|-----------|
| `AgentStart` | Agent session begins | ❌ |
| `PreToolUse` | Before tool call | ✅ (exit 2) |
| `PostToolUse` | After tool call | ❌ |

`PreToolUse` and `PostToolUse` support a `matcher` field (case-sensitive tool name pattern).

### Configuration Example

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": "$FACTORY_PROJECT_DIR/hooks/validate.sh" }
        ]
      }
    ],
    "PostToolUse": [
      {
        "hooks": [
          { "type": "command", "command": "/usr/local/bin/audit-log.sh" }
        ]
      }
    ]
  }
}
```

### Hook Input (stdin JSON)

```json
{
  "event": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": { "command": "..." },
  "session_id": "...",
  "project_dir": "/Users/user/project"
}
```

### Exit Code Behavior

| Code | Meaning |
|------|---------|
| `0` | Continue |
| `2` | Block tool (PreToolUse only); stderr sent to Droid |
| Other | Warning shown |

### Path Best Practices

- Use `$FACTORY_PROJECT_DIR/path/to/script.sh` for project-relative scripts.
- Use absolute paths (`/usr/local/bin/script.sh`, `~/.factory/hooks/script.sh`) for global scripts.
- Never use relative paths — Droid's working directory can change during execution.

## Common Hook Use Cases

- **Notifications**: Custom notification on task completion
- **Auto-formatting**: Run Prettier/Black after file edits
- **Logging**: Audit trail of executed commands
- **Feedback**: Enforce code conventions
- **Permissions**: Block production file modifications

## Built-in Tools

| Tool | Description |
|------|-------------|
| `Bash` | Execute shell commands |
| `Read` | Read file contents |
| `Write` | Write files |
| `Edit` | Apply edits |
| `Glob` | File pattern matching |
| `Grep` | Search files |

## MCP Support

Configure MCP servers under `mcpServers` in `settings.json`.

## Custom Droids (Subagents)

Define specialized sub-agents in `.factory/droids/` with custom system prompts, tool access, and hooks.

## Plugins

Plugins extend Droid with new tools, hooks, and integrations. Place in `.factory/plugins/` or load from npm.

## Sources (Official)

| Topic | URL |
|-------|-----|
| Hooks guide | https://docs.factory.ai/cli/configuration/hooks-guide |
| Hooks reference | https://docs.factory.ai/reference/hooks-reference |
| CLI reference | https://docs.factory.ai/reference/cli-reference |
| Custom droids (subagents) | https://docs.factory.ai/cli/configuration/custom-droids |
| Plugins | https://docs.factory.ai/cli/configuration/plugins |
| Building plugins | https://docs.factory.ai/guides/building/building-plugins |
| Quickstart | https://docs.factory.ai/cli/getting-started/quickstart |
