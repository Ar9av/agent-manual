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
| Vercel AI SDK | Vercel AI SDK | Vercel | AI app SDK | TypeScript | `npm install ai` |
| LlamaIndex | LlamaIndex | LlamaIndex | Data / agent framework | Python, TypeScript | `pip install llama-index` |
| Semantic Kernel | Semantic Kernel | Microsoft | Agent SDK / app framework | C# (.NET), Python, Java | `pip install semantic-kernel` / NuGet / Maven |
| smolagents | smolagents | Hugging Face | Lightweight agent framework | Python | `pip install 'smolagents[toolkit]'` |
| Agno | Agno | Agno AGI (formerly Phidata) | Agent platform SDK | Python | `pip install agno` |
| Haystack | Haystack | deepset | AI pipeline framework | Python | `pip install haystack-ai` |
| BeeAI | BeeAI Framework | IBM Research / Linux Foundation AI & Data | Agent framework | Python, TypeScript | `pip install beeai-framework` / `npm install beeai-framework` |
| DSPy | DSPy | Stanford NLP | Prompt optimization / AI system framework | Python | `pip install -U dspy` |
| Letta | Letta | Letta AI (formerly MemGPT) | Stateful agent platform | Python, TypeScript | `pip install letta-client` / `npm install @letta-ai/letta-client` |
| LlamaStack | LlamaStack | Meta | Agentic API server / inference stack | Python, TypeScript | `pip install llama-stack` |
| CAMEL-AI | CAMEL | CAMEL-AI | Multi-agent research framework | Python | `pip install camel-ai[all]` |
| MetaGPT | MetaGPT | FoundationAgents | Multi-agent software engineering framework | Python | `pip install metagpt` |
| TaskWeaver | TaskWeaver | Microsoft | Code-first agent framework for data analytics | Python | `git clone https://github.com/microsoft/TaskWeaver && pip install -r requirements.txt` |
| Flowise | Flowise | FlowiseAI | Low-code visual LLM workflow builder | JavaScript/TypeScript | `npm install -g flowise` |
| Langflow | Langflow | langflow-ai | Low-code visual agent and RAG builder | Python (backend), TypeScript (frontend) | `pip install langflow` |
| Composio | Composio | ComposioHQ | Tool integration platform for AI agents | Python, TypeScript | `pip install composio-core` |
| Instructor | Instructor | jxnl | Structured LLM output extraction library | Python, TypeScript, Go, Ruby | `pip install instructor` |
| Marvin | Marvin | PrefectHQ | Ambient AI library for structured outputs and tasks | Python | `pip install marvin` |

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
| Vercel AI SDK | TypeScript SDK for building AI-powered web and server applications | Streaming text and structured output, multi-step agents, tool use, built-in providers (100+ models via OpenAI, Anthropic, Google, etc.), React hooks, built-in fallbacks | https://ai-sdk.dev/ |
| LlamaIndex | Framework for building LLM applications over custom data | Data loaders/connectors, vector stores, query engines, agents, workflows, RAG pipelines, multi-modal support, TypeScript counterpart LlamaIndex.TS | https://developers.llamaindex.ai/python/framework/ |
| Semantic Kernel | Enterprise-grade SDK for integrating AI models into applications | Agents, plugins, planners, memory/vector stores, function calling, process framework, multi-agent orchestration, .NET/Python/Java support | https://learn.microsoft.com/en-us/semantic-kernel/overview/ |
| smolagents | Minimal, code-first agent framework from Hugging Face | Code agents (write Python to act), tool-calling agents, multi-agent orchestration, MCP tool support, guided generation, model-agnostic | https://huggingface.co/docs/smolagents/ |
| Agno | High-performance agent platform SDK | Agents, teams (multi-agent), sessions, memory, knowledge bases, tools, reasoning models, multimodal support, evals, built-in observability | https://docs.agno.com/ |
| Haystack | Composable AI pipeline framework | Pipelines, components, document stores, RAG, agents, tool calling, async support, YAML pipeline serialization, 60+ integrations | https://docs.haystack.deepset.ai/ |
| BeeAI Framework | Open-source agent framework from IBM Research | Agents, workflows, memory, tools, ReAct/Granite agent patterns, Python and TypeScript SDKs, Linux Foundation AI & Data project | https://framework.beeai.dev/ |
| DSPy | Framework for algorithmically optimizing LLM prompts and weights | Signatures, modules, optimizers (compile), assertions, retrieval models, multi-hop programs, evaluation, model-agnostic | https://dspy.ai/ |
| Letta | Stateful long-term memory agent platform | Agents with persistent memory, memory blocks (core/archival/recall), tool calling, multi-agent support, REST API server, Python and TypeScript clients | https://docs.letta.com/ |
| LlamaStack | Standardized agentic API server and inference stack from Meta | Inference, agents, memory, safety, tool groups, eval tasks, REST API spec, multiple distribution backends (Ollama, Together, Fireworks, etc.) | https://github.com/meta-llama/llama-stack |
| CAMEL | Multi-agent framework for role-playing, task decomposition, and agent society research | Role assignment, Society framework, multi-agent task coordination, MCP multi-agent support, RAG, web tools, research-oriented scaling experiments | https://docs.camel-ai.org/ |
| MetaGPT | Multi-agent framework modeled as a software engineering company | One-line requirement → full PRD/design/tasks/repo, SOP-encoded agent roles (PM, architect, engineer, QA), assembly-line task decomposition, collaborative code review | https://github.com/FoundationAgents/MetaGPT |
| TaskWeaver | Code-first data analytics agent framework from Microsoft | Sandbox code execution, task decomposition and progress tracking, reflective execution (mid-flight adjustment), handles tabular/high-dimensional data, preserves chat and code history | https://microsoft.github.io/TaskWeaver/ |
| Flowise | Low-code visual builder for LLM workflows and agents | Drag-and-drop canvas, Agentflow (multi-agent) and Chatflow (single-agent/chatbot) modes, modular component library, Docker deployment, quick prototyping to production | https://docs.flowiseai.com/ |
| Langflow | Low-code visual builder for agents and RAG pipelines | Visual component editor, Python-based and model/database agnostic, built-in API and MCP server export, supports all major LLMs and vector databases | https://docs.langflow.org/ |
| Composio | Tool integration and orchestration platform for AI agents | 1000+ app/tool integrations, MCP server architecture, managed auth, sandboxed workbench, integrates with Claude Code/Cursor/VS Code, tool search and context management | https://docs.composio.dev/ |
| Instructor | Structured LLM output extraction library | Pydantic model-based schema definition, 15+ provider support, automatic validation and retries, streaming (Iterables/Partial), TypeScript/Go/Ruby ports, 3M+ monthly downloads | https://python.useinstructor.com/ |
| Marvin | Ambient AI library for structured outputs and task-based workflows | Tasks as fundamental unit, multi-agent assignment, Threads for workflow orchestration, Pydantic AI backend, SQLite message history, cast/classify/extract/generate primitives | https://askmarvin.ai/ |

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

