# Agent → Tools, Hooks & Config Reference

Comprehensive per-agent reference. All data sourced from official documentation or confirmed primary sources. Entries without a source label are inferred or from community docs — see each tool's README for per-topic source table.

> **Source key used inline:** `[official]` = confirmed from vendor docs | `[github]` = confirmed from vendor repo | `[installer-src]` = confirmed from the installer scripts or directory structures

---

## Claude Code

**Vendor:** Anthropic | **Config format:** JSON | **Instruction file:** `CLAUDE.md`
**Source:** https://code.claude.com/docs/en/hooks [official]

### Config Files
| File | Scope |
|------|-------|
| `~/.claude/settings.json` | Global |
| `.claude/settings.json` | Project |
| `.claude/settings.local.json` | Project personal (git-ignored) |
| `CLAUDE.md` | Natural-language instructions (loaded recursively from repo root) |

### Built-in Tools
| Tool | Description |
|------|-------------|
| `Bash` | Execute shell commands |
| `Read` | Read files (text, images, PDF, Jupyter) |
| `Write` | Write/overwrite files |
| `Edit` | Targeted string replacement |
| `MultiEdit` | Multiple edits in one call |
| `Glob` | File pattern matching |
| `Grep` | Regex search in files |
| `LS` | List directory |
| `WebFetch` | Fetch a URL |
| `WebSearch` | Web search |
| `NotebookRead` | Read Jupyter notebooks |
| `NotebookEdit` | Edit Jupyter notebooks |
| `TodoRead` / `TodoWrite` | In-session task tracking |
| `AskUserQuestion` | Collect user input via structured questions |
| `Agent` | Spawn subagents |
| `Skill` | Invoke a skill |

### Hook Events
| Event | When | Can Block | Notes |
|-------|------|-----------|-------|
| `SessionStart` | Session begins or resumes | ❌ | |
| `Setup` | `--init-only` / maintenance flag | ❌ | |
| `UserPromptSubmit` | User submits a prompt, before processing | ✅ exit 2 erases prompt | 30 s timeout override |
| `UserPromptExpansion` | User-typed slash command expands into prompt | ✅ | 30 s timeout override |
| `PreToolUse` | Before every tool call | ✅ | |
| `PermissionRequest` | Permission dialog triggered | ✅ exit 2 denies | |
| `PermissionDenied` | Tool denied by auto-mode classifier | ❌ | Use JSON `retry: true` instead |
| `PostToolUse` | After tool call succeeds | ❌ | Tool already ran |
| `PostToolUseFailure` | After tool call fails | ❌ | |
| `PostToolBatch` | After a parallel tool-call batch resolves | ✅ stops loop | |
| `Notification` | Agent sends a notification | ❌ | |
| `MessageDisplay` | While assistant message text renders | ❌ | 10 s timeout override |
| `SubagentStart` | Subagent is spawned | ❌ | |
| `SubagentStop` | Subagent finishes | ✅ | |
| `TaskCreated` | Task created via `TaskCreate` | ✅ rolls back | |
| `TaskCompleted` | Task marked completed | ✅ | |
| `Stop` | Claude finishes a turn | ✅ continues conversation | |
| `StopFailure` | Turn ends due to API error | ❌ | Output/exit code ignored |
| `TeammateIdle` | Agent-team teammate about to go idle | ✅ | |
| `InstructionsLoaded` | `CLAUDE.md` or `.claude/rules/*.md` loaded | ❌ | Exit code ignored |
| `ConfigChange` | Config file changes during session | ✅ | Except `policy_settings` |
| `CwdChanged` | Working directory changes | ❌ | |
| `FileChanged` | Watched file changes on disk | ❌ | |
| `WorktreeCreate` | Worktree created | ✅ any non-zero fails creation | |
| `WorktreeRemove` | Worktree removed | ❌ | |
| `PreCompact` | Before context compaction | ✅ | |
| `PostCompact` | After context compaction completes | ❌ | |
| `Elicitation` | MCP server requests user input | ✅ exit 2 denies | |
| `ElicitationResult` | User responds to MCP elicitation | ✅ becomes decline | |
| `SessionEnd` | Session terminates | ❌ | |

**Stdin fields (universal):** `session_id`, `hook_event_name`, `transcript_path`, `cwd`
**PreToolUse adds:** `tool_name`, `tool_input`
**PostToolUse adds:** `tool_name`, `tool_input`, `tool_response`
**Stop/SubagentStop add:** `stop_hook_active`
**PreCompact adds:** `trigger`, `custom_instructions`

**Hook types:** `command`, `http`, `prompt`, `agent`
**Async hooks:** Add `"async": true` to run without blocking
**Exit codes:** `0` = continue | `2` = block (stderr to Claude) | other = warning

**hookSpecificOutput JSON stdout:**
```json
{
  "continue": true,
  "suppressOutput": false,
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "additionalContext": "string injected into Claude's context"
  }
}
```

