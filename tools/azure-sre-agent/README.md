# Azure SRE Agent

> Microsoft's cloud SRE (Site Reliability Engineering) agent that autonomously investigates incidents and operates Azure resources. Unlike the coding CLIs in this catalog, it is a hosted Azure service configured through the portal UI or REST API v2 — there is no local config file or terminal binary.

**Vendor:** Microsoft (Azure) | **License:** Proprietary (Azure service) | **Runtime:** Hosted / cloud

## Links

- Docs (Agent Hooks): https://learn.microsoft.com/en-us/azure/sre-agent/agent-hooks
- Create & manage hooks (portal): https://learn.microsoft.com/en-us/azure/sre-agent/create-manage-hooks-ui
- Configure hooks (REST API v2): https://learn.microsoft.com/en-us/azure/sre-agent/tutorial-agent-hooks

---

## Installation

Not installed locally. Azure SRE Agent is provisioned as an Azure resource and used through the Azure portal. Hooks are configured either in the portal or via the management REST API — see below.

## Configuration Files

There is **no local config file**. Hooks and agent settings are stored in the Azure service and edited through:

| Surface | Scope | Purpose |
|---------|-------|---------|
| **Builder → Hooks** (portal UI) | Agent level | Applies to the entire agent, all threads and all custom agents |
| **Agent Canvas → Custom agent → Manage Hooks** (portal UI) | Custom-agent level | Applies only when that specific custom agent runs |
| **REST API v2** — `PUT /api/v2/extendedAgent/agents/{agentName}` | Agent / custom-agent | Full YAML configuration schema (see below) |

Both levels can coexist. When an agent-level hook and a custom-agent-level hook match the same event, **both run**, and **agent-level hooks fire first**.

> The **Agent Canvas YAML** tab shows the v1 format and does **not** display hooks. Use the **Builder → Hooks** page to view/manage hooks.

## Instruction File

No repo-level instruction file (`AGENTS.md`/`CLAUDE.md` equivalent). The agent's standing instructions live in the `spec.instructions` field of the ExtendedAgent YAML.

## Hooks

Two hook events are currently supported. Each hook is implemented as either an LLM **prompt** or a **command** (shell/Python script running in a sandboxed code interpreter).

### Supported Events

| Event | When | Can Block? |
|-------|------|-----------|
| `Stop` | Agent is about to return its final response | ✅ — reject to force the agent to continue |
| `PostToolUse` | A tool finishes executing successfully | ✅ — block the result, or inject `additionalContext` |

### Hook Types

| Type | How it works | Best for |
|------|--------------|----------|
| `prompt` | An LLM evaluates the prompt and returns a JSON decision | Nuanced validation ("Is this response complete?") |
| `command` | A bash or Python script runs in a sandboxed environment | Deterministic checks, policy enforcement, auditing |

**Prompt hooks** use the `$ARGUMENTS` placeholder to receive the full hook context; if `$ARGUMENTS` is absent, the context is appended automatically. When a conversation transcript is available, prompt hooks also receive `ReadFile` and `GrepSearch` tools to reason over the full history.

### Hook Input (context schema)

Prompt hooks receive context via `$ARGUMENTS`; command hooks receive it as JSON on **stdin**. In both cases `execution_summary` is a **file path** to the transcript (not inline content).

**Common fields (all hooks):**
```json
{
  "hook_event_name": "Stop",
  "agent_name": "my_agent",
  "current_turn": 5,
  "max_turns": 50,
  "execution_summary": "/path/to/transcript.txt"
}
```

**`Stop` adds:**
```json
{ "final_output": "Here is my response...", "stop_hook_active": false, "stop_rejection_count": 0 }
```

**`PostToolUse` adds:**
```json
{ "tool_name": "ExecutePythonCode", "tool_input": { "code": "print(2+2)" }, "tool_result": "4", "tool_succeeded": true }
```

### Hook Output (stdout JSON)

**Simple format** (recommended for prompt hooks):
```json
{"ok": true}
{"ok": false, "reason": "Please include more details."}
```

**Expanded format** (recommended for command hooks):
```json
{"decision": "allow"}
{"decision": "block", "reason": "Dangerous command detected."}
{"decision": "allow", "hookSpecificOutput": {"additionalContext": "Tool audit logged."}}
```

`additionalContext` is injected into the conversation as a user message. If multiple `PostToolUse` hooks emit `additionalContext`, the **last** one wins.

> ⚠️ For `Stop` hooks, a rejection **without a `reason`** is treated as **approval** — always include `reason` when rejecting.

### Exit Code Behavior (command hooks)

| Code | Meaning |
|------|---------|
| `0` with no output | Allow (no objection) |
| `0` with JSON | Parse JSON for decision |
| `2` | Always block; stderr becomes the reason |
| Other | Falls back to `failMode` (`allow` or `block`) |