### Vercel AI SDK

- Official positioning: a unified TypeScript SDK for building AI apps across React, Next.js, Vue, Svelte, and Node.js. The canonical docs site is now `https://ai-sdk.dev/` (the old `sdk.vercel.ai` redirects there).
- Core primitives: `generateText`, `streamText`, `generateObject`, `streamObject` — all model-agnostic via a unified provider interface.
- Verified features: multi-step (agentic) tool use, 100+ model support via separate `@ai-sdk/<provider>` packages, built-in fallbacks for production reliability, and React streaming hooks (`useChat`, `useCompletion`).
- Ships as `ai` (core) on npm; provider packages follow the `@ai-sdk/<provider>` naming convention.

### LlamaIndex

- Official docs have moved to `https://developers.llamaindex.ai/python/framework/` (the old `docs.llamaindex.ai` redirects there).
- Official docs position LlamaIndex as a data framework for building LLM applications over custom/private data.
- Core primitives: data loaders, vector stores, indices, query engines, and agents.
- Verified features include RAG pipelines, structured extraction, multi-modal support, workflows, agentic tool use, and observability integrations.
- Has a TypeScript counterpart (`llamaindex` npm package) maintained as LlamaIndex.TS.

### Semantic Kernel

- Microsoft positions Semantic Kernel as an enterprise-grade SDK for integrating AI models into existing .NET, Python, and Java applications.
- Verified features: plugins (wrapping functions or OpenAPI specs as AI tools), planners, kernel memory / vector store connectors, function calling, and a process framework for stateful workflows.
- Official docs also cover multi-agent orchestration via the Agent framework, and deep Azure OpenAI integration.
- Primary install paths: NuGet (`Microsoft.SemanticKernel`) for .NET, `pip install semantic-kernel` for Python, Maven for Java.

