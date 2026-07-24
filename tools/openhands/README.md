# OpenHands (formerly OpenDevin)

> Community-driven, open-source platform for AI-driven software development — CLI, local GUI, self-hostable enterprise/cloud offerings, and a composable Python "Software Agent SDK."

**Vendor:** All Hands AI (OSS project, org `OpenHands` on GitHub) | **License:** MIT (core repo; the `enterprise/` directory ships under a separate license) | **Runtime:** Python (CLI installable via `uv`/binary installer; agent execution happens in a sandboxed workspace, typically a Docker container)

> **Naming history:** The project started in March 2024 as **OpenDevin**, an homage to Cognition's Devin, created by Xingyao Wang, Graham Neubig, and collaborators including members of the Qwen team. It was rebranded to **OpenHands** in mid/late 2024 when the maintainers formed the company All Hands AI. The canonical GitHub org moved to `github.com/OpenHands/OpenHands` (previously `All-Hands-AI/OpenHands`).

## Links

- Docs (root): https://docs.openhands.dev/
- CLI installation: https://docs.openhands.dev/openhands/usage/cli/installation
- Headless mode: https://docs.openhands.dev/openhands/usage/cli/headless
- MCP servers (CLI): https://docs.openhands.dev/openhands/usage/cli/mcp-servers
- Hooks: https://docs.openhands.dev/openhands/usage/customization/hooks
- Skills / microagents overview: https://docs.openhands.dev/overview/skills
- Software Agent SDK: https://docs.openhands.dev/sdk
- SDK skill format: https://docs.openhands.dev/sdk/guides/skill
- SDK MCP guide: https://docs.openhands.dev/sdk/guides/mcp
- GitHub (main app): https://github.com/OpenHands/OpenHands
- GitHub (lightweight CLI binary): https://github.com/OpenHands/OpenHands-CLI
- GitHub (Software Agent SDK): https://github.com/OpenHands/software-agent-sdk
- GitHub (community skills registry): https://github.com/OpenHands/skills

---

## Installation

**CLI via `uv` (recommended):**
```sh
uv tool install openhands --python 3.12
openhands
```
Upgrade with `uv tool upgrade openhands --python 3.12`. Requires Python 3.12+ and `uv`.

**CLI via install script (binary executable):**
```sh
curl -fsSL https://install.openhands.dev/install.sh | sh
openhands
```
On macOS, the app may need to be allowed under System Settings → Privacy & Security → Security on first run.

**Docker (agent server / sandbox):**
```sh
# Configure ~/.openhands/agent_settings.json with LLM settings first, then:
docker run -it --rm \
  -e SANDBOX_VOLUMES=$PWD:/workspace:rw \
  -e SANDBOX_USER_ID=$(id -u) \
  -v ~/.openhands:/root/.openhands \
  docker.all-hands.dev/all-hands-ai/openhands:0.x \
  # (agent-server image tag referenced in docs as e.g. 1.26.0-python)
```
❓ Exact Docker image/tag naming conventions vary by release; consult the installation page for the current tag before copy-pasting.

**Headless mode (CI / scripting):**
```sh
openhands --headless -t "Your task here"
openhands --headless -f task.txt          # load task from a file
openhands --headless --json -t "Add unit tests" > output.jsonl   # JSONL event stream
```
Headless mode always runs in "always-approve" mode — the agent executes all actions without confirmation prompts; `--llm-approve` is not available in this mode. Headless mode also **requires existing saved settings** — running it before any interactive first-run configuration exits immediately with "Headless mode requires existing settings." To bypass interactive setup entirely (e.g. in CI), pass `--override-with-envs` with `LLM_MODEL`, `LLM_API_KEY`, and optionally `LLM_BASE_URL` set as environment variables:
```sh
LLM_MODEL=openai/gpt-4o-mini LLM_API_KEY=sk-... openhands --headless --override-with-envs -t "Your task here"
```
Verified live on 2026-07-23: this successfully runs a full headless session against the OpenAI API without any prior `openhands` interactive setup.

