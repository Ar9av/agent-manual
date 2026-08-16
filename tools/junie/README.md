# Junie

> JetBrains' AI coding agent — ships as both a JetBrains-IDE plugin and a standalone terminal CLI, and can also run in GitHub Actions / GitLab CI.

**Vendor:** JetBrains s.r.o. | **License:** Proprietary (JetBrains AI Service Terms of Service; usage gated by JetBrains AI subscription or BYOK) | **Runtime:** Native cross-platform CLI binary (Linux/macOS/Windows) for the CLI; JVM-hosted plugin inside JetBrains IDEs for the IDE variant

## Links

- Docs (CLI + agent reference): https://junie.jetbrains.com/docs/
- Docs (CLI quickstart): https://junie.jetbrains.com/docs/junie-cli.html
- Docs (IDE plugin): https://junie.jetbrains.com/docs/junie-ide-plugin.html
- IntelliJ IDEA Help (Junie page): https://www.jetbrains.com/help/idea/junie.html
- GitHub (distribution/installers): https://github.com/JetBrains/junie
- Product page: https://junie.jetbrains.com/
- Marketing page: https://www.jetbrains.com/junie/

---

## Installation

> ❓ Went GA June 2026 per task brief; exact GA announcement/changelog page was not independently located during this research pass — installation commands below are taken directly from official docs/homepage.

### CLI (Linux / macOS)
```sh
curl -fsSL https://junie.jetbrains.com/install.sh | bash
```

### CLI (Windows, PowerShell)
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -Command "iex (irm 'https://junie.jetbrains.com/install.ps1')"
```

### CLI (Homebrew / npm)

Confirmed via the `github.com/JetBrains/junie` repo README:
```sh
# Homebrew
brew tap jetbrains-junie/junie
brew install junie

# npm
npm install -g @jetbrains/junie
```
The installed `junie` binary is a version-management shim: `junie --eap`, `junie --nightly`, `junie --experimental`, and `junie --release` select an update channel (fetching that channel's latest build on first use), and `--use-version=<build>` pins a specific build (e.g. `junie --eap --use-version=122.1`).

### IDE Plugin

Via JetBrains Marketplace: open **Settings → Plugins**, search for "Junie by JetBrains", click **Install**. Alternatively, download the `.zip` from Marketplace and use **Settings → Plugins → gear icon → Install Plugin from Disk** (manually-installed plugins require manual updates).

Supported JetBrains IDEs (by version):
- 2024.3.2+: IntelliJ IDEA Ultimate, PyCharm Pro, WebStorm, GoLand
- 2025.1+: IntelliJ IDEA Community, PhpStorm, RubyMine, RustRover
- 2025.2.1+: CLion, Rider
- Also available: Android Studio

Requires a JetBrains AI subscription (Free tier with limited credits, AI Pro, AI Ultimate) or bring-your-own-key (BYOK) for third-party model providers.

---

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.junie/config.json` | Global (user) | Default model/provider, flags, hook definitions, custom skill/agent/command/model locations |
| `.junie/config.json` | Project | Team-shared config; loaded only for **trusted** projects (interactive trust prompt otherwise) |
| `~/.junie/settings.json` | Global (user) | User settings, sits between CLI flags and project config in precedence |
| `~/.junie/allowlist.json` | Global (user) | Action Allowlist — approved commands / patterns that skip permission prompts |
| `~/.junie/trust` | Global (user) | Project-trust markers, stored via OS keychain / Credential Manager / Secret Service |
| `.junie/mcp/mcp.json` | Project | Project-level MCP server definitions (can be version-controlled) |
| `~/.junie/mcp/mcp.json` | Global (user) | User-level (private) MCP server definitions |
| `.junie/AGENTS.md` | Project | Preferred location for project guidelines/instructions |
| `.aiignore` | Project | gitignore-syntax file restricting Junie's file access (IDE plugin) |
| `.junie/agents/`, `.agents/` | Project/User | Custom subagent definitions |
| `.junie/skills/`, `~/.junie/skills/` | Project/User | Agent Skills (`SKILL.md` folders) |

**Config precedence (high → low):** CLI flags → `~/.junie/settings.json` → project `.junie/config.json` (if trusted) → `~/.junie/config.json`.

---

## Instruction File

