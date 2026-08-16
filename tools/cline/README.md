# Cline

> Open-source autonomous coding agent, shipped as a VS Code/JetBrains extension, a standalone terminal CLI ("Cline CLI 2.0"), and a Node.js SDK — all three built on a shared agent engine.

**Vendor:** Cline Bot Inc. | **License:** Apache-2.0 | **Runtime:** Node.js (CLI: `npm install -g cline`; SDK: `npm install @cline/sdk`)

## Links

- Docs: https://docs.cline.bot
- CLI overview: https://docs.cline.bot/cline-cli/getting-started
- CLI reference: https://docs.cline.bot/cli/cli-reference
- Hooks (in-repo docs): https://github.com/cline/cline/blob/main/.clinerules/hooks/README.md
- Rules: https://docs.cline.bot/customization/cline-rules
- Skills: https://docs.cline.bot/customization/skills
- Subagents: https://docs.cline.bot/features/subagents
- MCP config: https://docs.cline.bot/mcp/configuring-mcp-servers
- GitHub: https://github.com/cline/cline
- CLI 2.0 announcement: https://cline.bot/blog/introducing-cline-cli-2-0
- Hooks announcement (v3.36): https://cline.bot/blog/cline-v3-36-hooks

---

## CLI vs. VS Code Extension vs. SDK

Cline ships as three distinct products from one monorepo (`cline/cline`), sharing an underlying agent engine:

- **VS Code / JetBrains extension** (`apps/vscode/`) — the original sidebar chat UI, Plan/Act mode, diff review, checkpoints.
- **CLI 2.0** (`apps/cli/`, installed as the `cline` npm package) — a **separate product**, not a thin wrapper around the extension. It runs as an interactive TUI (`-i`/`--tui`) or headless (`--auto-approve`, `--json`, piped stdin/stdout), is designed to be composed into shell pipelines/CI, and supports parallel isolated instances (e.g. one per tmux pane) and an Agent Client Protocol (`--acp`) mode for editor interop (Zed, Neovim, JetBrains, Emacs).
- **SDK** (`sdk/`, `@cline/sdk`) — the programmatic Node.js API/engine (`ClineCore`) that both the extension and CLI are built on; used to build custom agents, tools, and plugins.

Settings, rules, skills, and hooks are largely shared/unified across all three surfaces (per official docs, feature toggles like `use_subagents` "apply across all editors (VS Code, JetBrains, CLI)").

---

## Installation

```sh
npm install -g cline
cline auth
```

