# Tool Name

> One-line description of the tool.

**Vendor:** | **License:** | **Runtime:**

## Links

- Docs: 
- GitHub: 
- Changelog: 

---

## Installation

```sh
# install command
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.config/<tool>/config.json` | Global | User-level settings |
| `./<tool>.json` | Project | Project-level overrides |

## Instruction File

The agent reads natural-language instructions from: (e.g., `AGENTS.md`, `.tool/rules.md`)

## Hooks

> If no hook system exists, explain the workaround (MCP, rules, etc.) here.

### Supported Events

| Event | When | Can Block? |
|-------|------|-----------|
| `PreToolUse` | Before every tool call | ✅ (exit 2) |
| `PostToolUse` | After every tool call | ❌ |

### Hook Input (stdin JSON)

```json
{
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": { "command": "..." },
  "session_id": "..."
}
```

### Hook Output (stdout JSON, optional)

```json
{
  "decision": "block",
  "reason": "Reason shown to LLM"
}
```

### Exit Code Behavior

| Code | Meaning |
|------|---------|
| `0` | Success, continue |
| `2` | Block tool (PreToolUse only); stderr sent to LLM |
| Other | Warning; execution continues |

### Example Config

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{ "type": "command", "command": "./hooks/validate.sh" }]
      }
    ]
  }
}
```

## Built-in Tools

| Tool | Description |
|------|-------------|
| `Bash` | Run shell commands |
| `Read` | Read file contents |
| `Write` | Write files |
| `Edit` | Patch files |

## MCP Support

Config location:

```json
{
  "mcpServers": {
    "my-server": { "command": "node", "args": ["./mcp/server.js"] }
  }
}
```

## Tool Substitution

> Can built-in tools be turned OFF in favor of MCP-provided ones — not just supplemented? See `_shared/mcp-tool-substitution.md` for the cross-tool reference and methodology.

- **Server trust**: does a project-declared MCP server auto-load, or need approval? Can a project-shared config file grant that trust, or only a user-local one?
- **Native tool disablement**: can built-ins be turned off so only MCP tools remain, or is MCP strictly additive?
- **MCP tool naming + permissioning**: what is a connected MCP tool called (e.g. `mcp__server__tool`)? How do you pre-allow it without an interactive prompt?
- **Headless behaviour**: what happens in CI/non-interactive mode when a tool call still needs approval — clean failure, hang, or silent auto-approve?

## Skills / Commands

- Skills location: 
- Format: 

## Agent / Subagent Configuration

## Notes

## Sources (Official)

| Topic | URL |
|-------|-----|
| Hooks | |
| Config | |
| GitHub repo | |
