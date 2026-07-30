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
| Strands Agents SDK | Strands Agents | AWS / Amazon | Agent SDK / runtime | Python, TypeScript | `pip install strands-agents strands-agents-tools` |
| AG2 | AG2 | ag2ai (AutoGen community fork) | Multi-agent framework | Python | `pip install 'ag2[openai]'` |
| Griptape | Griptape | Griptape AI | Multi-agent / agentic workflow framework | Python | `pip install griptape` |
| Swarms | Swarms | kyegomez / Swarm Corporation | Multi-agent orchestration framework | Python | `pip3 install -U swarms` |
| Atomic Agents | Atomic Agents | Eigenwise (formerly BrainBlend-AI) | Lightweight, schema-driven agent framework | Python | `pip install atomic-agents` |
| uAgents | uAgents | Fetch.ai | Agent SDK / runtime (decentralized agents) | Python | `pip install uagents` |
| Parlant | Parlant | Emcie | Conversational agent behavior/guardrail framework | Python | `pip install parlant` |
| Restack | Restack | Restack | Agent workflow orchestration engine (Temporal-based) | Python | `pip install restack_ai` |
| Inngest AgentKit | AgentKit | Inngest | Agent SDK / multi-agent toolkit | TypeScript | `npm install @inngest/agent-kit inngest` |
| Relevance AI | Relevance AI | Relevance AI | Low-code AI agent platform | Python (SDK) | `pip install relevanceai` |
| Dify | Dify | LangGenius | Low-code LLM app/agent platform | Python, TypeScript | `git clone https://github.com/langgenius/dify` + Docker Compose |
| Botpress | Botpress | Botpress Inc. | Agent SDK / hybrid low-code conversational agent platform | TypeScript | `npm install --save @botpress/sdk` |
| elizaOS | elizaOS | elizaOS (rebranded from ai16z) | Agentic operating system / multi-agent runtime | TypeScript | `bun add -g elizaos@beta` |
| LangChain | LangChain | LangChain, Inc. | LLM application / component framework | Python, TypeScript | `pip install langchain` |
| AutoGPT | AutoGPT Platform | Significant Gravitas | Agent building/deployment platform | Python (core), TypeScript (UI) | `curl -fsSL https://setup.agpt.co/install.sh \| bash` |
| n8n | n8n | n8n GmbH | Workflow automation platform with AI Agent nodes | TypeScript | `npm install n8n -g` |
| Herdr | Herdr | herdrdev | Terminal-based agent multiplexer / orchestration platform | Rust | `curl -fsSL https://herdr.dev/install.sh \| sh` |

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
| Strands Agents SDK | AWS's model-driven agent SDK | Decorator-based tools (`@tool`), conversation-manager memory (sliding-window/summarizing), multi-agent patterns (Agent-as-Tool, Swarm), native MCP client support, hook-based guardrails, built-in decision tracing, multi-provider (Bedrock, Anthropic, OpenAI, Gemini) | https://strandsagents.com/ |
| AG2 | AutoGen community fork focused on agent networks | Protocol-driven Network (Hub + typed channels), `@tool`-decorated tools, persistent memory/knowledge stores, human-in-the-loop, structured outputs, telemetry, eval framework, MCP integration | https://ag2.ai/ |
| Griptape | Modular Python framework for agents, pipelines, and workflows | Agents/pipelines/workflows, conversation/task/meta memory, large Driver ecosystem (LLM/embedding/vector/search), RAG/extraction/eval engines, Rulesets, MCP tool support, observability drivers | https://docs.griptape.ai/ |
| Swarms | Enterprise-grade multi-agent orchestration framework | 60+ orchestration structures, agents, tool integration, multiple memory implementations, native MCP support, observability/dashboard, interop with LangChain/AutoGen/CrewAI agents | https://github.com/kyegomez/swarms |
| Atomic Agents | Schema-first, lightweight agent-pipeline framework | Schema-validated agent I/O (Instructor + Pydantic), pluggable memory (`BaseChatHistory`), tool library ("Atomic Forge"), multi-agent chaining via schema alignment, hooks for monitoring/retries, multimodal | https://github.com/Eigenwise/atomic-agents |
| uAgents | Fetch.ai's SDK for decentralized autonomous agents | Agents register on the Almanac, cryptographically secured agent-to-agent messaging, multi-agent composition, LLM-wrapped agents via ASI:One | https://uagents.fetch.ai/docs |
| Parlant | Behavioral control framework for conversational agents | Guidelines/Journeys/Relationships for behavioral control, context-aware Tools, Glossary/variables for state, full OpenTelemetry tracing; used in regulated-industry production | https://github.com/emcie-co/parlant |
| Restack | Durable execution engine for agent workflows | Event-driven workflows on a Temporal-based durable execution core with Kubernetes deploy, agents, tool/integration connectors, MCP support, real-time logs/audit trails | https://docs.restack.io/ |
| Inngest AgentKit | TypeScript multi-agent toolkit built on Inngest | Single agents through multi-agent "Networks," Router (deterministic/autonomous), shared Network State (memory), Tools, MCP integration, visual debugging via Inngest dev server | https://agentkit.inngest.com/ |
| Relevance AI | Hosted "AI workforce" agent platform | Visual builder (Invent) plus Python SDK, multi-agent coordination (per-agent memory/tools/scope), MCP support, SOC 2/SSO enterprise controls, 2000+ integrations | https://relevanceai.com/docs/changelog |
| Dify | Low-code platform combining visual workflows, RAG, and agents | Visual workflow canvas plus RAG and agent capabilities, LLM function-calling/ReAct agents with 50+ prebuilt tools, MCP Registry, workflow-based context/memory, observability integrations (Opik, Langfuse, Arize Phoenix) | https://github.com/langgenius/dify |
| Botpress | Code-first and visual platform for conversational agents | Code-first ADK/SDK plus a visual Studio for conversational agents, channel/service integrations, hosted Cloud + API | https://botpress.com/docs |
| elizaOS | Plugin-based agentic operating system | Plugin/app-based multi-agent runtime, personal-assistant memory, document RAG, browser/computer-use tooling, non-custodial crypto wallet support, MCP-related app catalog; rebranded from "ai16z" | https://github.com/elizaOS/eliza |
| LangChain | Component library for LLM applications | Model I/O, retrieval, vector store, and chat abstractions with 1000+ integrations; official docs position it as complementary to LangGraph (LangChain = components, LangGraph = orchestration) | https://github.com/langchain-ai/langchain |
| AutoGPT | Agent building and deployment platform | Conversational/visual agent builder, scheduled/triggered/on-demand execution, 45+ integrations, 100+ model support, agent marketplace, dashboard; the older `classic/` autonomous-loop package is retired | https://github.com/Significant-Gravitas/AutoGPT |
| n8n | Workflow automation platform with a native AI Agent node | LangChain-powered "Tools Agent" node consolidating ReAct/Plan-and-Execute patterns, dedicated memory nodes (Simple/Window Buffer Memory); core identity remains general workflow automation, not a dedicated agent framework | https://docs.n8n.io/ |
| Herdr | Terminal multiplexer that runs and manages many coding agents from one session | Persistent PTY sessions that survive detach/SSH disconnect, real terminal panes with mouse support/tabs/workspaces, per-agent state tracking (blocked/working/done/idle), pure socket API so agents can spawn panes and coordinate with each other, plugin marketplace, single Rust binary; works with 150+ terminal-based coding agents out of the box | https://herdr.dev/ |

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

