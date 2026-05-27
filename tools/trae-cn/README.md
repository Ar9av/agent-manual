# Trae CN

> ByteDance's AI-first development tool, Chinese-market edition.

**Vendor:** ByteDance | **License:** Proprietary | **Runtime:** Electron

## Links

- Docs: https://docs.trae.ai (shared with Trae global)
- CN-specific portal: https://www.trae.com.cn
- Agent guide: https://docs.trae.ai/ide/agent
- Rules: https://docs.trae.ai/ide/rules

---

## Overview

Trae CN is the Chinese-market edition of [Trae IDE](../trae/README.md). The architecture, configuration format, and extensibility model are identical to the global version. Key differences:

| Aspect | Trae (Global) | Trae CN |
|--------|--------------|---------|
| Default models | Claude, GPT-4o | Doubao, Qwen, DeepSeek |
| API endpoints | Global Anthropic/OpenAI | ByteDance domestic endpoints |
| Compliance | Standard | Chinese data residency |
| UI language | English (primary) | Chinese (primary) |

## Configuration

Identical to [Trae (Global)](../trae/README.md) — same configuration files, rules format, and MCP support.

```
project_rules.md   # Project-level rules (Markdown)
user_rules.md      # User-level rules
.trae/mcp.json     # MCP server configuration
```

## Hooks

Same situation as Trae global — no native pre/post tool use hooks. Use MCP servers for lifecycle extensibility.

## Supported Models (CN edition, 2026)

- Doubao (ByteDance)
- Qwen (Alibaba)
- DeepSeek
- (International models may require VPN or separate API keys)

## Notes

- For full configuration reference, see [Trae (Global)](../trae/README.md).
- Trae CN and Trae Global share the same codebase; regional differences are configuration-only.

## Sources (Official)

| Topic | URL |
|-------|-----|
| Trae CN portal | https://www.trae.com.cn |
| Agent guide (shared docs) | https://docs.trae.ai/ide/agent |
| Rules (shared docs) | https://docs.trae.ai/ide/rules |
| Models | https://docs.trae.ai/ide/models |