On first run, the CLI walks you through LLM configuration, saved to `~/.openhands/agent_settings.json`.

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.openhands/agent_settings.json` | Global | LLM provider/model settings, saved on first run (confirmed via `openhands_cli/locations.py`'s `AGENT_SETTINGS_PATH`; earlier drafts of this page incorrectly called it `settings.json`) |
| `~/.openhands/conversations/` | Global | Saved conversation history |
| `~/.openhands/mcp.json` | Global | MCP server registrations (JSON; TOML format used pre-1.0.0 is no longer supported) |
| `.openhands/hooks.json` | Project | Lifecycle hook definitions |
| `.openhands/hooks/` | Project | Hook shell scripts referenced by `hooks.json` |
| `.agents/skills/<name>/SKILL.md` | Project | Current/recommended location for repo skills (microagents) |
| `~/.agents/skills/<name>/SKILL.md` | Global (user) | User-level skills, loaded across all repos |
| `.openhands/skills/` | Project | Deprecated skills path, still read |
| `.openhands/microagents/` | Project | Deprecated (legacy "microagents") path, still read |
| `AGENTS.md` (repo root) | Project | Always-on, permanent repository context injected into the system prompt |
| `CLAUDE.md`, `GEMINI.md` (repo root) | Project | Optional model-specific variants of the always-on instructions file |

Skill-directory loading precedence (first match wins): `.agents/skills/` → `.openhands/skills/` → `.openhands/microagents/`.

## Instruction File

The agent reads repository-wide, always-on instructions from a root-level **`AGENTS.md`**. Model-specific variants (`CLAUDE.md`, `GEMINI.md`) are also recognized for tailoring instructions per underlying LLM. This is distinct from (and simpler than) the trigger-based skill/microagent system below.

## Microagents / Skills (extensibility)

OpenHands' primary customization/extensibility mechanism has historically been called **microagents**; current docs have largely renamed the concept to **skills** (the file format tracks the general "SKILL.md" convention seen elsewhere in the agent ecosystem), while `.openhands/microagents/` remains a supported, deprecated path. There is no plugin/extension API beyond skills, hooks, and MCP.

### Types / trigger mechanisms

| Trigger type | Frontmatter field | Behavior |
|---|---|---|
| Always-on | none (no frontmatter required) | Content is injected into the system prompt at the start of every conversation (same role as `AGENTS.md`) |
| Keyword-triggered | `triggers: [kw1, kw2, ...]` | Skill is loaded into context only when a keyword appears in the user's (or, per some docs, the agent's) message |
| Path-triggered | `paths: [glob1, glob2, ...]` | Skill is deterministically injected when the agent reads/edits/creates a file matching the glob |
| Manual | ❓ (referenced in community sources as a "manual" trigger for user-invoked-only microagents) | Only loaded when explicitly invoked by the user |

A skill file can declare `paths:` or `triggers:`, but not both meaningfully combined — if both are present, `paths:` wins.

### SKILL.md frontmatter schema

```yaml
---
name: string                    # required: skill identifier (lowercase + hyphens)
description: string             # required: shown to the agent, explains what the skill does
license: string                 # optional
compatibility: string           # optional: environment/requirements note
metadata:                       # optional: arbitrary key-value pairs
  author: string
  version: string
triggers:                       # optional: keyword-triggered activation
  - string
paths:                          # optional: glob patterns for path-triggered activation
  - string
---
```

### Example (keyword-triggered)

```markdown
---
name: encryption-helper
description: >
  Provides ROT13 encryption and decryption utilities using the encrypt.sh script.
license: MIT
triggers:
  - encrypt
  - decrypt
  - cipher
---

# Encryption Helper

Use `./scripts/encrypt.sh "message"` to encrypt text via ROT13.
```

### Example (path-triggered)

```markdown
---
name: api-validation
description: Enforces input validation on all API endpoint handlers using zod schema.
paths:
  - "src/api/**/*.ts"
  - "**/*.route.ts"
---

