# Warp (Agent Mode)

> An agentic terminal application ("Agentic Development Environment") with an embedded AI agent, plus a companion `oz` CLI for headless/cloud agent runs.

**Vendor:** Warp (warp.dev) | **License:** Warp client is source-available under AGPL-3.0 (UI framework under MIT); Oz cloud-orchestration platform and cloud infra are **not** open source | **Runtime:** Native desktop app (Rust-based terminal) + separate `oz` CLI binary for headless/CI use

## Links

- Docs home: https://docs.warp.dev/
- Agent platform overview: https://docs.warp.dev/agent-platform/
- Oz CLI reference: https://docs.warp.dev/reference/cli/
- GitHub (client): https://github.com/warpdotdev/warp
- Downloads: https://www.warp.dev/download

---

## What Warp Is (and Isn't)

Warp is **primarily a GUI terminal application**, not a CLI-only tool like the other entries in this catalog. Its "Agent Mode" is an AI agent embedded in that terminal UI — you install and run Warp itself (macOS/Linux/Windows desktop app), not a standalone `npm install -g` style CLI, for the interactive experience.

However, Warp also ships **`oz`**, a separate command-line tool for running/managing Warp's **cloud agents** headlessly — "the command-line tool for running and managing Warp's cloud agents from any terminal, script, or CI pipeline," usable in CI pipelines, headless servers, VMs, Codespaces, and containers via `WARP_API_KEY` non-interactive auth. `oz agent run` executes locally; `oz agent run-cloud` dispatches to Warp's cloud infrastructure. See the Notes section for the full CLI-vs-GUI distinction. [official]

## Installation

**Warp desktop app (GUI, required for interactive Agent Mode):**
```sh
# macOS (Homebrew)
brew install --cask warp

# macOS/Linux/Windows: download installer from
# https://www.warp.dev/download
# Linux also ships .deb, .rpm, and AppImage packages
```

**`oz` CLI (headless/cloud agents):**
```sh
# macOS (Homebrew)
brew install --cask oz

# Linux: apt / yum / pacman packages available
# Windows: bundled with the Warp app install
# The oz binary is also auto-distributed alongside the Warp desktop app
```

