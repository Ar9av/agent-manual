# Auggie CLI

> Augment Code's agentic terminal CLI — brings Augment's agent, Context Engine, and tool integrations to the command line.

**Vendor:** Augment Code (augmentcode.com) | **License:** Proprietary (requires an Augment account/login; npm package not open-source) ❓ exact license text not published in docs | **Runtime:** Node.js 20+ (npm package `@augmentcode/auggie`)

## Links

- Docs (overview): https://docs.augmentcode.com/cli/overview
- CLI reference (flags): https://docs.augmentcode.com/cli/reference
- Install guide: https://docs.augmentcode.com/cli/setup-auggie/install-auggie-cli
- GitHub (mirror/issue tracker): https://github.com/augmentcode/auggie
- npm package: https://www.npmjs.com/package/@augmentcode/auggie

---

## Installation

```sh
npm install -g @augmentcode/auggie
```

Requirements: Node.js 20 or later; supported on macOS, Windows (via WSL), and Linux; shell must be zsh, bash, or fish. Interactive mode needs a terminal that supports ANSI escape codes (Ghostty, iTerm2, macOS Terminal, Windows Terminal, Alacritty, Kitty recommended).

Authenticate after install:

```sh
auggie login
```

Update to the latest version:

```sh
auggie upgrade
```

❓ The docs describe only the npm install method — no Homebrew formula or curl script is documented as official.

## Configuration Files

Settings load from these locations (highest → lowest precedence per the hooks docs; the same `settings.json` file also carries MCP servers, rules, and tool permissions):

| File | Scope | Purpose |
|------|-------|---------|
| `/etc/augment/settings.json` (macOS/Linux) or `C:\ProgramData\Augment\settings.json` (Windows) | System-wide | Immutable org policy |
| `<workspace>/.augment/settings.local.json` | Project (personal) | Local overrides, not shared |
| `<workspace>/.augment/settings.json` | Project (shared) | Team-shared config, MCP servers, permissions |
| `~/.augment/settings.json` | Global (user) | User-level settings, MCP servers, hooks |
| `~/.augment/rules/` | Global | User rules (always applied) |
| `.augment/rules/` | Project | Workspace rules (frontmatter-controlled) |
| `.augment/commands/`, `.claude/commands/` (project & user variants) | Both | Custom slash commands |
| `.augment/skills/`, `.claude/skills/`, `.agents/skills/` (project & user variants) | Both | Agent Skills |
| `.augment/agents/`, `~/.augment/agents/` | Both | Subagent definitions |

## Instruction File

Auggie resolves "rules" (instruction files) in this precedence order:

1. Custom rules file passed via `--rules <path>`
2. `CLAUDE.md` (Claude Code compatible)
3. `AGENTS.md` (Cursor-compatible format)
4. Workspace guidelines file `.augment-guidelines`
5. Workspace rules folder `.augment/rules/` (recursive)
6. User rules folder `~/.augment/rules/` (recursive)

`AGENTS.md` and `CLAUDE.md` are discovered **hierarchically**: Auggie walks up from the current file's directory to the workspace root, including every matching file along that path (but not sibling directories). `.augment/rules/*.md` files support YAML frontmatter (`type: always_apply` or `type: agent_requested` with a `description`); user-level rules under `~/.augment/rules/` are always treated as `always_apply`.

Source: https://docs.augmentcode.com/cli/rules

## Hooks

Auggie has a native hook system (`"hooks"` key inside `settings.json`, layered across the same five settings-file locations listed above).

### Supported Events

| Event | When | Can Block? |
|-------|------|-----------|
| `PreToolUse` | Before a tool executes | ✅ (exit code 2, or JSON `permissionDecision: "deny"`) |
| `PostToolUse` | After a tool completes | ❌ (feedback only) |
| `Stop` | When the agent wants to stop | ✅ (can prevent completion) |
| `SessionStart` | At session initialization | ❌ (stdout is injected as agent context) |
| `SessionEnd` | When a session ends | ❌ |

`PreToolUse`, `PostToolUse`, and `Stop` hooks run **synchronously**. MCP tools are matched via the naming pattern `{toolName}_{serverName}` and an `"mcp:"` matcher prefix.

