<p align="center">
  <img src="https://img.shields.io/badge/Agentic%20Tools%20Almanac-%E2%9C%A7-blueviolet?style=for-the-badge" alt="Agentic Tools Almanac">
  <a href="_shared/agent-tools-hooks-config.md"><img src="https://img.shields.io/badge/Config%20Reference-Shared-FFD700?style=for-the-badge" alt="Config Reference"></a>
  <a href="_shared/hook-event-comparison.md"><img src="https://img.shields.io/badge/Hooks%20Matrix-Cross%20Tool-5865F2?style=for-the-badge" alt="Hooks Matrix"></a>
  <a href="frameworks/README.md"><img src="https://img.shields.io/badge/Frameworks%20%26%20SDKs-Sourced-2E8B57?style=for-the-badge" alt="Frameworks and SDKs"></a>
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
| [Claude Code](tools/claude-code/) | Anthropic | JSON | ✅ Full (31 events) | ✅ | [code.claude.com/docs/en/hooks](https://code.claude.com/docs/en/hooks) |
| [Codex CLI](tools/codex/) | OpenAI | TOML | ✅ Full (10 events) | ✅ | [developers.openai.com/codex](https://developers.openai.com/codex) |
| [GitHub Copilot](tools/github-copilot/) | GitHub/Microsoft | JSON | ✅ Full (13 events) | ✅ | [docs.github.com/en/copilot/reference/hooks-reference](https://docs.github.com/en/copilot/reference/hooks-reference) |
| [Gemini CLI](tools/gemini-cli/) | Google | JSON | ✅ Full (11 events) | ✅ | [geminicli.com/docs](https://geminicli.com/docs) |
| [Factory Droid](tools/factory-droid/) | Factory.ai | JSON | ✅ Full (9 events) | ✅ | [docs.factory.ai](https://docs.factory.ai) |
| [Kiro IDE/CLI](tools/kiro/) | Amazon (AWS) | YAML | ✅ Full | ✅ | [kiro.dev/docs](https://kiro.dev/docs) |
| [Kimi Code](tools/kimi-code/) | Moonshot AI | TOML | ✅ Full — Beta (16 events) | ✅ | [kimi.com/code/docs](https://www.kimi.com/code/docs/en/kimi-code-cli/) |
| [Cursor](tools/cursor/) | Anysphere | JSON | ✅ Full (23 events) | ✅ | [cursor.com/docs/hooks](https://cursor.com/docs/hooks) |
| [Hermes](tools/hermes/) | NousResearch | YAML | ✅ Full (❓ events) | ✅ | [hermes-agent.nousresearch.com](https://hermes-agent.nousresearch.com/docs) |
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
| Which pages still rely on community or inferred sources? | [`_shared/source-audit.md`](_shared/source-audit.md) |
| Do we track orchestration frameworks and SDKs too? | [`frameworks/README.md`](frameworks/README.md) |

---

## Frameworks & SDKs

These are tracked separately from the coding-agent matrices because they are developer frameworks/runtimes rather than end-user CLI agents with a shared hooks/config surface.

| Framework / SDK | Vendor | Notes | Docs |
|------|--------|-------|------|
| [LangGraph](frameworks/README.md#langgraph) | LangChain | Low-level stateful agent orchestration | https://github.com/langchain-ai/langgraph |
| [CrewAI](frameworks/README.md#crewai) | CrewAI, Inc. | Crews, flows, memory, guardrails | https://docs.crewai.com/ |
| [AutoGen](frameworks/README.md#autogen-stable) | Microsoft | Stable docs tracked instead of unverified "2.0" branding | https://microsoft.github.io/autogen/stable/ |
| [OpenAI Agents SDK](frameworks/README.md#openai-agents-sdk) | OpenAI | Agent runtime with handoffs, guardrails, MCP | https://openai.github.io/openai-agents-python/ |
| [Claude Code Agent SDK](frameworks/README.md#claude-code-agent-sdk) | Anthropic | Official product name for the requested "Anthropic Agent SDK" | https://code.claude.com/docs/en/agent-sdk/overview |
| [Google ADK](frameworks/README.md#adk) | Google | Agent Development Kit | https://adk.dev/ |

---

## Repository Structure

```
.
├── README.md                          # Master index + comparative benchmarks
├── frameworks/
│   └── README.md                      # Sourced framework / SDK catalog
├── tools/
│   └── <tool-name>/
│       └── README.md                 # Per-tool configuration & command references
├── _shared/
│   ├── activity-agent-matrix.md      # Capabilities matrix (natively supported tools)
│   ├── agent-tools-hooks-config.md   # Unified agent specifications & schemas
│   ├── source-audit.md               # Source coverage audit and unresolved gaps
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
| After tool fails | `PostToolUseFailure` | — | — | `postToolUseFailure` | `PostToolUseFailure` | — | `postToolUseFailure` | — | — |
| Prompt submitted | `UserPromptSubmit` | `UserPromptSubmit` | `BeforeAgent` | `userPromptSubmitted` | `UserPromptSubmit` | `UserPromptSubmit` | `beforeSubmitPrompt` | `UserPromptSubmit` | — |
| Agent turn ends | `Stop` | `Stop` | `AfterAgent` | `agentStop` | `Stop` | `Stop` | `stop` | `Stop` | — |
| Subagent starts | `SubagentStart` | `SubagentStart` | — | `subagentStart` | `SubagentStart` | — | `subagentStart` | — | — |
| Session starts | `SessionStart` | `SessionStart` | `SessionStart` | `sessionStart` | `SessionStart` | `SessionStart` | `sessionStart` | `SessionStart` | `session.created` |
| Session ends | `SessionEnd` | — | `SessionEnd` | `sessionEnd` | — | `SessionEnd` | `sessionEnd` | — | — |
| Context compacted | `PreCompact` | `PreCompact` | `PreCompress` | `preCompact` | `PreCompact` | `PreCompact` | `preCompact` | — | — |
| Permission request | `PermissionRequest` | `PermissionRequest` | — | `permissionRequest` | — | — | — | `PermissionRequest` | — |
| LLM call | — | — | `BeforeModel` / `AfterModel` | — | — | — | — | — | — |
| Notification | `Notification` | — | `Notification` | `notification` | — | `Notification` | — | — | — |

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
| **[official mirror]** | Vendor-maintained mirror of official docs |
| **[community]** | Community-maintained project (e.g., oh-my-pi) — may diverge from vendor |
| **[third-party]** | Non-vendor package index, mirror, or independently maintained page |
| **[press]** | News or press coverage, useful for release context but not normative product behavior |
| **[installer-src]** | Inferred from the installer scripts or directory structures |

---

## Contributing

See [`_templates/tool-template.md`](_templates/tool-template.md) for the standard page format.

When adding or updating a tool:
1. Source everything to an official URL — no guessing.
2. Mark uncertain items with ❓.
3. Use the `[official]` / `[github]` / `[community]` labels in the Sources table.
4. Update the master table in this README.
5. Update `_shared/agent-tools-hooks-config.md` with the agent's section.

---

## Data Sources & Disclaimer

The Hook Event Cross-Reference table and event counts in the Tools Covered table are sourced directly from each tool's official documentation at the time of last update. Hook APIs evolve rapidly — vendor docs are the authoritative source and should always be consulted before relying on counts or event names listed here. Entries marked ❓ could not be verified against live official documentation at the time of writing.
