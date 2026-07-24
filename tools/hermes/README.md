# Hermes Agent

> NousResearch's self-improving AI agent with shell-script hooks and a plugin system.

**Vendor:** NousResearch | **License:** Open Source | **Runtime:** Python

## Links

- Docs: https://hermes-agent.nousresearch.com/docs
- Configuration: https://hermes-agent.nousresearch.com/docs/user-guide/configuration
- Event Hooks: https://hermes-agent.nousresearch.com/docs/user-guide/features/hooks
- CLI commands reference: https://hermes-agent.nousresearch.com/docs/reference/cli-commands
- GitHub: https://github.com/NousResearch/hermes-agent
- CLI config example: https://github.com/NousResearch/hermes-agent/blob/main/cli-config.yaml.example
- AGENTS.md: https://github.com/NousResearch/hermes-agent/blob/main/AGENTS.md
- Community docs: https://github.com/mudrii/hermes-agent-docs

---

## Installation

**Desktop (Windows/macOS):** Download the Hermes Desktop installer from the official website.

**Linux/macOS/WSL2/Android (Termux):**
```sh
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

**Windows (PowerShell):**
```powershell
iex (irm https://hermes-agent.nousresearch.com/install.ps1)
```

After installation, run `hermes setup --portal` to configure OAuth for model access and Tool Gateway features.

**pip (Python):**
```sh
pip install hermes-agent
```
As of v0.14.0 (May 16, 2026), `pip install hermes-agent` is an officially documented installation method. The binary is placed at `~/.local/bin/hermes` via `console_scripts`.

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.hermes/config.yaml` | Global | User settings, shell hooks, model config |
| `~/.hermes/.env` | Global | API keys and secrets |
| `~/.hermes/auth.json` | Global | OAuth provider credentials |
| `~/.hermes/shell-hooks-allowlist.json` | Global | Trusted hook hashes |
| `~/.hermes/SOUL.md` | Global | Agent identity (system prompt slot #1) |
| `AGENTS.md` | Project | Natural-language instructions |

## Hooks

Hermes provides three hook systems at different lifecycle points:

| System | Registration | Scope | Purpose |
|--------|--------------|-------|---------|
| Shell hooks | `hooks:` block in `~/.hermes/config.yaml` | CLI + Gateway | Drop-in scripts: blocking, formatting, context injection |
| Plugin hooks | `ctx.register_hook()` in plugin | CLI + Gateway | Tool interception, metrics, guardrails |
| Gateway hooks | `HOOK.yaml` + `handler.py` in `~/.hermes/hooks/` | Gateway only | Logging, alerts, webhooks |

Shell hooks require no Python authoring — they run as subprocesses and communicate over stdin/stdout JSON.

### Plugin Hook Events (17 events — CLI + Gateway)

| Event | When |
|-------|------|
| `pre_tool_call` | Before tool execution; can block with `{"action": "block", "message": str}` |
| `post_tool_call` | After tool returns |
| `transform_tool_result` | Post-execution, pre-model; rewrites result string |
| `transform_terminal_output` | Inside `terminal` tool, pre-truncation/redaction |
| `pre_llm_call` | Once per turn, before tool loop; injects context via `{"context": str}` |
| `post_llm_call` | Once per turn, after tool loop completes |
| `pre_verify` | Before the agent finishes after editing code |
| `on_session_start` | New session created (first turn only) |
| `on_session_end` | End of every `run_conversation()` call |
| `on_session_finalize` | Session teardown before identity is discarded |
| `on_session_reset` | Gateway swaps in fresh session key (user invoked `/new` or `/reset`) |
| `pre_gateway_dispatch` | After internal-event guard, before auth/dispatch |
| `pre_approval_request` | Before approval shown to user; observer-only |
| `post_approval_response` | After user responds to approval prompt |
| `subagent_start` | Before a child agent (via `delegate_task`) is constructed and run |
| `subagent_stop` | After `delegate_task` child exits |
| `transform_llm_output` | After tool loop, before final response delivery |

> **Note:** The event previously listed as `on_subagent_complete` does not exist. The correct name is `subagent_stop`.

### Gateway-Exclusive Events (10 events)

| Event | When |
|-------|------|
| `gateway:startup` | Gateway process initialization |
| `session:start` | New messaging session created |
| `session:end` | Session termination (before reset) |
| `session:reset` | User invoked `/new` or `/reset` |
| `agent:start` | Agent begins processing a message |
| `agent:step` | Each tool-calling loop iteration |
| `agent:end` | Agent finishes processing |
| `reaction:added` | Emoji reaction added to a message (Slack) |
| `reaction:removed` | Emoji reaction removed from a message |
| `command:*` | Any slash command executed (wildcard matching supported) |

### Shell Hook Configuration

Shell hooks are declared in `~/.hermes/config.yaml` (not `cli-config.yaml`):

```yaml
# ~/.hermes/config.yaml
hooks:
  pre_tool_call:
    - matcher: "<regex>"         # Optional; pre/post_tool_call only
      command: ~/.hermes/agent-hooks/validate-tool.sh
      timeout: 30                # Optional; default 60s, max 300s
  post_tool_call:
    - command: ~/.hermes/agent-hooks/audit.sh
  on_session_start:
    - command: ~/.hermes/agent-hooks/setup-context.sh
hooks_auto_accept: false         # Skip consent prompts when true
```

**Wire protocol:**
- **stdin:** `{"hook_event_name", "tool_name", "tool_input", "session_id", "cwd", "extra"}`
- **stdout:** `{"decision": "block", "reason": "..."}` or `{"context": "..."}` or `{}`

### Hook Security / Approval Model

- Each unique `(event, command)` pair prompts for consent on first run.
- Trust persisted to `~/.hermes/shell-hooks-allowlist.json` keyed by hook hash.
- **Script edits do NOT automatically trigger re-approval.** Use `hermes hooks doctor` to detect mtime drift and `hermes hooks revoke` to reset consent.
- Default timeout: 60 seconds. Maximum: 300 seconds (values above 300 are clamped with a warning). Non-blocking errors are logged; they never crash the agent.

**Bypassing consent prompts:**
1. `hermes --accept-hooks chat` — CLI flag
2. `HERMES_ACCEPT_HOOKS=1` — environment variable
3. `hooks_auto_accept: true` in `~/.hermes/config.yaml`

### Hook Use Cases

- **Block**: Reject dangerous terminal commands
- **Run after**: Auto-format files, log API calls
- **Inject context**: Git status into next LLM turn
- **Observe**: Track subagent completion, session lifecycle

## Built-in Tools

| Tool | Description |
|------|-------------|
| `bash` | Execute shell commands |
| `read_file` | Read file contents |
| `write_file` | Write files |
| `edit_file` | Apply edits |
| `search` | Search codebase |
| `web_fetch` | Fetch URLs |

## MCP Support

Configure MCP servers in `~/.hermes/config.yaml` under `mcp_servers`.

## Subagents

Hermes supports spawning subagents for parallel workloads via `delegate_task`. The `subagent_stop` hook fires when a subagent exits.

## `hermes hooks` CLI

```sh
hermes hooks list                          # Show configured hooks with matcher, timeout, consent status
hermes hooks test <event> [--for-tool X] [--payload-file F]  # Fire hooks against synthetic payload
hermes hooks revoke <command>              # Remove allowlist entries (aliases: remove, rm)
hermes hooks doctor                        # Audit exec bit, allowlist, mtime drift, JSON validity, timing
```

> **Note:** `hermes hooks trust <id>` and `hermes hooks disable <id>` do not exist. Use `revoke` (aliases: `remove`, `rm`) to remove allowlist entries, and `doctor` to audit hook health.

## `hermes config` CLI

```sh
hermes config           # (show) View current configuration
hermes config edit      # Open config.yaml in your editor
hermes config set KEY VAL  # Set specific values
hermes config path      # Print the config file path
hermes config env-path  # Print the .env file path
hermes config check     # Verify all options after updates
hermes config migrate   # Interactively add missing options
```

## AGENTS.md

Place `AGENTS.md` at repo root for persistent natural-language instructions that persist across sessions.

## Notes

- **RESOLVED:** Both `pre_tool_call`/`post_tool_call` (tool-level) AND `pre_llm_call`/`post_llm_call` (loop-level) exist — they are different lifecycle stages, not duplicates.
- Shell hooks run as subprocesses; keep them fast. Default timeout 60s, max 300s.
- `hooks_auto_accept: true` or env `HERMES_ACCEPT_HOOKS=1` skips the per-hook consent prompt.
- `SOUL.md` defines agent identity/personality — unique to Hermes. Located at `~/.hermes/SOUL.md`.
- Six terminal backends: local, Docker, SSH, Daytona, Singularity, Modal.
- Machine-readable docs: `/llms.txt` (~17 KB) and `/llms-full.txt` (~1.8 MB).

## Sources

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| Installation | https://hermes-agent.nousresearch.com/docs/getting-started/installation | 2026-07-23 | [official] |
| Event hooks | https://hermes-agent.nousresearch.com/docs/user-guide/features/hooks | 2026-07-23 | [official] |
| CLI commands reference | https://hermes-agent.nousresearch.com/docs/reference/cli-commands | 2026-07-23 | [official] |
| Configuration | https://hermes-agent.nousresearch.com/docs/user-guide/configuration | 2026-07-23 | [official] |
| Hooks (GitHub source) | https://github.com/NousResearch/hermes-agent/blob/main/website/docs/user-guide/features/hooks.md | — | [github] |
| CLI config example | https://github.com/NousResearch/hermes-agent/blob/main/cli-config.yaml.example | — | [github] |
| AGENTS.md | https://github.com/NousResearch/hermes-agent/blob/main/AGENTS.md | — | [github] |
| GitHub repo | https://github.com/NousResearch/hermes-agent | — | [github] |