### Strands Agents SDK

- AWS's official model-driven agent SDK, open-sourced under `strands-agents` on GitHub.
- Verified features: decorator-based tools, sliding-window and summarizing conversation managers for memory, Agent-as-Tool and Swarm multi-agent patterns, native MCP client support, hook-based guardrails, and built-in decision tracing.
- Multi-provider by design: works with Amazon Bedrock, Anthropic, OpenAI, and Gemini models.

### AG2

- Community fork of Microsoft's AutoGen, now maintained by the independent `ag2ai` organization under the `ag2.ai` brand — explicitly distinct from Microsoft's `autogen-agentchat` line already tracked above.
- v1.0 rearchitected the framework around a protocol-driven Network (a Hub plus typed channels) rather than the older group-chat model.
- Verified features: `@tool`-decorated tools, persistent memory/knowledge stores, human-in-the-loop, structured outputs, telemetry, an eval framework, and MCP integration (including running AG2 itself as an MCP server).

### Griptape

- Griptape AI's modular Python framework for building agents, pipelines, and workflows around a "Driver" abstraction layer.
- Verified features: conversation/task/meta memory, a large Driver ecosystem (LLM, embedding, vector store, web search), RAG and extraction/eval engines, Rulesets for behavior constraints, MCP tool support (v1.10.0+), and observability drivers.
- Positions itself as enterprise-oriented, with structured data handling as a first-class concern.

