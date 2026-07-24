# Configuration File Locations

Quick reference for where each agentic tool stores its configuration.

## Global (User-Level) Config

| Tool | Path |
|------|------|
| Claude Code | `~/.claude/settings.json` |
| Codex CLI | `~/.codex/config.toml` |
| Gemini CLI | `~/.gemini/settings.json` |
| Hermes | `~/.hermes/config.yaml` |
| Kiro | `~/.kiro/config.json` |
| Pi Agent | `~/.pi/agent/settings.json` |
| Devin CLI | `~/.config/devin/config.json` |
| OpenCode | `~/.config/opencode/opencode.json` |
| Factory Droid | `~/.factory/settings.json` |
| Kimi Code | `~/.kimi-code/config.json` |
| Cursor | `~/.cursor/settings.json` |
| Trae | IDE Settings UI |
| OpenClaw | `~/.openclaw/openclaw.json` |
| GitHub Copilot | `~/.config/gh-copilot/config.json` |
| Aider | `~/.aider.conf.yml` |
| Google Antigravity | `~/.config/antigravity/config.toml` |
| Amazon Q Developer CLI | `~/.aws/amazonq/cli-agents/` (per-agent JSON) |
| Amp | `~/.config/amp/settings.json` |
| Goose | `~/.config/goose/config.yaml` (Windows: `%APPDATA%\Block\goose\config\config.yaml`) |
| OpenHands | `~/.openhands/settings.json` |
| Crush | `~/.config/crush/crush.json` |
| Continue CLI | `~/.continue/config.yaml` |
| Auggie CLI | `~/.augment/settings.json` (system-wide: `/etc/augment/settings.json`) |
| Qwen Code | `~/.qwen/settings.json` |
| Warp | ❓ GUI Settings (no documented global settings file; rules via `~/.warp/.mcp.json` for MCP) |

## Project-Level Config

| Tool | Path |
|------|------|
| Claude Code | `.claude/settings.json` |
| Codex CLI | `.codex/config.toml` |
| Gemini CLI | `.gemini/settings.json` |
| Hermes | `cli-config.yaml` |
| Kiro | `.kiro/config.json` |
| Pi Agent | `.pi/settings.json` |
| Devin CLI | `config.json` + `hooks.v1.json` |
| OpenCode | `opencode.json` |
| Factory Droid | `.factory/settings.json` |
| Kimi Code | `.kimi-code/config.json` |
| Cursor | `.cursor/` directory |
| Trae | `.trae/rules/project_rules.md` |
| OpenClaw | `openclaw.json` |
| Aider | `.aider.conf.yml` |
| Google Antigravity | `.agents/hooks.json` |
| Amazon Q Developer CLI | `.amazonq/cli-agents/` (per-agent JSON) |
| Amp | `.amp/settings.json` |
| Goose | ❓ no documented separate project file — `.goosehints` provides project instructions |
| OpenHands | `.openhands/hooks.json` (also `.openhands/settings.json`) |
| Crush | `crush.json` / `.crush.json` |
| Continue CLI | `.continue/config.yaml` (or `--config` override) |
| Auggie CLI | `.augment/settings.json` (+ git-ignored `.augment/settings.local.json`) |
| Qwen Code | `.qwen/settings.json` |
| Warp | `.warp/.mcp.json` (MCP only; rules via `WARP.md`/`AGENTS.md`) |

## MCP Config Locations

| Tool | Path |
|------|------|
| Claude Code | `.claude/mcp.json` or via `settings.json` |
| Cursor | `.cursor/mcp.json` or `~/.cursor/mcp.json` |
| Gemini CLI | `settings.json` (`mcpServers` key) |
| Kiro | `.kiro/config.json` (`mcpServers` key) |
| Devin CLI | `config.json` (`mcpServers` key) |
| OpenCode | `opencode.json` (`mcp.servers` key) |
| Trae | `.trae/mcp.json` |
| GitHub Copilot | `.github/mcp.json` |
| Kimi Code | `config.json` (`mcpServers` key) |
| Google Antigravity | `~/.gemini/config/mcp_config.json` or `~/.gemini/antigravity-cli/mcp_config.json` |
| Amazon Q Developer CLI | Per-agent `mcpServers` field, or legacy `~/.aws/amazonq/mcp.json` / `.amazonq/mcp.json` |
| Amp | `settings.json` (`amp.mcpServers` key) |
| Goose | `config.yaml` (`extensions:` key) |
| OpenHands | `~/.openhands/mcp.json` |
| Crush | `crush.json` (`mcp` key) |
| Continue CLI | `.continue/mcpServers/*.yaml` or `*.json` |
| Auggie CLI | `settings.json` (`mcpServers` key), or `--mcp-config` |
| Qwen Code | `settings.json` (`mcpServers` key) |
| Warp | `.warp/.mcp.json` (project) or `~/.warp/.mcp.json` (global) |

## Skills Directories

These paths are sourced directly from installer scripts and verified directories for skill file locations.

