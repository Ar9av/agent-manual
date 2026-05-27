<p align="center">
  <img src="https://img.shields.io/badge/Agentic%20Tools%20Almanac-%E2%9C%A7-blueviolet?style=for-the-badge" alt="Agentic Tools Almanac">
  <a href="_shared/agent-tools-hooks-config.md"><img src="https://img.shields.io/badge/Config%20Reference-Shared-FFD700?style=for-the-badge" alt="Config Reference"></a>
  <a href="_shared/hook-event-comparison.md"><img src="https://img.shields.io/badge/Hooks%20Matrix-Cross%20Tool-5865F2?style=for-the-badge" alt="Hooks Matrix"></a>
  <a href="https://github.com/Ar9av/agent-manual"><img src="https://img.shields.io/badge/Repo-GitHub-green?style=for-the-badge" alt="GitHub Repo"></a>
</p>

# Agentic Tools Almanac ☤

**The ultimate developer's index for AI coding agents.** A community-driven reference for configuring, extending, and intercepting every major agentic development tool. Easily compare lifecycle hooks (e.g., `PreToolUse` and `PostToolUse`), built-in capabilities, Model Context Protocol (MCP) servers, system instruction file paths, and customized skills across 17+ different developer agents.

<table>
<tr><td><b>Universal Hooks Index</b></td><td>Standardized naming, exit-code blocking rules, and in-depth event execution behaviors for intercepting actions.</td></tr>
<tr><td><b>Comprehensive Matrix</b></td><td>Side-by-side comparisons of file reads, shell commands, web searches, browser automation, and planning modes.</td></tr>
<tr><td><b>MCP Config Reference</b></td><td>Direct paths, environment variables, local/remote stdio transports, and tool discovery rules.</td></tr>
<tr><td><b>Conventions & Rules</b></td><td>Where to write custom instruction files (e.g., <code>CLAUDE.md</code>, <code>CONVENTIONS.md</code>, <code>.cursor/rules/</code>) to customize agent behaviors.</td></tr>
</table>

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
| [OpenClaw](tools/openclaw/) | OpenClaw | YAML | ✅ Full (18 events via Plugin SDK) | ✅ | [docs.openclaw.ai](https://docs.openclaw.ai) |
| [Trae](tools/trae/) | ByteDance | Markdown rules | ⚠️ Via MCP only | ✅ | [docs.trae.ai](https://docs.trae.ai) |
| [Trae CN](tools/trae-cn/) | ByteDance | Markdown rules | ⚠️ Via MCP only | ✅ | [docs.trae.ai](https://docs.trae.ai) |
| [Aider](tools/aider/) | Aider-AI | YAML | ❌ Not shipped | ❌ | [aider.chat/docs](https://aider.chat/docs/) |
| [Google Antigravity](tools/google-antigravity/) | Google | JSON / TOML | ✅ Full (5 events) | ✅ | [antigravity.google](https://antigravity.google) |

**Legend:** ✅ Full = shipped, documented hook system with blocking · ⚠️ = partial or workaround · ❌ = not available · ❓ = unknown

---

## Quick Navigation

| Question | Go to |
|----------|-------|
| Which agents support activity X? | [`_shared/activity-agent-matrix.md`](_shared/activity-agent-matrix.md) |
| What are all of Agent Y's hooks/tools/config? | [`_shared/agent-tools-hooks-config.md`](_shared/agent-tools-hooks-config.md) |
| What is PreToolUse called in each tool? | [`_shared/hook-event-comparison.md`](_shared/hook-event-comparison.md) |
| Where does Tool Y store its config and skills? | [`_shared/config-file-locations.md`](_shared/config-file-locations.md) |

---

## Repository Structure

```
.
├── README.md                          # Master index + comparative benchmarks
├── tools/
│   └── <tool-name>/
│       └── README.md                 # Per-tool configuration & command references
├── _shared/
│   ├── activity-agent-matrix.md      # Capabilities matrix (natively supported tools)
│   ├── agent-tools-hooks-config.md   # Unified agent specifications & schemas
│   ├── hook-event-comparison.md      # Cross-agent hook event name lookup
│   ├── mcp-support.md                # MCP client capabilities & transport settings
│   └── config-file-locations.md      # Global/local paths & git-ignored overrides
└── _templates/
    └── tool-template.md              # Blueprint for documenting new developer tools
```

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

---

## Contributing

See [`_templates/tool-template.md`](_templates/tool-template.md) for the standard page format.

When adding or updating a tool:
1. Source everything to an official URL — no guessing.
2. Mark uncertain items with ❓.
3. Use the `[official]` / `[github]` / `[community]` labels in the Sources table.
4. Update the master table in this README.
5. Update `_shared/agent-tools-hooks-config.md` with the agent's section.
