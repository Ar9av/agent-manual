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

```sh
pip install hermes-agent
hermes
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.hermes/config.yaml` | Global | User settings, hooks |
| `cli-config.yaml` | Project | Project config |
| `~/.hermes/shell-hooks-allowlist.json` | Global | Trusted hook hashes |
| `AGENTS.md` | Project | Natural-language instructions |

## Hooks

Hooks are shell scripts declared in `cli-config.yaml` that fire at plugin-hook events in both CLI and gateway sessions. No Python plugin authoring required.

### Supported Events

| Event | When |
|-------|------|
| `pre_llm_call` | Before sending prompt to LLM |
| `post_llm_call` | After receiving LLM response |
| `pre_tool_call` | Before tool execution |
| `post_tool_call` | After tool execution |
| `on_session_start` | Session initialization |
| `on_session_end` | Session teardown |
| `on_subagent_complete` | Subagent finishes |

### Configuration Example

```yaml
# cli-config.yaml
hooks:
  pre_tool_call:
    - command: ~/.hermes/hooks/validate-tool.sh
      events: ["pre_tool_call"]
  post_tool_call:
    - command: ~/.hermes/hooks/audit.sh
  on_session_start:
    - command: ~/.hermes/hooks/setup-context.sh
```

### Hook Security / Approval Model

- Each unique `(event, command)` pair prompts for approval on first run.
- Trust persisted to `~/.hermes/shell-hooks-allowlist.json` keyed by hook hash.
- Changed hooks re-prompt for approval.
- Use `hermes hooks` CLI command to inspect, test, and manage hook trust.

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

Configure MCP servers in `cli-config.yaml` under `mcp_servers`.

## Subagents

Hermes supports spawning subagents for parallel workloads. The `on_subagent_complete` hook fires when a subagent finishes.

## `hermes hooks` CLI

```sh
hermes hooks list        # Show all configured hooks
hermes hooks test <event> # Send synthetic payload to test hooks
hermes hooks trust <id>   # Trust a specific hook
hermes hooks disable <id> # Disable a hook
```

## AGENTS.md

Place `AGENTS.md` at repo root for persistent natural-language instructions that persist across sessions.

## Notes

- **RESOLVED:** Both `pre_tool_call`/`post_tool_call` (tool-level) AND `pre_llm_call`/`post_llm_call` (loop-level) exist — they are different lifecycle stages, not duplicates.
- Shell hooks run as subprocesses; keep them fast. Max timeout is 300s.
- `hooks_auto_accept: true` or env `HERMES_ACCEPT_HOOKS=1` skips the per-hook consent prompt.
- `SOUL.md` defines agent identity/personality — unique to Hermes.

## Sources (Official)

| Topic | URL |
|-------|-----|
| Event hooks | https://hermes-agent.nousresearch.com/docs/user-guide/features/hooks |
| Tools reference | https://hermes-agent.nousresearch.com/docs/reference/tools-reference |
| Configuration | https://hermes-agent.nousresearch.com/docs/user-guide/configuration |
| CLI commands reference | https://hermes-agent.nousresearch.com/docs/reference/cli-commands |
| Hooks (GitHub source) | https://github.com/NousResearch/hermes-agent/blob/main/website/docs/user-guide/features/hooks.md |
| CLI config example | https://github.com/NousResearch/hermes-agent/blob/main/cli-config.yaml.example |
| AGENTS.md | https://github.com/NousResearch/hermes-agent/blob/main/AGENTS.md |
| GitHub repo | https://github.com/NousResearch/hermes-agent |