### smolagents

- Hugging Face positions smolagents as a minimal, code-first agent library focused on simplicity.
- Two core agent types: `CodeAgent` (writes and executes Python to act) and `ToolCallingAgent` (function-call style).
- Verified features: multi-agent orchestration, MCP tool support, guided generation for structured output, and a model-agnostic design (works with any HF model or API-backed model).
- Designed to keep the core library small while remaining extensible via tools and custom agents.

### Agno

- Formerly Phidata; rebranded to Agno AGI.
- Official docs position Agno as a high-performance agent platform SDK emphasizing speed and multi-modal support.
- Verified features: agents, teams (multi-agent collaboration), sessions, memory (user + session), knowledge bases (vector-backed), tools, and built-in evals and observability.
- Supports reasoning models and multimodal inputs; Python-first.

### Haystack

- deepset positions Haystack as a composable AI pipeline framework for building production-grade AI applications.
- Core primitive: Pipelines composed of reusable, interchangeable Components (LLM nodes, document stores, embedders, rankers, etc.).
- Verified features: RAG pipelines, agents with tool calling, async component execution, YAML pipeline serialization/deserialization, and 60+ integration packages.
- Document stores integrate with major vector databases; agents support multi-step tool use.

### BeeAI Framework

- Developed by IBM Research, now a Linux Foundation AI & Data project; GitHub organization is `i-am-bee`.
- Official docs are at `https://framework.beeai.dev/` (the old `beeai.dev/docs` URL is broken).
- Verified features: agents (ReAct and Granite agent patterns), structured workflows, memory, 10+ LLM provider support (Ollama, OpenAI, Watsonx.ai, etc.), and feature-parity Python and TypeScript SDKs.
- Available as `beeai-framework` (pip) and `beeai-framework` (npm).

### DSPy

- Stanford NLP's framework for algorithmically optimizing LM prompts and weights rather than writing prompts by hand.
- Core abstractions: Signatures (define input/output), Modules (compose reasoning steps), Optimizers/`compile()` (tune prompts or fine-tune weights against a metric).
- Verified features: assertions/constraints, retrieval model integration, multi-hop programs, built-in evaluation, and model-agnostic design.
- Positioned as a programmatic alternative to hand-crafted prompt engineering.

### Letta

- Formerly MemGPT; rebranded to Letta, focused on stateful long-term memory agents.
- Official docs verify a REST API server (`letta-server`) plus Python (`letta-client`) and TypeScript (`@letta-ai/letta-client`) client SDKs.
- Verified features: agents with persistent memory blocks (core memory, archival memory, recall memory), tool calling, human-in-the-loop via memory editing, and multi-agent support.
- Agents retain state across sessions by design; memory is first-class rather than bolted on.

### LlamaStack

- Meta's standardized agentic API server and inference stack, designed to make Llama models deployable across providers. GitHub is `meta-llama/llama-stack`.
- The old `llama-stack.readthedocs.io` URL is defunct; versioned docs live at `llamastack.github.io/v<version>/`.
- Verified REST API spec covers inference, agents, memory, safety (shields), tool groups, and eval tasks.
- Install the server with `pip install llama-stack`; install only the client with `pip install llama-stack-client`.
- Verified distribution backends include Ollama, Together AI, Fireworks, NVIDIA, and others.

### CAMEL

- CAMEL (Communicative Agents for "Mind" Exploration of Large Language Model Society) is a research-oriented multi-agent framework from the CAMEL-AI community.
- Official docs verify role-playing agent coordination, a Society framework for task decomposition, code execution via built-in interpreters (Python, shell, browser), and RAG and web tools.
- Research focus includes scaling laws for agent societies, synthetic data generation, and world simulation.
- Install all toolkits and interpreters with `pip install camel-ai[all]`; use partial extras (e.g., `[rag]`, `[web_tools]`) to keep installs lean.

### MetaGPT

- FoundationAgents' framework models a software engineering company as a multi-agent system, where a one-line requirement produces a complete PRD, design, task plan, and code repository.
- Core design: SOP-encoded agent roles (product manager, architect, engineer, QA) collaborate in an assembly-line paradigm.
- Includes collaborative code review and a research track (ICLR 2025 AFlow paper).
- Primarily documented via the GitHub repo rather than a standalone docs site.

### TaskWeaver

