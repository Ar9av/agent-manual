# OpenClaw

> Open-source AI coding agent with gateway architecture and hook-based safety rails.

**Vendor:** OpenClaw (community) | **License:** Open Source

## Links

- Docs: https://docs.openclaw.ai
- Hooks: https://docs.openclaw.ai/automation/hooks
- Configuration: https://docs.openclaw.ai/gateway/configuration
- Configuration reference: https://docs.openclaw.ai/gateway/configuration-reference
- Agent config: https://docs.openclaw.ai/gateway/config-agents
- GitHub: https://github.com/openclaw/openclaw
- AGENTS.md: https://github.com/openclaw/openclaw/blob/main/AGENTS.md
- Tools docs: https://github.com/openclaw/openclaw/blob/main/docs/tools/index.md
- CLI agents docs: https://github.com/openclaw/openclaw/blob/main/docs/cli/agents.md
- Pi integration: https://docs.openclaw.ai/pi

---

## Installation

```sh
# Via npm
npm install -g openclaw

# Via binary
curl -fsSL https://openclaw.ai/install | bash
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.openclaw/config.yaml` | Global | User settings |
| `config.yaml` | Project | Project settings |
| `AGENTS.md` | Project | Natural-language instructions |

## Hooks

> ⚠️ **IMPORTANT:** `PreToolUse`/`PostToolUse` with exit-code blocking are **open feature requests** (issues #60943, #12311, #1733) — **NOT yet shipped** as of April 2026. The information below reflects the current shipped hook system.

OpenClaw's current hook system is event-based (session, command, and message lifecycle events) with **no blocking mechanism**. Handlers execute asynchronously.

### Current Hook Events (No Blocking)

| Event | When |
|-------|------|
| `command:new` | `/new` command issued |
| `command:reset` | `/reset` command issued |
| `command:stop` | `/stop` command issued |
| `command` | Any command (general listener) |
| `session:compact:before` | Before context compaction |
| `session:compact:after` | After compaction completes |
| `session:patch` | Session properties modified |
| `agent:bootstrap` | Before workspace bootstrap files injected |
| `gateway:startup` | After channels start and hooks loaded |
| `gateway:shutdown` | Gateway shutdown begins |
| `gateway:pre-restart` | Before expected gateway restart |
| `message:received` | Inbound message from any channel |
| `message:transcribed` | After audio transcription |
| `message:preprocessed` | After media/link preprocessing |
| `message:sent` | Outbound message delivered |

**Input format:** Event object with `type`, `action`, `sessionKey`, `timestamp`, `messages[]`, and `context` (event-specific).

### Planned (Feature Requests)

Pre/post tool use blocking hooks (like Claude Code's `PreToolUse`) are proposed to use:
```yaml
# PROPOSED — not yet shipped
hooks:
  preToolUse:
    - ./hooks/validate-exec.sh
  postToolUse:
    - ./hooks/audit-external.sh
```
With env vars `HOOK_TOOL_NAME`, `HOOK_TOOL_INPUT`, `HOOK_EVENT`. Track issue #60943 for status.

## Built-in Tools

| Tool | Description |
|------|-------------|
| `bash` | Execute shell commands |
| `read_file` | Read files |
| `write_file` | Write files |
| `edit_file` | Apply edits |
| `search` | Search codebase |
| `web_fetch` | Fetch URLs |

## MCP Support

OpenClaw supports MCP servers configured in `config.yaml` under `mcpServers`.

## Agent Configuration

```yaml
# config.yaml
agents:
  my-agent:
    model: claude-opus-4
    system: "You are a security-focused code reviewer"
    tools:
      - read_file
      - search
    hooks:
      preToolUse:
        - ./hooks/read-only-guard.sh
```

## Gateway Mode

OpenClaw includes a gateway mode for team deployments, allowing centralized hook enforcement and agent routing policies.

## Notes

- **PreToolUse/PostToolUse blocking hooks are NOT yet shipped** — open feature requests in issues #60943, #12311, #1733 as of April 2026.
- Current hooks are session/message lifecycle events with no blocking mechanism.
- Gateway configuration allows admin-level hook enforcement across teams.

## Sources (Official)

| Topic | URL |
|-------|-----|
| Hooks | https://docs.openclaw.ai/automation/hooks |
| Configuration | https://docs.openclaw.ai/gateway/configuration |
| Configuration reference | https://docs.openclaw.ai/gateway/configuration-reference |
| Agent configuration | https://docs.openclaw.ai/gateway/config-agents |
| AGENTS.md (GitHub) | https://github.com/openclaw/openclaw/blob/main/AGENTS.md |
| Tools docs (GitHub) | https://github.com/openclaw/openclaw/blob/main/docs/tools/index.md |
| CLI agents docs (GitHub) | https://github.com/openclaw/openclaw/blob/main/docs/cli/agents.md |
| PreToolUse feature request | https://github.com/openclaw/openclaw/issues/60943 |