`cline auth` configures a provider (Cline's own provider, ClinePass, or a bring-your-own-key provider such as OpenAI/Anthropic/Bedrock/Vertex).

VS Code / JetBrains extension: installed from the respective marketplace (not covered here — this page focuses on the CLI/SDK surface per repo docs).

SDK:
```sh
npm install @cline/sdk
```

---

## Configuration Files

| File / Directory | Scope | Purpose |
|---|---|---|
| `~/.cline/data/settings/providers.json` | Global | API keys / provider config |
| `~/.cline/data/settings/global-settings.json` | Global | General settings |
| `~/.cline/data/settings/cline_mcp_settings.json` | Global | MCP server config (extension) |
| `~/.cline/mcp.json` | Global (CLI) | MCP server config (CLI) |
| `~/.cline/rules/`, `~/.cline/skills/`, `~/.cline/hooks/`, `~/.cline/agents/`, `~/.cline/plugins/`, `~/.cline/cron/` | Global | Global component directories |
| `~/.cline/data/workflows/` | Global | Global workflows directory (note: nested under `data/`, unlike the other `~/.cline/<component>/` dirs above) |
| `~/Documents/Cline/Rules/`, `~/Documents/Cline/Hooks/`, `~/Documents/Cline/Workflows/`, `~/Documents/Cline/Plugins/` | Global (alternate/legacy path, Windows: `Documents\Cline\Rules` etc.) | Global rules/hooks/workflows/plugins |
| `.cline/` (rules, skills, hooks, agents, plugins, cron) | Project | Project-specific config, meant to be committed (keep secrets out) |
| `.clinerules` (single file) or `.clinerules/` (directory) | Project | Project instruction rules |
| `.clinerules/hooks/` | Project | Project-level hook scripts |
| `.clinerules/workflows/` | Project | Project-level workflow files (slash commands) |
| `CLINE_DATA_DIR` env var | — | Overrides `~/.cline/data/` |
| `CLINE_HOOKS_DIR` env var | — | Overrides hooks directory |
| `CLINE_COMMAND_PERMISSIONS` env var | — | JSON allow/deny policy restricting shell commands |

❓ The exact relationship/precedence between the newer `.cline/` project directory and the older `.clinerules` / `.clinerules/` convention is not fully spelled out in the fetched docs — both are documented as current, and `.clinerules` is explicitly still the hooks/workflows location.

---

## Instruction File

Project rules: a single `.clinerules` file, or a `.clinerules/` directory at the project root containing `.md`/`.txt` files (Cline merges all of them into one context block; numeric prefixes like `01-coding.md` are an optional ordering convention).

Global rules: `~/Documents/Cline/Rules/` (path varies by OS; macOS/Linux: `~/Documents/Cline/Rules`).

Individual rule files support YAML frontmatter with a `paths` glob field for conditional/scoped activation (e.g. only active when files under `src/components/**` are open/edited). Rules without frontmatter are always active.

Cline also recognizes instruction files from other tools/conventions: `.cursorrules` (Cursor), `.windsurfrules` (Windsurf), and `AGENTS.md` / `~/.agents/AGENTS.md` (cross-tool standard).

Workspace rules take precedence over global rules when they conflict.

---

## Hooks

Cline has a genuine, documented hook/lifecycle-event system (script-based, similar in spirit to Claude Code's hooks). It is off by default and must be enabled in settings ("Enable Hooks" under Feature Settings). Per the official in-repo docs, hooks currently run on **macOS/Linux only — Windows is not supported**.

Hook scripts are placed in:
- Global: `~/Documents/Cline/Hooks/<HookName>`
- Workspace/project: `.clinerules/hooks/<HookName>`

Files must be named exactly after the hook event (no extension, e.g. `PreToolUse`), must start with a shebang line (e.g. `#!/usr/bin/env bash`), and must be made executable (`chmod +x`) on Unix-like systems — a git-hooks-style convention.

### Supported Events

| Event | When | Can Block? | Notes |
|---|---|---|---|
| `TaskStart` | A new task starts (not on resume) | ✅ (`cancel: true`) | |
| `TaskResume` | An existing task is resumed | ✅ | |
| `TaskCancel` | A task is cancelled / aborted mid-work | ❌ (documented as "NOT cancellable") | Only fires if there was active/started work |
| `TaskComplete` | A task is marked complete | ✅ (per shared schema) | Documented as "coming soon" in the source README |
| `UserPromptSubmit` | User submits a prompt (initial task, resume, or feedback) | ✅ | |
| `PreToolUse` | Before a tool executes | ✅ | Cannot alter the tool's already-decided parameters, only block or inject context for future turns |
| `PostToolUse` | After a tool completes | ❌ (informational; can still return `cancel`, but tool already ran) | |
| `PreCompact` | Before conversation context is compacted/truncated | ❓ | Documented as "coming soon" in the source README |

❓ `TaskComplete` and `PreCompact` are documented but explicitly marked "coming soon" in the official hooks README as of the fetched date — treat as not-yet-live.

### Hook Input (stdin JSON)

```json
{
  "clineVersion": "string",
  "hookName": "PreToolUse",
  "timestamp": "string",
  "taskId": "string",
  "workspaceRoots": ["string"],
  "userId": "string",
  "preToolUse": {
    "toolName": "write_to_file",
    "parameters": { "path": "src/index.ts" }
  }
}
```

Each event nests its own payload key (`taskStart`, `taskResume`, `taskCancel`, `taskComplete`, `userPromptSubmit`, `preToolUse`, `postToolUse`, `preCompact`) alongside the shared base fields above. `postToolUse` additionally includes `result`, `success`, and `executionTimeMs`.

### Hook Output (stdout JSON)

```json
{
  "cancel": false,
  "contextModification": "string, optional — injected into the NEXT AI turn, not the current one",
  "errorMessage": "string, optional — shown to the user if cancel is true"
}
```

There is no separate "approve" bypass value documented for the script-based hook system in the official README fetched here (some secondary/third-party sources mention an "approve" option for `PreToolUse` — ❓ unconfirmed against the primary source, treat as unverified).

### Exit Code Behavior

The documented mechanism is **JSON-field-based, not exit-code-based** — hooks signal blocking via the `cancel: true` field in their stdout JSON, not via process exit code. If multiple hooks (global + per-workspace-root) fire for the same event, they may run concurrently; results are combined: `cancel` blocks if **any** hook returns `true`, and `contextModification`/`errorMessage` strings are concatenated across all fired hooks.

| Signal | Meaning |
|---|---|
| stdout JSON `"cancel": false` (or omitted) | Allow execution to continue |
| stdout JSON `"cancel": true` | Block execution; `errorMessage` shown to user |
| Hook exceeds 30 s (default; `HOOK_EXECUTION_TIMEOUT_MS`) | Times out |
| `contextModification` > 50 KB (default; `MAX_CONTEXT_MODIFICATION_SIZE`) | Truncated/limited |

### Example Config

No JSON hook-registration config exists — hooks are auto-discovered by filename/path, not declared in a settings file. Example script (`.clinerules/hooks/PreToolUse`):

```bash
#!/usr/bin/env bash
input=$(cat)
tool_name=$(echo "$input" | jq -r '.preToolUse.toolName')
path=$(echo "$input" | jq -r '.preToolUse.parameters.path // ""')

if [[ "$tool_name" == "write_to_file" && "$path" == *.js ]]; then
  echo '{"cancel": true, "errorMessage": "Project rule: use .ts files only"}'
  exit 0
fi

echo '{"cancel": false}'
```

Separately, the SDK (`@cline/sdk`) exposes a **lower-level plugin hook system** with different event names (`session_start`, `run_start`, `iteration_start`, `turn_start`, `before_agent_start`, `tool_call_before`, `tool_call_after`, `turn_end`, `stop_error`, `iteration_end`, `run_end`, `session_shutdown`, `error`), configurable per-hook with `mode: "blocking"|"async"`, `timeoutMs`, `retries`, `retryDelayMs`, `failureMode: "fail_open"|"fail_closed"`, `maxConcurrency`, and `queueLimit`. This is a programmatic API for building custom agents/plugins, distinct from the file-based `.clinerules/hooks/` feature above. ❓ Full input/output schema for this SDK-level hook API was not available from the fetched docs pages.

---

## Built-in Tools

Per the classic/extension tool surface (XML-style tool names used in the agent loop):

| Tool | Description |
|---|---|
| `read_file` | Read file contents |
| `write_to_file` | Create or overwrite a file |
| `replace_in_file` | Targeted diff-style edits to a file |
| `search_files` | Regex search across files |
| `list_files` | List directory contents |
| `list_code_definition_names` | List top-level code definitions in a directory |
| `execute_command` | Run a shell/CLI command |
| `use_mcp_tool` | Call a tool exposed by a connected MCP server |
| `access_mcp_resource` | Read a resource exposed by an MCP server |
| `ask_followup_question` | Ask the user a clarifying question |
| `use_skill` | Load and activate a Skill (`SKILL.md`) |
| `use_subagents` | Spawn parallel read-only research subagents |

The SDK's `ClineCore` engine documents a smaller, renamed built-in toolset for programmatic use: `bash`, `editor`, `read_files`, `apply_patch`, `search`, `fetch_web`, `ask_question`. The tools-reference docs now explicitly address this: "older docs/examples reference XML-style names like `read_file`, `replace_in_file`, or `execute_command`. Current SDK/ClineCore runtime uses the built-in tool names listed above" — i.e. `read_files`, `apply_patch`, and `bash` are the modern equivalents of `read_file`/`replace_in_file`(+`write_to_file`)/`execute_command`. ❓ A full 1:1 mapping for every remaining tool (e.g. `search_files`/`list_files`/`list_code_definition_names` vs. `search`) is still not spelled out.

---

## MCP Support

Config locations:
- CLI: `~/.cline/mcp.json`
- Extension: `~/.cline/data/settings/cline_mcp_settings.json` (edited via the MCP Servers panel → Configure → "Configure MCP Servers", or by hand)

```json
{
  "mcpServers": {
    "local-server": {
      "command": "node",
      "args": ["/path/to/server.js"],
      "env": { "API_KEY": "your_api_key" },
      "disabled": false,
      "autoApprove": []
    },
    "remote-server": {
      "type": "streamableHttp",
      "url": "https://example.com/mcp",
      "headers": { "Authorization": "Bearer token" },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

Supported transports: local **STDIO** (`command`/`args`/`env`), **Streamable HTTP** (recommended for hosted/remote servers), and legacy **SSE** (backward compatibility only).

CLI-specific: `cline mcp` opens an interactive wizard to list/add/edit/enable/disable/delete servers; `cline config mcp [--json]` supports non-interactive querying/scripting.

---

## Skills / Commands

**Skills** — location: `~/.cline/skills/` (global; `C:\Users\USERNAME\.cline\skills\` on Windows) or project-level skill directories (`.cline/skills/`, `.clinerules/skills/`, and — for cross-tool interop — `.claude/skills/`). Format: a directory per skill containing a required `SKILL.md` (YAML frontmatter + instructions, recommended under ~5k tokens) plus optional `docs/` and `scripts/` subdirectories. Cline uses progressive disclosure — it sees only the skill's frontmatter `description` at first, and calls `use_skill` to load the full `SKILL.md` when the description matches the user's request.

**Workflows (slash commands)** — location: `.clinerules/workflows/` (project) or `~/.cline/data/workflows/` (global, primary; `~/Documents/Cline/Workflows/` is also discovered as an alternate/legacy global path). Format: one Markdown file per workflow, e.g. `pr-review.md` becomes `/pr-review`. Typing the slash command wraps that file's content in `explicit_instructions` tags and injects it into the current message; it runs once and does not persist like a rule. Project workflows take precedence over global ones on a name collision.

---

## Agent / Subagent Configuration

Cline supports **subagents** — parallel, read-only research agents spawned via the `use_subagents` tool, available in both VS Code and the CLI, enabled by default. Each subagent gets its own prompt, its own context window/token budget, and can use `read_file`, `list_files`, `search_files` (regex), `list_code_definition_names`, safe read-only shell commands (`ls`, `grep`, `git log`, `git diff`, etc.), and `use_skill`. Subagents explicitly **cannot** write/edit files, apply patches, use the browser, call MCP servers, web search, or spawn nested subagents. Toggle: Settings → Features → Agent → `use_subagents` (applies uniformly across VS Code, JetBrains, and CLI).

Separately, **CLI 2.0** supports running multiple fully-isolated top-level `cline` instances in parallel (e.g. separate tmux panes), each with its own state/conversation/model config — this is process-level parallelism, distinct from the in-task `use_subagents` feature.

The **SDK** additionally exposes an `AgentPlugin` model for building custom multi-agent/tool/hook bundles registered into `ClineCore` via an `extensions` array or `pluginPaths`. ❓ Beyond this SDK-level building block, no separate "subagent definition file" format (comparable to Claude Code's `.claude/agents/*.md`) was found in the fetched official docs.

---

## Notes

- Cline is fully open source (Apache-2.0), unlike several competitors in this catalog that are proprietary CLIs with open-source-adjacent SDKs only.
- The CLI is explicitly positioned by the vendor as "more than just Cline in the terminal" — a scriptable agent layer meant for CI/CD and shell composition. The CLI 2.0 announcement blog described a `-y` flag for full autonomy, but the current official CLI reference (fetched 2026-08-15) documents this as `--auto-approve <boolean>` instead (default `true`; defaults to `false` in `--acp` mode) — no bare `-y` flag appears in the current flag table, so treat `-y` as superseded/renamed. `--json` (structured output) and piped stdin/stdout are both current.
- `--acp` flag makes the CLI speak the Agent Client Protocol, for use inside editors like Zed/JetBrains/Neovim/Emacs without a Cline-specific plugin.
- The hooks feature (file-based, `.clinerules/hooks/`) is explicitly Unix-only (macOS/Linux); there is no Windows hook support as of the fetched docs.
- `CLINE_COMMAND_PERMISSIONS` env var provides allow/deny policy control over `execute_command`/shell usage, independent of the hooks system.
- "Free to use" models were being offered for a limited time on the Cline provider at the time of the CLI 2.0 announcement (Minimax M2.5, Kimi K2.5) — a promotional detail, not a permanent pricing feature.

---

## Sources (Official)

| Topic | URL | Fetched | Label |
|---|---|---|---|
| CLI 2.0 announcement / features | https://cline.bot/blog/introducing-cline-cli-2-0 | 2026-08-15 | [official] |
| CLI overview / getting started | https://docs.cline.bot/cline-cli/getting-started | 2026-08-15 | [official] |
| CLI command reference / flags | https://docs.cline.bot/cli/cli-reference | 2026-08-15 | [official] |
| Global/project config file locations | https://docs.cline.bot/getting-started/config | 2026-08-15 | [official] |
| Config file locations by component (canonical table incl. workflows path) | https://docs.cline.bot/customization/overview | 2026-08-15 | [official] |
| Rules (`.clinerules`) reference | https://docs.cline.bot/customization/cline-rules | 2026-08-15 | [official] |
| Hooks feature page (redirects to SDK plugins for schema) | https://docs.cline.bot/features/hooks | 2026-08-15 | [official] |
| Hooks — full event list, JSON schema, examples (in-repo docs) | https://github.com/cline/cline/blob/main/.clinerules/hooks/README.md | 2026-08-15 | [github] |
| Hooks v3.36 announcement | https://cline.bot/blog/cline-v3-36-hooks | 2026-08-15 | [official] |
| SDK plugins / low-level hook stages | https://docs.cline.bot/sdk/plugins | 2026-08-15 | [official] |
| Subagents reference | https://docs.cline.bot/features/subagents | 2026-08-15 | [official] |
| MCP server configuration | https://docs.cline.bot/mcp/configuring-mcp-servers | 2026-08-15 | [official] |
| Skills reference | https://docs.cline.bot/customization/skills | 2026-08-15 | [official] |
| Tools reference (extension tool list) | https://docs.cline.bot/tools-reference/all-cline-tools | 2026-08-15 | [official] |
| Workflows / slash commands (behavior; does not document file locations — see config-locations row above) | https://docs.cline.bot/core-workflows/using-commands | 2026-08-15 | [official] |
| GitHub repo (license, monorepo layout: apps/cli, apps/vscode, sdk/) | https://github.com/cline/cline | 2026-08-15 | [github] |
| "Return to the primitives" CLI philosophy post | https://cline.bot/blog/cline-cli-return-to-the-primitives | 2026-08-15 | [official] |
