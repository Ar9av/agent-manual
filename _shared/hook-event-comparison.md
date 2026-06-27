# Hook Event Cross-Tool Comparison

A unified view of lifecycle hook events across all major agentic tools.

## Event Matrix

| Hook Event | Claude Code | Codex CLI | Gemini CLI | Kiro | Kimi Code | Factory Droid | Hermes | Pi Agent | OpenClaw | Devin CLI | Cursor |
|---|---|---|---|---|---|---|---|---|---|---|---|
| **Pre Tool Use** | `PreToolUse` | `PreToolUse` | `BeforeTool` | `preToolUse` | `PreToolUse` | `PreToolUse` | `pre_tool_call` | `tool.before.*` | `preToolUse` | `PreToolUse` | `preToolUse` |
| **Post Tool Use** | `PostToolUse` | `PostToolUse` | `AfterTool` | `postToolUse` | `PostToolUse` | `PostToolUse` | `post_tool_call` | `tool.after.*` | `postToolUse` | `PostToolUse` | `postToolUse` |
| **Session Start** | — | `SessionStart` | `SessionStart` | `agentSpawn` | `SessionStart` | `SessionStart` | `on_session_start` | `session.created` | — | `SessionStart` | — |
| **Session End** | — | — | `SessionEnd` | — | `SessionEnd` | `SessionEnd` | `on_session_end` | `session.deleted` | — | `SessionEnd` | — |
| **Prompt Submit** | — | `UserPromptSubmit` | — | `userPromptSubmit` | `UserPromptSubmit` | `UserPromptSubmit` | `pre_llm_call` | — | — | — | — |
| **Post LLM Response** | — | — | `AfterModel` | — | — | — | `post_llm_call` | `after_provider_response` | — | — | — |
| **Pre LLM Call** | — | — | `BeforeModel` | — | — | — | — | — | — | — | — |
| **Tool Selection** | — | — | `BeforeToolSelection` | — | — | — | — | — | — | — | — |
| **Agent Start** | — | — | `BeforeAgent` | — | — | — | — | — | — | — | — |
| **Agent End** | — | — | `AfterAgent` | — | — | — | — | — | — | — | — |
| **Context Compact** | `PreCompact` | `PreCompact` | `PreCompress` | — | `PreCompact` | `PreCompact` | — | — | — | `PostCompaction` | — |
| **Post Compact** | — | `PostCompact` | — | — | `PostCompact` | — | — | — | — | — | — |
| **Notification** | `Notification` | — | `Notification` | — | `Notification` | `Notification` | — | `notify` action | — | — | — |
| **Subagent Start** | — | `SubagentStart` | — | — | `SubagentStart` | — | — | — | — | — | — |
| **Subagent Done** | `SubagentStop` | `SubagentStop` | — | — | `SubagentStop` | `SubagentStop` | `subagent_stop` | — | — | — | — |
| **Permission Request** | `PermissionRequest` | `PermissionRequest` | — | — | `PermissionRequest` | — | — | — | — | `PermissionRequest` | — |
| **Turn End / Stop** | `Stop` | `Stop` | `AfterAgent` | `stop` | `Stop` | `Stop` | `post_llm_call` | — | — | `Stop` | `stop` |
| **File Changed** | — | — | — | — | — | — | — | `file.changed` | — | — | — |
| **Session Idle** | — | — | — | — | — | — | — | `session.idle` | — | — | — |

## OpenCode Event Matrix

OpenCode uses dot-namespaced event names in its plugin SDK:

| Hook Event | OpenCode |
|---|---|
| **Pre Tool Use** | `tool.execute.before` |
| **Post Tool Use** | `tool.execute.after` |
| **Session Start** | `session.created` |
| **Session Idle** | `session.idle` |
| **Session End** | `session.deleted` |
| **Context Compact** | `session.compacted` |

## Can-Block Comparison

