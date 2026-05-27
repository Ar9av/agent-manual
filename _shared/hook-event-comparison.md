# Hook Event Cross-Tool Comparison

A unified view of lifecycle hook events across all major agentic tools.

## Event Matrix

| Hook Event | Claude Code | Codex CLI | Gemini CLI | Kiro | Kimi Code | Factory Droid | Hermes | Pi Agent | OpenClaw | Devin CLI | Cursor |
|---|---|---|---|---|---|---|---|---|---|---|---|
| **Pre Tool Use** | `PreToolUse` | `pre_tool` | `before_tool_call` | `preToolUse` | `PreToolUse` | `PreToolUse` | `pre_tool_call` | `tool.before.*` | `preToolUse` | `PreToolUse` | `onPreEdit` |
| **Post Tool Use** | `PostToolUse` | `post_tool` | `after_tool_call` | `postToolUse` | `PostToolUse` | `PostToolUse` | `post_tool_call` | `tool.after.*` | `postToolUse` | `PostToolUse` | `onPostEdit` |
| **Session Start** | — | `session_start` | `on_session_start` | `agentSpawn` | `AgentSpawn` | `AgentStart` | `on_session_start` | `session.created` | — | `AgentStart` | — |
| **Session End** | — | `session_end` | `on_session_end` | — | — | — | `on_session_end` | `session.deleted` | — | `AgentStop` | — |
| **Prompt Submit** | — | `pre_prompt` | — | `userPromptSubmit` | `UserPromptSubmit` | — | `pre_llm_call` | — | — | — | — |
| **Post LLM Response** | — | `post_response` | — | — | — | — | `post_llm_call` | `after_provider_response` | — | — | — |
| **Pre Commit** | — | — | — | — | — | — | — | — | — | — | `onPreCommit` |
| **Diff Approved** | — | — | — | — | — | — | — | — | — | — | `onApprove` |
| **Context Compact** | `PreCompact` | — | — | — | — | — | — | — | — | — | — |
| **Notification** | `Notification` | — | — | — | — | — | — | `notify` action | — | — | — |
| **Subagent Done** | `SubagentStop` | — | — | — | — | — | `on_subagent_complete` | — | — | — | — |
| **File Changed** | — | — | — | — | — | — | — | `file.changed` | — | — | — |
| **Session Idle** | — | — | — | — | — | — | — | `session.idle` | — | — | — |

## Can-Block Comparison

| Tool | Block Mechanism | How to Block |
|------|----------------|-------------|
| Claude Code | Exit code `2` in `PreToolUse` | `exit 2` + message on stderr |
| Codex CLI | Non-zero exit in `pre_tool` | `exit 1` |
| Gemini CLI | Non-zero exit in `before_tool_call` | `exit 1` |
| Kiro | Exit code `2` in `preToolUse` | `exit 2` |
| Kimi Code | Exit code `2` in `PreToolUse` | `exit 2` |
| Factory Droid | Exit code `2` in `PreToolUse` | `exit 2` |
| Hermes | Return value from `pre_tool_call` | `{ block: true }` |
| Pi Agent | Exit code `2` in `tool.before.*` | `exit 2` |
| OpenClaw | Non-zero exit in `preToolUse` | `exit 1` |
| Devin CLI | Exit code `2` in `PreToolUse` | `exit 2` |
| Cursor | Non-zero exit in `onPreEdit` | `exit 1` |
| Aider | ❌ No hook system | N/A |
| Trae | ❌ No hook system | N/A (use MCP) |
| OpenCode | Plugin return value | `{ block: true }` |
| GitHub Copilot | Plugin return value | `{ block: true }` |

## Hook Input Format Comparison

| Tool | Format | Delivery |
|------|--------|---------|
| Claude Code | JSON | stdin |
| Codex CLI | JSON env vars | `CODEX_TOOL_NAME`, `CODEX_TOOL_INPUT` |
| Gemini CLI | JSON | stdin |
| Kiro | JSON | stdin |
| Kimi Code | JSON | stdin |
| Factory Droid | JSON | stdin |
| Hermes | YAML config → subprocess | stdin |
| Pi Agent | JSON context | stdin / template vars |
| OpenClaw | JSON env vars | `HOOK_TOOL_NAME`, `HOOK_TOOL_INPUT`, `HOOK_EVENT` |
| Devin CLI | JSON | stdin |

## Config File Format Comparison

| Tool | Format | Primary Hook File |
|------|--------|------------------|
| Claude Code | JSON | `settings.json` |
| Codex CLI | TOML | `config.toml` |
| Gemini CLI | JSON | `settings.json` |
| Kiro | YAML | `config.yaml` / agent YAML |
| Kimi Code | JSON | `config.json` |
| Factory Droid | JSON | `settings.json` |
| Hermes | YAML | `cli-config.yaml` |
| Pi Agent | YAML | `hooks.yaml` (via package) |
| OpenClaw | YAML | `config.yaml` |
| Devin CLI | JSON | `hooks.v1.json` |
| Cursor | JSON | `settings.json` |
| Aider | YAML | `.aider.conf.yml` (no hooks) |
| Trae | Markdown | `project_rules.md` (no hooks) |
| OpenCode | JSON + JS | `opencode.json` + plugin files |

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

## Sources (Official)

| Tool | Hook docs URL |
|------|--------------|
| Claude Code | https://docs.anthropic.com/en/docs/claude-code/hooks |
| Codex CLI | https://developers.openai.com/codex/hooks |
| Gemini CLI | https://geminicli.com/docs/hooks |
| Kiro | https://kiro.dev/docs/hooks |
| Kimi Code | https://moonshotai.github.io/kimi-code/hooks |
| Factory Droid | https://docs.factory.ai/hooks |
| Hermes | https://hermes-agent.nousresearch.com/docs/hooks |
| Cursor | https://cursor.com/docs/hooks |
| Pi Agent | https://github.com/can1357/oh-my-pi/blob/main/docs/hooks.md |
| OpenClaw | https://docs.openclaw.ai/automation/hooks |