### Configuration Reference

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `type` | string | `prompt` | `prompt` or `command` |
| `prompt` | string | — | LLM prompt text (required for prompt hooks); use `$ARGUMENTS` for context injection |
| `command` | string | — | Inline shell command (mutually exclusive with `script`) |
| `script` | string | — | Multi-line script (mutually exclusive with `command`) |
| `matcher` | string | — | Regex for tool names (required for `PostToolUse`). `*` matches all. Anchored as `^(pattern)$`, **case-sensitive**. Empty/null matches nothing |
| `timeout` | int | `30` | Execution timeout in seconds (positive; >300 flagged during CLI validation) |
| `failMode` | string | `allow` | How to handle hook errors: `allow` or `block` |
| `model` | string | `ReasoningFast` | Model for prompt hooks (scenario or deployment name) |
| `maxRejections` | int | `3` | Max rejections before forcing stop (1–25). Prompt-type `Stop` hooks only; command-type `Stop` hooks have no implicit limit. Max value used when multiple prompt hooks differ |

### Example Config (YAML, REST API v2)

```yaml
api_version: azuresre.ai/v2
kind: ExtendedAgent
metadata:
  name: my_hooked_agent
spec:
  instructions: |
    You are a helpful assistant.
  handoffDescription: ""
  enableVanillaMode: true
  hooks:
    Stop:
      - type: prompt
        prompt: |
          Check if the response ends with "Task complete."
          $ARGUMENTS
          Respond with:
          - {"ok": true} if it does
          - {"ok": false, "reason": "End your response with 'Task complete.'"} if not
        timeout: 30
    PostToolUse:
      - type: command
        matcher: "Bash|ExecuteShellCommand"
        timeout: 30
        failMode: block
        script: |
          #!/usr/bin/env python3
          import sys, json, re
          context = json.load(sys.stdin)
          command = context.get('tool_input', {}).get('command', '')
          dangerous = [r'\brm\s+-rf\b', r'\bsudo\b', r'\bchmod\s+777\b']
          for pattern in dangerous:
              if re.search(pattern, command):
                  print(json.dumps({"decision": "block", "reason": f"Blocked: {pattern}"}))
                  sys.exit(0)
          print(json.dumps({"decision": "allow"}))
```

### Model Tiers (prompt hooks)

| Tier | Best for | Trade-off |
|------|----------|-----------|
| **Reasoning** | Complex policy enforcement, multistep validation | Highest quality, higher cost/latency |
| **Fast Reasoning** (default) | Most hooks, response validation, audit/safety checks | Good reasoning, low latency |
| **General Purpose** | Simple format checks, basic compliance | Balanced accuracy/cost/speed |
| **Fast** | Lightweight presence/format checks | Lowest cost, fastest |
| **Long Context** | Large outputs, full-document analysis | Handles larger input, higher cost |

### Limits

| Limit | Value |
|-------|-------|
| Script size | 64 KB max |
| Timeout | 1–300 seconds |
| Max rejections (prompt `Stop` hooks) | 1–25 (default 3) |
| Supported shebangs | `#!/bin/bash`, `#!/usr/bin/env python3` |
| Execution environment | Sandboxed code interpreter |

## Built-in Tools

The agent invokes tools during incident investigation; hooks intercept them via the `matcher` field. Tool names seen in official examples include `Bash`, `ExecuteShellCommand`, and `ExecutePythonCode`. ❓ A complete, canonical tool list is not enumerated on the hooks page.

## MCP Support

❓ Not documented on the agent-hooks page.

## Skills / Commands

Not applicable in the CLI-skill sense. Behavior is shaped by `spec.instructions` and by **custom agents** defined on the Agent Canvas.

## Agent / Subagent Configuration

- **Agent** — the top-level SRE agent (`kind: ExtendedAgent`).
- **Custom agents** — specialized agents defined on the Agent Canvas, each able to carry their own hooks.
- Hooks complement (do not replace) **run mode** safety controls: run modes govern *what* the agent may do; hooks govern *how well* it does it and *what happens* with results.

## Notes

- Cloud service — no local install, no terminal binary, no local config file.
- Only `Stop` and `PostToolUse` events exist today (vs. the broader lifecycle surfaces of the CLI agents in this catalog).
- Best practices from the docs: always provide a `reason` when rejecting; keep timeouts short; prefer `failMode: allow` unless strict enforcement is required; be specific with matchers; log debug output to **stderr** (stdout is parsed as the hook result).

## Sources

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| Agent Hooks (concept) | https://learn.microsoft.com/en-us/azure/sre-agent/agent-hooks | 2026-09-12 | [official] |
| Create & manage hooks (portal) | https://learn.microsoft.com/en-us/azure/sre-agent/create-manage-hooks-ui | 2026-09-12 | [official] |
| Configure hooks (REST API v2) | https://learn.microsoft.com/en-us/azure/sre-agent/tutorial-agent-hooks | 2026-09-12 | [official] |
</content>
</invoke>
