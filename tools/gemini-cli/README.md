# Gemini CLI

> Google's open-source AI agent for the terminal, powered by Gemini models.

**Vendor:** Google | **License:** Apache 2.0 | **Runtime:** Node.js

## Links

- Docs: https://geminicli.com/docs
- Hooks overview: https://geminicli.com/docs/hooks/
- Hooks reference: https://geminicli.com/docs/hooks/reference/
- Writing hooks: https://geminicli.com/docs/hooks/writing-hooks/
- GitHub: https://github.com/google-gemini/gemini-cli
- Google blog: https://developers.googleblog.com/tailor-gemini-cli-to-your-workflow-with-hooks/

---

## Installation

```sh
npm install -g @google/gemini-cli
gemini
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.gemini/settings.json` | Global | User settings, hooks |
| `.gemini/settings.json` | Project | Project-level settings |

Configs are merged, with project settings taking precedence.

## Hooks

Hooks act as middleware for the agentic loop, running synchronously at defined lifecycle points.

### Supported Events

| Event | When | Can Block? |
|-------|------|-----------|
| `before_tool_call` | Before every tool call | ✅ |
| `after_tool_call` | After every tool call | ❌ |
| `on_session_start` | Session initialization | ❌ |
| `on_session_end` | Session teardown | ❌ |

### Configuration Example

```json
{
  "hooks": {
    "before_tool_call": [
      {
        "matcher": "run_shell_command",
        "command": "~/.gemini/hooks/validate-cmd.sh"
      }
    ],
    "after_tool_call": [
      {
        "command": "~/.gemini/hooks/audit-log.sh"
      }
    ]
  }
}
```

### Hook Input (stdin JSON)

```json
{
  "event": "before_tool_call",
  "tool_name": "run_shell_command",
  "tool_input": { "command": "ls -la" },
  "session_id": "abc123"
}
```

### Exit Code Behavior

| Code | Meaning |
|------|---------|
| `0` | Continue |
| Non-zero | Block (before_tool_call); warning (after) |

### Best Practices

- Keep hooks fast — they run synchronously and block execution.
- Use specific matchers to limit execution scope.
- Hooks run with your user privileges; validate external inputs carefully.

## Built-in Tools

| Tool | Description |
|------|-------------|
| `run_shell_command` | Execute shell commands |
| `read_file` | Read file contents |
| `write_file` | Write files |
| `edit_file` | Apply edits |
| `list_directory` | List directory |
| `glob` | File pattern matching |
| `grep` | Search file contents |
| `web_fetch` | Fetch URLs |
| `web_search` | Web search (via Google) |

## MCP Support

- Configure MCP servers in `settings.json` under `mcpServers`
- Extensions can also contribute hooks via `settings.json`

## Extensions

Extensions can provide additional tools and hook configurations. Hooks from installed extensions are merged with user/project hooks.

## Notes

- Tool context injection (git commits, tickets, docs) is a primary hook use case.
- Dynamic tool filtering via hooks allows narrowing available tools per-session.

## Sources (Official)

| Topic | URL |
|-------|-----|
| Hooks overview | https://geminicli.com/docs/hooks/ |
| Hooks reference | https://geminicli.com/docs/hooks/reference/ |
| Writing hooks | https://geminicli.com/docs/hooks/writing-hooks/ |
| Hooks index (GitHub) | https://github.com/google-gemini/gemini-cli/blob/main/docs/hooks/index.md |
| Hooks reference (GitHub) | https://github.com/google-gemini/gemini-cli/blob/main/docs/hooks/reference.md |
| Writing hooks (GitHub) | https://github.com/google-gemini/gemini-cli/blob/main/docs/hooks/writing-hooks.md |
| Tools reference | https://geminicli.com/docs/tools/file-system/ |
| Tools (GitHub pages) | https://google-gemini.github.io/gemini-cli/docs/tools/ |
| Google blog (hooks intro) | https://developers.googleblog.com/tailor-gemini-cli-to-your-workflow-with-hooks/ |
| Main docs | https://geminicli.com/docs/ |
| GitHub repo | https://github.com/google-gemini/gemini-cli |
