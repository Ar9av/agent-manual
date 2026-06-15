# Claude Code

> Anthropic's official AI coding agent for the terminal, IDE, desktop app, and browser.

**Vendor:** Anthropic | **License:** Proprietary | **Runtime:** Node.js (CLI); native binary also available

## Links

- Docs: https://docs.anthropic.com/en/docs/claude-code (redirects to https://code.claude.com/docs)
- Hooks reference: https://code.claude.com/docs/en/hooks
- Settings reference: https://code.claude.com/docs/en/settings
- GitHub (community): https://github.com/anthropics/claude-code
- Changelog: https://code.claude.com/docs/en/changelog

---

## Installation

### Native installer (recommended — auto-updates)

**macOS / Linux / WSL:**
```sh
curl -fsSL https://claude.ai/install.sh | bash
```

**Windows PowerShell:**
```powershell
irm https://claude.ai/install.ps1 | iex
```

**Windows CMD:**
```batch
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

### Homebrew
```sh
brew install --cask claude-code
# or for latest channel:
brew install --cask claude-code@latest
```
Note: Homebrew does **not** auto-update. Run `brew upgrade claude-code` manually.

### WinGet
```powershell
winget install Anthropic.ClaudeCode
```

### npm (alternative)
```sh
npm install -g @anthropic-ai/claude-code
claude
```

Linux package managers (apt, dnf, apk) are also supported — see the [setup page](https://code.claude.com/docs/en/setup).

---

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.claude/settings.json` | Global (user) | User-level settings, hooks, permissions |
| `.claude/settings.json` | Project (shared) | Project-level settings, committed to git |
| `.claude/settings.local.json` | Project (personal) | Personal project overrides, git-ignored |
| `managed-settings.json` / plist / registry | Org (managed) | Admin-controlled, highest precedence |
| `CLAUDE.md` | Project | Natural-language instructions loaded into context |
| `~/.claude.json` / `.mcp.json` | User / Project | MCP server configuration |

**Scope precedence (high → low):** Managed → Command line → Local → Project → User

Most settings hot-reload on file change without a session restart (including `permissions` and `hooks`).

---

## Hooks

Hook events are defined in `settings.json` under the `hooks` key.

### Supported Events

| Event | When it fires | Can Block? | Notes |
|-------|--------------|-----------|-------|
| `SessionStart` | Session begins or resumes | ❌ | |
| `Setup` | `--init-only` / maintenance flag | ❌ | |
| `UserPromptSubmit` | User submits a prompt, before processing | ✅ (exit 2 erases prompt) | 30 s timeout override |
| `UserPromptExpansion` | User-typed slash command expands into prompt | ✅ | 30 s timeout override |
| `PreToolUse` | Before every tool call | ✅ | |
| `PermissionRequest` | Permission dialog triggered | ✅ (exit 2 denies) | |
| `PermissionDenied` | Tool denied by auto-mode classifier | ❌ | Use `hookSpecificOutput: { hookEventName: "PermissionDenied", retry: true }` to retry |
| `PostToolUse` | After tool call succeeds | ❌ | Tool already ran |
| `PostToolUseFailure` | After tool call fails | ❌ | |
| `PostToolBatch` | After a parallel tool-call batch resolves | ✅ (stops loop) | |
| `Notification` | Agent sends a notification | ❌ | |
| `MessageDisplay` | While assistant message text renders | ❌ | 10 s timeout override |
| `SubagentStart` | Subagent is spawned | ❌ | |
| `SubagentStop` | Subagent finishes | ✅ | |
| `TaskCreated` | Task created via `TaskCreate` | ✅ (rolls back) | |
| `TaskCompleted` | Task marked completed | ✅ | |
| `Stop` | Claude finishes a turn | ✅ (continues conversation) | |
| `StopFailure` | Turn ends due to API error | ❌ | Output/exit code ignored |
| `TeammateIdle` | Agent-team teammate about to go idle | ✅ | |
| `InstructionsLoaded` | `CLAUDE.md` or `.claude/rules/*.md` loaded | ❌ | Exit code ignored |
| `ConfigChange` | Config file changes during session | ✅ | Except `policy_settings` |
| `CwdChanged` | Working directory changes | ❌ | |
| `FileChanged` | Watched file changes on disk | ❌ | |
| `WorktreeCreate` | Worktree created | ✅ (any non-zero fails creation) | |
| `WorktreeRemove` | Worktree removed | ❌ | |
| `PreCompact` | Before context compaction | ✅ | |
| `PostCompact` | After context compaction completes | ❌ | |
| `Elicitation` | MCP server requests user input | ✅ (exit 2 denies) | |
| `ElicitationResult` | User responds to MCP elicitation | ✅ (becomes decline) | |
| `SessionEnd` | Session terminates | ❌ | |

