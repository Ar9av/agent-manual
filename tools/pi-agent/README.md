# Pi Coding Agent

> Terminal AI coding agent with a rich hook system, subagents, browser, and LSP integration.

**Vendor:** earendil-works / can1357 | **License:** Open Source | **Runtime:** Node.js / TypeScript

## Links

- GitHub (pi-mono): https://github.com/earendil-works/pi
- GitHub (oh-my-pi): https://github.com/can1357/oh-my-pi
- Pi YAML hooks package: https://pi.dev/packages/pi-yaml-hooks
- Hook docs (oh-my-pi): https://github.com/can1357/oh-my-pi/blob/main/docs/hooks.md
- Extensions docs: https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/docs/extensions.md
- Awesome Pi Agent: https://github.com/qualisero/awesome-pi-agent

---

## Installation

```sh
npm install -g @earendil/pi
pi
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.pi/agent/settings.json` | Global | User settings, model, theme |
| `.pi/settings.json` | Project | Project settings |
| `hooks.yaml` | Project | YAML-based hook config (via `pi-yaml-hooks` package) |

## Hooks

Pi supports two hook mechanisms: TypeScript extensions (full power) and the `pi-yaml-hooks` package (YAML config, no code).

### YAML Hooks (via `pi-yaml-hooks` package)

Install as a Pi package, then configure in `hooks.yaml`.

```yaml
# hooks.yaml
hooks:
  - on: tool.before.bash
    action:
      bash: |
        if echo "$TOOL_INPUT" | grep -q "rm -rf"; then
          echo "Blocked" >&2
          exit 2
        fi

  - on: tool.after.*
    action:
      notify:
        title: "Tool completed"
        body: "{{tool_name}} finished"

  - on: session.created
    action:
      bash: ~/.pi/hooks/setup.sh

  - on: file.changed
    action:
      bash: "prettier --write '{{file}}'"
      setStatus: "Formatted {{file}}"
```

### Supported Events

| Event | When | Can Block? |
|-------|------|-----------|
| `tool.before.*` | Before tool call (glob matches tool name) | ✅ (exit 2) |
| `tool.after.*` | After tool call | ❌ |
| `file.changed` | File modified | ❌ |
| `session.created` | Session start | ❌ |
| `session.idle` | Session goes idle | ❌ |
| `session.deleted` | Session end | ❌ |

### Available Hook Actions

| Action | Description |
|--------|-------------|
| `bash` | Run shell command |
| `tool` | Call another Pi tool |
| `notify` | Show UI notification |
| `confirm` | Prompt user for confirmation |
| `setStatus` | Update status bar |

### TypeScript Extension Hooks

```typescript
// .pi/extensions/my-extension.ts
export default {
  hooks: {
    'tool.before.bash': async ({ input, session }) => {
      if (input.command.includes('rm -rf')) {
        return { block: true, reason: 'Destructive command blocked' }
      }
    },
    'after_provider_response': async ({ status, headers }) => {
      // Inspect provider HTTP response
    }
  },
  tools: { /* custom tools */ },
  commands: { /* custom slash commands */ }
}
```

### `after_provider_response` Hook

New hook (2026) that lets extensions inspect provider HTTP status codes and headers immediately after response creation, before stream consumption.

## Built-in Tools

| Tool | Description |
|------|-------------|
| `bash` | Execute shell commands (with `spawn` hook for adjusting cmd/cwd/env) |
| `read_file` | Read file contents |
| `write_file` | Write files |
| `edit_file` | Apply edits |
| `glob` | File pattern matching |
| `grep` | Search files |
| `browser` | Browser automation |
| `lsp` | Language server queries |
| `subagent` | Spawn subagents |

## MCP Support

Configure MCP servers in `settings.json` under `mcp`.

## Subagents

Pi supports subagents as first-class tools. Use the `subagent` tool to delegate tasks to specialized agents.

## Settings Options

```json
{
  "model": "claude-opus-4",
  "theme": "dark",
  "compaction": { "enabled": true, "threshold": 0.8 },
  "retry": { "maxAttempts": 3 },
  "mcp": { "servers": {} }
}
```

## Notes

- `pi-yaml-hooks` (v2026.5.12) is the recommended no-code hook approach.
- TypeScript extensions give full access to the agent internals.
- The `bash` tool's `spawn` hook lets you mutate command, cwd, and env before execution.
- Pi includes browser automation and LSP integration as first-party tools.

## Sources

> **Note on hook documentation:** The primary hook docs are from `oh-my-pi` (`can1357/oh-my-pi`), which is a **community-maintained** project, not the primary vendor repo (`earendil-works/pi`). Hook behavior may diverge between the two. The `pi-yaml-hooks` package and TypeScript extensions docs are from the primary vendor ecosystem.

| Topic | Source type | URL |
|-------|-------------|-----|
| pi-yaml-hooks package | Vendor (pi.dev) | https://pi.dev/packages/pi-yaml-hooks |
| Hooks docs (oh-my-pi) | Community | https://github.com/can1357/oh-my-pi/blob/main/docs/hooks.md |
| Extensions docs | Vendor (GitHub) | https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/docs/extensions.md |
| GitHub (pi-mono) — primary | Vendor | https://github.com/earendil-works/pi |
| GitHub (oh-my-pi) | Community | https://github.com/can1357/oh-my-pi |
| Awesome Pi Agent | Community | https://github.com/qualisero/awesome-pi-agent |