| Tool | Block Mechanism | How to Block |
|------|----------------|-------------|
| Claude Code | Exit code `2` in `PreToolUse` | `exit 2` + message on stderr |
| Codex CLI | Exit code `2` in `PreToolUse` | `exit 2` (or stdout JSON `"permissionDecision": "deny"`) |
| Gemini CLI | Exit code `2` in `BeforeTool` | `exit 2` + message on stderr |
| Kiro | Exit code `2` in `preToolUse` | `exit 2` |
| Kimi Code | Exit code `2` in `PreToolUse` | `exit 2` |
| Factory Droid | Exit code `2` in `PreToolUse` | `exit 2` |
| Hermes | Return value from `pre_tool_call` | `{ block: true }` |
| Pi Agent | Exit code `2` in `tool.before.*` | `exit 2` |
| OpenClaw | Plugin SDK `before_tool_call` return | Synchronous block decision / throw |
| Devin CLI | Exit code `2` in `PreToolUse` | `exit 2` |
| Cursor | Exit code `2` in `preToolUse` | `exit 2` |
| Aider | ❌ No hook system | N/A |
| Trae | ❌ No hook system | N/A (use MCP) |
| OpenCode | Plugin return value from `tool.execute.before` | `{ block: true }` |
| GitHub Copilot | stdout JSON in `preToolUse` | `{"permissionDecision": "deny"}` |
| Google Antigravity | Exit code `2` or `{"decision":"block"}` stdout | `exit 2` or return JSON block |

## Hook Input Format Comparison

| Tool | Format | Delivery |
|------|--------|---------|
| Claude Code | JSON | stdin |
| Codex CLI | JSON | stdin |
| Gemini CLI | JSON | stdin |
| Kiro | JSON | stdin |
| Kimi Code | JSON | stdin |
| Factory Droid | JSON | stdin |
| Hermes | YAML config → subprocess | stdin |
| Pi Agent | JSON context | stdin / template vars |
| OpenClaw | TypeScript plugin SDK | in-process event object |
| Devin CLI | JSON | stdin |
| Google Antigravity | JSON | stdin |

## Config File Format Comparison

| Tool | Format | Primary Hook File |
|------|--------|------------------|
| Claude Code | JSON | `settings.json` |
| Codex CLI | TOML | `config.toml` |
| Gemini CLI | JSON | `settings.json` |
| Kiro | YAML | `config.yaml` / agent YAML |
| Kimi Code | TOML | `~/.kimi-code/config.toml` |
| Factory Droid | JSON | `settings.json` |
| Hermes | YAML | `~/.hermes/config.yaml` |
| Pi Agent | YAML | `hooks.yaml` (via package) |
| OpenClaw | YAML | `config.yaml` |
| Devin CLI | JSON | `hooks.v1.json` |
| Cursor | JSON | `settings.json` |
| Aider | YAML | `.aider.conf.yml` (no hooks) |
| Trae | Markdown | `project_rules.md` (no hooks) |
| OpenCode | JSON + JS | `opencode.json` + plugin files |
| Google Antigravity | JSON | `hooks.json` |

## Tool Instruction Files

Files you place in a repo to give the agent persistent natural-language instructions:

| Tool | File |
|------|------|
| Claude Code | `CLAUDE.md` |
| Codex CLI | `AGENTS.md` |
| OpenClaw | `AGENTS.md` |
| Hermes | `AGENTS.md` |
| Trae | `project_rules.md` / `user_rules.md` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Cursor | `.cursor/rules/*.mdc` |
| Gemini CLI | `.gemini/GEMINI.md` |
| Kiro | `.kiro/agents/*.yaml` (system prompt) |
| Aider | `CONVENTIONS.md` |
| Google Antigravity | `.agents/rules/` and `.agents/workflows/` |

## Sources

| Tool | Hook docs URL | Fetched | Label |
|------|--------------|---------|-------|
| Claude Code | https://docs.anthropic.com/en/docs/claude-code/hooks | 2026-06-26 | [official] |
| Codex CLI | https://developers.openai.com/codex/hooks | 2026-06-26 | [official] |
| Gemini CLI | https://geminicli.com/docs/hooks | 2026-06-26 | [official] |
| Kiro | https://kiro.dev/docs/hooks | 2026-06-26 | [official] |
| Kimi Code | https://moonshotai.github.io/kimi-code/hooks | 2026-06-26 | [official mirror] |
| Factory Droid | https://docs.factory.ai/hooks | 2026-06-26 | [official] |
| Hermes | https://hermes-agent.nousresearch.com/docs/hooks | 2026-06-26 | [official] |
| Cursor | https://cursor.com/docs/hooks | 2026-06-26 | [official] |
| Pi Agent | https://github.com/can1357/oh-my-pi/blob/main/docs/hooks.md | 2026-06-26 | [community] |
| OpenClaw | https://docs.openclaw.ai/automation/hooks | 2026-06-26 | [official] |
| Google Antigravity | https://antigravity.google/docs/hooks | 2026-06-26 | [official] |
