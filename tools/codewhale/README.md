# Codewhale

> Provider-neutral Rust terminal coding agent with a TUI-scoped shell hook system, fleets, workflows, and a cloud dispatch path.

**Vendor:** Hmbown (community, independently maintained) | **License:** MIT | **Runtime:** Rust (single checksummed binary)

## Links

- Docs / site: https://codewhale.net/
- Hooks: https://github.com/Hmbown/Codewhale/blob/HEAD/docs/HOOKS.md
- MCP: https://github.com/Hmbown/Codewhale/blob/HEAD/docs/MCP.md
- Configuration: https://github.com/Hmbown/Codewhale/blob/HEAD/docs/CONFIGURATION.md
- Providers / local models: https://github.com/Hmbown/Codewhale/blob/HEAD/docs/PROVIDERS.md
- GitHub: https://github.com/Hmbown/Codewhale

---

## Installation

```sh
curl -fsSL https://codewhale.net/install.sh | sh
```

**Live-verified on st3ve (Ubuntu 24.04, 2026-09-10):** installed v0.9.12
(`dcd4c200f72f`). The installer verifies checksums and installs two commands,
`codewhale` and the short alias `codew`, into `~/.local/bin`. `codewhale update`
updates a release binary in place; package-managed installs get migration
instructions instead.

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.codewhale/config.toml` | Global | Provider, auth mode, hooks, telemetry (TOML) |
| `~/.codewhale/secrets/secrets.json` | Global | File-based secret store for API keys |
| `~/.codewhale/builtin-plugins/` | Global | Bundled plugins (ships `computer-use`) |
| `~/.codewhale/telemetry/` | Global | Telemetry buffer, install id, state |
| `AGENTS.md` | Project | Instruction file — `codewhale init` scaffolds a default |

Config is TOML, read and written through `codewhale config get|set|list`.
Credential lookup order is **config → secret store → env**, reported by
`codewhale auth get` / `auth status` without ever printing the credential.

## Instruction File

`AGENTS.md` in the project root. `codewhale init` creates a default one.

## Hooks

Hooks run a shell command when Codewhale reaches a lifecycle point. They receive
context through environment variables, some receive a JSON payload on stdin, and
three of them can steer what Codewhale does next.

> ⚠️ **Hooks are a TUI-runtime feature only.** This is the single most important
> fact about Codewhale hooks for anyone building an interception layer.

| Surface | Fires hooks |
|---------|-------------|
| `codewhale` / `codew` interactive TUI | ✅ yes |
| `codewhale exec` (headless one-shot) | ❌ no |
| the CLI dispatcher and its subcommands | ❌ no |
| app-server / ACP | ❌ no |
| the `workflow` tool and sub-agent *internals* | ❌ no — but the TUI fires `subagent_spawn` / `subagent_complete` around them |
| public API | there is none |

A CI or headless automation path therefore gets **no hook coverage at all**.

> The `crates/hooks` event-sink crate in the repository is an unrelated internal
> mechanism sharing no configuration, event names, or contract with these hooks.

### Supported Events

| Event | When | Can Block / Steer? |
|-------|------|-------------------|
| `session_start` | once, after the engine is up and before the first draw | observer |
| `session_end` | once, on graceful shutdown | observer |
| `turn_end` | after a turn completes and post-turn state is updated | observer |
| `message_submit` | before a submitted message reaches history or the model | ✅ **can replace or block the text** |
| `tool_call_before` | before each tool call executes | ✅ **allow / deny / ask, rewrite input, add context** |
| `tool_call_after` | after each tool result settles | observer |
| `mode_change` | on every applied Plan/Work/Operate transition (`Act` is a compatibility alias for Work) | observer |
| `on_error` | on transport, capacity, and auth errors, and on tool failures | observer |
| `subagent_spawn` | when a sub-agent starts | observer |
| `subagent_complete` | when a sub-agent completes, fails, or is cancelled | observer |
| `shell_env` | immediately before each `exec_shell` invocation | ✅ **contributes environment variables** |
| `session_idle` | when the session settles back to idle — no prompt, approval, or continuation outstanding | observer |
| `session_error` | when a turn ends in a terminal failure; transient tool failures the agent absorbs never fire it | observer |

❓ **Documentation discrepancy:** the configuration section describes `event` as
"one of the 11 names below" while the event table lists **13**. Treat 13 as the
implemented set and the "11" as stale prose.

### Example Config

```toml
# ~/.codewhale/config.toml
[hooks]
enabled = true                 # global switch; false suppresses every hook
default_timeout_secs = 30      # see the timeout note below
working_dir = "/path/to/dir"   # default: the session workspace

