# Codex CLI

> OpenAI's lightweight coding agent that runs in your terminal.

**Vendor:** OpenAI | **License:** Apache 2.0 | **Runtime:** Rust

## Links

- Docs: https://developers.openai.com/codex
- CLI reference: https://developers.openai.com/codex/cli/reference
- Hooks: https://developers.openai.com/codex/hooks
- Skills: https://developers.openai.com/codex/skills
- AGENTS.md guide: https://developers.openai.com/codex/guides/agents-md
- GitHub: https://github.com/openai/codex
- Config schema: https://github.com/openai/codex/blob/main/docs/config.md

---

## Installation

```sh
npm install -g @openai/codex
# or
cargo install codex
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.codex/config.toml` | Global | User settings, model, hooks |
| `.codex/config.toml` | Project | Project overrides |
| `requirements.toml` | Managed | Admin-enforced policies |
| `AGENTS.md` | Project | Natural-language instructions |

## Hooks

Hooks are an extensibility framework for injecting scripts into the agentic loop.

### Enabling/Disabling

```toml
[features]
hooks = true   # default; set false to disable all hooks
```

To force disable at admin level:
```toml
# requirements.toml
allow_managed_hooks_only = true
```

### Supported Events

| Event | When |
|-------|------|
| `pre_tool` | Before a tool executes |
| `post_tool` | After a tool executes |
| `session_start` | Agent session begins |
| `session_end` | Agent session ends |
| `pre_prompt` | Before sending prompt to LLM |
| `post_response` | After receiving LLM response |

### Hook Security Model

- Each unique `(event, command)` pair prompts for user approval on first run.
- Trust persisted to `~/.codex/shell-hooks-allowlist.json` keyed by hash.
- Changed hooks re-prompt for approval.
- Use `/hooks` in CLI to inspect, trust, or disable hooks.

### Configuration Example

```toml
[[hooks]]
event = "pre_tool"
command = "~/.codex/hooks/validate-tool.sh"

[[hooks]]
event = "post_tool"
command = "~/.codex/hooks/log-tool.sh"
```

### Hook Environment Variables

| Variable | Description |
|----------|-------------|
| `CODEX_TOOL_NAME` | Name of the tool being called |
| `CODEX_TOOL_INPUT` | JSON-encoded tool input |
| `CODEX_SESSION_ID` | Current session ID |

## Built-in Tools

| Tool | Description |
|------|-------------|
| `shell` | Execute shell commands |
| `read_file` | Read file contents |
| `write_file` | Write files |
| `patch_file` | Apply diffs |
| `list_files` | List directory |
| `search` | Search files |

## MCP Support

- Configure MCP servers in `config.toml` under `[mcp]`
- Servers provide additional tools to the agent

## Agent Skills

Skills are reusable, on-demand expertise modules. Configure in `config.toml` or place in `.codex/skills/`.

## AGENTS.md

Place `AGENTS.md` at the repo root to give the agent persistent natural-language instructions. Supports sections for tools, style, and constraints.

## GitHub Action

Codex ships a first-party GitHub Action for running automated coding tasks in CI:
https://developers.openai.com/codex/github-action

## Notes

- `codex_hooks` is a deprecated alias for `[features].hooks`.
- Managed hooks from `requirements.toml` are always allowed even when `allow_managed_hooks_only = true`.

## Sources (Official)

| Topic | URL |
|-------|-----|
| Hooks | https://developers.openai.com/codex/hooks |
| CLI reference | https://developers.openai.com/codex/cli/reference |
| CLI features | https://developers.openai.com/codex/cli/features |
| Config reference | https://developers.openai.com/codex/config-reference |
| Advanced config | https://developers.openai.com/codex/config-advanced |
| Config file schema (GitHub) | https://github.com/openai/codex/blob/main/docs/config.md |
| AGENTS.md guide | https://developers.openai.com/codex/guides/agents-md |
| Skills | https://developers.openai.com/codex/skills |
| GitHub Action | https://developers.openai.com/codex/github-action |
| Main docs | https://developers.openai.com/codex |
| GitHub repo | https://github.com/openai/codex |