### Hook Config Format
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": "/path/to/script.sh", "async": false }
        ]
      }
    ]
  }
}
```

### Permissions
```json
{
  "permissions": {
    "allow": ["Bash(git *)", "Read(**)"],
    "deny": ["Bash(rm -rf *)"]
  }
}
```

### Skills (Slash Commands)
- Location: `~/.claude/skills/*/SKILL.md` (global), `.claude/skills/*/SKILL.md` (project)
- Format: Markdown with YAML frontmatter (`name`, `description`, `trigger`)
- Registered via `CLAUDE.md` section

---

## Codex CLI

**Vendor:** OpenAI | **Config format:** TOML | **Instruction file:** `AGENTS.md`
**Sources:** https://developers.openai.com/codex/hooks [official] · https://developers.openai.com/codex/config-reference [official] · https://github.com/openai/codex/blob/main/docs/config.md [github]

### Config Files
| File | Scope |
|------|-------|
| `~/.codex/config.toml` | Global |
| `.codex/config.toml` | Project (requires trust) |
| `~/.codex/hooks.json` | Global hooks |
| `.codex/hooks.json` | Project hooks |
| `requirements.toml` | Admin-enforced policy |
| `AGENTS.md` | Natural-language instructions |

### Built-in Tools
| Tool | Description |
|------|-------------|
| `shell` | Execute shell commands |
| `read_file` | Read file contents |
| `write_file` | Create/modify files |
| `apply_patch` | Apply unified diffs |
| `exec_command` | Long-lived PTY / interactive REPL |
| `write_stdin` | Feed stdin to existing exec_command session |
| `update_plan` | Manage plan/TODO state |
| `multi_tool_use` / `multi_tool_use.parallel` | Parallel tool calls |
| `web_search` | Web search (cached) |
| `browser` | Web browsing |
| `image_generation` | Generate/edit images |

### Hook Events
| Event | When | Can Block |
|-------|------|-----------|
| `SessionStart` | Session begins | ❌ |
| `SubagentStart` | Before subagent launches | ❌ |
| `PreToolUse` | Before tool execution | ✅ — deny or rewrite input |
| `PermissionRequest` | Before approval prompt | ✅ — allow/deny |
| `PostToolUse` | After tool completes | ✅ — replace result |
| `PreCompact` | Before context compaction | ✅ |
| `PostCompact` | After context compaction | ✅ |
| `UserPromptSubmit` | Before user prompt sent | ✅ |
| `SubagentStop` | Subagent stops | ✅ — can continue |
| `Stop` | Turn ends | ✅ — can continue |

**Input format:** JSON via stdin
**Universal stdin fields:** `session_id`, `cwd`, `hook_event_name`, `model`, `transcript_path`, `turn_id`
**Plugin env vars:** `PLUGIN_ROOT`, `PLUGIN_DATA` (and `CLAUDE_PLUGIN_ROOT`/`CLAUDE_PLUGIN_DATA` for compat)
**Exit codes:** `0` = success | `2` = block (stderr as reason) | other = failed

**stdout response:**
```json
{
  "continue": boolean,
  "stopReason": "string",
  "systemMessage": "string",
  "suppressOutput": boolean
}
```
`PreToolUse` also supports `"deny"` or `{"updatedInput": {...}}` to rewrite tool input.

### Hook Config Format (TOML)
```toml
[[hooks.PreToolUse]]
matcher = "^shell$"

