# Kiro IDE / CLI

> AI-native IDE and CLI with deep hook support for agent lifecycle control.

**Vendor:** Amazon (Amazon Kiro, built on Amazon Bedrock) | **License:** Proprietary | **Runtime:** Electron (IDE) / Node.js (CLI)

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
- CLI installation: https://kiro.dev/docs/cli/installation/
- AWS documentation overview: https://aws.amazon.com/documentation-overview/kiro/

---

## Installation

### IDE

Download the installer from https://kiro.dev/ and follow OS-specific instructions.

**System requirements:**
- macOS: Intel and Apple silicon
- Windows: 10/11 (64-bit only; ARM not supported)
- Linux: glibc 2.39+ (Ubuntu 24+, Debian 13+, Fedora 40+, Arch, Linux Mint 22+)

### CLI

> **Note:** The Kiro CLI is not available via Homebrew. Use the platform-specific installers below.

**macOS / Linux:**

```sh
curl -fsSL https://cli.kiro.dev/install | bash
```

**Windows (PowerShell / Windows Terminal — not Command Prompt):**

See the [CLI installation docs](https://kiro.dev/docs/cli/installation/) for the dedicated PowerShell install command. Windows 11 is the supported version.

**Linux system requirement:** glibc 2.34+ (standard on most distros since 2021). For older distros, a musl binary is available — check the [CLI installation docs](https://kiro.dev/docs/cli/installation/) for the musl download command. Note: the IDE has a stricter glibc 2.39+ requirement; the CLI glibc floor is lower.

**Launch command:**

```sh
kiro-cli
```

When no subcommand is specified, `kiro-cli` defaults to interactive chat. If the command router integration is installed, `kiro` also works as an alias.

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.kiro/config.yaml` | Global | User settings |
| `.kiro/config.yaml` | Project | Project settings |
| `.kiro/agents/` | Project | Custom agent definitions (JSON) |

## Hooks

Hooks execute custom commands at specific points during agent lifecycle and tool execution.

### CLI Hook Events

| Event | When | Provides | Can Block? |
|-------|------|----------|-----------|
| `agentSpawn` | Agent activated | No tool context | ❌ |
| `userPromptSubmit` | User submits prompt | `prompt` field | ❌ (output added to context) |
| `preToolUse` | Before tool execution | Tool name + input | ✅ (exit 2) |
| `postToolUse` | After tool execution | Tool name + input + `tool_response` | ❌ |
| `stop` | Assistant finishes responding (end of each turn) | `assistant_response` | ✅ (JSON stdout) |

The `stop` hook blocks agent termination by writing JSON to stdout:

```json
{ "decision": "block", "reason": "Tests failed — re-run and fix before stopping." }
```

The `reason` is injected as a new user message, continuing the conversation.

### Hook Input (stdin JSON)

All events receive:

```json
{
  "hook_event_name": "preToolUse",
  "cwd": "/Users/user/project",
  "session_id": "abc123"
}
```

Tool events (`preToolUse`, `postToolUse`) additionally include:

```json
{
  "tool_name": "execute_bash",
  "tool_input": { "command": "rm -rf dist/" }
}
```

`postToolUse` additionally includes:

```json
{
  "tool_response": { "success": true, "output": "..." }
}
```

`userPromptSubmit` additionally includes:

```json
{
  "prompt": "user's input prompt"
}
```

`stop` additionally includes:

```json
{
  "assistant_response": "The text content of the assistant's last response"
}
```

### Exit Code Behavior

| Code | Meaning |
|------|---------|
| `0` | Success, continue |
| `2` | Block tool execution (`preToolUse` only); stderr sent to LLM |
| Other | Hook failed; stderr shown as warning; execution continues |

### Hook Configuration (timeout / caching)

- Default timeout: 30 seconds (30,000 ms). Override with `timeout_ms` field.
- Caching: `cache_ttl_seconds: 0` = no caching (default). Values > 0 cache successful results. `agentSpawn` hooks are never cached.

### Hook Matcher Syntax

Matchers for `preToolUse` / `postToolUse` accept snake_case canonical names, aliases, wildcards, and MCP notation:

| Matcher | Matches |
|---------|---------|
| `"read"` or `"fs_read"` or `"fsRead"` | File read tool |
| `"write"` or `"fs_write"` or `"fsWrite"` | File write tool |
| `"shell"` or `"execute_bash"` or `"execute_cmd"` | Shell execution |
| `"aws"` or `"use_aws"` | AWS CLI tool |
| `"@git"` | All git MCP server tools |
| `"@git/status"` | Specific MCP tool |
| `"@builtin"` | All built-in tools |
| `"*"` | All tools |

### Agent Hook Configuration

Agent config files are **JSON** (not YAML), located in `.kiro/agents/`:

```json
{
  "name": "my-agent",
  "model": "claude-sonnet-4",
  "prompt": "You are a careful assistant...",
  "tools": ["read", "write", "shell"],
  "hooks": {
    "agentSpawn": [
      { "command": "~/.kiro/hooks/setup.sh" }
    ],
    "preToolUse": [
      { "matcher": "execute_bash", "command": "~/.kiro/hooks/validate-bash.sh" }
    ],
    "postToolUse": [
      { "command": "~/.kiro/hooks/audit.sh" }
    ],
    "stop": [
      { "command": "~/.kiro/hooks/run-tests.sh" }
    ]
  }
}
```

### IDE Hook Types

The IDE has 10 hook types (configured in the IDE UI, not in agent JSON):

| Hook Type | Trigger | Can Block? |
|-----------|---------|-----------|
| Prompt Submit | User submits a prompt (`USER_PROMPT` env var available) | ✅ |
| Agent Stop | Agent finishes its turn | ✅ |
| Pre Tool Use | Agent about to invoke a tool (filterable by tool name) | ✅ |
| Post Tool Use | Agent completed tool invocation | ✅ |
| File Create | New files matching pattern created | ✅ |
| File Save | Files matching pattern saved | ✅ |
| File Delete | Files matching pattern deleted | ✅ |
| Pre Task Execution | Spec task status → in_progress | ✅ |
| Post Task Execution | Spec task status → completed | ✅ |
| Manual Trigger | User manually executes on demand | N/A |

IDE hooks support two action types: **Ask Kiro** (agent prompt) or **Run Command** (shell).

## Built-in Tools

Tool names are **snake_case**. `Edit` and `LS` do not exist as built-in tools.

| Tool | Aliases | Description |
|------|---------|-------------|
| `read` | `fs_read`, `fsRead` | Read files, folders, and images |
| `glob` | — | File discovery using glob patterns (respects .gitignore) |
| `grep` | — | Fast content search using regex (respects .gitignore) |
| `write` | `fs_write`, `fsWrite` | Create and edit files |
| `shell` | `execute_bash`, `execute_cmd` | Execute bash commands |
| `aws` | `use_aws` | Make AWS CLI calls |
| `web_search` | — | Search the web |
| `web_fetch` | — | Fetch content from a URL |
| `code` | — | Code intelligence: symbol search and LSP integration |
| `introspect` | — | Self-awareness about Kiro CLI features via official docs |
| `tool_search` | — | Find and load MCP tools on demand |
| `delegate` | — | Delegate tasks to background agents asynchronously |
| `subagent` | `use_subagent` | Delegate to specialized subagents running in parallel |
| `report` | — | Open browser to pre-filled GitHub issue template |
| `session` | — | Temporarily override CLI settings for current session |
| `goal` | — | Manage goal-driven iterative loops with built-in verification |
| `knowledge` | — | Store/retrieve information across sessions *(experimental)* |
| `thinking` | — | Internal reasoning for complex tasks *(experimental)* |
| `todo` | — | Create and manage ToDo lists for multi-step tasks *(experimental)* |

## MCP Support

Kiro CLI supports MCP servers for extending tool capabilities. Configure in `.kiro/config.yaml` under `mcpServers`. MCP tools are referenced in hooks and tool lists with `@server-name` or `@server-name/tool-name` notation.

## Custom Agents

Agent definitions are **JSON files** in `.kiro/agents/`. The system prompt field is `"prompt"` (not `"system"`).

```json
{
  "name": "code-reviewer",
  "model": "claude-opus-4",
  "prompt": "You are a strict code reviewer...",
  "tools": ["read", "grep", "glob"],
  "hooks": {
    "preToolUse": [
      { "matcher": "write", "command": "./hooks/block-writes.sh" }
    ]
  }
}
```

Additional agent config fields: `description`, `toolAliases`, `allowedTools`, `toolsSettings`, `mcpServers`, `resources`, `keyboardShortcut`, `welcomeMessage`, `includeMcpJson`.

The `prompt` field accepts inline text or a `file://` URI (e.g. `"file://./prompts/expert.md"`).

## Notes

- Kiro is built on Amazon Bedrock and is an AWS/Amazon product.
- `userPromptSubmit` hook output is injected into the conversation context, not shown to the user.
- IDE hooks and CLI hooks share the same event model but use different config locations and configuration methods (IDE UI vs. agent JSON).
- CLI `kiro-cli` is distinct from the IDE; the IDE is a GUI application (Electron-based).

## Sources

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| CLI hooks reference | https://kiro.dev/docs/cli/hooks/ | 2026-06-13 | [official] |
| IDE hooks overview | https://kiro.dev/docs/hooks/ | 2026-06-13 | [official] |
| IDE hook types | https://kiro.dev/docs/hooks/types/ | 2026-06-13 | [official] |
| CLI built-in tools | https://kiro.dev/docs/cli/reference/built-in-tools/ | 2026-06-13 | [official] |
| CLI commands reference | https://kiro.dev/docs/cli/reference/cli-commands/ | 2026-06-13 | [official] |
| Agent config reference | https://kiro.dev/docs/cli/custom-agents/configuration-reference/ | 2026-06-13 | [official] |
| CLI installation | https://kiro.dev/docs/cli/installation/ | 2026-06-13 | [official] |
| IDE installation | https://kiro.dev/docs/getting-started/installation/ | 2026-06-13 | [official] |
| AWS documentation overview (vendor confirmation) | https://aws.amazon.com/documentation-overview/kiro/ | 2026-06-13 | [official] |
| Arm install guide (curl install command) | https://learn.arm.com/install-guides/kiro-cli/ | 2026-06-13 | [official] |
