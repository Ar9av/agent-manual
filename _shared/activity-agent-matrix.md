# Activity → Agent Matrix

Every activity and which agents support it natively (built-in tools, no MCP required).

> Data sourced from official docs where available. See each tool's README for per-source citations. Key official sources used: [Codex tools](https://developers.openai.com/codex/cli/features) · [Gemini CLI tools](https://google-gemini.github.io/gemini-cli/docs/tools/) · [Kiro CLI tools](https://kiro.dev/docs/cli/reference/built-in-tools/) · [Kimi Code agents](https://moonshotai.github.io/kimi-cli/en/customization/agents.html) · [Factory Droid droids](https://docs.factory.ai/cli/configuration/custom-droids) · [Hermes tools](https://hermes-agent.nousresearch.com/docs/reference/tools-reference) · [Cursor tools](https://cursor.com/docs/agent/tools) · [VS Code Copilot](https://code.visualstudio.com/docs/copilot/agents/overview)

Legend: ✅ Built-in | 🔌 Via MCP | ⚠️ Partial/Limited | ❌ Not supported | ❓ Unknown

> **Notes:** OpenClaw is a gateway/routing product (not a standalone coding agent) — its tool capabilities are mediated through gateway-attached agents. Kiro is an Amazon/AWS product (not an independent vendor). Google Antigravity is Generally Available (not Preview). Pi Agent MCP support and subagent spawning require extensions (`pi-yaml-hooks` package or TypeScript extensions), not built-in.

---

## File Operations

| Activity | Claude Code | Codex | Gemini CLI | Kiro CLI | Kimi Code | Factory Droid | Hermes | Pi Agent | OpenClaw | Devin | Cursor | OpenCode | Aider | GitHub Copilot |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Read file | ✅ Read | ✅ read_file | ✅ read_file | ✅ read | ✅ ReadFile | ✅ Read | ✅ read_file | ✅ read_file | ✅ read | ✅ Read | ✅ Read files | ✅ read | ✅ (chat) | ✅ read_file |
| Read multiple files | ✅ Read (loop) | ✅ multi_tool_use | ✅ read_many_files | ✅ read | ✅ ReadFile | ✅ Read | ✅ read_file | ✅ read_file | ✅ read | ✅ Read | ✅ Read files | ✅ read | ✅ | ✅ |
| Read images | ✅ Read | ⚠️ | ✅ read_file | ✅ read | ✅ ReadMediaFile | ❌ | ✅ vision_analyze | ✅ | ❓ | ✅ | ✅ | ❓ | ❌ | ✅ |
| Read PDF | ✅ Read | ❌ | ✅ read_file | ✅ read | ✅ ReadFile | ❌ | ✅ | ✅ | ❓ | ✅ | ❌ | ❓ | ❌ | ❌ |
| Read video/audio | ❌ | ❌ | ✅ read_file | ❌ | ✅ ReadMediaFile | ❌ | ✅ video_analyze | ❌ | ❓ | ❓ | ❌ | ❓ | ❌ | ❌ |
| Write/create file | ✅ Write | ✅ write_file | ✅ write_file | ✅ write | ✅ WriteFile | ✅ Create | ✅ write_file | ✅ write_file | ✅ write | ✅ Write | ✅ Edit files | ✅ write | ✅ (diff) | ✅ create_file |
| Edit/patch file | ✅ Edit/MultiEdit | ✅ apply_patch | ✅ replace | ✅ write | ✅ StrReplaceFile | ✅ Edit/ApplyPatch | ✅ patch | ✅ edit_file | ✅ edit | ✅ Edit | ✅ Edit files | ✅ edit | ✅ (diff) | ✅ insert_edit |
| List directory | ✅ LS | ✅ read_file | ✅ list_directory | ✅ read (dirs) | ✅ Glob | ✅ LS | ✅ search_files | ✅ ls | ✅ ls | ✅ LS | ✅ Search | ✅ ls | ❌ | ✅ list_directory |
| Pattern match files | ✅ Glob | ❌ | ✅ glob | ✅ glob | ✅ Glob | ✅ Glob | ✅ search_files | ✅ glob | ✅ glob | ✅ Glob | ✅ Search files | ✅ glob | ❌ | ✅ |
| Search file content | ✅ Grep | ❌ | ✅ grep_search | ✅ grep | ✅ Grep | ✅ Grep | ✅ search_files | ✅ grep | ✅ grep | ✅ Grep | ✅ Semantic search | ✅ grep | ❌ | ✅ |
| Apply unified diff | ❌ | ✅ apply_patch | ❌ | ❌ | ❌ | ✅ ApplyPatch | ✅ patch | ❌ | ✅ apply_patch | ❌ | ❌ | ❌ | ✅ (native) | ❌ |
| Watch files (auto-rebuild) | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ⚠️ --watch-files | ❌ |

