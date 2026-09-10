# Reasonix (DeepSeek-Reasonix)

> DeepSeek-native Go/TypeScript terminal agent engineered around prefix-cache stability, with Claude-Code-shaped JSON hooks and a fail-closed bash sandbox.

**Vendor:** ESEngine (community) | **License:** MIT | **Runtime:** Go core + npm-delivered prebuilt native binary

## Links

- Docs / site: http://reasonix.io/
- Hooks: https://github.com/esengine/DeepSeek-Reasonix/blob/HEAD/docs/DESKTOP_HOOKS.zh-CN.md
- Config paths: https://github.com/esengine/DeepSeek-Reasonix/blob/HEAD/docs/CONFIG_PATHS.md
- Provider catalog: https://github.com/esengine/DeepSeek-Reasonix/blob/HEAD/docs/PROVIDER_CATALOG.md
- Reasoning providers: https://github.com/esengine/DeepSeek-Reasonix/blob/HEAD/docs/REASONING_PROVIDERS.md
- Subagent profiles: https://github.com/esengine/DeepSeek-Reasonix/blob/HEAD/docs/SUBAGENT_PROFILES.md
- Tool contract: https://github.com/esengine/DeepSeek-Reasonix/blob/HEAD/docs/TOOL_CONTRACT.md
- GitHub: https://github.com/esengine/DeepSeek-Reasonix

---

## Installation

```sh
npm i -g reasonix                          # any OS; pulls the prebuilt native binary
brew install esengine/reasonix/reasonix    # macOS
```

**Live-verified on st3ve (Ubuntu 24.04, 2026-09-10):** `npm i -g reasonix`
installed v1.38.3 as 2 packages — the npm package is a thin fetcher for the
prebuilt binary, not a source build.

## Configuration Files

Reasonix uses **one** user-facing home directory shared by CLI and desktop.

| File | Scope | Purpose |
|------|-------|---------|
| `~/.reasonix/config.toml` | Global | Non-secret config: providers, plugins, UI, tools, skills, sandbox, permissions, bot, agent (TOML) |
| `~/.reasonix/.env` | Global | Provider credentials — **secrets live here, never in `config.toml`** |
| `~/.reasonix/settings.json` | Global | Hooks |
| `<workspace>/.reasonix/settings.json` | Project | Project hooks |
| `./reasonix.toml` | Project | Project config overrides |
| `~/.reasonix/commands/` | Global | Slash commands |
| `~/.reasonix/skills/` | Global | Skills |
| `~/.reasonix/projects/<slug>/sessions/` | Global | Per-workspace session JSONL + event index |
| `~/.reasonix/remote/known_hosts` | Global | Remote-SSH managed known_hosts |

| Platform | Reasonix home |
|----------|---------------|
| macOS / Linux | `~/.reasonix` |
| Windows | `%APPDATA%\reasonix` |

Resolution order: **flag > `./reasonix.toml` > `~/.reasonix/config.toml` >
built-in defaults.** `REASONIX_HOME` overrides the home entirely and makes the
runtime fully self-contained (legacy migration and OS-convention scanning are
skipped, so nothing leaks in from a system-wide install).
`REASONIX_STATE_HOME` moves sessions/archives/memory only — **not** global
config or provider credentials.

### Provider Configuration

Provider entries store the *name of the credential variable* in `api_key_env`,
never the secret value:

```toml
config_version = 1
default_model = "openai/gpt-5.4-mini"

[[providers]]
name        = "openai"
kind        = "openai"
base_url    = "https://api.openai.com/v1"
models      = ["gpt-5.4-mini", "gpt-4.1-mini"]
default     = "gpt-5.4-mini"
api_key_env = "OPENAI_API_KEY"
```

with the value in `~/.reasonix/.env`:

```
OPENAI_API_KEY=sk-...
```

> ⚠️ **Defining `[[providers]]` replaces the built-in presets** rather than
> extending them. Adding an OpenAI entry removes `deepseek-flash`/`deepseek-pro`
> from the model list unless you re-declare them.

**Credential hardening (notable):** saved provider and bot credential variables
are stripped from every model-controlled child-process environment, and the
global credential `.env` is hidden from Reasonix's own file readers, sandboxed
shell commands, and MCP servers. A project's ordinary `.env` is unaffected.

## Instruction File

`AGENTS.md` — `reasonix init` prints how to generate project memory.

## Hooks

Claude-Code-shaped JSON hooks that execute local shell commands at session,
user-input, tool-call, model-return, and compaction points. The desktop app's
Settings → Hooks editor reads and writes the same `settings.json`.

### Supported Events