### Hook Input (stdin JSON)

Common fields on every event: `hook_event_name`, `conversation_id`, `workspace_roots`.

`PreToolUse` / `PostToolUse` add: `tool_name`, `tool_input`, `tool_output` (PostToolUse only), `tool_error` (PostToolUse only), `file_changes` (PostToolUse only), `is_mcp_tool`.

`Stop` adds: `agent_stop_cause` (`"end_turn"`, `"interrupted"`, `"max_iterations"`, `"error"`).

Optional `metadata` flags (`includeUserContext`, `includeMCPMetadata`, `includeConversationData`) opt in to extra context (`context`, `mcp_metadata`, `conversation` objects) — privacy-first / opt-in by default.

```json
{
  "hook_event_name": "PreToolUse",
  "conversation_id": "...",
  "workspace_roots": ["/path/to/project"],
  "tool_name": "launch-process",
  "tool_input": { "command": "rm -rf /" },
  "is_mcp_tool": false
}
```

### Hook Output (stdout JSON, optional)

```json
{
  "continue": true,
  "stopReason": "optional reason",
  "suppressOutput": false,
  "systemMessage": "user-facing message",
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "reason text"
  }
}
```

### Exit Code Behavior

| Code | Meaning |
|------|---------|
| `0` | Success; stdout/stderr shown to user, SessionStart stdout injected as context |
| `2` | Blocking error (PreToolUse only); stderr shown to agent |
| Other | Non-blocking error |

