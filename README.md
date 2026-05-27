# Agentic Tools Almanac

A community reference for AI coding agent configurations — lifecycle hooks (PreToolUse / PostToolUse), built-in tools, MCP support, config file paths, and skill systems — across every major agent.

> **Goal:** Be the single place developers go to understand how to configure, extend, and hook into any agentic coding tool.

---

## Tools Covered

| Tool | Vendor | Config format | Hooks | MCP | Official docs |
|------|--------|--------------|-------|-----|--------------|
| [Claude Code](tools/claude-code/) | Anthropic | JSON | ✅ Full (12 events) | ✅ | [docs.anthropic.com/claude-code](https://docs.anthropic.com/en/docs/claude-code) |
| [Codex CLI](tools/codex/) | OpenAI | TOML | ✅ Full (10 events) | ✅ | [developers.openai.com/codex](https://developers.openai.com/codex) |
| [GitHub Copilot](tools/github-copilot/) | GitHub/Microsoft | JSON | ✅ Full (12 events) | ✅ | [docs.github.com/copilot/hooks](https://docs.github.com/en/copilot/concepts/agents/hooks) |
| [Gemini CLI](tools/gemini-cli/) | Google | JSON | ✅ Full (11 events) | ✅ | [geminicli.com/docs](https://geminicli.com/docs) |
| [Factory Droid](tools/factory-droid/) | Factory.ai | JSON | ✅ Full (9 events) | ✅ | [docs.factory.ai](https://docs.factory.ai) |
| [Kiro IDE/CLI](tools/kiro/) | AWS | YAML | ✅ Full | ✅ | [kiro.dev/docs](https://kiro.dev/docs) |
| [Kimi Code](tools/kimi-code/) | Moonshot AI | TOML | ✅ Full — Beta (13 events) | ✅ | [kimi.com/code/docs](https://www.kimi.com/code/docs/en/kimi-code-cli/) |
| [Cursor](tools/cursor/) | Anysphere | JSON | ✅ Full (21 events) | ✅ | [cursor.com/docs/hooks](https://cursor.com/docs/hooks) |
| [Hermes](tools/hermes/) | NousResearch | YAML | ✅ Full | ✅ | [hermes-agent.nousresearch.com](https://hermes-agent.nousresearch.com/docs) |
| [Devin CLI](tools/devin-cli/) | Cognition AI | JSON | ✅ Full (7 events) | ✅ | [cli.devin.ai/docs/extensibility/hooks](https://cli.devin.ai/docs/extensibility/hooks/overview) |
| [Pi Coding Agent](tools/pi-agent/) | earendil-works | JSON + YAML | ✅ Full (7 events) | ✅ | [pi.dev](https://pi.dev) |
| [OpenCode](tools/opencode/) | OpenCode.ai | JSON | ✅ Via plugins | ✅ | [opencode.ai/docs](https://opencode.ai/docs) |
| [OpenClaw](tools/openclaw/) | OpenClaw | YAML | ⚠️ Session/message only — PreToolUse not shipped | ✅ | [docs.openclaw.ai](https://docs.openclaw.ai) |
| [Trae](tools/trae/) | ByteDance | Markdown rules | ⚠️ Via MCP only | ✅ | [docs.trae.ai](https://docs.trae.ai) |
| [Trae CN](tools/trae-cn/) | ByteDance | Markdown rules | ⚠️ Via MCP only | ✅ | [docs.trae.ai](https://docs.trae.ai) |
| [Aider](tools/aider/) | Aider-AI | YAML | ❌ Not shipped | ❌ | [aider.chat/docs](https://aider.chat/docs/) |
| [Google Antigravity](tools/google-antigravity/) | Google | — | ⚠️ Workflows + Rules (no PreToolUse) | ❓ | Gated / invite-only |

**Legend:** ✅ Full = shipped, documented hook system with blocking · ⚠️ = partial or workaround · ❌ = not available · ❓ = unknown

---

## Repository Structure

```
agentic-tools-almanac/
├── README.md                          # This file — master index + comparison tables
├── tools/
│   └── <tool-name>/
│       └── README.md                 # Per-tool: install, config, hooks, tools, MCP, skills
├── _shared/
│   ├── activity-agent-matrix.md      # Activity → which agents support it
│   ├── agent-tools-hooks-config.md   # Agent → complete tools/hooks/config reference
│   ├── hook-event-comparison.md      # Cross-tool hook event naming matrix
│   ├── mcp-support.md                # MCP support across tools
│   └── config-file-locations.md      # Config file + skills directory paths
└── _templates/
    └── tool-template.md              # Template for adding a new tool
```

---

## Quick Navigation

| Question | Go to |
|----------|-------|
| Which agents support activity X? | [`_shared/activity-agent-matrix.md`](_shared/activity-agent-matrix.md) |
| What are all of Agent Y's hooks/tools/config? | [`_shared/agent-tools-hooks-config.md`](_shared/agent-tools-hooks-config.md) |
| What is PreToolUse called in each tool? | [`_shared/hook-event-comparison.md`](_shared/hook-event-comparison.md) |
| Where does Tool Y store its config and skills? | [`_shared/config-file-locations.md`](_shared/config-file-locations.md) |

---

## Hook Event Cross-Reference

How the same lifecycle moment is named across tools:

| Lifecycle moment | Claude Code | Codex | Gemini CLI | GitHub Copilot | Kimi Code | Factory Droid | Cursor | Devin CLI | Pi Agent |
|---|---|---|---|---|---|---|---|---|---|
| Before any tool | `PreToolUse` | `PreToolUse` | `BeforeTool` | `preToolUse` | `PreToolUse` | `PreToolUse` | `preToolUse` | `PreToolUse` | `tool.before.*` |
| After any tool | `PostToolUse` | `PostToolUse` | `AfterTool` | `postToolUse` | `PostToolUse` | `PostToolUse` | `postToolUse` | `PostToolUse` | `tool.after.*` |
| After tool fails | `PostToolUse` | — | — | `postToolUseFailure` | `PostToolUseFailure` | — | — | — | — |
| Prompt submitted | — | `UserPromptSubmit` | `BeforeAgent` | `userPromptSubmitted` | `UserPromptSubmit` | `UserPromptSubmit` | `beforeSubmitPrompt` | `UserPromptSubmit` | — |
| Agent turn ends | `Stop` | `Stop` | `AfterAgent` | `agentStop` | `Stop` | `Stop` | `stop` | `Stop` | — |
| Subagent starts | `SubagentStart` | `SubagentStart` | — | `subagentStart` | `SubagentStart` | — | `subagentStart` | — | — |
| Session starts | — | `SessionStart` | `SessionStart` | `sessionStart` | `SessionStart` | `SessionStart` | `sessionStart` | `SessionStart` | `session.created` |
| Context compacted | `PreCompact` | `PreCompact` | `PreCompress` | `preCompact` | `PreCompact` | `PreCompact` | — | — | — |
| Permission request | — | `PermissionRequest` | — | `permissionRequest` | — | — | — | `PermissionRequest` | — |
| LLM call | — | — | `BeforeModel` / `AfterModel` | — | — | — | — | — | — |

---

## Blocking Mechanism Comparison

How each tool's `PreToolUse`-equivalent blocks a tool call:

| Tool | Block method | Stderr / reason goes to |
|------|-------------|------------------------|
| Claude Code | Exit 2 | LLM as context |
| Codex CLI | Exit 2 or `{"decision":"block"}` stdout | LLM |
| GitHub Copilot | `{"permissionDecision":"deny"}` stdout | LLM (via `permissionDecisionReason`) |
| Gemini CLI | Exit 2 | LLM |
| Factory Droid | `{"permissionDecision":"deny"}` stdout | LLM |
| Kimi Code | Exit 2 or `{"permissionDecision":"deny"}` stdout | LLM |
| Cursor | Exit 2 + `failClosed` option | LLM |
| Hermes | `{"action":"block","message":"..."}` stdout | LLM |
| Devin CLI | Exit 2 or `{"decision":"block"}` stdout | LLM |
| Kiro | Exit 2 | LLM |
| Pi Agent | Exit 2 | LLM |

---

## Source Confidence Key

Each tool README labels sources:

| Label | Meaning |
|-------|---------|
| **[official]** | Published by the vendor on their own docs site |
| **[github]** | Vendor GitHub repo (README, source code) |
| **[community]** | Community-maintained project (e.g., oh-my-pi) — may diverge from vendor |
| **[graphify-src]** | Inferred from `github.com/safishamsi/graphify` installer source code |

---

## Contributing

See [`_templates/tool-template.md`](_templates/tool-template.md) for the standard page format.

When adding or updating a tool:
1. Source everything to an official URL — no guessing.
2. Mark uncertain items with ❓.
3. Use the `[official]` / `[github]` / `[community]` labels in the Sources table.
4. Update the master table in this README.
5. Update `_shared/agent-tools-hooks-config.md` with the agent's section.