---

## Shell & Execution

| Activity | Claude Code | Codex | Gemini CLI | Kiro CLI | Kimi Code | Factory Droid | Hermes | Pi Agent | OpenClaw | Devin | Cursor | OpenCode | Aider | GitHub Copilot |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Run shell command | ✅ Bash | ✅ shell | ✅ run_shell_command | ✅ shell | ✅ Shell | ✅ Execute | ✅ terminal | ✅ bash | ✅ exec | ✅ Bash | ✅ Run shell | ✅ bash | ⚠️ via lint/test | ✅ run_in_terminal |
| Long-lived / PTY session | ❌ | ✅ exec_command | ❌ | ❌ | ❌ | ❌ | ✅ process | ❌ | ✅ process | ❌ | ❌ | ❌ | ❌ | ❌ |
| Feed stdin to process | ❌ | ✅ write_stdin | ❌ | ❌ | ❌ | ❌ | ✅ process | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Manage background process | ❌ | ❌ | ✅ list_background_processes | ❌ | ✅ TaskList/TaskStop | ❌ | ✅ process | ❌ | ✅ process | ❌ | ❌ | ❌ | ❌ | ❌ |
| Execute code (sandbox) | ❌ | ⚠️ code_interpreter | ❌ | ❌ | ❌ | ❌ | ✅ execute_code | ❌ | ✅ code_execution | ❌ | ❌ | ❌ | ❌ | ❌ |
| Run tests | ⚠️ via Bash | ⚠️ via shell | ⚠️ via shell | ⚠️ via shell | ⚠️ via Shell | ⚠️ via Execute | ⚠️ via terminal | ⚠️ via bash | ⚠️ | ✅ run_tests | ✅ run_tests | ⚠️ | ✅ --test-cmd | ✅ run_tests |
| Scheduled/cron tasks | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ cronjob | ❌ | ✅ cron | ❌ | ❌ | ❌ | ❌ | ❌ |
| AWS CLI calls | ❌ | ❌ | ❌ | ✅ aws | ❌ | ❌ | 🔌 MCP | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |

---

## Web & Network

| Activity | Claude Code | Codex | Gemini CLI | Kiro CLI | Kimi Code | Factory Droid | Hermes | Pi Agent | OpenClaw | Devin | Cursor | OpenCode | Aider | GitHub Copilot |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Web search | ✅ WebSearch | ✅ web_search | ✅ google_web_search | ✅ web_search | ✅ SearchWeb | ✅ WebSearch | ✅ web_search | ✅ fetch | ✅ web_search | ✅ WebSearch | ✅ Web | ✅ fetch | ❌ | ✅ web_search |
| Fetch URL content | ✅ WebFetch | ❌ | ✅ web_fetch | ✅ web_fetch | ✅ FetchURL | ✅ FetchUrl | ✅ web_extract | ✅ fetch | ✅ web_fetch | ✅ WebFetch | ❌ | ✅ fetch | ❌ | ❌ |
| Search X/Twitter | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ x_search | ❌ | ✅ x_search | ❌ | ❌ | ❌ | ❌ | ❌ |