| Tool | Global Skills Path | Project Skills Path |
|------|-------------------|---------------------|
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| Claude Desktop / claude.ai app | _upload-only_ (Settings → Capabilities → Skills) | _upload-only_ — no filesystem discovery |
| Codex CLI | `~/.agents/skills/` | `.agents/skills/` |
| **Google Antigravity** | `~/.gemini/skills/` or `~/.gemini/antigravity-cli/skills/` | `.agents/skills/` |
| OpenCode | `~/.config/opencode/skills/` | `.config/opencode/skills/` |
| GitHub Copilot | `~/.copilot/skills/` | `.copilot/skills/` |
| OpenClaw | `~/.openclaw/skills/` | `.openclaw/skills/` |
| Factory Droid | `~/.factory/skills/` | `.factory/skills/` |
| Trae | `~/.trae/skills/` | `.trae/skills/` |
| Trae CN | `~/.trae-cn/skills/` | `.trae-cn/skills/` |
| Hermes | `~/.hermes/skills/` | `.hermes/skills/` |
| Kiro | `~/.kiro/skills/` | `.kiro/skills/` |
| Pi Agent | `~/.pi/agent/skills/` | `.pi/agent/skills/` |
| Kimi Code | `~/.kimi-code/skills/` | `.kimi-code/skills/` |
| Devin CLI | `~/.config/devin/skills/` | `.devin/skills/` |
| Aider | `~/.aider/` | `.aider/` (non-standard, no skills convention) |
| Gemini CLI | `~/.gemini/skills/` | `.gemini/skills/` |
| Amp | `~/.agents/skills/` | `.agents/skills/` |
| Crush | `~/.config/crush/skills/` (also reads `.claude/skills/` for compat) | `.crush/skills/` |
| Warp | ❓ not documented (global) | `.agents/skills/` |
| OpenHands | ❓ not documented (global) | `.openhands/microagents/` (being renamed "skills") |

**Key finding:** Antigravity and Codex both use `.agents/skills/` — they share the same skill registration convention.

**Key finding:** Hermes uses the same skill format as OpenClaw (`skill-claw.md`) — they have essentially identical skill architectures.

**Key finding:** Kimi Code uses the same skill format as Claude Code (`skill.md`).

**Key finding:** The Claude **Desktop / claude.ai app** shares the `SKILL.md` format with Claude Code but has **no filesystem auto-discovery** — skills are added manually by uploading a `.zip` (with `SKILL.md` at the zip root) via Settings → Capabilities → Skills, and require the Code-execution sandbox to be enabled. A skill sitting in `~/.claude/skills/` is visible to Claude Code but *not* to the app.

## Hook / Extension Directories

| Tool | Path | Type |
|------|------|------|
| Claude Code | `.claude/commands/` | Skills (Markdown) |
| OpenCode | `.opencode/plugins/` | JS plugins |
| Kiro | `.kiro/agents/` | Agent JSON |
| Pi Agent | `~/.pi/agent/hook/` (global) or `./.pi/hook/` (project) | Shell hooks |
| Factory Droid | `.factory/plugins/` | Plugins |
| Factory Droid | `.factory/droids/` | Custom droids |
| Cursor | `.cursor/rules/` | MDC rule files |
| OpenClaw | `hooks/` | Shell scripts |
| Google Antigravity | `.agents/rules/` and `.agents/workflows/` | Declarative Rules/Workflows |
| Amp | `.amp/plugins/*.ts` (project) or `~/.config/amp/plugins/*.ts` (global) | TypeScript Plugin API |
| Goose | `~/.agents/plugins/<name>/` (global) or `<project>/.agents/plugins/<name>/` (project); hook config at `<plugin>/hooks/hooks.json` | Plugin scripts (Open Plugins spec) — confirmed by live test 2026-07-23 |
| OpenHands | `.openhands/hooks.json` | JSON hooks config |

## Git-Ignored / Personal Override Files

Files meant for personal settings that shouldn't be committed:

| Tool | File |
|------|------|
| Claude Code | `.claude/settings.local.json` |
| Devin CLI | `config.local.json` (any file with `.local.`) |
| Codex CLI | `~/.codex/` (user-level) |
| Auggie CLI | `.augment/settings.local.json` |

## Sources

| Tool | Config docs URL | Fetched | Label |
|------|----------------|---------|-------|
| Claude Code | https://docs.anthropic.com/en/docs/claude-code/settings | 2026-06-26 | [official] |
| Codex CLI | https://developers.openai.com/codex/configuration | 2026-06-26 | [official] |
| Gemini CLI | https://geminicli.com/docs/configuration | 2026-06-26 | [official] |
| Kiro | https://kiro.dev/docs/configuration | 2026-06-26 | [official] |
| Kimi Code | https://moonshotai.github.io/kimi-code/configuration | 2026-06-26 | [official mirror] |
| Factory Droid | https://docs.factory.ai/configuration | 2026-06-26 | [official] |
| Hermes | https://hermes-agent.nousresearch.com/docs/configuration | 2026-06-26 | [official] |
| Cursor | https://cursor.com/docs/rules | 2026-06-26 | [official] |
| Pi Agent | https://github.com/earendil-works/pi | 2026-06-26 | [github] |
| Trae / Trae CN | https://docs.trae.ai/ide/ide-settings-overview | 2026-06-26 | [official] |
| OpenCode | https://opencode.ai/docs/config/ | 2026-06-26 | [official] |
| GitHub Copilot | https://code.visualstudio.com/docs/copilot/overview | 2026-06-26 | [official] |
| Devin CLI | https://docs.devin.ai/ | 2026-06-26 | [official] |
| Aider | https://aider.chat/docs/config.html | 2026-06-26 | [official] |
| Amazon Q Developer CLI | https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-installing.html | 2026-07-23 | [official] |
| Amp | https://ampcode.com/manual | 2026-07-23 | [official] |
| Goose | https://goose-docs.ai/ | 2026-07-23 | [official] |
| OpenHands | https://docs.openhands.dev/ | 2026-07-23 | [official] |
| Crush | https://github.com/charmbracelet/crush | 2026-07-23 | [github] |
| Continue CLI | https://docs.continue.dev/cli/quickstart | 2026-07-23 | [official] |
| Auggie CLI | https://docs.augmentcode.com/cli/overview | 2026-07-23 | [official] |
| Qwen Code | https://qwenlm.github.io/qwen-code-docs/en/users/overview | 2026-07-23 | [official] |
| Warp | https://docs.warp.dev/agent-platform/ | 2026-07-23 | [official] |
