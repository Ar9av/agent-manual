# Hook Event Cross-Tool Comparison

A unified view of lifecycle hook events across all major agentic tools.

## Event Matrix

| Hook Event | Claude Code | Codex CLI | Gemini CLI | Kiro | Kimi Code | Factory Droid | Hermes | Pi Agent | OpenClaw | Devin CLI | Cursor | Amazon Q Dev CLI | Goose | OpenHands | Continue CLI | Auggie CLI | Qwen Code | Crush | Cline | Junie | Grok Build | jcode | Muse Code |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **Pre Tool Use** | `PreToolUse` | `PreToolUse` | `BeforeTool` | `preToolUse` | `PreToolUse` | `PreToolUse` | `pre_tool_call` | `tool.before.*` | `preToolUse` | `PreToolUse` | `preToolUse` | `preToolUse` | `PreToolUse` | `PreToolUse` | `PreToolUse` | `PreToolUse` | `PreToolUse` | `PreToolUse` | `PreToolUse` | `PreToolUse` | `PreToolUse` | `pre_tool` | `PreToolUse` |
| **Post Tool Use** | `PostToolUse` | `PostToolUse` | `AfterTool` | `postToolUse` | `PostToolUse` | `PostToolUse` | `post_tool_call` | `tool.after.*` | `postToolUse` | `PostToolUse` | `postToolUse` | `postToolUse` | `PostToolUse` | `PostToolUse` | `PostToolUse` | `PostToolUse` | `PostToolUse` | — | `PostToolUse` | — | `PostToolUse` | `post_tool` | `PostToolUse` |
| **Session Start** | — | `SessionStart` | `SessionStart` | `agentSpawn` | `SessionStart` | `SessionStart` | `on_session_start` | `session.created` | — | `SessionStart` | — | `agentSpawn` | `SessionStart` | `SessionStart` | `SessionStart` | `SessionStart` | `SessionStart` | — | — | `SessionStart` | `SessionStart` | `session_start` | `SessionStart` |
| **Session End** | — | — | `SessionEnd` | — | `SessionEnd` | `SessionEnd` | `on_session_end` | `session.deleted` | — | `SessionEnd` | — | — | `SessionEnd` | `SessionEnd` | `SessionEnd` | `SessionEnd` | `SessionEnd` | — | — | `SessionEnd` | `SessionEnd` | `session_end` | — |
| **Prompt Submit** | — | `UserPromptSubmit` | — | `userPromptSubmit` | `UserPromptSubmit` | `UserPromptSubmit` | `pre_llm_call` | — | — | — | — | `userPromptSubmit` | `UserPromptSubmit` | `UserPromptSubmit` | `UserPromptSubmit` | — | `UserPromptSubmit` | — | `UserPromptSubmit` | `UserPromptSubmit` | `UserPromptSubmit` | — | `UserPromptSubmit` |
| **Post LLM Response** | — | — | `AfterModel` | — | — | — | `post_llm_call` | `after_provider_response` | — | — | — | — | — | — | — | — | — | — | — | — | — | — | `PostLLMCall` |
| **Pre LLM Call** | — | — | `BeforeModel` | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | `PreLLMCall` |
| **Tool Selection** | — | — | `BeforeToolSelection` | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — |
| **Agent Start** | — | — | `BeforeAgent` | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — |
| **Agent End** | — | — | `AfterAgent` | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — |
| **Context Compact** | `PreCompact` | `PreCompact` | `PreCompress` | — | `PreCompact` | `PreCompact` | — | — | — | `PostCompaction` | — | — | — | — | `PreCompact` | — | `PreCompact` | — | `PreCompact`(❓) | — | `PreCompact` | — | `PreCompact` |
| **Post Compact** | — | `PostCompact` | — | — | `PostCompact` | — | — | — | — | — | — | — | — | — | `PostCompact`(❓) | — | `PostCompact` | — | — | — | `PostCompact` | — | `PostCompact` |
| **Notification** | `Notification` | — | `Notification` | — | `Notification` | `Notification` | — | `notify` action | — | — | — | — | — | — | `Notification`(❓) | — | `Notification` | — | — | — | `Notification` | — | — |
| **Subagent Start** | — | `SubagentStart` | — | — | `SubagentStart` | — | — | — | — | — | — | — | — | — | `SubagentStart`(❓) | — | `SubagentStart` | — | — | — | `SubagentStart` | — | `SubagentStart` |
| **Subagent Done** | `SubagentStop` | `SubagentStop` | — | — | `SubagentStop` | `SubagentStop` | `subagent_stop` | — | — | — | — | — | — | — | `SubagentStop`(❓) | — | `SubagentStop` | — | — | — | `SubagentStop` | — | `SubagentStop` |
| **Permission Request** | `PermissionRequest` | `PermissionRequest` | — | — | `PermissionRequest` | — | — | — | — | `PermissionRequest` | — | — | — | — | `PermissionRequest`(❓) | — | `PermissionRequest` | — | — | `PermissionRequest` | `PermissionDenied` | — | `PermissionRequest` |
| **Turn End / Stop** | `Stop` | `Stop` | `AfterAgent` | `stop` | `Stop` | `Stop` | `post_llm_call` | — | — | `Stop` | `stop` | `stop` | `Stop` | `Stop` | `Stop` | `Stop` | `Stop` | — | `TaskComplete`(❓) | `Stop` | `Stop` | `turn_end` | `Stop` |
| **File Changed** | — | — | — | — | — | — | — | `file.changed` | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — |
| **Session Idle** | — | — | — | — | — | — | — | `session.idle` | — | — | — | — | — | — | `TeammateIdle`(❓) | — | — | — | — | — | — | — | — |
| **Post Tool Use Failure** | — | — | — | — | — | — | — | — | — | — | — | — | `PostToolUseFailure` | — | `PostToolUseFailure`(❓) | — | `PostToolUseFailure`(❓) | — | — | — | `PostToolUseFailure` | — | — |
| **Before Read File** | — | — | — | — | — | — | — | — | — | — | — | — | `BeforeReadFile` | — | — | — | — | — | — | — | — | — | — |
| **After File Edit** | — | — | — | — | — | — | — | — | — | — | — | — | `AfterFileEdit` | — | — | — | — | — | — | — | — | — | — |
| **Before Shell Exec** | — | — | — | — | — | — | — | — | — | — | — | — | `BeforeShellExecution` | — | — | — | — | — | — | — | — | — | — |
| **After Shell Exec** | — | — | — | — | — | — | — | — | — | — | — | — | `AfterShellExecution` | — | — | — | — | — | — | — | — | — | — |
| **Todo Created/Completed** | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | — | `TodoCreated` / `TodoCompleted` | — | — | — | — | — | — |