[[hooks.PreToolUse.hooks]]
type = "command"
command = "path/to/script"
timeout = 600
statusMessage = "optional UI text"
```

### Skills
- Location: `~/.agents/skills/*/SKILL.md` (global), `.agents/skills/*/SKILL.md` (project)
- Same `.agents/skills/` path used by **Google Antigravity**

---

## Gemini CLI

**Vendor:** Google | **Config format:** JSON | **Instruction file:** `GEMINI.md`
**Sources:** https://geminicli.com/docs/hooks/reference/ [official] · https://google-gemini.github.io/gemini-cli/docs/tools/ [official] · https://github.com/google-gemini/gemini-cli/blob/main/docs/hooks/reference.md [github]

### Config Files
| File | Scope |
|------|-------|
| `~/.gemini/settings.json` | Global |
| `.gemini/settings.json` | Project |

### Built-in Tools
| Internal Name | Description |
|---------------|-------------|
| `read_file` | Read file (text, image, audio, PDF) |
| `read_many_files` | Read multiple files/dirs at once |
| `write_file` | Create or overwrite file |
| `replace` | Replace text within file |
| `list_directory` | List directory contents |
| `glob` | Find files by glob pattern |
| `grep_search` | Regex search in file contents |
| `run_shell_command` | Execute shell commands |
| `web_fetch` | Fetch URL content |
| `google_web_search` | Google web search |
| `save_memory` | Persist info across sessions |
| `ask_user` | Request clarification from user |
| `invoke_agent` | Spawn subagent |
| `activate_skill` | Load a skill |
| `enter_plan_mode` | Switch to plan mode |
| `update_topic` | Update topic context |
| `list_background_processes` | List background processes |
| `read_background_output` | Read from background process |

### Hook Events
| Event | When | Can Block | Notes |
|-------|------|-----------|-------|
| `BeforeTool` | Before tool invocation | ✅ | exit 2 or `decision: deny` |
| `AfterTool` | After tool execution | ✅ | |
| `BeforeAgent` | After user submits, before planning | ✅ | exit 2 erases/retries prompt |
| `AfterAgent` | After model generates final response | ✅ | |
| `BeforeModel` | Before LLM request | ✅ | exit 2 skips LLM call |
| `AfterModel` | After LLM response chunk | ✅ | |
| `BeforeToolSelection` | Before LLM decides which tools | ✅ | no `decision`/`continue`; use `toolConfig` |
| `SessionStart` | Startup, resume, or `/clear` | ❌ | advisory only |
| `SessionEnd` | Exit or session clear | ❌ | advisory only |
| `Notification` | System alert | ❌ | advisory only |
| `PreCompress` | Before history summarization | ❌ | advisory only |

**Input format:** JSON via stdin
**Universal stdin fields:** `session_id`, `transcript_path`, `cwd`, `hook_event_name`, `timestamp`
**Exit codes:** `0` = success | `2` = immediate block | other = warning, continues

**stdout response:**
```json
{
  "decision": "allow | deny | block",
  "reason": "string",
  "systemMessage": "string",
  "suppressOutput": boolean,
  "continue": boolean,
  "hookSpecificOutput": {
    "hookEventName": "string",
    "additionalContext": "string",
    "tool_input": {},
    "toolConfig": { "mode": "ANY|AUTO|NONE", "allowedFunctionNames": [] },
    "clearContext": boolean
  }
}
```

### Hook Config Format
```json
{
  "hooks": {
    "BeforeTool": [
      {
        "matcher": "run_shell_command",
        "sequential": true,
        "hooks": [
          { "name": "id", "type": "command", "command": "path/to/script", "timeout": 60000 }
        ]
      }
    ]
  }
}
```
`sequential: true` = serial; `false` = parallel. `timeout` in milliseconds.

### Skills
- Location: `~/.gemini/skills/*/SKILL.md` (global, Linux/Mac), `~/.agents/skills/` (Windows)

---

## Kiro IDE / CLI

**Vendor:** Kiro.dev | **Config format:** YAML | **Instruction file:** `.kiro/steering/*.md`
**Sources:** https://kiro.dev/docs/cli/hooks/ [official] · https://kiro.dev/docs/hooks/types/ [official] · https://kiro.dev/docs/cli/reference/built-in-tools/ [official] · https://kiro.dev/docs/steering/ [official]

### Config Files
| File | Scope |
|------|-------|
| `~/.kiro/config.yaml` | Global |
| `.kiro/config.yaml` | Project |
| `.kiro/steering/*.md` | Workspace steering (Markdown + frontmatter) |
| `~/.kiro/steering/*.md` | Global steering |
| `.kiro/hooks.yaml` | IDE hook definitions |
| `.kiro/agents/*.yaml` | Custom CLI agent definitions |

**Steering frontmatter fields:** `inclusion` (always/auto/conditional/manual), `fileMatchPattern`, `name`, `description`

### Built-in Tools (CLI)
| Tool | Description |
|------|-------------|
| `read` | Read files, folders, images |
| `glob` | File discovery by pattern (respects .gitignore) |
| `grep` | Regex content search (respects .gitignore) |
| `write` | Create and edit files |
| `shell` | Execute bash commands |
| `aws` | AWS CLI calls (service + operation + params) |
| `web_search` | Search the web |
| `web_fetch` | Fetch URL content |
| `introspect` | Answer questions about Kiro CLI itself |
| `code` | Symbol search + LSP integration |
| `tool_search` | Find + load MCP tools on demand |
| `delegate` | Delegate tasks to background async agents |
| `report` | Submit GitHub issues/feature requests |
| `knowledge` | Semantic knowledge store (cross-session) |
| `thinking` | Internal reasoning (complex decomposition) |
| `todo` | Multi-step task list management |
| `session` | Override CLI settings for current session |
| `subagent` | Spawn specialized subagents in parallel |

### Hook Events — CLI
| Event | When | Can Block | Stdin fields |
|-------|------|-----------|-------------|
| `AgentSpawn` | Agent activated | ❌ | `hook_event_name`, `cwd`, `session_id` |
| `UserPromptSubmit` | User submits prompt | ❌ (output → context) | + `prompt` |
| `PreToolUse` | Before tool execution | ✅ exit 2 | + `tool_name`, `tool_input` |
| `PostToolUse` | After tool execution | ❌ | + `tool_name`, `tool_input`, `tool_response` |
| `Stop` | Agent finishes response | ❌ | `cwd`, `session_id` |

**Tool matchers:** internal tool names (`fs_read`, `fs_write`, `execute_bash`, `use_aws`), categories (`@builtin`, `@mcp`), or glob patterns.

### Hook Events — IDE
| Trigger | When | Can Block |
|---------|------|-----------|
| Prompt Submit | User submits prompt | ✅ |
| Agent Stop | Agent completes turn | ✅ |
| Pre Tool Use | Before tool invocation | ✅ |
| Post Tool Use | After tool | ❌ |
| File Create | New file matches glob | ❌ |
| File Save | File matching pattern saved | ❌ |
| File Delete | File matching pattern deleted | ❌ |
| Pre Task Execution | Before spec task → in_progress | ✅ |
| Post Task Execution | After spec task → completed | ❌ |
| Manual Trigger | On-demand | N/A |

**IDE hook action types:** `Shell Command` (local, free) or `Agent Prompt` (LLM, costs credits)
**Shell hook exit codes:** `0` = stdout → context | non-zero = stderr → agent (blocks for PreToolUse/PromptSubmit)
**Timeout:** default 60s (set 0 to disable)

### Hook Config Format (YAML)
```yaml
hooks:
  PreToolUse:
    - matcher: "shell"
      command: ~/.kiro/hooks/validate.sh
  AgentSpawn:
    - command: ~/.kiro/hooks/setup.sh
```

### Skills
- Location: `~/.kiro/skills/*/SKILL.md`

---

## Kimi Code CLI

**Vendor:** Moonshot AI | **Config format:** TOML | **Instruction file:** `KIMI.md` (inferred)
**Sources:** https://moonshotai.github.io/kimi-cli/en/customization/hooks.html [official] · https://moonshotai.github.io/kimi-cli/en/customization/agents.html [official] · https://moonshotai.github.io/kimi-cli/en/configuration/config-files.html [official]

### Config Files
| File | Scope |
|------|-------|
| `~/.kimi/config.toml` | Primary (auto-migrated from JSON) |
| `~/.kimi/config.json` | Legacy (backed up on migration) |
| Custom via `--config-file` | Override |

**Key config options:** `default_model`, `default_thinking`, `default_yolo`, `theme`, `providers`, `models`, `loop_control`, `background`, `mcp`, `services`

### Built-in Tools
| Tool | Description |
|------|-------------|
| `Agent` | Launch subagents (`coder`, `explore`, `plan` types) |
| `AskUserQuestion` | Collect user input via structured questions |
| `SetTodoList` | Task progress tracking |
| `Shell` | Execute shell commands (requires approval) |
| `ReadFile` | Read text file (up to 1000 lines/op) |
| `ReadMediaFile` | Read images or videos (up to 100MB) |
| `Glob` | Pattern-based file search |
| `Grep` | Regex content search |
| `WriteFile` | Create or overwrite files |
| `StrReplaceFile` | Edit via string replacement |
| `SearchWeb` | Web search |
| `FetchURL` | Retrieve webpage content |
| `Think` | Record reasoning process |
| `SendDMail` | Send delayed messages (checkpoint scenarios) |
| `EnterPlanMode` | Initiate planning mode |
| `ExitPlanMode` | Submit plan for user approval |
| `TaskList` | List background tasks in session |
| `TaskOutput` | Get status/output of background tasks |
| `TaskStop` | Cancel running background tasks |

### Hook Events (Beta)
| Event | When | Can Block | Stdin fields |
|-------|------|-----------|-------------|
| `PreToolUse` | Before tool execution | ✅ exit 2 | `tool_name`, `tool_input`, `tool_call_id` |
| `PostToolUse` | After tool succeeds | ❌ | + `tool_output` |
| `PostToolUseFailure` | After tool fails | ❌ | + `error` |
| `UserPromptSubmit` | Before user input processed | ❌ | `prompt` |
| `Stop` | Agent turn ends | ✅ via stdout JSON | `stop_hook_active` |
| `StopFailure` | Turn ends due to error | ❌ | `error_type`, `error_message` |
| `SessionStart` | Session created or resumed | ❌ | `source` (startup/resume) |
| `SessionEnd` | Session closed | ❌ | `reason` |
| `SubagentStart` | Subagent launches | ❌ | `agent_name`, `prompt` |
| `SubagentStop` | Subagent concludes | ❌ | `agent_name`, `response` |
| `PreCompact` | Before context compaction | ❌ | `trigger`, `token_count` |
| `PostCompact` | After context compaction | ❌ | `trigger`, `estimated_token_count` |
| `Notification` | Notification delivered | ❌ | `sink`, `notification_type`, `title`, `body`, `severity` |
| `Interrupt` | User interrupts current turn (Esc) | ❌ | — |
| `PermissionRequest` | Before user approval prompt is shown | ❌ | `tool_name` |
| `PermissionResult` | After user approval completes | ❌ | `tool_name` |

**Exit codes:** `0` = allow (stdout → context if non-empty) | `2` = block (stderr → LLM) | other = allow silently (stderr logged)
**Fail-open policy:** hook failures do not interrupt workflows.
**Stop blocking:** uses structured JSON stdout, not exit code 2.

### Skills
- Location: `~/.kimi/skills/*/SKILL.md`
- Same skill format as Claude Code (`skill.md`)

---

## Factory Droid

**Vendor:** Factory.ai | **Config format:** JSON | **Instruction file:** `FACTORY.md` (inferred)
**Sources:** https://docs.factory.ai/reference/hooks-reference [official] · https://docs.factory.ai/cli/configuration/custom-droids [official] · https://docs.factory.ai/cli/configuration/hooks-guide [official]

### Config Files
| File | Scope |
|------|-------|
| `~/.factory/settings.json` | Global |
| `.factory/settings.json` | Project |
| `.factory/settings.local.json` | Project personal (git-ignored) |

### Built-in Tools
| Tool | Description |
|------|-------------|
| `Read` | File content retrieval |
| `LS` | Directory listing |
| `Grep` | Text search in files |
| `Glob` | Pattern-based file matching |
| `Create` | New file generation |
| `Edit` | File modification |
| `ApplyPatch` | Patch application (auto-included with Edit for OpenAI models) |
| `Execute` | Shell command execution |
| `WebSearch` | Web search |
| `FetchUrl` | Web content retrieval |
| `TodoWrite` | Task tracking (auto-included for all droids) |

**Tool category shortcuts:** `read-only` (Read, LS, Grep, Glob) | `edit` (Create, Edit, ApplyPatch) | `execute` (Execute) | `web` (WebSearch, FetchUrl) | `mcp` (dynamic)
**MCP tool pattern:** `mcp__<server>__<tool>`

### Hook Events
| Event | When | Can Block | Exit 2 behavior |
|-------|------|-----------|----------------|
| `PreToolUse` | Before tool execution | ✅ | Blocks tool; stderr fed to Droid |
| `PostToolUse` | After tool completes | ⚠️ Partial | stderr shown to Droid; tool already ran |
| `UserPromptSubmit` | User submits prompt | ✅ | Blocks + erases prompt |
| `Notification` | Droid sends notification | ❌ | stderr shown to user only |
| `Stop` | Droid finishes responding | ✅ | Blocks stoppage; stderr to Droid |
| `SubagentStop` | Sub-droid Task completes | ✅ | Blocks; stderr to sub-droid |
| `PreCompact` | Before compact operation | ❌ | stderr shown to user only |
| `SessionStart` | Session start or resume | ❌ | stderr shown to user only |
| `SessionEnd` | Session terminates | ❌ | Cannot prevent termination |

**Stdin JSON payloads (confirmed schemas):**
```json
// PreToolUse
{ "session_id", "transcript_path", "cwd", "permission_mode", "hook_event_name", "tool_name", "tool_input" }
// PostToolUse — adds:
{ "tool_response" }
// UserPromptSubmit — replaces tool fields with:
{ "prompt" }
// Stop/SubagentStop — replaces tool fields with:
{ "stop_hook_active": boolean }
// PreCompact:
{ "trigger": "manual|auto", "custom_instructions" }
// SessionStart:
{ "source": "startup|resume|clear|compact" }
// SessionEnd:
{ "reason": "clear|logout|prompt_input_exit|other" }
```

**stdout JSON response (advanced):**
```json
{
  "continue": true,
  "suppressOutput": false,
  "systemMessage": "string",
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow|deny|ask",
    "permissionDecisionReason": "string",
    "updatedInput": {},
    "additionalContext": "string",
    "decision": "block|undefined"
  }
}
```
`updatedInput` on `PreToolUse` mutates tool parameters in-flight.

**Execution:** all matching hooks run in parallel; 60s default timeout; identical commands deduplicated; `$FACTORY_PROJECT_DIR` env var available.

### Hook Config Format
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Execute",
        "hooks": [{ "type": "command", "command": "/abs/path/script.sh", "timeout": 60 }]
      }
    ]
  }
}
```

### Skills
- Location: `~/.factory/skills/*/SKILL.md`

---

## Hermes Agent

**Vendor:** NousResearch | **Config format:** YAML | **Instruction file:** `HERMES.md` / `.hermes.md`
**Sources:** https://hermes-agent.nousresearch.com/docs/user-guide/features/hooks [official] · https://hermes-agent.nousresearch.com/docs/reference/tools-reference [official] · https://github.com/NousResearch/hermes-agent/blob/main/website/docs/user-guide/features/hooks.md [github]

### Config Files
| File | Scope |
|------|-------|
| `~/.hermes/config.yaml` | Primary config (model, hooks, all settings) |
| `~/.hermes/.env` | API keys/secrets |
| `~/.hermes/auth.json` | OAuth credentials |
| `~/.hermes/SOUL.md` | Agent identity/personality |
| `~/.hermes/shell-hooks-allowlist.json` | Hook consent store |
| `.hermes.md` / `HERMES.md` | Project-specific instructions |

### Built-in Tools (selected — Hermes has 50+ tools)
| Category | Tools |
|----------|-------|
| Files | `read_file`, `write_file`, `search_files`, `patch` |
| Shell | `terminal`, `process` |
| Web | `web_search`, `web_extract`, `x_search` |
| Browser | `browser_navigate`, `browser_snapshot`, `browser_click`, `browser_type`, `browser_scroll`, `browser_vision`, `browser_cdp`, `browser_console`, `browser_dialog`, `browser_get_images` |
| Code | `execute_code` |
| Media | `image_generate`, `video_generate`, `video_analyze`, `text_to_speech`, `vision_analyze` |
| Agent | `delegate_task`, `clarify`, `todo`, `session_search`, `skill_manage`, `skills_list` |
| Memory | `memory`, `cronjob` |
| Messaging | `send_message`, `discord`, `discord_admin`, `feishu_doc_read`, `feishu_drive_*` |
| Home | `ha_call_service`, `ha_get_state`, `ha_list_entities`, `ha_list_services` |
| Desktop | `computer_use` |
| Music | `spotify_*` (7 tools) |
| Project | `kanban_*` (9 tools) |
| LLM | `mixture_of_agents` |
| AI | `yb_*` (5 Yuanbao tools) |

### Hook Events

**RESOLVED CONFLICT:** Both `pre_tool_call`/`post_tool_call` AND `pre_llm_call`/`post_llm_call` exist — they are different lifecycle stages.

**Plugin hooks** (registered via `ctx.register_hook()` in Python plugins):
| Event | When | Can Block | Return |
|-------|------|-----------|--------|
| `pre_tool_call` | Before tool execution | ✅ | `{"action": "block", "message": str}` |
| `post_tool_call` | After tool returns | ❌ | ignored |
| `pre_llm_call` | Before tool-calling loop starts | ❌ | `{"context": str}` injects into message |
| `post_llm_call` | After tool loop completes | ❌ | ignored |
| `on_session_start` | First turn of session | ❌ | ignored |
| `on_session_end` | Conversation end | ❌ | ignored |
| `on_session_finalize` | Session teardown | ❌ | ignored |
| `on_session_reset` | Fresh session allocated | ❌ | ignored |
| `subagent_stop` | Child agent completes | ❌ | ignored |
| `pre_gateway_dispatch` | Before auth/dispatch (Gateway) | ✅ | `{"action": "skip|rewrite|allow"}` |
| `transform_tool_result` | After tool, pre-model | ❌ | `str` replaces result |
| `transform_terminal_output` | Terminal pre-truncation | ❌ | `str` replaces output |
| `transform_llm_output` | Final response pre-delivery | ❌ | `str` replaces response |

**Shell hooks** (in `~/.hermes/config.yaml`):
```yaml
hooks:
  pre_tool_call:
    - matcher: "<regex>"   # optional
      command: "<shell>"
      timeout: 60          # max 300
  pre_llm_call:
    - command: "<shell>"