| Event | When | Can Block? |
|-------|------|-----------|
| `SessionStart` | session begins | ❌ |
| `SessionEnd` | session ends | ❌ |
| `UserPromptSubmit` | user input submitted | ✅ |
| `PreToolUse` | before a tool call | ✅ |
| `PostToolUse` | after a tool call | ❌ |
| `PostLLMCall` | after a model response returns | ❓ |
| `PreCompact` | before context compaction | ❓ |
| `Stop` | turn finished | ❌ |
| `SubagentStop` | subagent finished | ❌ |
| `Notification` | notification raised | ❌ |

❓ Blocking semantics are confirmed for the tool/prompt gates; `PostLLMCall` and
`PreCompact` blocking was not verified.

### Example Config

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "match": "bash",
        "command": "node .reasonix/hooks/check-bash.js",
        "description": "Block dangerous shell commands",
        "timeout": 5000
      }
    ],
    "Stop": [
      { "command": "echo Reasonix turn finished" }
    ]
  }
}
```

### Scope and Ordering

| Scope | Path | Loading | Order |
|-------|------|---------|-------|
| Project | `<workspace>/.reasonix/settings.json` | automatic | **first** |
| Global | `<Reasonix home>/settings.json` | automatic | after project |

Project hooks run before global hooks; within a scope, array order. On a
blocking event, the **first** blocking hook stops the rest from running.

> ⚠️ Saving hooks requires a **restart** to take effect. `/new` starts a new
> conversation but does not re-read the hook configuration.

### Inspection

`reasonix hook list --json` and `reasonix hook status --json` return a redacted,
schema-versioned view — verified live:

```json
{"schema_version":1,"command":"hook.list","hooks":[]}
```

## Built-in Tools

Observed live in session JSONL on st3ve:

| Tool | Type | Verified |
|------|------|----------|
| `bash` | shell | ✅ |
| `read_file` | file read | ✅ |
| `write_file` | file write | ✅ |
| `use_capability` | capability/skill dispatch | ✅ |
| `edit_file` / `multi_edit` / `move_file` | file write (named in the sandbox docs as the file-writer set) | doc |

`[tools] enabled = []` means all built-ins; `bash_timeout_seconds` defaults to
120 with `0` for no tool-local cap.

## MCP Support

MCP servers are configured as `[[plugins]]` and contribute tools, prompts, and
resources. `reasonix mcp <add|remove|list|import>` manages them.

```toml
[[plugins]]
name    = "example"
command = "reasonix-plugin-example"
startup_timeout_seconds = 60
call_timeout_seconds = 600
tool_timeout_seconds = { "generate_video" = 1800 }