### Hook Types

| Type | Description | Available Events |
|------|-------------|-----------------|
| `command` | Run a shell command (stdin = hook JSON, exit code drives behavior) | All events |
| `http` | HTTP POST to a URL (body = hook JSON) | All events |
| `mcp_tool` | Call a tool on a connected MCP server | All events |
| `prompt` | Single-turn LLM evaluation (yes/no) — 30 s default timeout | `UserPromptSubmit`, `UserPromptExpansion`, `PreToolUse`, `PermissionRequest`, `PostToolUse`, `PostToolUseFailure`, `PostToolBatch`, `Stop`, `SubagentStart`, `SubagentStop` only |
| `agent` | Spawn a subagent with Read/Grep/Glob tools (experimental) — 60 s default timeout | `UserPromptSubmit`, `UserPromptExpansion`, `PreToolUse`, `PermissionRequest`, `PostToolUse`, `PostToolUseFailure`, `PostToolBatch`, `Stop`, `SubagentStart`, `SubagentStop` only |

> Note: `SessionStart` and `Setup` only support `command` and `mcp_tool` types. Most informational/lifecycle events (e.g. `Notification`, `MessageDisplay`, `InstructionsLoaded`, `SessionEnd`) support `command`, `http`, and `mcp_tool` but not `prompt` or `agent`.

### Hook Input (stdin JSON)

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.jsonl",
  "cwd": "/current/working/directory",
  "permission_mode": "default|plan|acceptEdits|auto|dontAsk|bypassPermissions",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": { "command": "rm -rf /tmp/test" }
}
```

### Exit Code Behavior

| Code | Meaning |
|------|---------|
| `0` | Success; stdout parsed for optional JSON output fields |
| `2` | Blocking error on supported events; stderr sent to Claude as error message |
| Other | Non-blocking warning; first line of stderr shown in transcript |

Exit code 2 is **not** limited to `PreToolUse` — it applies to all events marked ✅ in the table above.

### Command Hook Fields

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | `"command"`, `"http"`, `"mcp_tool"`, `"prompt"`, or `"agent"` |
| `command` | Yes (command) | Shell command to execute |
| `args` | No | Argument list; when present, command is exec'd directly (no shell) |
| `shell` | No | `"bash"` (default) or `"powershell"`; ignored when `args` is set |
| `timeout` | No | Seconds before cancellation (see defaults below) |
| `statusMessage` | No | Custom spinner message while hook runs |
| `async` | No | `true` → run in background, output discarded, Claude proceeds immediately |
| `asyncRewake` | No | `true` → run in background **and** wake Claude if exit code 2 (implies `async`). Stderr shown to Claude as system reminder |
| `if` | No | Permission-rule pattern filter, e.g. `"Bash(git *)"` |
| `once` | No | `true` → run once per session then deregister (skill frontmatter only) |

**`async` vs `asyncRewake`:** `async: true` is fire-and-forget (output discarded). `asyncRewake: true` also runs in the background but re-wakes Claude and sends hook output if exit code 2 — use for long background operations that may need human attention.

### HTTP Hook Fields

| Field | Required | Description |
|-------|----------|-------------|
| `url` | Yes | URL to HTTP POST |
| `headers` | No | Additional HTTP headers (key-value object) |
| `allowedEnvVars` | No | Env var names allowed for interpolation in headers |

To block via HTTP hook: return 2xx with JSON `{ "decision": "block" }` (non-2xx = non-blocking error).

### MCP Tool Hook Fields

| Field | Required | Description |
|-------|----------|-------------|
| `server` | Yes | Name of an already-connected MCP server |
| `tool` | Yes | Tool name on that server |
| `input` | No | Tool arguments; supports `${path}` substitution from hook JSON |

### JSON Output Fields (stdout from any hook)

These are the **top-level universal fields** returned in JSON from any hook's stdout:

| Field | Default | Description |
|-------|---------|-------------|
| `continue` | `true` | `false` stops all processing |
| `stopReason` | — | Message to user when `continue: false` |
| `suppressOutput` | `false` | Hide hook stdout from transcript |
| `systemMessage` | — | Warning shown to user |
| `terminalSequence` | — | Terminal escape sequence (OSC/BEL) |
| `hookSpecificOutput` | — | Nested object for event-specific decisions (requires `hookEventName` inside) |

`hookSpecificOutput` is a **nested object** — not a top-level field itself — containing event-specific fields. For example:

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PermissionDenied",
    "retry": true
  }
}
```