**Non-interactive auth (CI/headless):**
```sh
export WARP_API_KEY="wk-xxx..."
oz agent run --prompt "analyze this codebase"
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `AGENTS.md` | Project (root or subdirectory) | Preferred rule/instruction file — natural-language project rules for agents |
| `WARP.md` | Project (root or subdirectory, legacy) | Legacy rule file; still supported. If both `WARP.md` and `AGENTS.md` exist in the same directory, `WARP.md` takes priority |
| `.warp/.mcp.json` | Project | Project-scoped MCP server config |
| `~/.warp/.mcp.json` | Global | Global MCP server config |
| `.agents/skills/<name>/SKILL.md` | Project (recommended path) | Custom Skill definitions (also reads `.warp/skills/`, `.claude/skills/`, `.cursor/skills/` for compatibility) |
| `~/.agents/skills/<name>/SKILL.md` | Global | Global Skill definitions |
| Warp Drive (cloud-synced, not a local file) | Global/Team | Global Rules, Workflows, Notebooks, Environment Variables, Agent Profiles — managed via Settings UI, not plain files |

Filenames for rule files **must be all-caps** (`AGENTS.md`/`WARP.md`) — lowercase variants are not recognized. [official]

## Instruction File

Warp automatically applies the `AGENTS.md` (or `WARP.md`) found in the project root and in the current working directory, with best-effort inclusion of relevant subdirectory rule files. Two rule tiers exist:

- **Global Rules** — set via Warp Drive → Personal → Rules (workspace-wide, cloud-synced, not a repo file).
- **Project Rules** — the `AGENTS.md`/`WARP.md` file(s) in the codebase.

**Precedence order** (most specific wins):
1. Current subdirectory's project rules file
2. Root directory's project rules file
3. Global Rules

❓ A discrete `rules_enabled` boolean/flag was referenced in some third-party summaries but was **not confirmed** on an official docs page during this research — treat as unverified. Rules can be toggled off via Settings → Agents → Knowledge → "Warp Drive as Agent Mode Context" (enabled by default), which governs whether Warp Drive objects (Rules, Workflows, Notebooks, MCP Servers, Environment Variables) are used as context at all. [official]

## Hooks

**Warp has no discrete pre/post-tool-call hook system** (no `PreToolUse`/`PostToolUse`-style lifecycle events with JSON stdin/stdout and blocking exit codes, unlike Claude Code, Devin CLI, or similar tools). No official docs page describing such a system was found. ❓

Instead, extensibility/control is achieved through:
- **Agent Profiles & Permissions** — per-profile autonomy level, tool access (terminal use, code editing, web search), and permission scopes (repos, files, secrets), configured under Settings → Agents → Profiles.
- **Command allowlist/denylist** — regex-based rules under Settings → AI → Agents → Command allowlist; matched commands auto-execute without approval.
- **MCP allowlist/denylist** — grants or requires approval for specific MCP servers regardless of the general allowlist.
- **Rules** (`AGENTS.md`/`WARP.md`) — natural-language behavioral constraints, not a programmatic hook.
- **Skills** (`SKILL.md`) — reusable instruction sets, invocable via natural language or `/skill-name` slash commands, closer to prompt-level extensibility than a lifecycle hook.

There is no shell-script-executing hook mechanism analogous to Claude Code/Devin CLI hooks documented for Warp. [official, absence inferred from lack of docs coverage]

## Built-in Tools / Capabilities

| Capability | Description |
|------|-------------|
| Terminal command execution | Runs shell commands, chains multi-step plans, reads output and self-corrects |
| Full Terminal Use | Drives interactive terminal apps (psql/mysql shells, gdb/lldb, vim/nano, Python REPLs, dev servers, interactive git rebases) by reading the live PTY buffer and writing input |
| Computer Use | Interacts with the desktop GUI via screenshots, clicking, typing (beyond the terminal) |
| File search / code search | grep- and glob-style search over the working directory |
| Codebase Context | Semantic indexing of Git-tracked files for code-aware answers |
| Web Search | Agent can retrieve current information from the internet |
| Planning / Task Lists | Decomposes requests into editable, step-by-step plans with live progress tracking |
| Skills | Reusable, invokable instruction sets (`SKILL.md`) |
| Slash Commands | Quick actions/saved prompts (`/command`) |
| MCP tools | Any tools exposed by connected MCP servers |

## MCP Support

Warp supports MCP servers natively for local agents, and (via Oz) for cloud agents. Config lives in JSON files with an `mcpServers` key, supporting two transport shapes: a launched-process ("CLI") server (`command`, `args`, `env`, `working_directory`) or a Streamable HTTP/SSE server (`url`, `headers`). Servers can be added via the Warp Drive GUI (Settings → Agents → MCP servers) or by editing the file directly (also assistable via the bundled `/agent-add-mcp` skill).

**Config locations:**

| File | Scope |
|------|-------|
| `~/.warp/.mcp.json` | Global |
| `.warp/.mcp.json` | Project |

Warp also reads compatible third-party MCP config formats for interop: `~/.claude.json`/`.mcp.json` (Claude Code), `~/.codex/config.toml`/`.codex/config.toml` (Codex), and `~/.agents/.mcp.json`/`.agents/.mcp.json` (generic agents dir).

**Example (`.warp/.mcp.json`):**
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path"]
    },
    "externalDocs": {
      "url": "http://localhost:4000/mcp/stream",
      "headers": { "my-header": "value" }
    }
  }
}
```

MCP logs are stored at:
- macOS: `~/Library/Group Containers/2BBY89MBSN.dev.warp/Library/Application Support/dev.warp.Warp-Stable/mcp`
- Windows: `%LOCALAPPDATA%\warp\Warp\data\logs\mcp`
- Linux: `${XDG_STATE_HOME:-$HOME/.local/state}/warp-terminal/mcp`

## Skills / Commands

Skills are Markdown files with YAML frontmatter (`name`, `description`), named `SKILL.md`, each in its own subdirectory.

