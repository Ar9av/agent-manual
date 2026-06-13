# OpenClaw

> Self-hosted multi-channel gateway connecting messaging apps (Discord, Slack, WhatsApp, Telegram, and more) to AI coding agents, with extensible hooks and robust multi-agent tool sandboxing.

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
npm install -g openclaw@latest
```

### Via Binary Installer
```sh
curl -fsSL https://openclaw.ai/install | bash
```

Requires Node 24 (recommended) or Node 22 LTS (22.19+) and an API key from your chosen provider. After install:
```sh
openclaw onboard --install-daemon
openclaw dashboard
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.openclaw/openclaw.json` | Global | User-level settings (JSON5 format — comments and trailing commas allowed) |
| `~/.openclaw/hooks/` | Global | Managed hook definitions shared across workspaces |
| `<workspace>/hooks/` | Project | Workspace-level hook overrides |

> **Config format is JSON5**, not YAML. The main config file is `~/.openclaw/openclaw.json`. Configuration can be split across multiple files using the `$include` directive.

Additional files:
- `auth-profiles.json` — Per-agent authentication credentials
- `sessions.json` — Session state and cron history
- `.env` / `~/.openclaw/.env` — Environment variables
- `secrets.json` — Optional secrets storage (when file-based secret provider is configured)

## Instruction Files

OpenClaw loads natural-language system instructions and steering prompts from workspace files. Confirmed bootstrap files (seeded during `openclaw onboard`):

| File | Purpose |
|------|---------|
| `AGENTS.md` | Agent tool instructions and reference |
| `SOUL.md` | Agent persona, tone, and core behavioral boundaries |
| `IDENTITY.md` | Agent name, ID, role label, and routing metadata |
| `USER.md` | User preferences and context collected during bootstrap |
| `TOOLS.md` | Tool usage instructions injected at bootstrap ❓ (confirmed in GitHub repo, not official start/bootstrapping page) |
| `BOOTSTRAP.md` | First-run ritual orchestrator; removed after bootstrap completes |

> **BOOT.md is not confirmed in live docs.** It appears in a bundled hook example (`boot-md`) but is not a standard instruction file. Removed from the primary list.

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

> **Note:** `openclaw hooks install` is deprecated. Use `openclaw plugins install <package>` instead. Plugin-managed internal hooks appear in `openclaw hooks list` with a `plugin:<id>` prefix and must be enabled/disabled at the plugin level.

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
| `command:new` | `/new` command issued | `sessionEntry`, `previousSessionEntry`, `commandSource`, `workspaceDir`, `cfg` |
| `command:reset` | `/reset` command issued | `sessionEntry`, `previousSessionEntry`, `commandSource`, `workspaceDir`, `cfg` |
| `command:stop` | `/stop` command issued; user cancellation, not agent finalization | Same as above |
| `command` | Any command event | Universal command context |
| `session:compact:before` | Before contexts compact | `messageCount`, `tokenCount` |
| `session:compact:after` | After compaction completes | `compactedCount`, `summaryLength`, `tokensBefore`, `tokensAfter` |
| `session:patch` | Session fields patched | Privileged patches, `sessionEntry`, `patch`, `cfg` |
| `agent:bootstrap` | Before agent workspace bootstrap files are injected | Mutable `bootstrapFiles` array, `agentId` |
| `gateway:startup` | After channels start and hooks are loaded | Startup environment state |
| `gateway:shutdown` | Gateway process begins exit | `reason`, `restartExpectedMs` (5s default wait budget) |
| `gateway:pre-restart` | Before scheduled gateway restart | `reason`, `restartExpectedMs` (10s default wait budget) |
| `message:received` | Inbound chat message arrives | `from`, `content`, `channelId`, `metadata` (`senderId`, `senderName`, `guildId`) |
| `message:sent` | Outbound message delivered | `to`, `content`, `success`, `channelId` |
| `message:transcribed` | Audio file transcribes | `transcript`, `from`, `channelId`, `mediaPath` |
| `message:preprocessed` | Inbound media/link processed | Final enriched `bodyForAgent`, `from`, `channelId` |

> **Replyable surfaces:** Only `command:*` and `message:received` support pushed replies via `event.messages`. Lifecycle-only events (`agent:bootstrap`, `session:*`, `gateway:*`, `message:sent`) ignore pushed messages.

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

Handlers run sequentially in descending priority order; same-priority hooks maintain registration order.

#### Agent Turn Hooks

| SDK Hook | When it fires | Can Block? | Notes |
|----------|---------------|------------|-------|
| `before_model_resolve` | Pre-session, before provider resolution AND before session messages load | ❌ No | Override provider/model; receives only current prompt and attachment metadata. Requires `allowConversationAccess: true` in non-bundled plugins. |
| `agent_turn_prepare` | After queued injections are consumed, before prompt hooks | ❌ No | Add same-turn context before prompt build |
| `before_prompt_build` | Post-session load (messages available), before model call | ❌ No | Inject dynamic `prependContext` or system prompts |
| `before_agent_start` | Compatibility alias | ❌ No | Deprecated; prefer `before_model_resolve` and `before_prompt_build`. Requires `allowConversationAccess: true` in non-bundled plugins. |
| `before_agent_run` | After prompt is finalized, before model submission | ✅ Yes | Inspect final prompt/messages; can block before submission |
| `before_agent_reply` | Before the LLM generation call | ✅ Yes | Can return synthetic reply or silence turn entirely |
| `before_agent_finalize` | Agent finishes reasoning turn | ✅ Yes | Can inspect messages and request extra pass |
| `agent_end` | Final turn completed | ❌ No | Observation-only; inspect final message lists and metadata |
| `heartbeat_prompt_contribution` | Heartbeat cycles only | ❌ No | Background monitors and lifecycle plugins |

#### Observation Hooks

| SDK Hook | When it fires | Can Block? | Notes |
|----------|---------------|------------|-------|
| `model_call_started` | Before each provider model call | ❌ No | Sanitized provider metadata and timing |
| `model_call_ended` | After each provider model call completes | ❌ No | Completion with bounded request-id hashes |
| `llm_input` | Observe provider input | ❌ No | System prompt, prompt, history (requires `allowConversationAccess`) |
| `llm_output` | Observe provider output | ❌ No | Output, usage, resolved context token budget (requires `allowConversationAccess`) |

#### Tool Hooks

| SDK Hook | When it fires | Can Block? | Notes |
|----------|---------------|------------|-------|
| `before_tool_call` | Before any agent tool executes | ✅ Yes | Rewrite arguments, block call (`{ block: true }`), or require approval |
| `after_tool_call` | After tool completes execution | ❌ No | Observe tool results, errors, execution duration |
| `resolve_exec_env` | Before exec tool invocations | ❌ No | Contribute environment variables |
| `tool_result_persist` | Before tool result is written to transcript | ✅ Yes | Rewrite the assistant message produced from a tool result |
| `before_message_write` | During in-progress message write | ✅ Yes | Inspect or block (rare usage) |

#### Message Hooks

| SDK Hook | When it fires | Can Block? | Notes |
|----------|---------------|------------|-------|
| `inbound_claim` | Before inbound message is routed to agent | ✅ Yes | Claim a message before agent routing |
| `message_received` | Inbound message received | ❌ No | Telemetry for incoming channels |
| `message_sending` | Outbound message about to be sent | ✅ Yes | Rewrite outbound content or cancel delivery (`{ cancel: true }`) |
| `reply_payload_sending` | Normalized reply payload before delivery | ✅ Yes | Mutate or cancel reply payloads |
| `message_sent` | Outbound message delivered | ❌ No | Observation-only; final delivery success or failure |
| `before_dispatch` | Before channel handoff | ✅ Yes | Inspect or rewrite outbound dispatch |
| `reply_dispatch` | Final reply-dispatch pipeline | ❌ No | Observation/participation hook; non-blocking |

#### Session Hooks

| SDK Hook | When it fires | Can Block? | Notes |
|----------|---------------|------------|-------|
| `session_start` | Session initialized | ❌ No | Track session lifecycle onset; `reason` field shows trigger type |
| `session_end` | Session finalized or gateway stopped | ❌ No | Track session lifecycle boundaries; `reason` includes idle, compaction, reset |
| `before_compaction` | Before context compaction begins | ❌ No | Observe compaction trigger and token counts |
| `after_compaction` | After compaction completes | ❌ No | Inspect compacted summaries |
| `before_reset` | Session reset via `/reset` or programmatic call | ❌ No | Observe reset events |

#### Subagent Hooks

| SDK Hook | When it fires | Can Block? | Notes |
|----------|---------------|------------|-------|
| `subagent_spawned` | Subagent launches | ❌ No | Observe subagent launch and completion |
| `subagent_ended` | Subagent completes | ❌ No | Resolved model/provider data |
| `subagent_spawning` | Subagent spawn event | ❌ No | Deprecated compatibility hook; non-blocking |
| `subagent_delivery_target` | Subagent delivery routing | ❌ No | Compatibility hook for completion delivery routing |

#### Lifecycle Hooks

| SDK Hook | When it fires | Can Block? | Notes |
|----------|---------------|------------|-------|
| `gateway_start` | Gateway process starts | ❌ No | Global startup initialization; start plugin-owned services |
| `gateway_stop` | Gateway process begins shutdown | ❌ No | Global cleanup sequence; clean up long-running resources |
| `deactivate` | Gateway shutdown (deprecated alias) | ❌ No | Deprecated alias for `gateway_stop`; non-blocking |
| `cron_changed` | Gateway-owned cron lifecycle events | ❌ No | added, updated, removed, started |
| `before_install` | Before skill or plugin installation | ✅ Yes | Prevent untrusted execution (`{ block: true }`) |

#### Hook Decision Rules
- **Tool Guard (`before_tool_call`):** Returning `{ block: true }` immediately stops execution of lower-priority handlers and blocks the tool. Can also return `requireApproval` with title, description, severity, timeout behavior, and `onResolution` callback.
- **Install Guard (`before_install`):** Returning `{ block: true }` immediately cancels the installation.
- **Message Outbound (`message_sending`):** Returning `{ cancel: true }` suppresses and cancels message delivery.
- **`tool_result_persist`:** Tool result details are runtime metadata, not prompt content. Oversized details are replaced with a summary and `persistedDetailsTruncated: true` flag.
- **`allowConversationAccess`:** Required for `llm_input`, `llm_output`, `before_model_resolve`, `before_agent_reply`, `before_agent_run`, `before_agent_finalize`, and `agent_end` to read raw conversation content in non-bundled plugins.

## Agent Loop & Concurrency Lifecycle

An agentic loop in OpenClaw represents a serialized, single-lane execution pipeline per session (intake → context assembly → model inference → tool execution → streaming replies → persistence).

### Concurrency and Locking
- **Session Lanes:** Execution runs are serialized per session key to avoid race conditions.
- **Session File Lock:** Workspace transcript writes are guarded by a process-aware, file-based session write lock. Transcript writers wait up to `session.writeLock.acquireTimeoutMs` (default: `60000` ms) before reporting busy.
- **Reentrancy:** Session locks are non-reentrant by default. Nested lock acquisitions must explicitly pass `allowReentrant: true`.

### Timeouts & Diagnostics
- **RPC Wait Timeout:** `agent.wait` default wait timeout is `30` seconds (overridable via `timeoutMs`).
- **Agent Runtime Timeout:** Runtime timeout defaults to `172800` seconds (48 hours) via `agents.defaults.timeoutSeconds`.
- **Model Idle Timeout:** OpenClaw watchdogs model requests and aborts if no chunks arrive; bounded by `models.providers.<id>.timeoutSeconds`.
- **Stuck/Stalled Sessions:** The diagnostics engine monitors processing lanes. Session stalls are reported as `session.stalled` after `diagnostics.stuckSessionWarnMs`. Active stale runs are abort-drained and released only after `diagnostics.stuckSessionAbortMs` (default: 5 minutes/3x warning threshold) to prevent stuck lanes.

### Event Streams
Three event streams bridge the runtime to clients:
- `lifecycle` — phase: start/end/error
- `assistant` — streamed deltas
- `tool` — tool execution events

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

### Tool Profiles

| Profile | Scope |
|---------|-------|
| `minimal` | `session_status` only |
| `coding` | File system, runtime, web, sessions, memory, cron, media (default for local configs) |
| `messaging` | Messaging and session management |
| `full` | No restrictions |

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
Configured in `openclaw.json` under `tools.exec.applyPatch`:
- `tools.exec.applyPatch.enabled`: Toggle tool access (`true` by default for OpenAI/Codex models).
- `tools.exec.applyPatch.workspaceOnly`: Confines patch actions to the active workspace (`true` by default).
- `tools.exec.applyPatch.allowModels`: Optional array to gate tool access by model.

## MCP Support

OpenClaw supports the Model Context Protocol (MCP). Integrate external servers by declaring them in `openclaw.json` (JSON5 format). MCP servers are exposed as plugin-owned tools under the `bundle-mcp` provider ID:
```json5
{
  mcp: {
    servers: {
      "sqlite-db": {
        command: "npx",
        args: ["-y", "@modelcontextprotocol/server-sqlite", "--db", "app.db"]
      }
    }
  },
  tools: {
    sandbox: {
      tools: { alsoAllow: ["bundle-mcp"] }
    }
  }
}
```

## Agent / Subagent Configuration

Agents and custom providers are declared in `openclaw.json` (JSON5). Multi-agent coordination is supported via subagent spawning and specialized ACP agent protocols.

```json5
{
  agents: {
    "developer-agent": {
      model: "claude-3-5-sonnet",
      system: "You are an expert fullstack software engineer.",
      tools: ["bash", "read_file", "write_file", "apply_patch"],
      hooks: {
        // Configure active plugin hooks for this agent
        before_tool_call: ["./hooks/read-only-guard.ts"]
      }
    }
  }
}
```

## Notes

- **Event-Driven vs. Lifecycle Hooks:** Operator-level internal hooks in folders (discovered via `HOOK.md`) are asynchronously non-blocking. Intercepting or blocking tool commands requires compiling in-process plugins via the SDK.
- **Trace Context:** Agent lifecycle hooks receive W3C-compatible `trace` contexts inside events for OpenTelemetry logging correlation.
- **Plugin Hook Access Control:** `plugins.entries.<id>.hooks.allowPromptInjection` controls whether core blocks `before_prompt_build` prompt-mutating fields from legacy `before_agent_start`. `plugins.entries.<id>.hooks.allowConversationAccess` gates raw conversation access in typed hooks.
- **Supported Channels:** Discord, Slack, WhatsApp, Telegram, Google Chat, iMessage, Signal, Microsoft Teams, Matrix, IRC, Feishu, LINE, Mattermost, Nextcloud Talk, Nostr, Synology Chat, Tlon, Twitch, Zalo, Zalo Personal, WeChat, QQ, WebChat.

## Sources

| Topic | URL | Label |
|-------|-----|-------|
| Hooks Overview | https://docs.openclaw.ai/automation/hooks | [official] |
| Plugin Hooks Reference | https://docs.openclaw.ai/plugins/hooks | [official] |
| apply_patch tool | https://docs.openclaw.ai/tools/apply-patch | [official] |
| CLI hooks commands | https://docs.openclaw.ai/cli/hooks | [official] |
| Configuration Reference | https://docs.openclaw.ai/gateway/configuration-reference | [official] |
| Tools & Custom Providers Config | https://docs.openclaw.ai/gateway/config-tools | [official] |
| Agent Loop Concepts | https://docs.openclaw.ai/concepts/agent-loop | [official] |
| Agent Bootstrapping | https://docs.openclaw.ai/start/bootstrapping | [official] |
| GitHub Repository | https://github.com/openclaw/openclaw | [github] |