# API Validation Rule
**REQUIREMENT**: Validate all request inputs with zod before processing.
```

Skills can also ship supporting `scripts/`, `references/`, and `assets/` subdirectories alongside `SKILL.md`. A public community registry of shareable skills is maintained at https://github.com/OpenHands/skills, and organizations can define org-wide skills for OpenHands Cloud/Enterprise.

## Hooks

OpenHands has a dedicated, documented hooks system, configured per-repository via `.openhands/hooks.json` plus a `.openhands/hooks/` directory of scripts.

### Supported Events

| Event | When | Can Block? |
|-------|------|-----------|
| `PreToolUse` (`pre_tool_use`) | Before the agent executes a tool | ✅ (exit code 2) |
| `PostToolUse` (`post_tool_use`) | After a tool finishes executing | ❌ |
| `UserPromptSubmit` | Before a user message is processed | ✅ |
| `Stop` | When the agent attempts to finish/stop | ✅ (e.g., to enforce quality gates like passing lint) |
| `SessionStart` | At the start of a conversation | ❌ |
| `SessionEnd` | At the end of a conversation | ❌ |

Hooks marked `"async": true` run in the background and can never block, regardless of event type.

### Config Format (`.openhands/hooks.json`)

```json
{
  "pre_tool_use": [
    {
      "matcher": "terminal|*|/regex/",
      "hooks": [
        {
          "type": "command",
          "command": ".openhands/hooks/validate.sh",
          "timeout": 60,
          "async": false
        }
      ]
    }
  ]
}
```

Matcher patterns: `*` (all tools), an exact tool name (e.g. `"terminal"`), or a `/regex/`.

> **Live-verified 2026-07-23:** Confirmed end-to-end on Ubuntu 24.04 with `openhands` v1.21.0 (installed via `uv tool install openhands`). A `PreToolUse` hook in `.openhands/hooks.json` matching `"terminal"` correctly intercepted a shell command and blocked it via `exit 2` — the CLI printed "✓ Hooks loaded" at startup and the agent reported "unable to run the command due to restrictions in place." One useful confirmation for anyone building a hook: the real `tool_name` sent on stdin for the shell tool is exactly `"terminal"` (matches this page's example) — a matcher of `"execute_bash"` or `"bash"` (names used by *other* agents in this catalog) will silently never match.

### Hook Environment Variables

`OPENHANDS_EVENT_TYPE`, `OPENHANDS_TOOL_NAME`, `OPENHANDS_PROJECT_DIR`, `OPENHANDS_SESSION_ID`

### Hook stdin Schema

```json
{
  "event_type": "PreToolUse",
  "tool_name": "terminal",
  "tool_input": { "command": "example" },
  "session_id": "abc-123",
  "working_dir": "/workspace"
}
```

### Hook stdout Schema (optional)

```json
{
  "decision": "deny",
  "reason": "Human-readable explanation",
  "additionalContext": "Context injected into the agent's prompt"
}
```

### Exit Code Behavior

| Code | Meaning |
|------|---------|
| `0` | Allow the operation to proceed |
| `2` | Block the operation |
| Other | Error is logged but execution proceeds |

### Use Cases (per docs)

Blocking dangerous commands before execution, enforcing quality gates before the agent is allowed to stop (e.g., linting must pass), auditing/logging tool usage, and injecting extra context (like `git status`) into prompts.

## Built-in Tools

The Software Agent SDK ships ready-to-use tools that the CLI/GUI/Cloud products build on:

| Tool | Import path | Description |
|------|-------------|-------------|
| `TerminalTool` | `openhands.tools.terminal` | Execute bash/shell commands |
| `FileEditorTool` | `openhands.tools.file_editor` | Read, write, and edit files in the workspace |
| `TaskTrackerTool` | `openhands.tools.task_tracker` | Track and manage sub-task progress during a session |
| `BrowserToolSet` | `openhands.tools.browser_use` | Automated browser control — navigate, click, fill forms, extract content |
| MCP tools | via `mcp_config` on `Agent` | Any tool exposed by a configured MCP server is auto-discovered and added to the toolset |

❓ Additional built-in tools (e.g., grep/glob-style search, planning) are referenced generically ("and more") in SDK docs but not individually itemized on the pages fetched; consult `docs.openhands.dev/llms.txt` for the complete, current list.

## MCP Support

### CLI (headless/interactive terminal app)

Config file: `~/.openhands/mcp.json` (JSON; the pre-1.0.0 TOML format is no longer used — configs must be redone after upgrading past 1.0.0).

```json
{
  "mcpServers": {
    "local-tools": {
      "command": "python",
      "args": ["-m", "my_mcp_tools"],
      "env": { "DEBUG": "true" }
    },
    "tavily-remote": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.tavily.com/mcp/?tavilyApiKey=your-api-key"]
    }
  }
}
```

CLI management commands:

```sh
openhands mcp add local-server --transport stdio --env "API_KEY=secret123" python -- -m my_mcp_server
openhands mcp add my-api --transport http --header "Authorization: Bearer token" https://api.example.com/mcp
openhands mcp list
openhands mcp get <name>
openhands mcp enable <name>
openhands mcp disable <name>
openhands mcp remove <name>
```

Supported `--transport` values: `stdio`, `http`, `sse`. Use `/mcp` inside a conversation to view active servers.

### Software Agent SDK (Python)

Follows the FastMCP configuration standard, passed directly to an `Agent`:

```python
mcp_config = {
    "mcpServers": {
        "server_name": {
            "command": "uvx",
            "args": ["mcp-server-name"]
        }
    }
}
# OAuth-enabled remote servers:
# "server_name": {"url": "https://mcp.notion.com/mcp", "auth": "oauth"}