Junie reads project guidelines/instructions with the following documented lookup order:
1. A custom path set in IDE project settings
2. `.junie/AGENTS.md` — "the most preferred standard location" (format follows the [agents.md](https://agents.md/) convention)
3. `AGENTS.md` in the project root
4. Legacy/deprecated: `.junie/guidelines.md` or a `.junie/guidelines/` directory

Note for this catalog: **`.junie/guidelines.md` is a legacy/deprecated path** as of the docs fetched for this entry — the current recommended file is `.junie/AGENTS.md`.

---

## Hooks

Junie CLI has a documented hook system (config-driven, similar in shape to Claude Code's). **Confirmed CLI-only**: the CLI hooks reference states hooks "are currently triggered from the interactive TUI host...and the batch host...ACP and server hosts do not yet invoke any hooks," and the JetBrains-IDE plugin docs make no mention of a hook system — treat hooks below as CLI-specific.

Hooks are configured under a `"hooks"` key in `~/.junie/config.json` (or a file loaded via `--config-location`). **Project-local `.junie/config.json` hooks are ignored by default for security** — hooks must come from the user-scope/global config.

### Supported Events

| Event | When it fires | Can Block? | Hosts |
|-------|---------------|-----------|-------|
| `SessionStart` | Session begins (startup, resume, `/clear`, `/compact`) | ❌ (runs in background) | TUI, Batch |
| `UserPromptSubmit` | User submits a prompt | ✅ | TUI only |
| `PreToolUse` | Before a tool executes | ✅ | TUI, Batch |
| `PermissionRequest` | Permission dialog is shown | ✅ | TUI, Batch |
| `Stop` | Before a task submission is finalized | ✅ (`blockOnError`, up to a consecutive-block cap) | TUI, Batch |
| `StopFailure` | An LLM/API call fails | ❌ (observability only) | TUI, Batch |
| `SessionEnd` | Session terminates | ❌ (cannot block) | TUI, Batch |

### Hook Input (stdin JSON)

```json
{ "hook_event_name": "PreToolUse", "tool_name": "Bash", "tool_input": { "command": "..." } }
```

Other events carry event-specific fields, e.g.:
```json
{ "hook_event_name": "UserPromptSubmit", "prompt": "..." }
{ "hook_event_name": "Stop", "stop_hook_active": false, "last_assistant_message": "..." }
{ "hook_event_name": "StopFailure", "error": "rate_limit", "error_details": "..." }
{ "hook_event_name": "SessionEnd", "reason": "prompt_input_exit" }
```

### Hook Output (stdout JSON, optional)

```json
{
  "decision": "allow|ask|block|deny",
  "reason": "human-readable explanation",
  "updatedInput": {},
  "additionalContext": "text injected for the model",
  "systemMessage": "message shown in the TUI",
  "continue": false,
  "stopReason": "halt explanation",
  "permissionDecision": "allow|deny"
}
```

### Exit Code Behavior

| Code | Meaning |
|------|---------|
| `0` | Success; stdout JSON processed normally |
| `2` | Explicit block (`PreToolUse`, `Stop`, `PermissionRequest`) |
| Other | Logged as a warning; execution proceeds unless `blockOnError: true` |
| Timeout exceeded | Hook process force-killed; logged/shown as an error |

Timeout defaults: 10s for `SessionStart`/`UserPromptSubmit`/`PermissionRequest`, 600s for `Stop`, 60s for `StopFailure`, 2s (total budget across matched entries) for `SessionEnd`. `Stop` hooks cap at 8 consecutive blocks per task (override via `JUNIE_STOP_HOOK_BLOCK_CAP`). Hooks support `"async": true` to run in the background without blocking the agent loop.

### Example Config

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{ "type": "command", "command": "~/.junie/hooks/check-bash-command.sh" }]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "~/.junie/hooks/validate-submission.sh",
            "blockOnError": true,
            "timeout": 600
          }
        ]
      }
    ]
  }
}
```

---

## Built-in Tools

Official docs do not publish one single canonical tool-name table; the following is assembled from the subagent "tool groups" reference and general capability descriptions:

| Tool | Description |
|------|-------------|
| `Read` | Read file contents |
| `Write` | Create files |
| `Edit` | Modify files (search/replace, patches) |
| `Bash` | Execute shell commands |
| `Glob` | File pattern search |
| `Grep` | Text/regex search across files |
| `WebSearch` | Internet search |
| `AskUserQuestion` | Prompt the user for input |

Junie can also "autonomously run terminal commands, create new files, write or edit code, run tests, and verify the changes" per the IDE docs, and additional tools are exposed by any configured MCP servers. Sensitive actions (terminal commands, MCP tools, file operations) require explicit user approval by default, governed by the Action Allowlist.

---

## MCP Support

Config location (CLI):
```json
{
  "mcpServers": {
    "ServerName": {
      "command": "npx",
      "args": ["-y", "@package/name"],
      "env": { "ENV_VAR": "value" }
    },
    "RemoteServer": {
      "url": "https://mcp.example.com/v1",
      "headers": { "Authorization": "Bearer token" }
    }
  }
}
```

- Project scope: `.junie/mcp/mcp.json` (can be committed to version control)
- User scope: `~/.junie/mcp/mcp.json` (private)
- CLI flags: `--mcp-location <path>` (repeatable) to add extra discovery folders; `--mcp-default-locations false` to disable default locations
- IDE plugin: MCP servers configured globally in Settings or per-project via `.junie/mcp/mcp.json`; server status shown under **Settings → Tools → Junie → MCP Settings**; the IDE plugin documents a practical limit of up to 100 tools total across connected servers
- MCP tool calls are treated as sensitive actions and prompt for approval unless allowlisted (an MCP-type Action Allowlist rule authorizes *all* MCP tools; per-server/per-tool allowlisting is not yet supported per docs)

---

## Skills / Commands

- Skills location: `.junie/skills/<skill-name>/SKILL.md` (project, version-controlled) or `~/.junie/skills/<skill-name>/SKILL.md` (user); project-level skills take precedence over same-named user-level skills
- Format: Agent Skills open format — Markdown with YAML frontmatter (`name` required, `description` optional — first body paragraph used as fallback), plus optional `scripts/`, `templates/`, `checklists/` subfolders
- Discovery: Junie scans names/descriptions first and only loads full skill content when relevant to the current task (unlike guidelines/instructions, which apply to every prompt)
- Commands: typing `/` in the CLI TUI lists available slash commands; `@` attaches a file/folder to the request context; `!` prefix runs a shell command inline

---

## Agent / Subagent Configuration

Subagents are Markdown files with YAML frontmatter, stored at:
- Project: `.junie/agents/` or `.agents/`
- User: `~/.junie/agents/` (or `~/.agents/`), Windows: `%USERPROFILE%\.junie\agents\`

Frontmatter fields:

| Field | Required | Purpose |
|-------|----------|---------|
| `name` | No | Identifier (defaults to filename) |
| `description` | **Yes** | Drives automatic delegation matching |
| `tools` | No | Allowlist of tool groups |
| `disallowedTools` | No | Denylist, applied after allowlist |
| `model` | No | Overrides default model for this subagent |
| `reasoningLevel` | No | low/medium/high |
| `maxTurns` | No | Step limit |
| `skills` | No | Agent Skill IDs to preload |
| `allowPromptArgument` | No | Exposes `$prompt` variable |
| `mcpServers` | No | Allowlist of MCP server names |

Subagents are invoked **automatically** by the main agent matching task intent against each subagent's `description` — there is no documented manual/slash-command invocation path. The main agent announces which subagent has started working, and each subagent runs independently in its own context before returning a result. Session transcripts for subagent runs are stored in the session's `subagents` folder.

---

## Notes

- Junie is dual-mode: a **JetBrains-IDE plugin** (IntelliJ IDEA, PyCharm, WebStorm, GoLand, PhpStorm, RubyMine, RustRover, CLion, Rider, Android Studio) *and* a **standalone terminal CLI** with its own TUI, plus GitHub Actions / GitLab CI integration — unlike the historical "IDE-only" framing some third-party writeups still use, the CLI is a first-class, separately documented product (`junie-cli.html`, `junie-cli-hooks.html`, `junie-cli-subagents.html`, etc.).
- Junie is explicitly LLM-agnostic / BYOK-capable, not limited to a single model vendor.
- Two operating modes are documented for the IDE plugin: **Code mode** (autonomous edits) and **Ask mode** (read-only analysis).
- Session/event data for the CLI is stored as `events.jsonl` and `transcript.md` files per session.
- Requires a JetBrains AI subscription (Free/Pro/Ultimate tiers) or BYOK; a JetBrains account and acceptance of the JetBrains AI Service Terms of Service is required to use Junie at all.
- ❓ This entry is a doc-only pass; no live install or execution of Junie was performed to verify these behaviors.

---

## Sources (Official)

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| Docs home / getting started | https://junie.jetbrains.com/docs/ | 2026-08-15 | [official] |
| CLI quickstart / installation | https://junie.jetbrains.com/docs/junie-cli.html | 2026-08-15 | [official] |
| CLI config.json reference | https://junie.jetbrains.com/docs/junie-cli-configuration.html | 2026-08-15 | [official] |
| CLI MCP configuration | https://junie.jetbrains.com/docs/junie-cli-mcp-configuration.html | 2026-08-15 | [official] |
| CLI hooks reference | https://junie.jetbrains.com/docs/junie-cli-hooks.html | 2026-08-15 | [official] |
| CLI custom subagents | https://junie.jetbrains.com/docs/junie-cli-subagents.html | 2026-08-15 | [official] |
| Agent skills | https://junie.jetbrains.com/docs/agent-skills.html | 2026-08-15 | [official] |
| IDE plugin overview / guidelines file | https://junie.jetbrains.com/docs/junie-ide-plugin.html | 2026-08-15 | [official] |
| MCP Settings (IDE plugin) | https://junie.jetbrains.com/docs/junie-plugin-mcp-settings.html | 2026-08-15 | [official] |
| Action Allowlist | https://junie.jetbrains.com/docs/action-allowlist.html | 2026-08-15 | [official] |
| Junie plugin settings | https://junie.jetbrains.com/docs/junie-plugin-settings.html | 2026-08-15 | [official, ⚠️ 404 as of re-verification — page returns HTTP 404; content below sourced from cached search snippets, not a live fetch] |
| IntelliJ IDEA Help — Junie page | https://www.jetbrains.com/help/idea/junie.html | 2026-08-15 | [official] |
| AI Assistant docs — Junie agent | https://www.jetbrains.com/help/ai-assistant/junie-agent.html | 2026-08-15 | [official] |
| Product / pricing / install command | https://junie.jetbrains.com/ | 2026-08-15 | [official] |
| Marketing page | https://www.jetbrains.com/junie/ | 2026-08-15 | [official] |
| GitHub distribution repo | https://github.com/JetBrains/junie | 2026-08-15 | [github] |
