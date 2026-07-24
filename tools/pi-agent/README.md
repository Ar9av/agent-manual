# Pi Coding Agent

> Terminal AI coding agent with a rich hook system, browser, and LSP integration.

**Vendor:** earendil-works / can1357 | **License:** Open Source | **Runtime:** Node.js / TypeScript

## Links

- GitHub (pi-mono): https://github.com/earendil-works/pi
- GitHub (oh-my-pi): https://github.com/can1357/oh-my-pi
- Pi YAML hooks package: https://pi.dev/packages/pi-yaml-hooks
- Hook docs (oh-my-pi): https://github.com/can1357/oh-my-pi/blob/main/docs/hooks.md
- Extensions docs: https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/extensions.md (repo formerly `badlogic/pi-mono`, which now 301-redirects here)
- Awesome Pi Agent: https://github.com/qualisero/awesome-pi-agent ⚠️ **Archived June 3, 2026 — outdated, read-only**

---

## Installation

Primary install (recommended):

```sh
curl -fsSL https://pi.dev/install.sh | sh
```

Via npm / pnpm / bun (note: `--ignore-scripts` required):

```sh
npm install -g --ignore-scripts @earendil-works/pi-coding-agent
# or
pnpm add -g --ignore-scripts @earendil-works/pi-coding-agent
# or
bun add -g --ignore-scripts @earendil-works/pi-coding-agent
```

> The package name is `@earendil-works/pi-coding-agent`. The name `@earendil/pi` is **incorrect**.

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.pi/agent/settings.json` | Global | User settings, model, theme |
| `.pi/settings.json` | Project | Project settings |
| `~/.pi/agent/hook/hooks.yaml` | Global | YAML-based hook config (via `pi-yaml-hooks` package) |
| `./.pi/hook/hooks.yaml` | Project | Project-scoped YAML hook config (trusted repos only) |

> `hooks.yaml` does **not** live at the project root. Global paths checked in order: `~/.pi/agent/hook/hooks.yaml`, then `~/.pi/agent/hooks.yaml`. Project paths checked in order: `./.pi/hook/hooks.yaml`, then `./.pi/hooks.yaml`.

## Hooks

Pi supports two hook mechanisms: TypeScript extensions (full power) and the `pi-yaml-hooks` package (YAML config, no code). These are **separate systems with different event naming conventions**.

### Hook System Comparison

| Aspect | `pi-yaml-hooks` | `pi-coding-agent` extensions (TypeScript) |
|--------|----------------|------------------------------------------|
| Event style | dot-notation (`tool.before.bash`) | snake_case (`tool_call`, `session_start`) |
| Config format | YAML (`hooks.yaml`) | TypeScript/JavaScript |
| Install | `pi install npm:pi-yaml-hooks` | Built into `@earendil-works/pi-coding-agent` |
| Vendor? | Community (KristjanPikhof, MIT) | Core package (earendil-works) |

### YAML Hooks (via `pi-yaml-hooks` package)

Install as a Pi package:

```sh
pi install npm:pi-yaml-hooks
```

Then configure at `~/.pi/agent/hook/hooks.yaml` (global) or `./.pi/hook/hooks.yaml` (project):

```yaml
# ~/.pi/agent/hook/hooks.yaml
hooks:
  - event: tool.before.bash
    actions:
      - bash: |
          if echo "$TOOL_INPUT" | grep -q "rm -rf"; then
            echo "Blocked" >&2
            exit 2
          fi

  - event: tool.after.*
    actions:
      - notify:
          title: "Tool completed"
          body: "{{tool_name}} finished"

  - event: session.created
    actions:
      - bash: ~/.pi/hooks/setup.sh

  - event: file.changed
    actions:
      - bash: "prettier --write '{{file}}'"
      - setStatus: "Formatted {{file}}"
```

> **Live-tested 2026-07-23 on pi-coding-agent v0.81.1 with `pi-yaml-hooks` (installed fresh via `pi install npm:pi-yaml-hooks`) and `gpt-4o-mini` via `--provider openai`.** Confirmed working: the hook loads (`[pi-yaml-hooks] Loaded 1 hook`), fires on every `tool.before.bash` call, and **unconditional `exit 2` genuinely blocks the tool call** (the model reported the command as blocked, and no output from the intended command ever appeared). **Not confirmed as documented:** the example above reads the triggering command via a `$TOOL_INPUT` environment variable inside the `bash:` action — in this test, `$TOOL_INPUT` was empty and, in fact, `env` inside the hook showed **no `TOOL_*`-prefixed variables at all**. Mustache-style templating in the bash string itself (`{{tool_input}}`, `{{command}}`, `{{input}}`, guessed from the `{{file}}` pattern used elsewhere in this same doc) also did not get substituted — the literal `{{...}}` text passed through unchanged. **Bottom line:** hook firing and blocking are real and confirmed; how to actually read the command/tool-input text from inside a `pi-yaml-hooks` bash action remains unconfirmed by this pass — don't copy the `$TOOL_INPUT` pattern from the example below without verifying it against your installed version first.

### Supported Events (pi-yaml-hooks — dot-notation)

| Event | When | Can Block? |
|-------|------|-----------|
| `tool.before.*` | Before tool call (glob matches tool name) | ✅ (`stop` action or `exit 2`) |
| `tool.after.*` | After tool call | ❌ |
| `file.changed` | File modified | ❌ |
| `session.created` | Session start | ❌ |
| `session.idle` | Agent turn finishes with no pending messages | ❌ |
| `session.deleted` | Session end / shutdown | ❌ |

Bash actions receive hook context as JSON on stdin and injected `PI_*` env vars (e.g. `PI_PROJECT_DIR`, `PI_SESSION_ID`).

### Supported Events (pi-coding-agent TypeScript extensions — snake_case)

The core `@earendil-works/pi-coding-agent` package exposes a richer event set registered via `pi.on(...)` in TypeScript extensions. These snake_case events are **not** from oh-my-pi — oh-my-pi is a separate community project that happens to use a similar convention:

`session_start`, `session_before_switch`, `session_switch`, `session_before_compact`, `session_compact`, `session_shutdown`, `before_agent_start`, `agent_start`, `agent_end`, `turn_start`, `turn_end`, `tool_call` (pre-execution), `tool_result` (post-execution), `auto_compaction_start`, `auto_compaction_end`, and others.

> Do not mix dot-notation events with TypeScript extensions, or snake_case events with pi-yaml-hooks — the two hook systems are incompatible with each other.

### Available Hook Actions (pi-yaml-hooks)

| Action | Description |
|--------|-------------|
| `bash` | Run shell command |
| `stop` | Block the tool call (primary blocking mechanism for `tool.before.*`) |
| `tool` | Send follow-up prompt into current session |
| `notify` | Show UI notification |
| `confirm` | Prompt user for confirmation |
| `setStatus` | Update status bar |

### TypeScript Extension Hooks

```typescript
// .pi/extensions/my-extension.ts
import { pi } from '@earendil-works/pi-coding-agent'

