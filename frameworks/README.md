# Agent Frameworks & SDKs

Sourced catalog for agent frameworks and SDKs that are adjacent to the coding-agent tools tracked elsewhere in this repo.

> Naming note: a few labels in the screenshot map to slightly different official product names. "Anthropic Agent SDK" is documented by Anthropic as the **Claude Code Agent SDK**. "Google ADK" is the **Agent Development Kit (ADK)**. "AutoGen 2.0" is listed below as **AutoGen (stable)** because the official docs I verified do not currently brand the stable line as "2.0".

## Coverage

| Project | Official name | Vendor | Category | Primary languages | Install / package |
|---|---|---|---|---|---|
| LangGraph | LangGraph | LangChain | Orchestration framework | Python, JavaScript/TypeScript | `pip install -U langgraph` |
| CrewAI | CrewAI | CrewAI, Inc. | Multi-agent framework | Python | Install via `uv` / CrewAI CLI |
| AutoGen 2.0 | AutoGen (stable) | Microsoft | Agent framework | Python, .NET | `pip install -U autogen-agentchat autogen-ext[openai]` |
| Mastra | Mastra | Mastra | Agent framework / app framework | TypeScript | `npm create mastra` |
| OpenAI Agents SDK | OpenAI Agents SDK | OpenAI | Agent SDK / runtime | Python | `pip install openai-agents` |
| PydanticAI | Pydantic AI | Pydantic | Agent framework | Python | `pip install pydantic-ai` |
| Anthropic Agent SDK | Claude Code Agent SDK | Anthropic | Agent SDK / runtime | Python, TypeScript | `pip install claude-agent-sdk` / `npm install @anthropic-ai/claude-agent-sdk` |
| Google ADK | Agent Development Kit (ADK) | Google | Agent framework / runtime | Python, TypeScript, Go, Java, Kotlin | See language-specific install docs |

## Comparison

| Project | What it is | Verified built-in concepts / capabilities | Sources |
|---|---|---|---|
| LangGraph | Low-level framework for long-running, stateful agents and workflows | Durable execution, human-in-the-loop, memory, deployment path, JS/TS counterpart called LangGraph.js | https://github.com/langchain-ai/langgraph |
| CrewAI | Framework for collaborative agents, crews, and flows | Agents, flows, memory, knowledge, guardrails, callbacks, human-in-the-loop triggers, enterprise deploy/monitoring docs | https://docs.crewai.com/ |
| AutoGen (stable) | Framework for building AI agents and applications | AgentChat, Core, Extensions, Studio, MCP extension support | https://microsoft.github.io/autogen/stable/ |
| Mastra | TypeScript framework for AI-powered applications and agents | Agents, workflows, memory, observability, evals, guardrails, MCP server support, deployers for Vercel/Netlify/Cloudflare | https://mastra.ai/ |
| OpenAI Agents SDK | Lightweight production SDK for agentic apps | Agent loop, handoffs, guardrails, function tools, MCP server tools, sessions, tracing, sandbox agents, realtime agents | https://openai.github.io/openai-agents-python/ |
| Pydantic AI | Python framework for production-grade agent applications and workflows | Agents, typed output, dependencies, hooks, toolsets, native tools, MCP client/server support, multi-agent patterns, durable execution, evals, graph workflows | https://pydantic.dev/docs/ai/overview/ |
| Claude Code Agent SDK | Claude Code packaged as a programmable SDK | File/tools/web/code capabilities from Claude Code, hooks, sessions, permissions, subagents, Python and TypeScript SDKs | https://code.claude.com/docs/en/agent-sdk/overview |
| ADK | Open-source framework for building, debugging, and deploying agents | Multi-tool agents, graph workflows, multi-agent workflows, MCP tools, sessions/memory, plugins, deploy-to-Google-Cloud path | https://adk.dev/ |

## Notes By Project

### LangGraph

- Official positioning: a low-level orchestration framework for long-running, stateful agents.
- Verified features from the official repo README: durable execution, human-in-the-loop, comprehensive memory, and deployment-oriented workflow support.
- Official repo also points to a separate JS/TS variant, `LangGraph.js`.

### CrewAI

- Official docs position CrewAI around three top-level concepts: agents, crews, and flows.
- Verified docs claims include guardrails, memory, knowledge, observability, persisted flows, and human-in-the-loop triggers.
- The docs homepage confirms CLI-oriented local setup, but I did not add package-level claims beyond the docs-verified `uv` installation guidance.

### AutoGen (stable)

- The current official docs present AutoGen as a framework with four main areas: AgentChat, Core, Extensions, and Studio.
- Official docs also show an MCP-oriented extension path via `McpWorkbench`.
- I did not label this "2.0" in the repo content because the verified docs are branded as `stable` and explicitly distinguish themselves from the old `0.2 Docs`.

### OpenAI Agents SDK

- Official docs describe it as a lightweight, production-ready SDK with a small set of primitives: agents, handoffs, and guardrails.
- Verified docs also cover built-in tracing, sessions, MCP tool integration, sandbox agents, and realtime agents.
- This is currently Python-first in the official docs I verified.

### Mastra

- Official site positions Mastra as an open-source TypeScript framework for building AI-powered applications and agents.
- Verified official claims include agents, workflows, memory, workspaces, observability, evals, guardrails, and MCP server support.
- Official docs/site also verify deployment paths for Node.js environments, including built-in deployers for Vercel, Netlify, and Cloudflare.

### Pydantic AI

- Official docs position Pydantic AI as the Pydantic team’s AI framework, with dedicated docs under `pydantic.dev`.
- Verified docs include agents, dependencies, structured output, capabilities, hooks, toolsets, native tools, and multi-agent patterns.
- The official docs also verify MCP client/server support, durable execution integrations, evals, graph workflows, and a CLI.

### Claude Code Agent SDK

- Anthropic documents this under Claude Code, not as a standalone "Anthropic Agent SDK" brand.
- Official docs verify Python and TypeScript SDKs, bundled tools, hooks, permissions, session management, and subagent support.
- The SDK exposes the same agent loop and context-management primitives that power Claude Code.

### ADK

- Google documents ADK as an open-source framework for building, debugging, and deploying agents.
- Verified docs show support for Python, TypeScript, Go, Java, and Kotlin.
- The docs also explicitly surface graph workflows, multi-agent workflows, MCP tools, sessions/memory, plugins, and Google Cloud deployment paths.

## Sources

| Project | Source | Fetched | Label |
|---|---|---|---|
| LangGraph | https://github.com/langchain-ai/langgraph | 2026-06-26 | [github] |
| CrewAI | https://docs.crewai.com/ | 2026-06-26 | [official] |
| AutoGen | https://microsoft.github.io/autogen/stable/ | 2026-06-26 | [official] |
| Mastra | https://mastra.ai/ | 2026-06-26 | [official] |
| OpenAI Agents SDK | https://openai.github.io/openai-agents-python/ | 2026-06-26 | [official] |
| Pydantic AI | https://pydantic.dev/docs/ai/overview/ | 2026-06-26 | [official] |
| Claude Code Agent SDK | https://code.claude.com/docs/en/agent-sdk/overview | 2026-06-26 | [official] |
| ADK | https://adk.dev/ | 2026-06-26 | [official] |