agent = Agent(..., mcp_config=mcp_config, filter_tools_regex="...")  # filter_tools_regex is optional
```

## Skills / Commands

- Skills location: `.agents/skills/<name>/SKILL.md` (project), `~/.agents/skills/<name>/SKILL.md` (user/global); deprecated: `.openhands/skills/`, `.openhands/microagents/`
- Format: Markdown with YAML frontmatter (`name`, `description` required; `triggers`, `paths`, `license`, `compatibility`, `metadata` optional) — see schema and examples above
- Community registry: https://github.com/OpenHands/skills

## Agent / Subagent Configuration

The Software Agent SDK is explicitly "a composable Python library" for building custom agents — tool sets, LLM config, and MCP servers are assembled programmatically via `Agent(...)` rather than through a declarative subagent-profile file format like some competitors. ❓ No standalone "subagent" file convention comparable to Devin's `AGENT.md` or Claude Code's subagents was found in the fetched docs.

> **Live-verified 2026-07-23 — real, working delegation confirmed.** The Python package ships dedicated `openhands/tools/delegate/` and `openhands/tools/task/` modules; the delegate tool accepts a `command` of `"spawn"` or `"delegate"`. Asked the CLI to "Delegate to a subagent/sub-task to create subagent-test.txt..." and it completed the task — inspecting the on-disk conversation state afterward showed a genuine `~/.openhands/conversations/<id>/subagents/<subagent-id>/` directory with its **own independent event log**, distinct from the parent conversation's events. This is a real isolated sub-conversation, not just a differently-worded tool call — comparable in substance to Claude Code's `Task` tool or Goose's `delegate` tool, and notably more real than the "subagent" scaffolding found (but confirmed non-functional) in Crush and Continue CLI during this same testing pass.

## Notes

- OpenHands ships four main products/surfaces per its docs homepage: **Agent Canvas** (browser-based UI/backend for running agents), **OpenHands Cloud** (managed service with GitHub/GitLab/Bitbucket integration), **OpenHands Enterprise** (self-hosted, org-scale), and the **Software Agent SDK** (the underlying Python library). The CLI and a "Local GUI" are referenced as legacy/lightweight entry points.
- A `SecurityAnalyzer` component subscribes to the agent's event stream and annotates actions with a `security_risk` rating asynchronously, without blocking the main pipeline (documented via GitHub issue discussion, not a dedicated docs page — treat as ❓ for exact user-facing configuration).
- The core repo is MIT-licensed; code under `enterprise/` uses a separate license defined in `enterprise/LICENSE`.

## Sources

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| Docs homepage / product overview | https://docs.openhands.dev/ | 2026-07-23 | [official] |
| CLI installation | https://docs.openhands.dev/openhands/usage/cli/installation | 2026-07-23 | [official] |
| Headless mode | https://docs.openhands.dev/openhands/usage/cli/headless | 2026-07-23 | [official] |
| MCP servers (CLI) | https://docs.openhands.dev/openhands/usage/cli/mcp-servers | 2026-07-23 | [official] |
| Hooks | https://docs.openhands.dev/openhands/usage/customization/hooks | 2026-07-23 | [official] |
| Skills / microagents overview | https://docs.openhands.dev/overview/skills | 2026-07-23 | [official] |
| SDK skill format (SKILL.md schema) | https://docs.openhands.dev/sdk/guides/skill | 2026-07-23 | [official] |
| SDK MCP guide | https://docs.openhands.dev/sdk/guides/mcp | 2026-07-23 | [official] |
| SDK getting started (built-in tools) | https://docs.openhands.dev/sdk/getting-started | 2026-07-23 | [official] |
| SDK browser-use guide | https://docs.openhands.dev/sdk/guides/agent-browser-use | 2026-07-23 | [official] |
| GitHub repo (main app) | https://github.com/OpenHands/OpenHands | 2026-07-23 | [official] |
| GitHub LICENSE file | https://raw.githubusercontent.com/OpenHands/OpenHands/main/LICENSE | 2026-07-23 | [official] |
| GitHub CLI repo README | https://github.com/OpenHands/OpenHands-CLI | 2026-07-23 | [official] |
| Community skills registry | https://github.com/OpenHands/skills | 2026-07-23 | [official] |
| `openhands_cli/locations.py` source (real settings filename `agent_settings.json`) | pip package `openhands` v1.21.0, installed via `uv tool install openhands` | 2026-07-23 | [live-verified] |
| Live headless smoke test on a real box (`ubuntu 24.04`, `uv tool install openhands`, `--override-with-envs` + OpenAI key) | n/a — first-hand verification, not a doc URL | 2026-07-23 | [live-verified] |
| SecurityAnalyzer discussion (GitHub issue) | https://github.com/OpenHands/OpenHands/issues/10525 | 2026-07-23 | [community] |
| OpenHands blog: "One Year of OpenHands" (rename/history context) | https://www.openhands.dev/blog/one-year-of-openhands-a-journey-of-open-source-ai-development | 2026-07-23 | [official] |