pi.on('tool_call', async (event, ctx) => {
  if (event.toolName === 'bash' && event.input.command.includes('rm -rf')) {
    return { block: true, reason: 'Destructive command blocked' }
  }
})

pi.on('after_provider_response', async (event, ctx) => {
  // Inspect provider HTTP response (status, headers)
})
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

## MCP Support

Pi does **not** include MCP support as a built-in feature. Per pi.dev: "No MCP." MCP can be added by building a custom extension that implements MCP connectivity.

> Remove the `mcp` key from `settings.json` — it is not a recognized built-in option.

## Subagents

Pi does **not** ship subagents as a first-class built-in tool. Per pi.dev: "No sub-agents." The recommended approach is to spawn Pi instances via tmux, or implement subagent behavior via extensions.

## Settings Options

```json
{
  "model": "claude-opus-4",
  "theme": "dark",
  "compaction": { "enabled": true, "threshold": 0.8 },
  "retry": { "maxAttempts": 3 }
}
```

> The `mcp.servers` key is not a built-in setting. MCP must be added via extensions.

## Notes

- `pi-yaml-hooks` (v2026.7.19) is the recommended no-code hook approach; install with `pi install npm:pi-yaml-hooks`.
- TypeScript extensions give full access to agent internals.
- The `bash` tool's `spawn` hook lets you mutate command, cwd, and env before execution.
- Pi includes browser automation and LSP integration as first-party tools.
- Project YAML hooks only load when the repo/worktree is trusted.
- Set `PI_YAML_HOOKS_ALLOW_GLOBAL_IMPORTS=1` to allow global hook file imports.
- Set `PI_YAML_HOOKS_ENABLE_USER_BASH=1` to enable user bash interception.
- Set `PI_YAML_HOOKS_ALLOW_PACKAGE_IMPORTS=1` to allow importing npm packages from hooks.
- Set `PI_YAML_HOOKS_ALLOW_PROJECT_IMPORTS_OUTSIDE_TRUST_ANCHOR=1` to allow project-local imports outside the trust anchor path.
- Set `PI_YAML_HOOKS_TRUST_PROJECT=1` to trust the current project directory for hook loading without an explicit trust prompt.

## Sources

> **Note on hook documentation:** `oh-my-pi` (`can1357/oh-my-pi`) is a **community-maintained** project, not the primary vendor repo (`earendil-works/pi`). Hook behavior and event naming differ between `pi-yaml-hooks` (dot-notation) and oh-my-pi (snake_case). The `pi-yaml-hooks` package is also community-maintained (KristjanPikhof, MIT license), referenced from the official pi.dev packages directory.

| Topic | Source type | URL | Fetched |
|-------|-------------|-----|---------|
| Installation & feature overview | Vendor (pi.dev) | https://pi.dev | 2026-07-23 [official] — install commands, MIT license, "No MCP"/"No sub-agents" stance all confirmed unchanged |
| pi-yaml-hooks package | Vendor index (pi.dev) | https://pi.dev/packages/pi-yaml-hooks | 2026-07-23 [official] (now at v2026.7.19) |
| pi-yaml-hooks source | Community (KristjanPikhof) | https://github.com/KristjanPikhof/pi-yaml-hooks | 2026-07-23 [github] — maintainer, MIT license, and dot-notation event list confirmed |
| Hooks docs (oh-my-pi) | Community | https://github.com/can1357/oh-my-pi/blob/main/docs/hooks.md | 2026-07-23 [github] — snake_case events confirmed, not archived |
| Extensions docs | Vendor (GitHub) | https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/extensions.md | 2026-07-23 [github] (old `badlogic/pi-mono` path 301s here); `after_provider_response` event confirmed present |
| GitHub (pi-mono) — primary | Vendor | https://github.com/earendil-works/pi | 2026-07-23 [github] — active, MIT, not archived |
| GitHub (oh-my-pi) | Community | https://github.com/can1357/oh-my-pi | 2026-07-23 [github] — active, not archived |
| Awesome Pi Agent | Community | https://github.com/qualisero/awesome-pi-agent | 2026-07-23 ⚠️ Archived Jun 3 2026, confirmed still read-only |