- Microsoft's code-first agent framework designed for data analytics tasks requiring code execution.
- **No pip package exists.** Install by cloning the repo: `git clone https://github.com/microsoft/TaskWeaver && cd TaskWeaver && pip install -r requirements.txt`. A specific tag can be installed via `pip install git+https://github.com/microsoft/TaskWeaver@<TAG>`.
- Verified features: sandbox Python execution, task decomposition with progress tracking, plugin architecture for domain customization, reflective mid-flight execution adjustment, and support for complex data structures (pandas DataFrames, tabular, high-dimensional data).
- Documented at `https://microsoft.github.io/TaskWeaver/`.

### Flowise

- FlowiseAI's low-code visual builder for LLM workflows, backed by Y Combinator.
- Three visual builder modes: **Assistant** (beginner-friendly, instruction-following + RAG), **Chatflow** (single-agent and advanced LLM flows), **Agentflow** (multi-agent orchestration).
- Install globally with `npm install -g flowise`; start with `npx flowise start` (runs on localhost:3000). Docker recommended for production.
- Backend is Node.js/TypeScript; frontend is React; 100+ data source integrations verified.

### Langflow

- langflow-ai's low-code visual builder for agents and RAG pipelines, Python-first.
- Verified features: visual component editor, model/database-agnostic design, built-in REST API export, and MCP server output.
- Supports all major LLMs and vector databases as interchangeable components.
- Backend is Python (FastAPI); frontend is React; deployable locally or via Docker.

### Composio

- ComposioHQ's tool integration and orchestration platform designed to give AI agents access to external apps and services.
- Verified features: 1000+ pre-built tool/app integrations, MCP server architecture, managed OAuth/auth flows, sandboxed workbench, and IDE integrations (Claude Code, Cursor, VS Code).
- Ships as `composio-core` (Python) plus an npm package for TypeScript.
- Distinct from agent frameworks — functions as an integration layer layered on top of existing agent frameworks.

### Instructor

- jxnl's structured output extraction library; often described as the standard approach for getting typed, validated data out of LLMs.
- Core primitive: decorate a function with a Pydantic model and the library handles schema generation, API call, validation, and automatic retries.
- Verified features: 15+ provider support, streaming (Iterables and Partial), and ports in TypeScript, Go, Ruby, Elixir, and Rust.
- Positioned as a utility library rather than a full agent framework; widely used as a building block inside agent systems.

### Marvin

- PrefectHQ's ambient AI library for structured outputs and task-based workflows.
- Core primitives: `cast`, `classify`, `extract`, `generate` for structured data; Tasks as the fundamental scheduling unit; Threads for multi-step workflow orchestration.
- Verified backend: Pydantic AI; verified persistence: SQLite-based message history.
- Multi-agent assignment is built in; designed to integrate naturally with Prefect workflows.

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
| Vercel AI SDK | https://ai-sdk.dev/ | 2026-06-27 | [official] |
| LlamaIndex | https://developers.llamaindex.ai/python/framework/ | 2026-06-27 | [official] |
| Semantic Kernel | https://learn.microsoft.com/en-us/semantic-kernel/overview/ | 2026-06-27 | [official] |
| smolagents | https://huggingface.co/docs/smolagents/ | 2026-06-27 | [official] |
| Agno | https://docs.agno.com/ | 2026-06-27 | [official] |
| Haystack | https://docs.haystack.deepset.ai/ | 2026-06-27 | [official] |
| BeeAI Framework | https://framework.beeai.dev/ | 2026-06-27 | [official] |
| DSPy | https://dspy.ai/ | 2026-06-27 | [official] |
| Letta | https://docs.letta.com/ | 2026-06-27 | [official] |
| LlamaStack | https://github.com/meta-llama/llama-stack | 2026-06-27 | [github] |
| CAMEL | https://docs.camel-ai.org/ | 2026-06-27 | [official] |
| MetaGPT | https://github.com/FoundationAgents/MetaGPT | 2026-06-27 | [github] |
| TaskWeaver | https://microsoft.github.io/TaskWeaver/ | 2026-06-27 | [official] |
| Flowise | https://docs.flowiseai.com/ | 2026-06-27 | [official] |
| Langflow | https://docs.langflow.org/ | 2026-06-27 | [official] |
| Composio | https://docs.composio.dev/ | 2026-06-27 | [official] |
| Instructor | https://python.useinstructor.com/ | 2026-06-27 | [official] |
| Marvin | https://askmarvin.ai/ | 2026-06-27 | [official] |
