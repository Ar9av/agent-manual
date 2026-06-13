# Trae CN

> ByteDance's AI-first development tool, Chinese-market edition.

**Vendor:** ByteDance | **License:** Proprietary | **Runtime:** Electron

## Links

- Docs: https://docs.trae.ai (shared with Trae global)
- CN-specific portal: https://www.trae.cn
- Agent guide: https://docs.trae.ai/ide/agent
- Rules: https://docs.trae.ai/ide/rules
- MCP: https://docs.trae.ai/ide/model-context-protocol

---

## Overview

Trae CN is the Chinese-market edition of [Trae IDE](../trae/README.md). The architecture, configuration format, and extensibility model are identical to the global version. Key differences:

| Aspect | Trae (Global) | Trae CN |
|--------|--------------|---------|
| Default models | Claude, GPT-4o | Doubao-1.5-Pro, DeepSeek (R1 + V3) |
| API endpoints | Global Anthropic/OpenAI | ByteDance domestic endpoints |
| Compliance | Standard | Chinese data residency |
| UI language | English (primary) | Chinese (primary) |

## Configuration

Identical to [Trae (Global)](../trae/README.md) — same configuration files, rules format, and MCP support.

```
.trae/rules/project_rules.md   # Project-level rules (Markdown)
.trae/rules/user_rules.md      # User-level rules
.trae/mcp.json                 # MCP server configuration
```

The `.trae/rules/` folder can also be placed in any project subdirectory to scope rules to that module. Project rules take precedence over user rules when there is a conflict.

## Hooks

Same situation as Trae global — no native pre/post tool use hooks. Use MCP servers for lifecycle extensibility.

## MCP Transport Types

Trae IDE supports three MCP transport types:

- **stdio** — local process, uses `command` + optional `env`
- **SSE** — remote HTTP, uses `url` + `"type": "sse"`
- **Streamable HTTP** — remote HTTP streaming (added in later releases)

## Supported Models (CN edition, 2026)

Built-in defaults (no additional API key required):
- Doubao-1.5-Pro (ByteDance)
- DeepSeek R1
- DeepSeek V3

Via SiliconCloud integration (requires SiliconCloud API key):
- Qwen (Alibaba) — e.g. Qwen2.5-Coder
- QWQ-32B
- Additional SiliconCloud-hosted models

Additional providers configurable via custom model settings: ❓ full CN provider list not confirmed from live docs.

## Notes

- For full configuration reference, see [Trae (Global)](../trae/README.md).
- Trae CN and Trae Global share the same codebase; regional differences are configuration-only.
- The CN portal `https://www.trae.com.cn` permanently redirects to `https://www.trae.cn`.

## Sources

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| CN portal (canonical) | https://www.trae.cn | 2026-06-13 | [official] |
| Rules documentation | https://docs.trae.ai/ide/rules?_lang=en | 2026-06-13 | [official] |
| MCP documentation | https://docs.trae.ai/ide/model-context-protocol | 2026-06-13 | [official] |
| Add MCP servers | https://docs.trae.ai/ide/add-mcp-servers | 2026-06-13 | [official] |
| Models / custom models | https://docs.trae.ai/ide/models | 2026-06-13 | [official] |
| Agent guide | https://docs.trae.ai/ide/agent | 2026-06-13 | [official] |
| Trae + SiliconCloud integration (Qwen, DeepSeek) | https://www.aibase.com/news/16237 | 2026-06-13 | [news] |
| Trae v1.3.0 MCP + .rules release notes | https://traeide.com/news/6 | 2026-06-13 | [community] |