[[hooks.hooks]]
event = "tool_call_before"     # required
command = "~/.codewhale/hooks/gate.sh"  # required; `sh -c` on Unix, `cmd /C` on Windows
name = "gate"                  # optional label for /hooks and log lines
timeout_secs = 30              # optional, default 30
background = false             # optional; foreground inside the hook worker
continue_on_error = true       # optional, default true
condition = { type = "tool_name", name = "exec_shell" }  # optional
```

### Gotchas

- **`default_timeout_secs` overrides, it does not default.** When
  `[hooks].default_timeout_secs` is set it *replaces* every hook's own
  `timeout_secs`. Leave it unset for per-hook timeouts to apply. `/hooks list`
  shows the timeout the runtime will actually use.
- **A timed-out hook never blocks the turn.**
- **`background = true` neuters gating.** A background hook is enqueued without
  blocking into a fixed 32-entry supervisor queue, carries no exit code, and
  "can never allow, deny, ask, or rewrite anything" — a `deny` gate written as a
  background hook is silently inert, which the docs themselves flag as the
  dangerous form.
- **`exit_code` conditions need a real exit code.** They match only on
  `tool_call_after`, or `on_error` for tool failures. A tool that reports no exit
  code never matches.
- `/hooks` lists what is configured, the global switch state, and entries
  rejected at load. `/hooks events` lists the event names. `/hooks revoke`
  blocks future and queued launches but does not stop already-running commands.

## Built-in Tools

Observed live in `exec --auto --output-format stream-json` on st3ve:

| Tool | Description | Verified |
|------|-------------|----------|
| `bash` | Shell execution | ✅ |
| `read` | Read file contents | ✅ |
| `write` | Create/overwrite a file | ✅ |
| `tool_search` | Search the available tool catalog (observed being called before an MCP tool) | ✅ |
| `exec_shell` | Shell invocation targeted by the `shell_env` hook and `tool_name` conditions | doc |

❓ The hooks doc references `exec_shell` while the live stream-json run reported
`bash`; a normalizer should accept both spellings until the tool table is
published.

## MCP Support

✅ Full, with the widest MCP CLI of the six tools in this pass.

Config lives in its **own file**, `~/.codewhale/mcp.json` — *not* `config.toml`.
`codewhale mcp init` writes a template:

```json
{
  "timeouts": { "connect_timeout": 10, "execute_timeout": 60, "read_timeout": 120 },
  "servers": {
    "example": {
      "command": "node",
      "args": ["./path/to/your-mcp-server.js"],
      "env": {}, "url": null,
      "connect_timeout": null, "execute_timeout": null, "read_timeout": null,
      "disabled": true, "enabled": true, "required": false,
      "enabled_tools": [], "disabled_tools": []
    }
  }
}
```

```
codewhale mcp list | init | connect | tools | add | login | logout
              | remove | enable | disable | validate | add-self