hooks_auto_accept: false   # or --accept-hooks flag / HERMES_ACCEPT_HOOKS=1
```

**Shell hook stdin JSON:**
```json
{ "hook_event_name": "pre_tool_call", "tool_name": "terminal", "tool_input": {"command": "..."}, "session_id": "...", "cwd": "/path", "extra": {} }
```

**Shell hook stdout JSON:** `{"decision": "block", "reason": "..."}` to block | `{"context": "..."}` to inject | `{}` = no-op

**Security:** Each unique `(event, command)` pair prompts for approval on first run; persisted to `~/.hermes/shell-hooks-allowlist.json` by hash.

### Skills
- Location: `~/.hermes/skills/*/SKILL.md`
- Uses same skill format as OpenClaw

---

## OpenClaw

**Vendor:** OpenClaw (community) | **Config format:** YAML | **Instruction file:** `AGENTS.md` (or `.agents/` or `packages/*/AGENTS.md`)
**Sources:** https://docs.openclaw.ai/automation/hooks [official] · https://docs.openclaw.ai/tools/apply-patch [official] · https://docs.openclaw.ai/plugins/hooks [official]

### Config Files
| File | Scope | Purpose |
|------|-------|---------|
| `~/.openclaw/config.yaml` | Global | User settings |
| `config.yaml` | Project | Project overrides |
| `~/.openclaw/hooks/` | Global | Managed hook definitions shared across workspaces |
| `<workspace>/hooks/` | Project | Workspace-level hook overrides |

### Built-in Tools
| Tool | Specification | Description |
|------|---------------|-------------|
| `bash` | `command` | Run terminal commands on the host |
| `read_file` | `path` | Retrieve raw contents of files (text/images) |
| `write_file` | `path`, `content` | Write or overwrite files in the workspace |
| `edit_file` | `path`, `edits` | Apply targeted edits to a file |
| `search` | `query`, `glob` | Find symbols and matches using ripgrep |
| `web_fetch` | `url` | Retrieve HTML/Markdown from remote pages |
| `apply_patch` | `input` | Apply multi-file diff patches via unified patch block |

#### `apply_patch` Format
Accepts unified patch blocks containing update, add, delete, and move operations inside `*** Begin Patch` and `*** End Patch`:
```
*** Begin Patch
*** Add File: path/to/file
+line 1
*** Update File: src/app.ts
@@
-old line
+new line
*** End Patch
```

### Hook Events
OpenClaw has two distinct hook surfaces:

1. **Internal Event Hooks (Non-blocking):** Asynchronously run side-effects for command and lifecycle events.
2. **Typed Plugin Hooks (In-process, Blocking):** Run synchronously via the Plugin SDK (`api.on(...)`) for middleware policy gates, canceling actions, or argument mutations.

#### Event Matrices

##### Internal Event Hooks (Non-blocking)
| Event | When | Context Details |
|-------|------|-----------------|
| `command:new` | `/new` command issued | `sessionEntry`, `workspaceDir` |
| `command:reset` | `/reset` command issued | `sessionEntry`, `previousSessionEntry` |
| `command:stop` | `/stop` command issued | Termination details |
| `command` | Any command event | |
| `session:compact:before` | Before context compaction | `messageCount`, `tokenCount` |
| `session:compact:after` | After compaction completes | `tokensBefore`, `tokensAfter` |
| `session:patch` | Session properties modified | Privileged patches |
| `agent:bootstrap` | Before workspace bootstrap | Mutable `bootstrapFiles` array |
| `gateway:startup` | Gateway processes start up | |
| `gateway:shutdown` | Gateway shutdown begins | `reason`, `restartExpectedMs` |
| `gateway:pre-restart` | Before scheduled restart | Bounded drain budget (10s limit) |
| `message:received` | Inbound message arrives | `from`, `content`, `channelId`, `metadata` |
| `message:transcribed` | Audio transcription done | `transcript`, `mediaPath` |
| `message:preprocessed` | Media/link preprocessed | Final enriched `bodyForAgent` |
| `message:sent` | Outbound message delivered | `to`, `content`, `success` |

##### Typed Plugin Hooks (Blocking)
| SDK Hook | When it fires | Can Block? | Notes |
|----------|---------------|------------|-------|
| `before_model_resolve` | Pre-session, before provider resolution | ❌ No | Deterministically override provider/model |
| `before_prompt_build` | Post-session load, before prompt submission | ❌ No | Inject dynamic `prependContext` or system prompts |
| `before_tool_call` | Before any agent tool executes | ✅ Yes | Can rewrite arguments or block call (`{ block: true }`) |
| `after_tool_call` | After tool completes execution | ❌ No | Inspect tool outputs/results |
| `tool_result_persist` | Before tool result is written to transcript | ❌ No | Synchronously transform outputs before log |
| `before_agent_reply` | Before the LLM generation call | ✅ Yes | Can return synthetic reply or silence turn entirely |
| `before_agent_finalize` | Agent finishes reasoning turn | ✅ Yes | Can inspect messages and request extra pass |
| `agent_end` | Final turn completed | ❌ No | Inspect final message lists and metadata |
| `before_compaction` | Before context compaction begins | ❌ No | Observe compaction trigger and token counts |
| `after_compaction` | After compaction completes | ❌ No | Inspect compacted summaries |
| `before_install` | Before skill or plugin installation | ✅ Yes | Prevent untrusted execution (`{ block: true }`) |
| `message_received` | Inbound message received | ❌ No | Telemetry/hooks for incoming channels |
| `message_sending` | Outbound message about to be sent | ✅ Yes | Cancel outbound message via `{ cancel: true }` |
| `message_sent` | Outbound message delivered | ❌ No | Telemetry for successful deliveries |
| `session_start` | Session initialized | ❌ No | Setup session boundaries |
| `session_end` | Session finalized or gateway stopped | ❌ No | Bounded cleanup drain, run on SIGTERM/restart |
| `gateway_start` | Gateway process starts | ❌ No | Global startup initialization |
| `gateway_stop` | Gateway process begins shutdown | ❌ No | Global cleanup sequence |

#### Hook Decision Rules
- **Tool Guard (`before_tool_call`):** Returning `{ block: true }` immediately stops execution of lower-priority handlers and blocks the tool.
- **Install Guard (`before_install`):** Returning `{ block: true }` immediately cancels the installation.
- **Message Outbound (`message_sending`):** Returning `{ cancel: true }` immediately suppresses and cancels message delivery.

### Hook Config Format
Internal hooks are enabled via YAML config:
```yaml
hooks:
  internal:
    enabled: true
    entries:
      session-memory:
        enabled: true
      command-logger:
        enabled: false