### Example Config

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "str-replace-editor|view",
        "hooks": [
          {
            "type": "command",
            "command": "/etc/augment/hooks/audit-files.sh",
            "timeout": 5000,
            "metadata": {
              "includeUserContext": true,
              "includeMCPMetadata": true,
              "includeConversationData": true
            }
          }
        ]
      }
    ]
  }
}
```

Only `"type": "command"` hooks are currently supported (`.sh`, `.ps1`, `.bat`, `.cmd`), default timeout 60000ms.

Sources: https://docs.augmentcode.com/cli/hooks , https://docs.augmentcode.com/cli/hooks-examples

## Built-in Tools

Per the permissions docs, native tools addressable by `toolPermissions` / `--permission`:

| Tool | Description |
|------|-------------|
| `terminal` | Execute shell commands (filterable via `shellInputRegex`, e.g. `launch-process`) |
| `read` | Read files (also referenced as `view` in hook matchers) |
| `edit` | Find/replace file editing (`str-replace-editor`) |
| `write` | Create/overwrite files |
| `web-search` | Web search queries |
| `web-fetch` | Fetch page content |

❓ This list is drawn from the permissions/hooks docs' tool-name references, not a single authoritative "all built-in tools" table; run `auggie tools list` (or `auggie tools schemas`) locally for the definitive, versioned list. MCP-provided tools follow the `{tool-name}_{server-name}` naming pattern and are managed the same way.

## MCP Support

Yes. MCP servers persist in `~/.augment/settings.json` (or the project-level `.augment/settings.json`), under `mcpServers`, and are checkable in-session with `/mcp`.

```json
{
  "mcpServers": {
    "my-server": {
      "type": "stdio",
      "command": "/path/to/executable",
      "args": ["arg1", "arg2"],
      "env": { "KEY": "value" }
    },
    "remote-server": {
      "type": "http",
      "url": "https://example.com/mcp",
      "headers": { "Authorization": "Bearer TOKEN" }
    }
  }
}
```

Transport types: `stdio` (local executable, `command`/`args`), `http`/`sse` (remote, `url` + optional `headers`). `${workspaceFolder}` expands in `command`, `args`, and `url` when run inside a workspace.

CLI management commands:

```sh
auggie mcp add <name> --command <path> --args <args> --transport stdio|http|sse
auggie mcp add-json <name> '<json>'
auggie mcp list [--json]
auggie mcp remove <name>
```

Also supports **inline/one-off** MCP config via `--mcp-config /path/to/mcp.json` or `--mcp-config '{json}'`, and **MCP Tool Search** — hides individual MCP tools behind two meta-tools (`find-tool`, `execute-tool`) to reduce context use, enabled with `--enable-tool-search` or `"enableToolSearch": true` in settings. Native (non-MCP) integrations for GitHub, Linear, and Notion are also available once configured.

Source: https://docs.augmentcode.com/cli/integrations

### `--mcp` (MCP server mode)

`auggie --mcp` runs Auggie itself **as** an MCP server (exposing Auggie's agent as a tool to other MCP clients), with `--mcp-auto-workspace` for dynamic workspace discovery and `-w /path` to specify the workspace to index. Source: https://docs.augmentcode.com/cli/reference

### `--acp` (Agent Client Protocol mode)

```sh
auggie --acp
```

Runs Auggie as a fully compliant [Agent Client Protocol](https://agentclientprotocol.com) agent — an open, editor-agnostic protocol (JSON-RPC over stdio) for connecting AI agents to editors/IDEs. Configure the client (editor) to launch `auggie --acp`; not all interactive-mode features are supported in ACP mode yet. See the client list at https://docs.augmentcode.com/cli/acp/clients.

Sources: https://docs.augmentcode.com/cli/acp/agent , https://docs.augmentcode.com/cli/acp/clients

## Skills / Commands

Auggie implements the open **[agentskills.io](https://agentskills.io/specification)** specification. Skills are self-contained `SKILL.md` packages discovered from six locations:

| Location | Scope |
|----------|-------|
| `~/.augment/skills/<name>/SKILL.md` | User |
| `~/.claude/skills/<name>/SKILL.md` | User (Claude-compatible) |
| `~/.agents/skills/<name>/SKILL.md` | User (agentskills.io generic) |
| `<workspace>/.augment/skills/<name>/SKILL.md` | Project |
| `<workspace>/.claude/skills/<name>/SKILL.md` | Project (Claude-compatible) |
| `<workspace>/.agents/skills/<name>/SKILL.md` | Project (agentskills.io generic) |

Required frontmatter:

```yaml
---
name: skill-identifier
description: 1-1024 character description
---
```

`name` must be 1–64 chars, lowercase alphanumeric + hyphens, no leading/trailing/consecutive hyphens, and must match the containing directory name. Skills are auto-registered as slash commands (`/<skill-name>`); browse all with `/skills`. When the same skill name exists in multiple locations, higher-precedence location wins.

Separately, **custom slash commands** (distinct from skills) are markdown files under `~/.augment/commands/`, `./.augment/commands/`, `~/.claude/commands/`, `./.claude/commands/`, invoked via `auggie command list/help/<command-name>` or `/<command-name>` in the TUI.

Sources: https://docs.augmentcode.com/cli/skills , https://docs.augmentcode.com/cli/custom-commands

## Agent / Subagent Configuration

Subagents are markdown files with YAML frontmatter:

| Location | Scope |
|----------|-------|
| `~/.augment/agents/<name>.md` | User (all workspaces) |
| `./.augment/agents/<name>.md` | Workspace only |

Frontmatter fields: `name` (required), `description`, `color` (ANSI color name for CLI display), `model` (defaults to CLI default), `tools` (allowlist), `disabled_tools` (denylist). Markdown body is the subagent's system prompt.

Invoke by name in a message (e.g. "Use the code-review agent to review my staged changes"), let Auggie auto-detect and offer a subagent for a task, or scaffold one interactively with `/agents`.

Source: https://docs.augmentcode.com/cli/subagents

## Permissions

Tool permissions are managed via `toolPermissions` in the same layered `settings.json` files (user `~/.augment/settings.json`, repo `.augment/settings.json` recommended for org policy).

- Rule types, most → least restrictive: `deny` > `webhook-policy` > `script-policy` > `allow`.
- Within one policy file, rules are evaluated top-down, first match wins; across policy files (system/project/user), the **most restrictive** applicable rule wins.
- `--permission <rule>` CLI flag (e.g. `--permission launch-process:allow`) supplies ad-hoc rules that combine with settings-file rules using the same most-restrictive-wins logic.
- Permission value must be an object with a `type` field (not a bare string) — malformed rules are dropped with a warning.

```json
{
  "toolPermissions": [
    { "toolName": "terminal", "permission": { "type": "deny" } },
    { "toolName": "terminal", "shellInputRegex": "\\bgit\\s+merge(\\s|$)", "permission": { "type": "deny" } },
    { "toolName": "read", "permission": { "type": "allow" } }
  ]
}
```

`webhook-policy` posts `{tool name, event type, details, timestamp}` to an external HTTP endpoint and expects `{ "allow": true/false, "output": "optional message" }` back. `script-policy` pipes the same JSON to a local script on stdin; exit code 0 allows, non-zero denies.

Enforced by the Auggie CLI and Cosmos cloud agents at startup — **not** enforced in the Augment IDE/editor extension.

Source: https://docs.augmentcode.com/cli/permissions

## Notes

- **Session modes:** `--print`/`-p` (single-shot, exits after answering), `--ask`/`-a` (retrieval-only, no edits), interactive (default). `--continue`/`-c` resumes the most recent session, `--resume`/`-r` picks a saved one; `auggie session list/delete/share/continue/resume` manage history.
- **Queueing:** `auggie --print --queue "Step 2" --queue "Step 3" "Step 1"` runs multiple instructions sequentially — useful for scripted pipelines.
- **Startup scripts:** `--startup-script`/`--startup-script-file` run a command before the agent starts (e.g. to prep an environment).
- **Auth:** `auggie login`/`logout`, `auggie token print/revoke`; non-interactive auth via `AUGMENT_SESSION_AUTH` env var or `--augment-session-json`. `auggie account status` shows billing/credit info.
- **Plugins:** `auggie plugin marketplace add/list/update/remove` and `auggie plugin list/install <id>` — a marketplace layer on top of skills/commands/MCP bundles.
- **Automation:** Documented first-class support for CI/service-account use (`https://docs.augmentcode.com/cli/automation/overview`, `.../automation/service-accounts`), plus a Python and TypeScript SDK (`https://docs.augmentcode.com/cli/sdk-python`, `.../sdk-typescript`).
- ❓ No official license string (MIT/Apache/proprietary) was found in the fetched docs pages; the CLI requires an authenticated Augment account/subscription to run, implying proprietary/closed licensing, but this is inferred rather than explicitly stated.

