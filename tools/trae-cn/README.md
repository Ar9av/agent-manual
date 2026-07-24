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
| Default models | Claude, GPT-4o | Dola-Seed-2.0-Code, GPT-5.x variants |
| API endpoints | Global Anthropic/OpenAI | ByteDance domestic endpoints |
| Compliance | Standard | Chinese data residency |
| UI language | English (primary) | Chinese (primary) |

## Configuration

Identical to [Trae (Global)](../trae/README.md) — same configuration files, rules format, and MCP support.

```
.trae/rules/<name>.md          # Project-level rules (Markdown, user-defined filenames)
.trae/mcp.json                 # MCP server configuration
```

The `.trae/rules/` folder can also be placed in any project subdirectory to scope rules to that module. Project rules take precedence over user rules when there is a conflict. Rule filenames within `.trae/rules/` are user-defined (e.g. `global-style.md`, `react-best-practices.md`); there is no canonical `user_rules.md` filename.

## Hooks

> **Update (2026-07-23):** Trae global's docs now show a native "Hooks" entry in the sidebar (see [Trae (Global) README](../trae/README.md#hooks)). Since Trae CN shares the same codebase, this likely also applies to Trae CN, but it was not independently confirmed on the CN docs portal during this audit — treat as ❓ pending verification, rather than assuming the old MCP-only workaround is still the only option.

## SOLO Mode

Trae CN supports SOLO mode, the same autonomous multi-agent mode available in Trae global. In SOLO mode the agent plans and executes multi-step tasks independently, spawning sub-agents as needed, with minimal human interruption.

## MCP Transport Types

Trae IDE supports three MCP transport types:

- **stdio** — local process, uses `command` + optional `env`
- **SSE** — remote HTTP, uses `url` + `"type": "sse"`
- **Streamable HTTP** — remote HTTP streaming

## Supported Models (CN edition, 2026)

Built-in defaults (no additional API key required):
- Dola-Seed-2.0-Code (ByteDance)
- GPT-5.x variants
- MiniMax-M3 / MiniMax-M2.7
- Kimi-K2.5
- DeepSeek-V3.2
- Gemini-3.1-Pro-Preview
- Gemini-3-Flash-Preview

Via preset CN provider integrations (each requires the respective provider's API key):
- Silicon Flow CN
- MiniMax CN
- Kimi CN
- Volcano Engine
- Alibaba Cloud
- Tencent Cloud
- Bigmodel
- Gitee
- PPIO
- Infinigence AI CN

Additional providers configurable via custom model settings.

## Notes

- For full configuration reference, see [Trae (Global)](../trae/README.md).
- Trae CN and Trae Global share the same codebase; regional differences are configuration-only.
- The CN portal `https://www.trae.com.cn` permanently redirects to `https://www.trae.cn`.

## Sources

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| CN portal (canonical) | https://www.trae.cn | 2026-07-23 | [official] — `trae.com.cn` still 301-redirects here |
| Rules documentation | https://docs.trae.ai/ide/rules?_lang=en | 2026-06-13 | [official] |
| MCP documentation | https://docs.trae.ai/ide/model-context-protocol | 2026-07-23 | [official] — transport types unchanged |
| Add MCP servers | https://docs.trae.ai/ide/add-mcp-servers | 2026-07-23 | [official] — config format unchanged |
| Models / custom models | https://docs.trae.ai/ide/models | 2026-06-13 | [official] ❓ — CN-specific model list not independently re-verified this pass; see [Trae Global README](../trae/README.md) for the confirmed current global built-in list, which now overlaps heavily with the CN list below |
| Agent guide | https://docs.trae.ai/ide/agent | 2026-06-13 | [official] |
| Trae v1.3.0 MCP + .rules release notes | https://traeide.com/news/6 | 2026-06-13 | [community] |
