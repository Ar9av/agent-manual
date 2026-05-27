# Kiro IDE / CLI

> AI-native IDE and CLI with deep hook support for agent lifecycle control.

**Vendor:** Kiro.dev | **License:** Proprietary | **Runtime:** Electron (IDE) / Node.js (CLI)

## Links

- Docs (CLI): https://kiro.dev/docs/cli/
- Hooks (CLI): https://kiro.dev/docs/cli/hooks/
- Hooks (IDE): https://kiro.dev/docs/hooks/
- Hook types (IDE): https://kiro.dev/docs/hooks/types/
- Built-in tools: https://kiro.dev/docs/cli/reference/built-in-tools/
- CLI commands: https://kiro.dev/docs/cli/reference/cli-commands/
- Agent config reference: https://kiro.dev/docs/cli/custom-agents/configuration-reference/
- Agent examples: https://kiro.dev/docs/cli/custom-agents/examples/
- Creating custom agents: https://kiro.dev/docs/cli/custom-agents/creating/

---

## Installation

```sh
npm install -g @kiro/cli
kiro
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.kiro/config.yaml` | Global | User settings |
| `.kiro/config.yaml` | Project | Project settings |
| `.kiro/agents/` | Project | Custom agent definitions |

## Hooks

Hooks execute custom commands at specific points during agent lifecycle and tool execution.

### Supported Events

| Event | When | Provides | Can Block? |
|-------|------|----------|-----------|
| `agentSpawn` | Agent activated | No tool context | ❌ |
| `userPromptSubmit` | User submits prompt | Prompt text | ❌ (output added to context) |
| `preToolUse` | Before tool execution | Tool name + input | ✅ (exit 2) |
| `postToolUse` | After tool execution | Tool name + result | ❌ |

### Hook Input (stdin JSON)

```json
{
  "hook_event_name": "preToolUse",
  "cwd": "/Users/user/project",
  "session_id": "abc123",
  "tool_name": "Bash",
  "tool_input": { "command": "rm -rf dist/" }
}
```

### Exit Code Behavior

| Code | Meaning |
|------|---------|
| `0` | Success, continue |
| `2` | Block tool execution (preToolUse only); stderr sent to LLM |
| Other | Hook failed; stderr shown as warning |

### Agent Hook Configuration

```yaml
# .kiro/agents/my-agent.yaml
name: my-agent
model: claude-sonnet-4
hooks:
  agentSpawn:
    - command: ~/.kiro/hooks/setup.sh
  preToolUse:
    - matcher: Bash
      command: ~/.kiro/hooks/validate-bash.sh
  postToolUse:
    - command: ~/.kiro/hooks/audit.sh
```

## Built-in Tools

| Tool | Description |
|------|-------------|
| `Bash` | Execute shell commands |
| `Read` | Read file contents |
| `Write` | Write files |
| `Edit` | Apply string replacements |
| `Glob` | File pattern matching |
| `Grep` | Search in files |
| `LS` | List directory |
| `WebFetch` | Fetch URLs |

## MCP Support

Kiro CLI supports MCP servers for extending tool capabilities. Configure in `.kiro/config.yaml` under `mcpServers`.

## Custom Agents

Create agents with specific system prompts, model selections, tool access, and hooks in `.kiro/agents/`.

```yaml
name: code-reviewer
model: claude-opus-4
system: "You are a strict code reviewer..."
tools:
  - Read
  - Grep
  - Glob
hooks:
  preToolUse:
    - command: ./hooks/block-writes.sh
```

## Notes

- Documentation last updated April 13, 2026.
- `userPromptSubmit` hook output is injected into the conversation context, not shown to the user.
- IDE hooks and CLI hooks share the same event model but use different config file locations.

## Sources (Official)

| Topic | URL |
|-------|-----|
| CLI hooks | https://kiro.dev/docs/cli/hooks/ |
| IDE hooks overview | https://kiro.dev/docs/hooks/ |
| IDE hook types | https://kiro.dev/docs/hooks/types/ |
| CLI built-in tools | https://kiro.dev/docs/cli/reference/built-in-tools/ |
| CLI commands | https://kiro.dev/docs/cli/reference/cli-commands/ |
| Agent config reference | https://kiro.dev/docs/cli/custom-agents/configuration-reference/ |
| Creating custom agents | https://kiro.dev/docs/cli/custom-agents/creating/ |
| Agent examples | https://kiro.dev/docs/cli/custom-agents/examples/ |
| Steering (IDE) | https://kiro.dev/docs/steering/ |
| Steering (CLI) | https://kiro.dev/docs/cli/steering/ |
| IDE changelog | https://kiro.dev/changelog/ide/ |