### Swarms

- kyegomez / Swarm Corporation's enterprise-grade multi-agent orchestration framework, one of the older and more actively iterated community agent frameworks.
- Verified features: 60+ named orchestration structures, tool integration, multiple memory implementations, native MCP support, an observability dashboard, and interoperability with agents built on LangChain, AutoGen, and CrewAI.
- GitHub's Releases tab looks stale relative to the live commit/PyPI history — treat commit activity, not the Releases tab, as the freshness signal.

### Atomic Agents

- Eigenwise's (formerly BrainBlend-AI) lightweight, schema-first agent framework built on Instructor and Pydantic.
- Verified features: schema-validated agent input/output, pluggable chat-history memory, a tool library called "Atomic Forge," multi-agent chaining via schema alignment, hooks for monitoring and retries, and multimodal support.
- Deliberately minimal core, designed to stay auditable rather than accumulate framework magic.

### uAgents

- Fetch.ai's SDK for building autonomous, decentralized agents that register on Fetch.ai's Almanac and communicate over cryptographically secured messaging.
- Verified features: multi-agent composition and LLM-wrapped agents via ASI:One.
- No documented built-in memory, guardrail, or MCP layer — it is closer to an agent-to-agent networking/identity SDK than a full agent-application framework.

### Parlant

- Emcie's framework for controlling conversational agent behavior in production, aimed at regulated industries.
- Core primitives: Guidelines, Journeys, and Relationships for behavioral control, plus context-aware Tools and a Glossary/variables system for state.
- Verified features: full OpenTelemetry tracing. No documented multi-agent orchestration or MCP support.

### Restack

- Restack's agent workflow orchestration engine, built on Temporal-style durable execution with Kubernetes-based deployment.
- Verified features: event-driven workflows, agents, tool/integration connectors, MCP support, and real-time logs/audit trails.
- Positions itself as infrastructure for long-running, fault-tolerant agent workflows rather than a prompting/agent-loop library.

### Inngest AgentKit

- Inngest's TypeScript toolkit for building single agents and multi-agent "Networks" on top of the Inngest event-driven runtime.
- Verified features: a Router supporting both deterministic and autonomous routing, shared Network State as memory, Tools, MCP integration, and visual debugging via the Inngest dev server.
- Ships as two packages together: `@inngest/agent-kit` and `inngest`.

### Relevance AI

- Relevance AI's hosted "AI workforce" platform, combining a visual builder (Invent) with a Python SDK.
- Verified features: multi-agent coordination with per-agent memory/tools/scope, MCP support, SOC 2/SSO enterprise controls, and 2000+ integrations.
- More of a managed agent platform than an open-source framework — no self-hosted install path documented beyond the SDK client.

### Dify

- LangGenius's low-code platform combining a visual workflow canvas, RAG pipelines, and agent capabilities.
- Verified features: LLM function-calling/ReAct agents with 50+ prebuilt tools, an MCP Registry, workflow-based context/memory, and observability integrations (Opik, Langfuse, Arize Phoenix).
- No pip/npm package — self-hosted via `git clone` plus Docker Compose, or used as a hosted cloud product.

### Botpress

- Botpress Inc.'s platform for conversational agents, offering both a code-first Agent Development Kit/SDK and a visual Studio.
- Verified features: channel/service integrations and a hosted Cloud + API.
- Memory, multi-agent orchestration, MCP, and guardrail primitives were not explicitly confirmed in the docs checked — treat as conversational-agent-focused rather than a general agent framework.