---

## Browser Automation

| Activity | Claude Code | Codex | Gemini CLI | Kiro CLI | Kimi Code | Factory Droid | Hermes | Pi Agent | OpenClaw | Devin | Cursor | OpenCode | Aider | GitHub Copilot |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Navigate to URL | ❌ | ⚠️ browser | ❌ | ❌ | ❌ | ❌ | ✅ browser_navigate | ✅ browser | ✅ browser | ✅ | ✅ Browser | ❌ | ❌ | ❌ |
| Screenshot / DOM snapshot | ❌ | ⚠️ browser | ❌ | ❌ | ❌ | ❌ | ✅ browser_snapshot | ✅ browser | ✅ browser | ✅ | ✅ Browser | ❌ | ❌ | ❌ |
| Click / interact with elements | ❌ | ⚠️ browser | ❌ | ❌ | ❌ | ❌ | ✅ browser_click | ✅ browser | ✅ browser | ✅ | ✅ Browser | ❌ | ❌ | ❌ |
| Type into form fields | ❌ | ⚠️ browser | ❌ | ❌ | ❌ | ❌ | ✅ browser_type | ✅ browser | ✅ browser | ✅ | ✅ Browser | ❌ | ❌ | ❌ |
| Get JS console output | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ browser_console | ✅ browser | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Raw CDP / DevTools | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ browser_cdp | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Vision analysis of page | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ browser_vision | ✅ browser | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |

---

## Media Generation & Analysis

| Activity | Claude Code | Codex | Gemini CLI | Kiro CLI | Kimi Code | Factory Droid | Hermes | Pi Agent | OpenClaw | Devin | Cursor | OpenCode | Aider | GitHub Copilot |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Image generation | ❌ | ✅ image_generation | ❌ | ❌ | ❌ | ❌ | ✅ image_generate | ❌ | ✅ image_generate | ❌ | ✅ Image generation | ❌ | ❌ | ❌ |
| Image analysis (vision) | ✅ Read (vision) | ❌ | ✅ read_file | ✅ read | ✅ ReadMediaFile | ❌ | ✅ vision_analyze | ✅ browser | ✅ image | ✅ | ✅ Read files | ❌ | ❌ | ❌ |
| Video generation | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ video_generate | ❌ | ✅ video_generate | ❌ | ❌ | ❌ | ❌ | ❌ |
| Video analysis | ❌ | ❌ | ❌ | ❌ | ✅ ReadMediaFile | ❌ | ✅ video_analyze | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Text-to-speech | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ text_to_speech | ❌ | ✅ tts | ❌ | ❌ | ❌ | ❌ | ❌ |
| Music generation | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ music_generate | ❌ | ❌ | ❌ | ❌ | ❌ |

---

## Code Intelligence

| Activity | Claude Code | Codex | Gemini CLI | Kiro CLI | Kimi Code | Factory Droid | Hermes | Pi Agent | OpenClaw | Devin | Cursor | OpenCode | Aider | GitHub Copilot |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| LSP / symbol lookup | ❌ | ❌ | ❌ | ✅ code | ❌ | ❌ | ❌ | ✅ lsp | ❌ | ✅ | ✅ Semantic search | ❌ | ❌ | ✅ get_errors |
| Semantic codebase search | ❌ | ❌ | ❌ | ✅ code | ❌ | ❌ | ❌ | ❌ | ✅ tool_search_code | ✅ | ✅ Semantic search | ❌ | ❌ | ✅ |
| Get editor diagnostics | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ get_errors |
| Repo map / graph | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ (built-in) | ❌ |

---

## Agent Coordination