or, to inject context into Claude's context window from a `PreToolUse` hook:

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "additionalContext": "This file is auto-generated — do not manually edit."
  }
}
```

`additionalContext` (a string injected into Claude's context window) is an event-specific field inside `hookSpecificOutput`, not a top-level field.

### Timeout Defaults

| Hook type | Default | Overrides |
|-----------|---------|-----------|
| `command` | 600 s | `UserPromptSubmit`/`UserPromptExpansion` → 30 s; `MessageDisplay` → 10 s |
| `http` | 600 s | `UserPromptSubmit`/`UserPromptExpansion` → 30 s |
| `mcp_tool` | 600 s | `UserPromptSubmit`/`UserPromptExpansion` → 30 s |
| `prompt` | 30 s | — |
| `agent` | 60 s | — |

### Configuration Example

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": "~/.claude/hooks/validate-bash.sh" }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          { "type": "command", "command": "~/.claude/hooks/format.sh", "async": true }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/background-validate.sh",
            "asyncRewake": true
          }
        ]
      }
    ]
  }
}
```

### Async Hooks

Add `"async": true` to run a hook in the background without blocking Claude's execution (released January 2026).

Add `"asyncRewake": true` to run in the background and re-wake Claude if the hook exits with code 2 (useful for deferred validation that may need to surface an error).

### Matcher Patterns

| Pattern | Evaluated as |
|---------|-------------|
| `*`, `""`, or omitted | Match all occurrences |
| Letters/digits/`_`/`\|` only | Exact string or `\|`-separated list |
| Any other characters | JavaScript regex |

MCP tool format: `mcp__<server>__<tool>` (e.g. `mcp__memory__create_entities`; all tools from a server: `mcp__memory__.*`).

---

## Built-in Tools

| Tool | Description |
|------|-------------|
| `Bash` | Execute shell commands |
| `Read` | Read file contents |
| `Write` | Write/overwrite files |
| `Edit` | Apply targeted string replacements |
| `MultiEdit` | Multiple edits in one call |
| `Glob` | File pattern matching |
| `Grep` | Search file contents |
| `LS` | List directory contents |
| `TodoRead` / `TodoWrite` | Manage in-session task lists |
| `Agent` | Spawn subagents |
| `WebFetch` | Fetch a URL |
| `WebSearch` | Web search |
| `NotebookRead` / `NotebookEdit` | Jupyter notebook support |

---

## MCP Support

- Config: `.mcp.json` (project) or `~/.claude.json` (user)
- Enable servers in settings under `mcpServers`
- Servers expose additional tools available in the agent loop
- MCP tool hooks use the `mcp_tool` hook type targeting connected servers

---

## Agent / Subagent Configuration

Claude Code supports spawning subagents via the `Agent` tool and via `agent`-type hooks. Subagents have access to Read, Grep, and Glob tools by default. Background agents and agent-team features (with `TeammateIdle` hook) are also available.