## Sources

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| CLI overview | https://docs.augmentcode.com/cli/overview | 2026-07-23 | [official] |
| CLI reference (flags) | https://docs.augmentcode.com/cli/reference | 2026-07-23 | [official] |
| Install guide | https://docs.augmentcode.com/cli/setup-auggie/install-auggie-cli | 2026-07-23 | [official] |
| Rules & Guidelines | https://docs.augmentcode.com/cli/rules | 2026-07-23 | [official] |
| Hooks | https://docs.augmentcode.com/cli/hooks | 2026-07-23 | [official] |
| Hooks Examples | https://docs.augmentcode.com/cli/hooks-examples | 2026-07-23 | [official] |
| Integrations & MCP | https://docs.augmentcode.com/cli/integrations | 2026-07-23 | [official] |
| Tool Permissions | https://docs.augmentcode.com/cli/permissions | 2026-07-23 | [official] |
| Agent Skills | https://docs.augmentcode.com/cli/skills | 2026-07-23 | [official] |
| Custom Slash Commands | https://docs.augmentcode.com/cli/custom-commands | 2026-07-23 | [official] |
| Subagents | https://docs.augmentcode.com/cli/subagents | 2026-07-23 | [official] |
| ACP Mode | https://docs.augmentcode.com/cli/acp/agent | 2026-07-23 | [official] |
| ACP Clients | https://docs.augmentcode.com/cli/acp/clients | 2026-07-23 | [official] |
| Docs llms.txt index | https://docs.augmentcode.com/llms.txt | 2026-07-23 | [official] |
| npm package page | https://www.npmjs.com/package/@augmentcode/auggie | 2026-07-23 | [official] |
| GitHub repo (mirror) | https://github.com/augmentcode/auggie | 2026-07-23 | [official] |
