# OpenClaw

> Open-source AI coding agent featuring a gateway architecture, extensible hooks, and robust multi-agent tool sandboxing.

**Vendor:** OpenClaw (community) | **License:** MIT | **Runtime:** Node.js / TypeScript

## Links

- Docs: https://docs.openclaw.ai
- Hooks Documentation: https://docs.openclaw.ai/automation/hooks
- Apply Patch Tool: https://docs.openclaw.ai/tools/apply-patch
- GitHub: https://github.com/openclaw/openclaw

---

## Installation

### Via npm
```sh
npm install -g openclaw
```

### Via Binary Installer
```sh
curl -fsSL https://openclaw.ai/install | bash
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.openclaw/config.yaml` | Global | User-level settings |
| `config.yaml` | Project | Project-level overrides |
| `~/.openclaw/hooks/` | Global | Managed hook definitions shared across workspaces |
| `<workspace>/hooks/` | Project | Workspace-level hook overrides |

## Instruction File

OpenClaw loads natural-language system instructions and steering prompts from:
- **`AGENTS.md`** (or `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md` in the project root)
- Custom boot commands can also be loaded via `BOOT.md` when the gateway launches.

### Loading Extra Instructions
Additional instructions can be automatically injected across multi-project setups via the bundled `bootstrap-extra-files` internal hook:
```json
{
  "hooks": {
    "internal": {
      "entries": {
        "bootstrap-extra-files": {
          "enabled": true,
          "paths": ["packages/*/AGENTS.md", "packages/*/TOOLS.md"]
        }
      }
    }
  }
}
```

## Hooks

OpenClaw features two distinct hook systems depending on the required level of automation: **Internal Hooks** for event-driven side effects and **Plugin Hooks** for in-process lifecycle policy enforcement.

---

### 1. Internal Hooks (Coarse Operator Automation)
Internal hooks are event-driven, non-blocking TypeScript/JavaScript scripts loaded by the gateway. They run asynchronously and are designed for side-effects (e.g., logging, notification). Active hooks can be listed, checked, and controlled via the CLI using `openclaw hooks [list|info|check|enable|disable]`.

#### Directory Structure
Each hook is a directory placed in a registered hooks directory:
```
my-hook/
├── HOOK.md          # Metadata, emoji, platforms, and required bins
└── handler.ts       # TypeScript handler implementation
```

#### Supported Internal Hook Events

| Event | When | Context Payload Highlights |
|-------|------|---------------------------|
| `command:new` | `/new` command issued | `sessionEntry`, `workspaceDir`, `cfg` |
| `command:reset` | `/reset` command issued | `sessionEntry`, `previousSessionEntry` |
| `command:stop` | `/stop` command issued | Diagnostic details |
| `command` | Any command event | Universal command context |
| `session:compact:before` | Before contexts compact | `messageCount`, `tokenCount` |
| `session:compact:after` | After compaction completes | `tokensBefore`, `tokensAfter` |
| `session:patch` | Session fields patched | Privileged patches, `sessionEntry` |
| `agent:bootstrap` | Before agent workspace starts | Mutable `bootstrapFiles` array |
| `gateway:startup` | Gateway processes start up | Startup environment state |
| `gateway:shutdown` | Gateway process begins exit | `reason`, `restartExpectedMs` |
| `gateway:pre-restart` | Before scheduled gateway restart | Bounded drain budget (10s limit) |
| `message:received` | Inbound chat message arrives | `from`, `content`, `channelId`, `metadata` |
| `message:sent` | Outbound message delivered | `to`, `content`, `success` |
| `message:transcribed` | Audio file transcribes | `transcript`, `mediaPath` |
| `message:preprocessed` | Inbound media/link processed | Final enriched `bodyForAgent` |

#### Handler Interface (TypeScript)
```typescript
import { promisify } from "util";
import { execFile } from "child_process";

const execFileAsync = promisify(execFile);

export default async function handler(event) {
  if (event.type !== "command" || event.action !== "new") return;
  
  // Custom logic
  console.log(`[my-hook] Session started: ${event.context.workspaceDir}`);
  
  // Push optional replies (only command:* or message:received can receive replies)
  event.messages.push("Custom operator hook executed successfully.");
}
```