[[plugins]]                                  # remote server over Streamable HTTP
name    = "stripe"
type    = "http"
url     = "https://mcp.stripe.com"
headers = { Authorization = "Bearer ${STRIPE_KEY}" }
```

`type` is `stdio` (default), `http`, or `sse`. `${VAR}` / `${VAR:-default}` are
expanded from the environment in `command`, `args`, `env`, `url`, and `headers`.
Timeouts: `mcp_startup_timeout_seconds` (default 30) and
`mcp_call_timeout_seconds` (default 300), with per-plugin overrides.

## Tool Substitution

- **Native tool disablement**: ✅ `[tools] enabled = [...]` is an explicit
  allowlist — an empty list means all built-ins, so a narrow list plus MCP
  plugins gets close to MCP-only operation.
- **Server trust**: plugins are declared in config; `${VAR}` expansion means a
  project config can reference environment credentials.
- ❓ MCP tool naming and headless pre-approval not live-verified.

## Permissions and Sandbox

Two independent layers, both worth knowing before writing a policy:

```toml
[permissions]
mode  = "ask"    # writer fallback when no rule matches: ask|allow|deny
# deny = ["Bash(rm -rf*)", "Bash(git push*)"]   # hard-blocked in every mode
# allow = ["Bash(go test:*)", "Bash(git status:*)"]
# ask = ["Edit(src/**)"]
```

Rules are `Tool` or `Tool(specifier)`. Precedence is **deny > ask > allow >
fallback**. Readers always default to allow.

```toml
[sandbox]
bash    = "enforce"   # enforce | off
network = true
# workspace_root = ""      # default: cwd
# allow_write = ["/tmp"]   # extra dirs writers may modify
# forbid_read = []
```

> ⚠️ **Reasonix fails closed on bash.** With `bash = "enforce"` (the default on
> macOS/Linux) each command is jailed in an OS sandbox, and **if no sandbox is
> available, bash execution is refused entirely.** Windows has no OS-level bash
> sandbox and fixes `bash = "off"`.

CLI permission modes: `--permission-mode ask|acceptEdits|yolo`, plus
`--allowed-tools` rules for `-p` runs.

## Skills / Commands

- Skills: `~/.reasonix/skills/`, extra roots via `[skills] paths`, hidden roots
  via `excluded_paths`, nested scan depth via `max_depth` (default 3).
- Slash commands: `~/.reasonix/commands/`.
- `disable_implicit_invocation` keeps skills available but stops automatic model
  invocation; `disabled_skills` hides individual ones.

## Agent / Subagent Configuration

`reasonix subagent <list|create|edit|delete|try|run>` manages isolated subagent
profiles. Config knobs:

| Key | Purpose |
|-----|---------|
| `subagent_model` / `subagent_models` | Default and per-skill subagent models |
| `subagent_effort` / `subagent_efforts` | Default and per-tool/skill effort |
| `max_subagent_depth` | Nested delegation depth (set 1 to disable nesting) |
| `max_subagent_concurrency` | Session-wide concurrency (task/fleet/skills), default 6 |
| `max_parallel_writers` | Concurrent writers with non-overlapping `write_paths` |
| `planner_model` | Two-model collaboration |
| `vision_model` | Image summarization for text-only models |
| `recovery_model` | Beyond rule-only recovery |

## Notes

- **Prefix-cache stability is the design goal** — "leave it running." The status
  bar can surface `cache`, `cache_avg`, and per-turn cost receipts.
- Ships far beyond a CLI: `reasonix web` (local Web UI), `serve` (HTTP+SSE with
  `none|token|password` auth), `acp` (Agent Client Protocol over stdio),
  `review` (diff review), a desktop Electron shell, and a **multi-channel IM bot
  gateway** (`bot`) for QQ, Feishu/Lark, WeChat, and DingTalk with allowlists,
  approvers, and a loopback control API.
- `[secrets] filter_subprocess_env` and `protect_sensitive_files` are **opt-in**
  and documented as breaking `gh`, HTTPS git push, and `npm publish` / legitimate
  edit workflows respectively.
- `reasonix doctor [--json]`, `doctor session <id> --zip`, and redacted
  `session`/`task`/`hook` JSON queries make it unusually introspectable for
  machine clients.
- Telemetry is content-free and configurable via
  `reasonix config telemetry auto|on|off`; crash reports are local until
  explicitly sent with `reasonix report send`.

## Live Verification (st3ve, 2026-09-10)

Ubuntu 24.04.4, Reasonix 1.38.3, OpenAI `gpt-5.4-mini`.

- ✅ Provider config accepted; `reasonix doctor` reported
  `openai  openai  api.openai.com  key:present` and
  `model  openai/gpt-5.4-mini`.
- ❌ **The bash sandbox blocked all shell execution out of the box.** `doctor`
  reported:

  ```
  bash  enforce (unavailable: no OS sandbox on this host; bash execution is
  refused. Install bubblewrap (`bwrap`) or set [sandbox] bash = "off" ...)
  ```

  Installing `bubblewrap` was **not sufficient** on Ubuntu 24.04: the distro
  ships `kernel.apparmor_restrict_unprivileged_userns=1`, so `bwrap` fails with
  `setting up uid map: Permission denied` and Reasonix still refuses to run
  unconfined. Setting `[sandbox] bash = "off"` restored execution. Reasonix
  fails **closed** here, which is the right default but will surprise anyone
  running it in a container or hardened host.
- ❌ `-p` alone still declined shell calls because `[permissions] mode = "ask"`
  cannot prompt non-interactively. `--permission-mode yolo` was required.
- ✅ With `bash = "off"` and `--permission-mode yolo`, both the simple task and
  the full five-part task completed; `summary.md` was written.
- Tool names observed in session JSONL: `bash`, `read_file`, `write_file`,
  `use_capability`.
- ⚠️ `reasonix run --events-jsonl` produced no parseable tool-name fields in
  this run — the redacted event stream did not expose them where the session
  JSONL did.
- ⚠️ Writing a minimal `config.toml` caused Reasonix to rewrite it into a fully
  annotated ~250-line file on next run. Hand-edited configs are normalized, so
  place top-level tables carefully — an appended `[sandbox]` block after an
  existing one was ignored until the file was rewritten with `[sandbox]` ahead
  of `[[providers]]`.

## Sources (Official)

| Topic | URL |
|-------|-----|
| Hooks | https://github.com/esengine/DeepSeek-Reasonix/blob/HEAD/docs/DESKTOP_HOOKS.zh-CN.md |
| Config paths | https://github.com/esengine/DeepSeek-Reasonix/blob/HEAD/docs/CONFIG_PATHS.md |
| Provider catalog | https://github.com/esengine/DeepSeek-Reasonix/blob/HEAD/docs/PROVIDER_CATALOG.md |
| Reasoning providers | https://github.com/esengine/DeepSeek-Reasonix/blob/HEAD/docs/REASONING_PROVIDERS.md |
| Subagent profiles | https://github.com/esengine/DeepSeek-Reasonix/blob/HEAD/docs/SUBAGENT_PROFILES.md |
| Tool approval modes | https://github.com/esengine/DeepSeek-Reasonix/blob/HEAD/docs/TOOL_APPROVAL_MODES.md |
| GitHub repo | https://github.com/esengine/DeepSeek-Reasonix |