### elizaOS

- Formerly branded "ai16z," now a fully rebranded "agentic operating system" maintained by the elizaOS organization.
- Verified features: a plugin/app-based multi-agent runtime, personal-assistant memory, document RAG, browser/computer-use tooling, non-custodial crypto wallet support, and an MCP-related app catalog.
- Retains roots in the Web3/crypto-agent space but has broadened scope considerably since its ai16z origins.

### LangChain

- LangChain, Inc.'s original component library for building LLM applications — distinct from LangGraph (already tracked above), which handles orchestration.
- Verified features: model I/O, retrieval, vector store, and chat-model abstractions across 1000+ integrations.
- Official docs explicitly position LangChain and LangGraph as complementary rather than competing: LangChain supplies components, LangGraph supplies the stateful orchestration layer.

### AutoGPT

- Significant Gravitas's agent building and deployment platform — the actively maintained "AutoGPT Platform," not the original 2023 autonomous-loop experiment.
- The original `classic/` package (the famous "give it a goal and let it loop" AutoGPT) is explicitly retired by the maintainers as "an experiment that has concluded."
- Verified features on the current platform: a conversational/visual agent builder, scheduled/triggered/on-demand execution, 45+ integrations, 100+ model support, an agent marketplace, and a dashboard.

### n8n

- n8n GmbH's general-purpose workflow automation platform, included here because it ships an official, LangChain-powered AI Agent node rather than treating agents as an afterthought.
- Verified features: a "Tools Agent" node consolidating ReAct/Plan-and-Execute-style patterns, and dedicated memory nodes (Simple Memory, Window Buffer Memory).
- Its core identity remains general workflow automation — list this as an agent-capable automation platform, not a dedicated agent framework.

### Herdr

- herdrdev's terminal-based multiplexer for running many coding agents (Claude Code, Codex, OpenCode, Cursor, Copilot, and 150+ others) from a single terminal, including over SSH.
- Verified features: persistent sessions that outlive the terminal/laptop closing, real PTY panes with mouse support/tabs/workspaces, per-agent state tracking (blocked/working/done/idle), a mobile-friendly remote UI, and a pure socket API letting agents spawn panes and coordinate with each other.
- Distinct from the agent frameworks/SDKs above — it doesn't build or run agent logic itself, it's session/process orchestration for existing terminal-based agents. Ships as a single Rust binary; extensible via a community plugin marketplace (GitHub-tagged `herdr-plugin`).
- No pip/npm package — install via curl script, Homebrew, mise, or direct binary download; Windows support is beta.

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
| Strands Agents SDK | https://strandsagents.com/ | 2026-07-24 | [official] |
| AG2 | https://github.com/ag2ai/ag2 | 2026-07-24 | [github] |
| Griptape | https://docs.griptape.ai/ | 2026-07-24 | [official] |
| Swarms | https://github.com/kyegomez/swarms | 2026-07-24 | [github] |
| Atomic Agents | https://github.com/Eigenwise/atomic-agents | 2026-07-24 | [github] |
| uAgents | https://uagents.fetch.ai/docs | 2026-07-24 | [official] |
| Parlant | https://github.com/emcie-co/parlant | 2026-07-24 | [github] |
| Restack | https://docs.restack.io/ | 2026-07-24 | [official] |
| Inngest AgentKit | https://agentkit.inngest.com/ | 2026-07-24 | [official] |
| Relevance AI | https://relevanceai.com/docs/changelog | 2026-07-24 | [official] |
| Dify | https://github.com/langgenius/dify | 2026-07-24 | [github] |
| Botpress | https://botpress.com/docs | 2026-07-24 | [official] |
| elizaOS | https://github.com/elizaOS/eliza | 2026-07-24 | [github] |
| LangChain | https://github.com/langchain-ai/langchain | 2026-07-24 | [github] |
| AutoGPT | https://github.com/Significant-Gravitas/AutoGPT | 2026-07-24 | [github] |
| n8n | https://docs.n8n.io/ | 2026-07-24 | [official] |
| Herdr | https://herdr.dev/ | 2026-07-30 | [official] |