---

### 2. Typed Plugin Hooks (In-Process Lifecycle Control)
For blocking execution, rewriting inputs, or preventing actions (e.g., denying tool calls or modifying LLM prompts), OpenClaw uses **Typed Plugin Hooks** registered via the Plugin SDK (`api.on(...)`).

| SDK Hook | When it fires | Can Block? | Notes |
|----------|---------------|------------|-------|
| `before_tool_call` | Before any agent tool call | ✅ Yes | Can rewrite `tool_input` or block execution |
| `before_agent_reply` | Before final reply is delivered | ✅ Yes | Can modify prompt or inject extra context |
| `before_agent_finalize` | Agent finishes reasoning | ✅ Yes | Can request another reasoning loop |
| `before_install` | Before plugin installation | ✅ Yes | Prevents untrusted plugin installation |
| `session_end` | Session is finalized/shutdown | ❌ No | Clean up connection locks and databases |

## Built-in Tools

| Tool | Parameters | Description |
|------|------------|-------------|
| `bash` | `command` | Run terminal commands on the host |
| `read_file` | `path` | Retrieve raw contents of files (text/images) |
| `write_file` | `path`, `content` | Write or overwrite files in the workspace |
| `edit_file` | `path`, `edits` | Apply targeted edits to a file |
| `search` | `query`, `glob` | Find symbols and matches using ripgrep |
| `web_fetch` | `url` | Retrieve HTML/Markdown from remote pages |
| `apply_patch` | `input` | Apply multi-file diff patches via unified patch block |

### `apply_patch` Tool Spec
Accepts a structured, multi-file patch block containing update, add, delete, and move operations inside `*** Begin Patch` and `*** End Patch`:
```
*** Begin Patch
*** Add File: src/config.json
+{ "enabled": true }
*** Update File: src/app.ts
@@
-const debug = false;
+const debug = true;
*** Move to: src/core/app.ts
*** Delete File: old-file.ts
*** End Patch
```
Configured in `config.yaml` under `tools.exec.applyPatch`:
- `tools.exec.applyPatch.enabled`: Toggle tool access (`true` by default for OpenAI/Codex).
- `tools.exec.applyPatch.workspaceOnly`: Confines patch actions to the active workspace (`true` by default).

## MCP Support

OpenClaw supports the Model Context Protocol (MCP). Integrate external servers by declaring them in the project `config.yaml`:
```yaml
mcpServers:
  sqlite-db:
    command: "npx"
    args: ["-y", "@modelcontextprotocol/server-sqlite", "--db", "app.db"]
```

## Agent / Subagent Configuration

Agents are declared dynamically inside `config.yaml`. Multi-agent coordination is supported via subagent spawning and specialized ACP agent protocols.

```yaml
# config.yaml
agents:
  developer-agent:
    model: claude-3-5-sonnet
    system: "You are an expert fullstack software engineer."
    tools:
      - bash
      - read_file
      - write_file
      - apply_patch
    hooks:
      # Configure active plugin hooks for this agent
      before_tool_call:
        - ./hooks/read-only-guard.ts
```

## Notes

- **Event-Driven vs. Lifecycle Hooks:** Operator-level internal hooks in folders (discovered via `HOOK.md`) are asynchronously non-blocking. Intercepting or blocking tool commands requires compiling in-process plugins via the SDK.
- **Trace Context:** Agent lifecycle hooks receive W3C-compatible `trace` contexts inside events for OpenTelemetry logging correlation.

## Sources (Official)

| Topic | URL |
|-------|-----|
| Hooks Overview | https://docs.openclaw.ai/automation/hooks |
| apply_patch tool | https://docs.openclaw.ai/tools/apply-patch |
| CLI commands | https://docs.openclaw.ai/cli |
| Plugin Hooks | https://docs.openclaw.ai/plugins/hooks |
| Workspace Bootstrap | https://docs.openclaw.ai/gateway |
