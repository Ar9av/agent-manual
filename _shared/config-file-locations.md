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

## Skills Directories

These paths are sourced directly from installer scripts and verified directories for skill file locations.

| Tool | Global Skills Path | Project Skills Path |
|------|-------------------|---------------------|
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| Claude Desktop / claude.ai app | _upload-only_ (Settings → Capabilities → Skills) | _upload-only_ — no filesystem discovery |
| Codex CLI | `~/.agents/skills/` | `.agents/skills/` |
| **Google Antigravity** | `~/.agents/skills/` | `.agents/skills/` |
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
| Google Antigravity | `.agent/rules/` and `.agent/workflows/` | Declarative Rules/Workflows |

## Git-Ignored / Personal Override Files

Files meant for personal settings that shouldn't be committed:

| Tool | File |
|------|------|
| Claude Code | `.claude/settings.local.json` |
| Devin CLI | `config.local.json` (any file with `.local.`) |
| Codex CLI | `~/.codex/` (user-level) |

## Sources (Official)

| Tool | Config docs URL |
|------|----------------|
| Claude Code | https://docs.anthropic.com/en/docs/claude-code/settings |
| Codex CLI | https://developers.openai.com/codex/configuration |
| Gemini CLI | https://geminicli.com/docs/configuration |
| Kiro | https://kiro.dev/docs/configuration |
| Kimi Code | https://moonshotai.github.io/kimi-code/configuration |
| Factory Droid | https://docs.factory.ai/configuration |
| Hermes | https://hermes-agent.nousresearch.com/docs/configuration |
| Cursor | https://cursor.com/docs/rules |
| Pi Agent | https://github.com/earendil-works/pi |
| Trae / Trae CN | https://docs.trae.ai/ide/ide-settings-overview |
| OpenCode | https://opencode.ai/docs/config/ |
| GitHub Copilot | https://code.visualstudio.com/docs/copilot/overview |
| Devin CLI | https://docs.devin.ai/ |
| Aider | https://aider.chat/docs/config.html |
