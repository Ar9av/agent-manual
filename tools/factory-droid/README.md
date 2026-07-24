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

Primary method (macOS/Linux):

```sh
curl -fsSL https://app.factory.ai/cli | sh
```

Alternative methods:

```sh
# npm
npm install -g droid

# Homebrew
brew install --cask droid

# Windows (PowerShell)
irm https://app.factory.ai/cli/windows | iex
```

Then launch from your project directory:

```sh
droid
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.factory/hooks.json` | Global | User-scoped hook definitions |
| `~/.factory/settings.json` | Global | User settings |
| `.factory/hooks.json` | Project | Project-scoped hook definitions |
| `.factory/settings.json` | Project | Project settings |
| — (Enterprise Controls) | Organization | Org-level managed hook policy |
| `$FACTORY_PROJECT_DIR` | Env var | Project root reference |

## Hooks

Hooks provide deterministic control over Droid's behavior — actions that always happen rather than relying on AI choices.

### Supported Events

| Event | When | Can Block? |
|-------|------|-----------|
| `SessionStart` | Session starts or resumes | ❌ |
| `SessionEnd` | Session ends | ❌ |
| `PreToolUse` | Before tool call | ✅ |
| `PostToolUse` | After tool call completes | ❌ (tool already ran; `decision` sends feedback only) |
| `UserPromptSubmit` | User submits a prompt, before processing | ✅ |
| `Notification` | Droid sends a notification | ❌ |
| `Stop` | Droid finishes responding | ✅ (via JSON `decision`) |
| `SubagentStop` | Sub-droid task completes | ✅ (via JSON `decision`) |
| `PreCompact` | Before context compaction | ❌ |

`PreToolUse` and `PostToolUse` support a `matcher` field (tool name string or regex pattern, e.g. `"Create|Edit|ApplyPatch"`).

### Configuration Example

Hooks are defined in `hooks.json` at the scope root (`~/.factory/hooks.json` user-level, `.factory/hooks.json` project-level, plus an org-level policy via Enterprise Controls). If no `hooks.json` is found, Droid falls back to a `hooks` key inside the matching `settings.json`. The older nested path `.factory/hooks/hooks.json` is still read for backward compatibility, but any new save now writes to `hooks.json` at the scope root:

```json
{
  "PreToolUse": [
    {
      "matcher": "Execute",
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
```

### Hook Input (stdin JSON)

All hook events receive:

```json
{
  "hook_event_name": "PreToolUse",
  "session_id": "...",
  "cwd": "/Users/user/project",
  "transcript_path": "/path/to/transcript.json",
  "permission_mode": "auto-medium"
}
```

Event-specific additional fields:

| Event | Extra fields |
|-------|-------------|
| `PreToolUse` | `tool_name`, `tool_input` |
| `PostToolUse` | `tool_name`, `tool_input`, `tool_response` |
| `UserPromptSubmit` | `prompt` |
| `Stop` / `SubagentStop` | `reason` |

`permission_mode` values: `"off"`, `"spec"`, `"auto-low"`, `"auto-medium"`, `"auto-high"`.

### Blocking Mechanisms

Hooks can block by either exit code or structured JSON output on stdout.

#### Exit Code Behavior

| Code | Meaning |
|------|---------|
| `0` | Continue; stdout shown in transcript (UserPromptSubmit/SessionStart inject stdout as context) |
| `2` | Block — stderr fed back to Droid. PreToolUse blocks the tool call; UserPromptSubmit blocks prompt; Stop/SubagentStop block stoppage |
| Other | Non-blocking error; stderr shown to user, execution continues |

#### JSON Output (advanced)

Return structured JSON on stdout for fine-grained control:

```json
{
  "continue": false,
  "stopReason": "Blocked by policy",
  "suppressOutput": false,
  "systemMessage": "Warning shown to user"
}
```

Event-specific JSON fields:

| Event | JSON field | Values / Notes |
|-------|-----------|----------------|
| `PreToolUse` | `permissionDecision` | `"allow"` / `"deny"` / `"ask"` |
| `PreToolUse` | `permissionDecisionReason` | Shown to user alongside the permission decision |
| `PreToolUse` | `updatedInput` | Override tool input |
| `PostToolUse` | `decision` | `"block"` to send feedback (tool already ran) |
| `PostToolUse` | `hookSpecificOutput.additionalContext` | Extra context for Droid (nested inside hookSpecificOutput) |
| `UserPromptSubmit` | `decision` | `"block"` to block |
| `UserPromptSubmit` | `hookSpecificOutput.additionalContext` | Inject context when prompt is not blocked |
| `SessionStart` | `additionalContext` | Inject context into session |
| `Stop` / `SubagentStop` | `decision` | `"block"` to block |

### Path Best Practices

- Use `$FACTORY_PROJECT_DIR/path/to/script.sh` for project-relative scripts.
- Use absolute paths (`/usr/local/bin/script.sh`, `~/.factory/hooks/script.sh`) for global scripts.
- Never use relative paths — Droid's working directory can change during execution.
- Default hook execution timeout: 60 seconds (configurable per hook).

## Common Hook Use Cases

- **Notifications**: Custom notification on task completion
- **Auto-formatting**: Run Prettier/Black after file edits
- **Logging**: Audit trail of executed commands
- **Feedback**: Enforce code conventions
- **Permissions**: Block production file modifications

## Built-in Tools

| Tool | Description |
|------|-------------|
| `Execute` | Execute shell commands |
| `Read` | Read file contents |
| `Create` | Write/create files |
| `Edit` | Apply edits to files |
| `ApplyPatch` | Apply patch diffs |
| `LS` | List directory contents |
| `Glob` | File pattern matching |
| `Grep` | Search files |
| `WebSearch` | Search the web |
| `FetchUrl` | Fetch a URL |

## MCP Support

MCP servers are configured in dedicated `mcp.json` files (an `mcpServers` object) — **not** `settings.json`: `~/.factory/mcp.json` (user scope) or `.factory/mcp.json` (project scope, also honored from any ancestor directory). Add servers via `droid mcp add <name> <url> --type <http|sse|stdio>` (goes to user config) or interactively via `/mcp`, which also browses a 40+ server registry.

## Custom Droids (Subagents)

Define specialized sub-agents as Markdown files with custom system prompts, tool access, and hooks. Two scopes: project (`<repo>/.factory/droids/`, shared via git) and personal (`~/.factory/droids/`, follows you across workspaces). They're exposed as `subagent_type` targets for the Task tool. Manage via `/droids` in the CLI.

## Plugins

Plugins extend Droid with new tools, hooks, and integrations.

- **User scope**: plugins install to `~/.factory/` (available across all projects)
- **Project scope**: plugins install to `<project>/.factory/` (shared via git)

Install via CLI:

```sh
droid plugin install <plugin@marketplace> [--scope user|project]
```

Or manage interactively with `/plugins`. Plugin manifests live at `.factory-plugin/plugin.json`; hook scripts at `hooks/hooks.json` within the plugin root. Use `${DROID_PLUGIN_ROOT}` to reference plugin files.

## Sources

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| Quickstart / Installation | https://docs.factory.ai/cli/getting-started/quickstart | 2026-07-23 | [official] — install commands unchanged |
| Hooks guide | https://docs.factory.ai/cli/configuration/hooks-guide | 2026-07-23 | [official] — added settings.json fallback + scope-root migration note |
| Hooks reference | https://docs.factory.ai/reference/hooks-reference | 2026-07-23 | [official] — 9 events unchanged, org-level scope added |
| CLI reference | https://docs.factory.ai/reference/cli-reference | 2026-07-23 | [official] — `droid mcp add`/`plugin` commands unchanged |
| Custom droids (subagents) | https://docs.factory.ai/cli/configuration/custom-droids | 2026-07-23 | [official] — added personal scope `~/.factory/droids/` |
| Plugins | https://docs.factory.ai/cli/configuration/plugins | 2026-06-13 | [official] |
| Building plugins | https://docs.factory.ai/guides/building/building-plugins | 2026-06-13 | [official] |
| Logging and analytics (hooks) | https://docs.factory.ai/guides/hooks/logging-analytics | 2026-06-13 | [official] |