| Activity | Claude Code | Codex | Gemini CLI | Kiro CLI | Kimi Code | Factory Droid | Hermes | Pi Agent | OpenClaw | Devin | Cursor | OpenCode | Aider | GitHub Copilot |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Spawn subagent | ✅ Agent | ✅ multi_tool_use | ✅ invoke_agent | ✅ subagent / delegate | ✅ Agent | ✅ (Task tool) | ✅ delegate_task | ⚠️ Via tmux/extension | ✅ subagents | ❓ | ✅ subagentStart | ❌ | ❌ | ❌ |
| Parallel tool calls | ✅ | ✅ multi_tool_use.parallel | ✅ | ✅ delegate | ✅ | ✅ | ✅ | ⚠️ Via extension | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| On-demand tool loading | ❌ | ❌ | ❌ | ✅ tool_search | ❌ | ❌ | ❌ | ❌ | ✅ tool_search | ❌ | ❌ | ❌ | ❌ | ❌ |
| Planning mode | ✅ (via prompt) | ❌ | ❌ | ❌ | ✅ EnterPlanMode | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Ask user clarification | ✅ AskUserQuestion | ❌ | ✅ ask_user | ❌ | ✅ AskUserQuestion | ❌ | ✅ clarify | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Task / todo list | ✅ TodoRead/Write | ✅ update_plan | ❌ | ✅ todo | ✅ SetTodoList | ✅ TodoWrite | ✅ todo | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Session-persistent memory | ❌ | ❌ | ✅ save_memory | ✅ knowledge | ❌ | ❌ | ✅ memory | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Skill activation | ✅ Skill tool | ✅ skills | ✅ activate_skill | ❌ | ✅ skills | ❌ | ✅ skill_manage | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |

---

## Messaging & Integrations

| Activity | Claude Code | Codex | Gemini CLI | Kiro CLI | Kimi Code | Factory Droid | Hermes | Pi Agent | OpenClaw | Devin | Cursor | OpenCode | Aider | GitHub Copilot |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Send Slack message | 🔌 MCP | 🔌 MCP | 🔌 MCP | 🔌 MCP | 🔌 MCP | 🔌 MCP | ✅ send_message | 🔌 MCP | ✅ message | 🔌 MCP | 🔌 MCP | 🔌 MCP | ❌ | 🔌 MCP |
| Discord management | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ discord | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Spotify control | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ spotify_* | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Home Assistant | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ ha_* | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Feishu/Lark docs | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ feishu_* | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Kanban board | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ kanban_* | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Multiple LLM routing | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ mixture_of_agents | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Delayed message (checkpoint) | ❌ | ❌ | ❌ | ❌ | ✅ SendDMail | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |

---

## Desktop / Computer Control

| Activity | Claude Code | Codex | Gemini CLI | Kiro CLI | Kimi Code | Factory Droid | Hermes | Pi Agent | OpenClaw | Devin | Cursor | OpenCode | Aider | GitHub Copilot |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Desktop / UI automation | ❌ | ⚠️ computer_use | ❌ | ❌ | ❌ | ❌ | ✅ computer_use | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |

---

## Notebook & Documents

| Activity | Claude Code | Codex | Gemini CLI | Kiro CLI | Kimi Code | Factory Droid | Hermes | Pi Agent | OpenClaw | Devin | Cursor | OpenCode | Aider | GitHub Copilot |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Read Jupyter notebook | ✅ NotebookRead | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Edit Jupyter notebook | ✅ NotebookEdit | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Google Workspace (Docs/Sheets) | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |

---

## Hooks by Activity

Which hook events fire for each agent activity:

| Activity | Claude Code | Codex | Gemini CLI | Kiro CLI | Kimi Code | Factory Droid |
|---|---|---|---|---|---|---|
| Any tool use | PreToolUse / PostToolUse | PreToolUse / PostToolUse | BeforeTool / AfterTool | PreToolUse / PostToolUse | PreToolUse / PostToolUse | PreToolUse / PostToolUse |
| Shell command specifically | PreToolUse(Bash) | PreToolUse(shell) | BeforeTool(run_shell_command) / beforeShellExecution | PreToolUse(shell) | PreToolUse(Shell) | PreToolUse(Execute) |
| File read specifically | PreToolUse(Read) | PreToolUse(read_file) | BeforeTool(read_file) / beforeReadFile | PreToolUse(read) | PreToolUse(ReadFile) | PreToolUse(Read) |
| File write/edit | PreToolUse(Write/Edit) | PreToolUse(write_file) | BeforeTool(write_file) / afterFileEdit | PreToolUse(write) | PreToolUse(WriteFile) | PreToolUse(Create/Edit) |
| MCP tool use | PreToolUse | PreToolUse | BeforeMCPExecution | PreToolUse(@mcp) | PreToolUse | PreToolUse(mcp__*) |
| Prompt submitted | — | UserPromptSubmit | BeforeAgent | UserPromptSubmit | UserPromptSubmit | UserPromptSubmit |
| LLM call | — | — | BeforeModel / AfterModel | — | — | — |
| Tool selection | — | — | BeforeToolSelection | — | — | — |
| Context compacted | PreCompact | PreCompact | PreCompress | — | PreCompact/PostCompact | PreCompact |
| Agent turn ends | Stop | Stop | AfterAgent | Stop | Stop | Stop |
| Subagent spawned | — | SubagentStart | — | — | SubagentStart | — |
| Subagent ends | SubagentStop | SubagentStop | — | — | SubagentStop | SubagentStop |
| Session starts | — | SessionStart | SessionStart | AgentSpawn | SessionStart | SessionStart |
| Session ends | — | — | SessionEnd | — | SessionEnd | SessionEnd |
| File saved (IDE) | — | — | — | File Save (IDE) | — | — |
| File created (IDE) | — | — | — | File Create (IDE) | — | — |
| File deleted (IDE) | — | — | — | File Delete (IDE) | — | — |
| Spec task starts | — | — | — | Pre Task Execution (IDE) | — | — |
| Tab edit (Cursor) | — | — | — | — | — | — |
| Workspace opens | — | — | — | — | — | — |

### Blocking Capability Notes

Not all hooks listed above can block execution. Post-tool hooks that **can** block:

| Agent | Post-tool hook | Can Block | Notes |
|-------|---------------|-----------|-------|
| Claude Code | PostToolUse | ❌ | Advisory only |
| Codex CLI | PostToolUse | ✅ | Can replace tool result |
| Gemini CLI | AfterTool | ✅ | exit 2 or `decision: deny` |
| Kiro CLI | PostToolUse | ❌ | Advisory only |
| Kimi Code | PostToolUse | ❌ | Advisory only |
| Factory Droid | PostToolUse | ⚠️ Partial | stderr shown to Droid; tool already ran |
| OpenClaw | after_tool_call | ❌ | Inspect only (gateway plugin hook) |

All Pre/Before tool hooks support blocking via exit code 2 (or equivalent) for all agents listed in this matrix.

## Sources (Official)

Activity capability data sourced from official docs for each tool:

| Tool | Primary source | Notes |
|------|---------------|-------|
| Claude Code | https://docs.anthropic.com/en/docs/claude-code/tools | |
| Codex CLI | https://developers.openai.com/codex/tools | |
| Gemini CLI | https://geminicli.com/docs/tools | |
| Kiro | https://kiro.dev/docs/tools | Amazon/AWS product (not an independent vendor) |
| Kimi Code | https://moonshotai.github.io/kimi-code/tools | |
| Factory Droid | https://docs.factory.ai/tools | |
| Hermes | https://hermes-agent.nousresearch.com/docs/reference/tools-reference | |
| Cursor | https://cursor.com/docs/agent/tools | |
| OpenClaw | https://docs.openclaw.ai/tools | Gateway/routing product, not a standalone coding agent; capabilities exposed through gateway-attached agents |
| Pi Agent | https://github.com/earendil-works/pi | MCP and subagent support are not built-in; require `pi-yaml-hooks` package or TypeScript extensions |
| Google Antigravity | https://antigravity.google/docs | Generally Available (GA), not Preview |
| GitHub Copilot | https://code.visualstudio.com/docs/copilot/agents/overview | |