| Location | Scope |
|----------|-------|
| `.agents/skills/<name>/SKILL.md` (recommended) or `.warp/skills/`, `.claude/skills/`, `.cursor/skills/` | Project |
| `~/.agents/skills/<name>/SKILL.md` (recommended) or `~/.warp/skills/`, etc. | Global |

Invocation: natural language ("use the deploy skill..."), slash command (`/deploy`), or slash command with arguments (`/explain-topic bears engineering fun` maps to `$0`/`$1`/`$2` or `$ARGUMENTS`).

## Agent / Subagent Configuration

Warp does not document discrete "subagent" profile files akin to Devin's `.devin/agents/`. Instead it exposes **Agent Profiles** (Settings → Agents → Profiles): named configurations of model, autonomy level, enabled tools (terminal use, code editing, web search), and permission scopes (repos/files/secrets access), selectable per session. Multiple agents can also be coordinated concurrently via **Oz** orchestration (local + cloud agents across machines/repos/teams), including cron-triggered, webhook-triggered, and CI/CD-integrated cloud agent runs.

## Notes

- **GUI vs. CLI**: Warp Agent Mode's interactive experience requires the full Warp desktop application — it is not a lightweight, install-anywhere CLI like Claude Code or Codex CLI. The separate `oz` CLI provides genuine headless/scriptable invocation for **cloud agents** (and local runs via `oz agent run`), authenticated with `WARP_API_KEY`, making Warp usable in CI/headless contexts despite the GUI-centric core product.
- Warp also runs **third-party CLI agents** (Claude Code, Codex, OpenCode) inside its terminal UI as an alternative to its own Agent Mode.
- The Warp client is source-available (AGPL-3.0 for the core, MIT for parts of the UI framework) at `github.com/warpdotdev/warp`; the Oz cloud orchestration platform and broader cloud infrastructure are proprietary/closed.
- No official hooks/lifecycle-event system (à la Claude Code `PreToolUse`/`PostToolUse`) was found in the docs; extensibility instead runs through Rules, Skills, Agent Profiles, and command/MCP allow-deny lists.
- Rule files (`AGENTS.md`/`WARP.md`) and skill directories (`.agents/skills/`) deliberately mirror competitor conventions (Claude Code, Cursor) for cross-tool compatibility.

## Sources

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| Docs home | https://docs.warp.dev/ | 2026-07-23 | [official] |
| Rules for agents | https://docs.warp.dev/agent-platform/capabilities/rules/ | 2026-07-23 | [official] |
| MCP (local agents) | https://docs.warp.dev/agent-platform/capabilities/mcp/ | 2026-07-23 | [official] |
| Agent platform overview | https://docs.warp.dev/agent-platform/ | 2026-07-23 | [official] |
| Full Terminal Use capability | https://docs.warp.dev/agent-platform/capabilities/full-terminal-use/ | 2026-07-23 | [official] |
| Capabilities overview | https://docs.warp.dev/agent-platform/capabilities/capabilities | 2026-07-23 | [official] |
| Skills for agents | https://docs.warp.dev/agent-platform/capabilities/skills/ | 2026-07-23 | [official] |
| Oz CLI reference | https://docs.warp.dev/reference/cli/ | 2026-07-23 | [official] |
| Agent Mode Context (Warp Drive) | https://docs.warp.dev/knowledge-and-collaboration/warp-drive/agent-mode-context/ | 2026-07-23 | [official] |
| Agent Profiles & Permissions | https://docs.warp.dev/agents/using-agents/agent-profiles-permissions | 2026-07-23 | [official] |
| Installation and setup | https://docs.warp.dev/getting-started/quickstart/installation-and-setup/ | 2026-07-23 | [official] |
| Pricing / plans and billing | https://docs.warp.dev/support-and-community/plans-and-billing/ | 2026-07-23 | [official] |
| GitHub repo (client, AGPL-3.0) | https://github.com/warpdotdev/warp | 2026-07-23 | [official] |
| Terminal & Agent modes | https://docs.warp.dev/agent-platform/local-agents/interacting-with-agents/terminal-and-agent-modes/ | 2026-07-23 | [official] |