```

`mcp add` takes flags, not positionals:
`codewhale mcp add <NAME> --command node --arg /path/server.js`. URL servers
support `--transport sse`, `--bearer-token-env-var`, `--oauth-client-id`,
`--oauth-resource`, and `--scope`. `mcp add-self` registers the Codewhale binary
itself as a local stdio MCP server.

## Tool Substitution

**Live-verified 2026-09-10 on st3ve.**

- **Server trust**: ❌ **none at all.** `codewhale mcp add` → `mcp enable` →
  `mcp tools` listed both tools, and a real `exec --auto` run called one, with no
  approval step anywhere in the chain.
- **Native tool disablement**: ❌ **not possible — MCP is strictly additive.** A
  binary string search found `disabled_tools` / `enabled_tools` (only in the
  *per-MCP-server* schema) and `allowed_tools` (the approval allowlist), but zero
  matches for `disable_builtin`, `available_tools`, `excluded_tools`,
  `builtin_tools`, or `no_builtin`. Confirmed behaviourally: an `exec --auto` run
  wrote a real file to `/tmp` with no way to remove the baseline write/shell
  tools.
- **MCP tool naming**: `mcp_<server>_<tool>`, hyphens preserved — e.g.
  **`mcp_weather-svc_get_forecast`**. Single underscore prefix, unlike omp's
  double.
- **Headless behaviour**: clean, no hang. But note hooks fire **only in the
  TUI**, so the headless path has no gating layer at all — no trust gate, no
  disablement, and no hooks.

## Skills / Commands

`codewhale setup` bootstraps skills directories alongside MCP config. Agent
roles are kept as readable files in the project or user config.

## Agent / Subagent Configuration

Sub-agents run through the `workflow` tool and a Lane Runtime backend:

- `fleet` — manage durable Agent fleet runs
- `workflow` — run checked-in Workflows through a Lane Runtime backend
- `lane` — manage running workflow instances (Lanes) and Runtime backends
- `dispatch` / `cloud-agent` — offload a coding agent to the Codewhale cloud;
  "never spends or pushes without `--confirm`"

## Notes

- **`exec` is not agentic by default.** Plain `codewhale exec "<prompt>"` is a
  one-shot model response with no tool use. `--auto` enables tool-backed agent
  mode with auto-approvals — this is the supported automation path used by
  stream-json wrappers. Verified live: without `--auto` the agent described the
  shell commands it *would* run instead of running them.
- **Flag order matters.** `--model` must precede the subcommand:
  `codewhale --model <m> exec "<prompt>"`. Placing it after `exec` errors out
  with an explicit message.
- **Telemetry is on by default** — Codewhale and PostHog process aggregate
  version/platform, session, feature, and error counts when delivery is
  configured; no content or IP. Disable with
  `codewhale config set telemetry false`.
- Ships a `speech` / `tts` command backed by Xiaomi MiMo TTS models, and a
  bundled `computer-use` plugin.
- Independently maintained and explicitly "not affiliated with any model
  provider" despite session-format compatibility with other harnesses.

## Live Verification (st3ve, 2026-09-10)

Ubuntu 24.04.4, Codewhale 0.9.12, OpenAI via `OPENAI_API_KEY`.

```sh
codewhale config set provider openai
printf '%s' "$OPENAI_API_KEY" | codewhale auth set --provider openai --api-key-stdin
codewhale --model gpt-4.1-mini exec --auto "<task>"
```

- ✅ `auth set --api-key-stdin` accepts a key on stdin and does not echo it;
  `auth status` reports source layer and last-4 only.
- ✅ Full five-part task (list dir, search TODO, read a file, write
  `summary.md`, run a shell command) completed with `--auto`.
- ✅ `--output-format stream-json` emits typed events plus a terminal
  `metadata` receipt with `input_tokens`, `prompt_cache_hit_tokens`,
  `approval_posture`, `sandbox_posture`, and the binary's `sha256`.
- ❌ **GPT-5-family models fail on the OpenAI route.** Codewhale 0.9.12 sends
  `max_tokens` on the chat-completions wire, which GPT-5 models reject:

  ```
  error: Invalid request (400): Unsupported parameter: 'max_tokens' is not
  supported with this model. Use 'max_completion_tokens' instead.
  ```

  `gpt-4.1-mini` works. `gpt-5.4-mini` does not. The receipt reports
  `codewhale_max_output_tokens_source: "uncatalogued"` for GPT-5 models, so the
  model catalog appears not to know the GPT-5 family yet.

## Sources (Official)

| Topic | URL |
|-------|-----|
| Hooks | https://github.com/Hmbown/Codewhale/blob/HEAD/docs/HOOKS.md |
| MCP | https://github.com/Hmbown/Codewhale/blob/HEAD/docs/MCP.md |
| Configuration | https://github.com/Hmbown/Codewhale/blob/HEAD/docs/CONFIGURATION.md |
| Providers | https://github.com/Hmbown/Codewhale/blob/HEAD/docs/PROVIDERS.md |
| GitHub repo | https://github.com/Hmbown/Codewhale |