---

## Permissions

```json
{
  "permissions": {
    "allow": ["Bash(git *)", "Read(**)"],
    "deny": ["Bash(rm -rf *)", "Read(./.env)"],
    "ask": ["Write(./config.json)"]
  }
}
```

- Deny rules take precedence over allow rules.
- Rules merge across scopes; higher-precedence sources can only add restrictions, not loosen them.

---

## Skills

Skills and slash commands are **two different things** in the Claude ecosystem:

| | Location | Shape | Invocation |
|---|---|---|---|
| **Skill** | `.claude/skills/<name>/SKILL.md` (project) · `~/.claude/skills/<name>/SKILL.md` (personal) | A folder whose `SKILL.md` has YAML frontmatter (`name`, `description`) + instructions | Auto-invoked by the model when the `description` matches intent |
| **Slash command** | `.claude/commands/<name>.md` | A single Markdown file; can reference `$ARGUMENTS` and embed tool calls | Typed explicitly as `/name` |

A skill is a directory, not a file. The `description` field is what the model uses to decide *when* to reach for it, so it should enumerate trigger phrases.

### Filesystem auto-discovery (Claude Code)

Claude Code (CLI, IDE extensions) **scans the filesystem** on startup and discovers every skill under `.claude/skills/` and `~/.claude/skills/` automatically. Drop a folder in, restart the session, and it's available. No registration step.

### Manual upload (Claude Desktop / claude.ai app)

The **Claude Desktop app and claude.ai web app do NOT read your local `.claude/skills/` folders.** They have no filesystem-discovery step at all — pointing them at a repo or a local path does nothing. Skills must be **added manually**, one at a time:

1. Enable the sandbox: **Settings → Capabilities** (a.k.a. Features) and turn on **Code execution** / file creation — Agent Skills run inside that sandbox, so they're invisible without it.
2. In the same **Capabilities → Skills** panel, click **Upload skill**.
3. Upload a **`.zip` of a single skill folder**, with `SKILL.md` at the **root of the zip** (zip the folder's *contents*, or the folder such that `SKILL.md` is at the top level — not nested under extra directories).
4. Repeat per skill — the uploader takes one skill per zip; there's no bulk "import a directory of skills" path.

**Common gotchas when a skill "doesn't show up" in the app:**

- **It's only on your disk.** A skill living in `.claude/skills/` works in Claude Code but is *not* synced to the app. The app only knows about skills you've explicitly uploaded.
- **Symlinks don't survive zipping.** If your `.claude/skills/<name>` entries are symlinks into another directory (a common pattern for sharing one source across tools), zipping them produces a broken link. Package from the **real** skill folder.
- **`SKILL.md` is buried.** If the zip expands to `myskill/myskill/SKILL.md` or the file isn't at the root, the app can't find the manifest and rejects it.
- **Sandbox off.** With Code execution disabled, uploaded skills exist but never trigger.

---

## Notes

- `CLAUDE.md` files are loaded recursively from the repo root and from `.claude/rules/*.md`.
- `disableAllHooks: true` in settings disables all hooks and the custom status line.
- Hook `agent` type has a 60 s default timeout; `prompt` type has 30 s; `command`/`http`/`mcp_tool` default to 600 s.
- Native installer (`curl`) automatically updates in the background; Homebrew/WinGet/npm do not.
- Path placeholders available in hook commands: `${CLAUDE_PROJECT_DIR}`, `${CLAUDE_PLUGIN_ROOT}`, `${CLAUDE_PLUGIN_DATA}`.

---

## Sources

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| Hooks reference | https://code.claude.com/docs/en/hooks | 2026-06-13 | [official] |
| Overview / installation | https://code.claude.com/docs/en/overview | 2026-06-13 | [official] |
| Settings reference | https://code.claude.com/docs/en/settings | 2026-06-13 | [official] |
| Canonical docs redirect | https://docs.anthropic.com/en/docs/claude-code → https://code.claude.com/docs | 2026-06-13 | [official] |