> Amp is excluded from this event-name matrix — it has no declarative hooks table, using a TypeScript Plugin API with lifecycle callbacks (`session.start`, `tool.call`, `tool.result`, `agent.start`, `agent.end`) instead. Warp is excluded — no documented hook/lifecycle-event system was found. Continue CLI's event list is not yet published on its official docs site (three open GitHub issues track the gap); it was reconstructed from CLI source (`[github]`-labeled), so entries marked (❓) are inferred from source rather than confirmed in docs.

## OpenCode Event Matrix

OpenCode uses dot-namespaced event names in its plugin SDK:

| Hook Event | OpenCode | Cline | Junie | Grok Build | jcode | Muse Code |
|---|---|---|---|---|---|---|
| **Pre Tool Use** | `tool.execute.before` | `PreToolUse` | `PreToolUse` | `PreToolUse` | `pre_tool` | `PreToolUse` |
| **Post Tool Use** | `tool.execute.after` | `PostToolUse` | — | `PostToolUse` | `post_tool` | `PostToolUse` |
| **Session Start** | `session.created` | — | `SessionStart` | `SessionStart` | `session_start` | `SessionStart` |
| **Session Idle** | `session.idle` | — | — | — | — | — |
| **Session End** | `session.deleted` | — | `SessionEnd` | `SessionEnd` | `session_end` | — |
| **Context Compact** | `session.compacted` | `PreCompact`(❓) | — | `PreCompact` | — | `PreCompact` |

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
| Amazon Q Developer CLI | Exit code `2` in `preToolUse` | `exit 2` + stderr message (no structured JSON decision) |
| Amp | Plugin API `tool.call` return action | Return `{ action: "reject" }` from TS plugin |
| Goose | Exit code `2` in `PreToolUse`/`Stop` | `exit 2` (only these two events can block) |
| OpenHands | Exit code `2` in `PreToolUse`/`UserPromptSubmit`/`Stop` | `exit 2` |
| Continue CLI | Exit code `2` (multiple events, per source) | `exit 2` |
| Auggie CLI | Exit code `2` in `PreToolUse`/`Stop` | `exit 2` |
| Qwen Code | stdout JSON `permissionDecision` in `PreToolUse` | `{"permissionDecision": "deny"}` |
| Crush | Exit codes `2` (deny) / `49` (halt turn) in `PreToolUse` | `exit 2` or `exit 49` |
| Warp | ❌ No hook system | N/A (use Agent Profiles/Permissions instead) |

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
| Amazon Q Developer CLI | Plain text/stderr (no structured JSON) | stdin (context) / stderr (block reason) |
| Amp | TypeScript object | in-process plugin callback |
| Goose | JSON | stdin |
| OpenHands | JSON | stdin |
| Continue CLI | JSON | stdin |
| Auggie CLI | JSON | stdin |
| Qwen Code | JSON | stdin |
| Crush | JSON | stdin |

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
| Amazon Q Developer CLI | JSON | Per-agent `cli-agents/*.json` |
| Amp | TypeScript | `.amp/plugins/*.ts` |
| Goose | YAML | `config.yaml` (plugin dir for hooks) |
| OpenHands | JSON | `.openhands/hooks.json` |
| Continue CLI | JSON | `settings.json` |
| Auggie CLI | JSON | `settings.json` |
| Qwen Code | JSON | `settings.json` |
| Crush | JSON | `crush.json` |
| Warp | — | ❌ No hooks |

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
| Amazon Q Developer CLI | `AmazonQ.md` (also `.amazonq/rules/**/*.md`) |
| Amp | `AGENTS.md` (also `AGENT.md`, `CLAUDE.md`) |
| Goose | `.goosehints` |
| OpenHands | `.openhands/microagents/repo.md` (repo-level microagent) |
| Continue CLI | `AGENTS.md`(❓ not explicitly confirmed for CLI) |
| Auggie CLI | `AGENTS.md` / `.augment-guidelines` / `.augment/rules/` |
| Qwen Code | `QWEN.md` |
| Warp | `WARP.md` (wins over `AGENTS.md` on conflict) |
| Crush | `AGENTS.md`(❓) |

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
| Amazon Q Developer CLI | https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-installing.html | 2026-07-23 | [official] |
| Amp | https://ampcode.com/manual/plugin-api | 2026-07-23 | [official] |
| Goose | https://goose-docs.ai/ | 2026-07-23 | [official] |
| OpenHands | https://docs.openhands.dev/ | 2026-07-23 | [official] |
| Continue CLI | `extensions/cli/src/hooks/types.ts` (continuedev/continue) | 2026-07-23 | [github] |
| Auggie CLI | https://docs.augmentcode.com/cli/hooks | 2026-07-23 | [official] |
| Qwen Code | https://qwenlm.github.io/qwen-code-docs/en/users/overview | 2026-07-23 | [official] |
| Crush | https://github.com/charmbracelet/crush | 2026-07-23 | [github] |
| Warp | https://docs.warp.dev/agent-platform/ | 2026-07-23 | [official] |
| Cline | https://docs.cline.bot | 2026-08-15 | [official] |
| Junie | https://junie.jetbrains.com/docs/ | 2026-08-15 | [official] |
| Grok Build | https://docs.x.ai/build/overview | 2026-08-15 | [official] |
| jcode | https://jcode.sh/docs | 2026-08-15 | [official] |