```
Plugin hooks are registered programmatically via the Plugin SDK.

### Concurrency and Locking
- **Session Lanes:** Execution runs are serialized per session key to avoid race conditions.
- **Session File Lock:** Workspace transcript writes are guarded by a process-aware, file-based session write lock. Transcript writers wait up to `session.writeLock.acquireTimeoutMs` (default: `60000` ms) before reporting busy.
- **Reentrancy:** Session locks are non-reentrant by default. Nested lock acquisitions must explicitly pass `allowReentrant: true`.

### Timeouts & Diagnostics
- **RPC Wait Timeout:** `agent.wait` default wait timeout is `30` seconds (overridable via `timeoutMs`).
- **Agent Runtime Timeout:** Runtime timeout defaults to `172800` seconds (48 hours) via `agents.defaults.timeoutSeconds` (enforced via abort timer in `runEmbeddedPiAgent`).
- **Model Idle Timeout:** OpenClaw watchdogs model requests and aborts if no chunks arrive (capped at `120` seconds by default; overridable via `models.providers.<id>.timeoutSeconds`).
- **Stuck/Stalled Sessions:** The diagnostics engine monitors processing lanes. Session stalls are reported as `session.stalled` after `diagnostics.stuckSessionWarnMs`. Active stale runs are abort-drained and released only after `diagnostics.stuckSessionAbortMs` (default: 5 minutes/3x warning threshold) to prevent stuck lanes.

### Skills
- Location: `~/.openclaw/skills/*/SKILL.md`
- Same skill format as Hermes

---

## Cursor

**Vendor:** Anysphere | **Config format:** JSON (hooks) + MDC (rules)
**Sources:** https://cursor.com/docs/hooks [official] · https://cursor.com/docs/agent/tools [official] · https://cursor.com/docs/rules [official] · https://cursor.com/docs/reference/third-party-hooks [official]

### Config Files
| File | Scope |
|------|-------|
| `~/.cursor/settings.json` | Global settings |
| `~/.cursor/hooks.json` | Global hooks |
| `.cursor/hooks.json` | Project hooks |
| `.cursor/rules/*.mdc` | Project rules (primary) |
| `.cursor/rules/*.md` | Project rules (no frontmatter) |
| `.cursorrules` | Legacy (ignored in Agent mode) |
| `AGENTS.md` | Universal agent instructions |

**MDC frontmatter fields:** `alwaysApply`, `description`, `globs`
**Hook env vars:** `CURSOR_PROJECT_DIR`, `CURSOR_VERSION`, `CURSOR_USER_EMAIL`, `CURSOR_TRANSCRIPT_PATH`, `CURSOR_CODE_REMOTE`

### Built-in Tools
| Tool | Description |
|------|-------------|
| `Semantic search` | Codebase search by meaning |
| `Search files and folders` | File name + keyword search |
| `Web` | Web search |
| `Fetch Rules` | Retrieve cursor rules |
| `Read files` | Read files (text + images) |
| `Edit files` | Suggest and apply edits |
| `Run shell commands` | Execute terminal commands |
| `Browser` | Full browser control (screenshots, navigate, interact) |
| `Image generation` | Generate images from prompts |
| `Ask questions` | Clarification while agent continues |

### Hook Events (21 events — all receive JSON via stdin)
| Event | When | Can Block |
|-------|------|-----------|
| `sessionStart` | New composer conversation | ❌ |
| `sessionEnd` | Conversation ends | ❌ |
| `preToolUse` | Before any tool | ✅ exit 2 |
| `postToolUse` | After tool succeeds | ❌ |
| `postToolUseFailure` | Tool fails/times out/denied | ❌ |
| `subagentStart` | Before spawning subagent | ✅ |
| `subagentStop` | Subagent completes/errors | ❌ |
| `beforeShellExecution` | Before shell command | ✅ |
| `afterShellExecution` | After shell command | ❌ |
| `beforeMCPExecution` | Before MCP tool | ✅ |
| `afterMCPExecution` | After MCP tool | ❌ |
| `beforeReadFile` | Before agent reads a file | ✅ |
| `afterFileEdit` | After agent edits a file | ❌ |
| `beforeTabFileRead` | Before Tab reads a file | ✅ |
| `afterTabFileEdit` | After Tab edits a file | ❌ |
| `beforeSubmitPrompt` | Before user prompt submitted | ✅ |
| `preCompact` | Before context compaction | ❌ |
| `afterAgentResponse` | After agent completes message | ❌ |
| `afterAgentThought` | After thinking block | ❌ |
| `stop` | Agent loop ends | ❌ |
| `workspaceOpen` | Workspace opens/folder changes | ❌ |

**Exit codes:** `0` = allow | `2` = block (stderr → LLM) | other = proceed (or block if `failClosed: true`)
`stop` + `subagentStop` support `followup_message` in stdout to continue loop.
`workspaceOpen` can return `pluginPaths`.

**Key stdin fields by event:**
- `preToolUse`: `tool_name`, `tool_input`, `tool_use_id`, `cwd`, `model`
- `beforeShellExecution`: `command`, `cwd`, `sandbox`
- `beforeMCPExecution`: `tool_name`, `tool_input`, `url` or `command`
- `beforeReadFile`: `file_path`, `content`, `attachments`
- `beforeSubmitPrompt`: `prompt`, `attachments`
- `sessionStart`: `session_id`, `is_background_agent`, `composer_mode`

### Hook Config Format
```json
{
  "event": "preToolUse",
  "command": ".cursor/hooks/validate.sh",
  "type": "command",
  "matcher": "Run shell commands",
  "timeout": 5000,
  "failClosed": true
}
```

---

## Devin CLI (Devin for Terminal)

**Vendor:** Cognition | **Config format:** JSON | **Instruction file:** `AGENTS.md` / `CLAUDE.md` (compatibility)
**Sources:** https://cli.devin.ai/docs/extensibility [official] · https://cli.devin.ai/docs/extensibility/hooks/overview [official]

### Config Files
| File | Scope |
|------|-------|
| `~/.config/devin/config.json` | Global |
| `.devin/config.json` | Project |
| `.devin/config.local.json` | Project personal (git-ignored) |
| `.devin/hooks.v1.json` | Hooks (standalone recommended file) |
| `.devin/skills/` | Custom skills |
| `.devin/agents/` | Custom subagent profiles |

Also reads: `AGENTS.md`, `AGENT.md`, `CLAUDE.md`, `.cursor/rules/*.md`, `.windsurf/rules/*.md`

### Hook Events (7 events — official)
| Event | When | Can Block |
|-------|------|-----------|
| `PreToolUse` | Before tool call | ✅ exit 2 or `{"decision":"block"}` |
| `PostToolUse` | After tool completes | ❌ |
| `PermissionRequest` | Permission decision needed | ✅ |
| `UserPromptSubmit` | User submits a message | ❌ |
| `Stop` | Agent wants to stop | ✅ |
| `SessionStart` | Session begins | ❌ |
| `SessionEnd` | Session ends | ❌ |

**Hook types:** `command` (shell) or `prompt` (LLM evaluation)
**stdin:** JSON with `hook_event_name`, `tool_name`, `tool_input`
**stdout:** `{"decision": "approve|block|deny", "reason": "..."}`
**Exit codes:** `0` = continue | `2` = block | other = error logged (fail-open)
**Env:** `DEVIN_PROJECT_DIR` set to project root

### Skills
- Global: `~/.config/devin/skills/<name>/`
- Project: `.devin/skills/<name>/`
- Format: Markdown + YAML frontmatter (`name`, `description`, `allowed-tools`, `triggers`)

---

## Aider

**Vendor:** Aider-AI | **Config format:** YAML | **Instruction file:** `CONVENTIONS.md` (or `.aider.conventions.md`)
**Sources:** https://aider.chat/docs/config/options.html [official] · https://aider.chat/docs/git.html [official] · https://github.com/Aider-AI/aider/issues/2045 [github — hooks feature request]

### Config Files
| File | Scope | Purpose |
|------|-------|---------|
| `~/.aider.conf.yml` | Global | User settings |
| `.aider.conf.yml` | Project | Project settings |
| `.env` | Project | Environment variables and API keys |
| `CONVENTIONS.md` | Project | Standing instructions and coding style conventions |

### Key Config Options
```yaml
model: claude-opus-4
editor-model: claude-haiku-4-5
auto-commits: true
lint-cmd: "eslint --fix {files}"
test-cmd: "npm test"
watch-files: true
```

### Hooks
- **No native pre/post tool use hook system** — tracked in issue #2045.
- **Git hooks integration:** Runs with `--no-verify` by default to skip slow local pre-commit hooks, but auto-generates descriptive commit messages on edits.
- **Functional quality gates (lint/test hooks):** Triggers `lint-cmd` and `test-cmd` automatically after edits, feeds non-zero exit codes back to LLM context, and prompts self-correction.
- **Workaround:** Wrapper shell script around `aider` command-line utility.

### Built-in Tools (In-Chat Commands)
Instead of tool-calling loops, Aider implements terminal slash commands:
- `/add <file>`: Add files to chat session for editing
- `/drop <file>`: Remove files from chat session
- `/read <file>`: Load file as read-only context (uses prompt caching)
- `/ls`: List repository files
- `/diff`: Show active changes since last commit
- `/commit`: Manually commit active changes to Git
- `/run <cmd>` (alias `!`): Run shell command (can optionally feed stdout to chat)
- `/test <cmd>`: Run command, add output to chat only if command fails
- `/lint`: Run linter on active files and self-correct errors
- `/model <model>`: Switch model
- `/ask <query>`: Ask questions without editing files

---

## Pi Coding Agent

**Vendor:** earendil-works / can1357 | **Config format:** JSON (settings) + YAML (hooks)
**Sources:** https://pi.dev/packages/pi-yaml-hooks [official] · https://github.com/can1357/oh-my-pi/blob/main/docs/hooks.md [github] · https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/docs/extensions.md [github]

### Config Files
| File | Scope |
|------|-------|
| `~/.pi/agent/settings.json` | Global |
| `.pi/settings.json` | Project |
| `hooks.yaml` | Hook config (via `pi-yaml-hooks` package) |
| `.pi/extensions/*.ts` | TypeScript extensions |

### Hook Events (via `pi-yaml-hooks` package)
| Event | When | Can Block |
|-------|------|-----------|
| `tool.before.*` | Before tool (glob matches tool name) | ✅ exit 2 |
| `tool.after.*` | After tool | ❌ |
| `file.changed` | File modified | ❌ |
| `session.created` | Session start | ❌ |
| `session.idle` | Session goes idle | ❌ |
| `session.deleted` | Session end | ❌ |
| `after_provider_response` | After provider HTTP response | ❌ |

**Hook actions:** `bash`, `tool`, `notify`, `confirm`, `setStatus`

### Skills
- Location: `~/.pi/agent/skills/*/SKILL.md`

---

## Trae / Trae CN

**Vendor:** ByteDance | No native hook system
**Sources:** https://docs.trae.ai/ide/agent [official] · https://docs.trae.ai/ide/rules [official] · https://github.com/bytedance/trae-agent [github]

### Config Files
| File | Scope |
|------|-------|
| `project_rules.md` | Project rules (Markdown) |
| `user_rules.md` | User rules |
| `.trae/mcp.json` | MCP servers |

**Extensibility:** Via MCP servers only. No native hook API.

### Skills
- Trae: `~/.trae/skills/*/SKILL.md`
- Trae CN: `~/.trae-cn/skills/*/SKILL.md`

---

## Google Antigravity

**Vendor:** Google | **Config format:** JSON / TOML | **Instruction file:** `.agents/rules/` and `.agents/workflows/`
**Sources:** https://antigravity.google/docs/home [official] · https://antigravity.google/docs/hooks [official] · https://discuss.ai.google.dev/t/does-antigravity-support-hooks-similar-to-the-hook-functionality-in-windsurf/121062 [community] · [installer-src] skill-path conventions

### Config Files
| File | Scope | Purpose |
|------|-------|---------|
| `~/.config/antigravity/config.toml` | Global | Model selection & API endpoints |
| `~/.gemini/antigravity-cli/settings.json` | Global | CLI interface settings |
| `~/.gemini/config/hooks.json` | Global | Global hooks configuration |
| `.agents/hooks.json` | Project | Project-specific hooks |
| `~/.gemini/config/mcp_config.json` | Global | Shared MCP configurations |

### Built-in Tools
| Tool | Description |
|------|-------------|
| `view_file` | Read and retrieve file contents |
| `replace_file_content` | Contiguous single-block file edits |
| `multi_replace_file_content` | Non-contiguous multiple-block file edits |
| `write_to_file` | Create new files |
| `list_dir` | List contents of a directory |
| `grep_search` | Search files for exact patterns |
| `run_command` | Execute shell/bash commands |
| `search_web` | Google web search |
| `read_url_content` | Fetch URL and convert HTML to markdown |

### Hook Events
Antigravity supports three categories of lifecycle hooks:
- **Inspect** (Read-Only, Non-Blocking)
- **Decide** (Read-Only, Blocking)
- **Transform** (Modifying, Blocking)

Execution order is strictly enforced: **Decide → Transform → Inspect** to prevent TOCTOU vulnerabilities.

| Event | When | Can Block | Notes |
|-------|------|-----------|-------|
| `PreToolUse` | Before a tool runs | ✅ | exit 2 or `{"decision":"block"}` |
| `PostToolUse` | After a tool completes | ❌ | Read-only |
| `PreInvocation` | Before calling the model | ✅ | For context/instruction injection |
| `PostInvocation` | After model calls finish | ❌ | |
| `Stop` | When the execution loop terminates | ✅ | |

**Input format:** JSON via stdin
**Universal stdin fields:** `hook_event_name`, `tool_name`, `tool_input`, `session_id`
**Exit codes:** `0` = success | `2` = block (stderr/reason sent to LLM) | other = warning/fails-open

**stdout response:**
```json
{
  "decision": "block",
  "reason": "Security policy violation"
}
```

### Hook Config Format
```json
{
  "safety-gate": {
    "enabled": true,
    "PreToolUse": [
      {
        "matcher": "run_command",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/safety-check.sh"
          }
        ]
      }
    ]
  }
}
```

### Skills
- Global: `~/.agents/skills/`
- Project: `.agents/skills/`
- Uses standard Markdown format with YAML frontmatter. Windows uses PowerShell.

---

## GitHub Copilot CLI / VS Code Chat

**Vendor:** GitHub/Microsoft | **Instruction file:** `.github/copilot-instructions.md`
**Sources:** https://docs.github.com/en/copilot/concepts/agents/hooks [official] · https://docs.github.com/en/copilot/reference/hooks-configuration [official] · https://docs.github.com/en/copilot/how-tos/copilot-sdk/use-hooks/pre-tool-use [official]

### Config Files
| File | Scope |
|------|-------|
| `.github/hooks/*.json` | Project (cloud agent reads only this) |
| `~/.copilot/hooks/` | Global user hooks |
| `.github/copilot/settings.json` | Inline hooks + settings |
| `~/.copilot/settings.json` | Global settings |

### Hook Events (12 events — official)
| Event | When | Can Block |
|-------|------|-----------|
| `preToolUse` | Before tool — can allow/deny/mutate args | ✅ |
| `postToolUse` | After tool — can modify result | Partial |
| `postToolUseFailure` | After tool fails | ❌ |
| `permissionRequest` | Before permission service — can approve/deny | ✅ |
| `agentStop` | Main agent finishes — can force another turn | ✅ |
| `subagentStop` | Subagent finishes — can force another turn | ✅ |
| `subagentStart` | Subagent starts | ❌ |
| `sessionStart` | Session start — can inject `additionalContext` | ❌ |
| `sessionEnd` | Session end | ❌ |
| `userPromptSubmitted` | After user input received | ❌ |
| `preCompact` | Before context compaction | ❌ |
| `notification` | On notification delivery | ❌ |

**Hook types:** `command` (bash/PowerShell), `http` (webhook), `prompt` (LLM, CLI-only)
**stdin:** JSON `{sessionId, timestamp, cwd, toolName, toolArgs}`
**stdout `preToolUse`:** `{permissionDecision, permissionDecisionReason, modifiedArgs, additionalContext}`
**stdout `agentStop`:** `{decision: "block|allow", reason}`
**Exit codes:** `0` = parse stdout | `2` = warning (deny for `permissionRequest`) | other = fail-open
**Cloud agent:** Linux only; `permissionRequest` does not fire; `ask` treated as `deny`

### Built-in Tools (VS Code)
| Tool | Description |
|------|-------------|
| `run_in_terminal` | Execute terminal commands |
| `read_file` | Read files |
| `create_file` / `insert_edit_into_file` | Write/edit files |
| `list_directory` | List files |
| `get_errors` | Get editor diagnostics |
| `run_tests` | Run test suite |
| `web_search` | Web search |

---

## OpenCode

**Vendor:** OpenCode.ai | **Config format:** JSON + JS plugins
**Sources:** https://opencode.ai/docs/config/ [official] · https://opencode.ai/docs/agents/ [official] · https://github.com/sst/opencode [github]

### Config Files
| File | Scope |
|------|-------|
| `~/.config/opencode/opencode.json` | Global |
| `opencode.json` | Project |
| `.opencode/plugins/` | Local plugins |

### Hook Events (via plugins)
| Event | When |
|-------|------|
| `before_tool` | Before tool execution |
| `after_tool` | After tool execution |
| `on_message` | Every LLM message |
| `on_session_start` | Session start |

Blocking: return `{ block: true, message: "..." }` from plugin handler.

### Skills
- Location: `~/.config/opencode/skills/*/SKILL.md`
