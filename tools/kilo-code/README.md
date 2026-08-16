# Kilo Code

> Open-source AI coding agent platform (VS Code, JetBrains, CLI, and Cloud) descended from the Cline → Roo Code lineage, with a separately-forked OpenCode-based CLI; reached a "1.0" CLI milestone in 2026.

**Vendor:** Kilo-Org (Kilo) | **License:** MIT | **Runtime:** VS Code / JetBrains extension (TypeScript, editor host) and a standalone CLI (`@kilocode/cli`, Node/Bun-based, cross-platform binary)

## Links

- Docs: https://kilo.ai/docs
- CLI docs: https://kilo.ai/docs/code-with-ai/platforms/cli
- CLI command reference: https://kilo.ai/docs/code-with-ai/platforms/cli-reference
- GitHub (main repo): https://github.com/Kilo-Org/kilocode
- GitHub (docs repo, ARCHIVED): https://github.com/Kilo-Org/docs — deprecated/archived 2025-08-27; docs content migrated into the main `kilocode` repo
- GitHub Releases (CLI binaries): https://github.com/Kilo-Org/kilocode/releases

---

## Installation

### CLI (npm — only method documented)

```sh
npm install -g @kilocode/cli
kilo --version
```

For CPUs without AVX support, a "baseline" binary variant can be downloaded directly from GitHub Releases and extracted. ❓ Docs reference curl/pnpm/bun/Homebrew/AUR install paths on the marketing site's search snippet, but the CLI platform doc page itself documents only the npm method — could not confirm the other install commands from an official page fetch.

### VS Code / JetBrains

Install "Kilo Code" from the VS Code Marketplace or the JetBrains Marketplace (extension IDs not independently re-verified here beyond the marketplace listing titles).

---

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.config/kilo/kilo.json[c]` | Global (user) | User-level settings, providers, MCP servers, agents |
| `./kilo.json[c]` or `./.kilo/` | Project | Project-level config/overrides, takes precedence over global |
| `.kilo/agents/*.md`, `.kilocode/agents/*.md` | Project | Markdown+YAML-frontmatter custom agent/mode definitions |
| `~/.kilo/skills/<name>/`, `.kilo/skills/<name>/` | Global / Project | Skill directories (see Skills below — path forms documented inconsistently, see note) |
| `KILO_CONFIG_CONTENT` env var | Session | Near-top-precedence config override (inline JSON string) |

Project config takes precedence over global config. The agent has a built-in capability for reading/updating `kilo.jsonc` directly (i.e., you can ask it in natural language to "add this MCP server" instead of hand-editing the file).

**Full config-value merge precedence** (lowest to highest, confirmed via the `kilo-config.md` skill file shipped in the main repo — deep-merged, later wins): remote well-known config → global config (`~/.config/kilo/kilo.json[c]`) → `KILO_CONFIG` env var (path to an additional config file) → project config (`./kilo.json[c]`) → `.kilo/kilo.json[c]` → `KILO_CONFIG_CONTENT` env var (inline JSON) → managed/MDM config (enterprise, highest). Legacy `opencode.json[c]` filenames are also recognized for backward compatibility.

❓ The docs show two different global-scope path conventions across pages (`~/.config/kilo/…` for settings/MCP, and `~/.kilo/skills/…` for skills). Both are as literally stated on their respective official pages; could not confirm from a single source whether these are meant to be the same root or genuinely different directories.

---

## Instruction File

Kilo supports the open **AGENTS.md** (or `AGENT.md` fallback) standard, placed at the project root, with per-directory `AGENTS.md` files in subdirectories dynamically loaded when the agent reads files in that directory (contents injected as system-reminder context). Both `AGENTS.md` and `AGENT.md` are write-protected — edits require explicit user approval.

Instruction precedence (highest to lowest), per official docs:
1. Per-agent prompt (agent/mode config)
2. Project-level `instructions` in `kilo.jsonc`
3. `AGENTS.md` (project root / per-directory)
4. Global `instructions` in `kilo.jsonc`
5. Skills (loaded on demand from `.kilo/skills/`)

The CLI additionally supports `.claude/` and `.agents/` directories for cross-tool compatibility (e.g., with Claude Code, Cursor, Windsurf conventions). ❓ Exact merge/precedence behavior of `.claude/`/`.agents/` relative to the numbered list above is not spelled out in the fetched source.

---

## Hooks

Kilo Code (CLI) has **no Claude-Code-style shell-hook system** (no JSON-over-stdin / exit-code-2-to-block contract). Instead, automation/extensibility is exposed through a **TypeScript/JavaScript Plugin API** — plugins are code modules that subscribe to internal lifecycle/event callbacks, not standalone shell scripts driven by exit codes. There is also a narrower **Agent Manager setup-script** mechanism for worktree bootstrap (not a general hook system).

### Supported Events (Plugin API lifecycle hooks)

| Hook / Event | When | Can Block? |
|---|---|---|
| `config` | Plugin receives fully-resolved config at startup | ❌ |
| `event` | Fires for every internal bus event (`session.created`, `session.idle`, `session.compacted`, message updates, tool execution, permissions, file changes, LSP diagnostics, server connections, etc.) | ❌ (observational) |
| `tool` | Register a custom tool | n/a |
| `tool.execute.before` / `tool.execute.after` | Intercept tool execution | ✅ (before) |
| `tool.definition` | Mutate a tool's description before it's sent to the model | n/a |
| `chat.message` / `chat.params` / `chat.headers` | Intercept outgoing chat request | ❌ (inspect/mutate only) |
| `permission.ask` | Custom permission decision | ✅ (auto-allow/auto-deny) |
| `command.execute.before` | Before a slash command executes (e.g. `/commit`) | ❌ (mutate resulting `parts` only; not documented as blockable) |
| `shell.env` | Modify shell environment | n/a |
| `auth` / `provider` | Register auth flows / dynamic model catalogs | n/a |

Confirmed via direct quotes from the Plugins doc: `tool.execute.before` demonstrates blocking with a documented example (`throw new Error('reading .env files is blocked')`), and `permission.ask` explicitly "auto-allow[s] or auto-den[ies] permission prompts." No other hook's description mentions a block/cancel/deny mechanism — `chat.*` and `command.execute.before` are described only as inspect/mutate hooks. There is still no formal "can block / exit-code" contract table the way Claude Code documents it; the above is derived from each hook's prose description, not a single canonical semantics reference.

### Plugin location / format

```
~/.config/kilo/plugin/         # global
.kilo/plugin/                  # project
.kilocode/plugin/              # legacy project path
```
or declared as an array in `kilo.json` / `opencode.jsonc` / `tui.jsonc`.

```ts
import type { Plugin } from "@kilocode/plugin"

const myPlugin: Plugin = async ({ project, client, $, directory, worktree }) => {
  return {
    // hook implementations, e.g. tool.execute.before, event, etc.
  }
}

export default { id: "my-plugin", server: myPlugin }
```

### Hook Input / Output

❓ No stdin-JSON / stdout-JSON contract exists (unlike Claude Code hooks) — the Plugin API is in-process TypeScript, not an external-process/exit-code protocol, so this section of the template does not apply as written.

### Exit Code Behavior

Not applicable — plugins run in-process; there is no documented exit-code contract for the Plugin API.

### Setup Scripts (separate, narrower mechanism)

The Agent Manager runs a setup script automatically whenever a new worktree is created:

| Platform | Filename |
|---|---|
| macOS/Linux | `.kilo/setup-script` or `.kilo/setup-script.sh` |
| Windows | `.kilo/setup-script.ps1`, `.kilo/setup-script.cmd`, or `.kilo/setup-script.bat` |

`WORKTREE_PATH` and `REPO_PATH` env vars are injected into the script's environment. This is documented as worktree bootstrap only — not a general pre/post-tool-call hook system.

---

## Built-in Tools

| Tool | Description |
|------|-------------|
| `read` | Retrieve file contents with line numbers |
| `glob` | Locate files matching a glob pattern |
| `grep` | Search file contents via regex |
| `edit` | Targeted text replacement in files |
| `write` | Create new files / fully replace existing ones |
| `apply_patch` | Apply unified-diff-format changes |
| `bash` | Run shell commands (configurable timeout/directory) |
| `webfetch` | Retrieve and return URL content |
| `websearch` | Web search via Exa or Parallel providers |
| `kilo-playwright_browser_navigate` | Open a URL in a browser |
| `kilo-playwright_browser_click` | Click a page element |
| `kilo-playwright_browser_type` | Enter text into a field |
| `kilo-playwright_browser_screenshot` | Capture a visual screenshot |
| `kilo-playwright_browser_snapshot` | Capture an accessibility snapshot |
| `question` | Prompt the user with selectable response options |
| `task` | Start a sub-agent session |
| `todowrite` / `todoread` | Manage session task lists |
| `plan` | Enter structured planning mode |
| `skill` | Invoke a reusable skill module |
| `agent_manager` | Manage VS Code sessions/worktrees |
| `{server}_{tool}` | Dynamically registered MCP tools |

---

## MCP Support

Config location: `~/.config/kilo/kilo.jsonc` (global) or `kilo.jsonc` / `.kilo/kilo.jsonc` in the project root, under the `mcp` key. Servers can also be installed from the Kilo Marketplace. MCP tools are subject to Kilo's `allow`/`ask`/`deny` permission rules.

**Local (stdio) server:**
```json
{
  "mcp": {
    "server-name": {
      "type": "local",
      "command": ["node", "/path/to/server.js"],
      "environment": { "API_KEY": "your_api_key" },
      "enabled": true,
      "timeout": 10000
    }
  }
}
```

**Remote (HTTP/SSE) server:**
```json
{
  "mcp": {
    "server-name": {
      "type": "remote",
      "url": "https://your-server-url.com/mcp",
      "headers": { "Authorization": "Bearer your-token" },
      "enabled": true,
      "timeout": 15000,
      "oauth": true
    }
  }
}
```

---

## Skills / Commands

- Skills location: `.kilo/skills/<name>/` (project scope), `~/.kilo/skills/<name>/` (global scope, per the marketplace/skills doc page — note the path-convention discrepancy flagged above)
- Format: one directory per skill; "task-specific instructions and resources that Kilo can load when relevant." Discovered and loaded automatically by the agent during a session based on context (no manual per-invocation slash command documented in the fetched pages). ❓ Exact required file inside a skill directory (e.g., a `SKILL.md` manifest) was not confirmed from the fetched source.
- Additional servers/skills can be installed via the **Marketplace** (`/docs/customize/marketplace`).

---

## Agent / Subagent Configuration

Kilo ships **seven built-in agents/modes**: `code`, `plan`, `debug`, `ask`, `orchestrator`, `explore`, and `general`. All can be overridden or extended.

Custom agents/subagents can be defined two ways:

**Markdown + YAML frontmatter** (recommended for CLI/VS Code):
```
.kilo/agents/my-agent.md         # project
.kilo/agent/my-agent.md          # project (alt. singular dir)
.kilocode/agents/my-agent.md     # project (legacy)
~/.config/kilo/agents/           # global, per the Custom Subagents doc
~/.config/kilo/agent/            # global, per the Custom Modes doc (singular — inconsistent with the above across doc pages)
```
Frontmatter fields include `description`, `mode` (`primary`/`subagent`/`all`), `color`, `model`, `temperature`, and `permission` rules; the markdown body becomes the agent's system prompt.

**JSON, inside `kilo.jsonc`:**
```json
{
  "agent": {
    "code-reviewer": {
      "description": "Reviews code for best practices",
      "mode": "subagent",
      "model": "anthropic/claude-sonnet-4-20250514",
      "prompt": "You are a code reviewer...",
      "permission": { "edit": "deny", "bash": "deny" }
    }
  }
}
```

Agent-definition precedence (highest to lowest, per the Custom Subagents doc): project agent markdown files (`.kilo/agents/`) → global agent markdown files (`~/.config/kilo/agents/`) → project `kilo.jsonc` `agent` config → global `kilo.jsonc` `agent` config → built-in agent defaults. (This is distinct from the general config-value merge precedence — including `KILO_CONFIG_CONTENT` — documented in the Configuration Files section above.)

Invocation: primary agents can auto-delegate to subagents via the `task` tool when a subagent's `description` matches the workload; users can also manually invoke an agent with `@agent-name`; `kilo agent list` lists all available agents from the CLI.

---

## Notes

- **Lineage:** Kilo Code (the VS Code/JetBrains extension) started in 2025 as a fork of **Roo Code**, which was itself a fork of **Cline** — the extension's git history is shared with Roo's. Roo Code's own upstream GitHub repo was archived (read-only) on 2026-05-15. The **Kilo CLI**, however, is a separately-sourced product described in official material as "a fork of OpenCode, enhanced to work within the Kilo agentic engineering platform" — so the CLI's plugin/hook architecture (in-process TypeScript lifecycle hooks, `kilo.jsonc` config, `opencode.jsonc` fallback file recognition) reflects OpenCode ancestry rather than the Cline/Roo VS Code extension's architecture. Treat "Kilo Code" as an umbrella brand covering at least two differently-sourced codebases (editor extension vs. CLI).
- Autonomous/non-interactive CLI runs use `kilo run --auto` per one sourced description, and `--auto` also appears as a general CLI flag for unattended execution; `--continue`/`-c` resumes the most recent session.
- The agent has a documented "self-configuring" capability: natural-language requests like "add this MCP server" or "add my Ollama endpoint" can directly edit `kilo.jsonc` on the user's behalf.
- Kilo also ships **KiloClaw** (an "always-on" agent product), a **Cloud Agent** (web-based, `app.kilo.ai/cloud`), and automated PR **Code Reviews** (`app.kilo.ai/code-reviews`) — these are adjacent products/surfaces beyond the CLI/extension scope of this page and were not deep-dived here.
- This is a documentation-only pass: no live install or execution of Kilo Code was performed to verify the above; all claims are sourced to the official docs/GitHub pages listed below.

---

## Sources (Official)

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| Docs home / overview | https://kilo.ai/docs | 2026-08-15 | [official] |
| Getting started | https://kilo.ai/docs/getting-started | 2026-08-15 | [official] |
| CLI platform docs | https://kilo.ai/docs/code-with-ai/platforms/cli | 2026-08-15 | [official] |
| Main GitHub repo (kilocode) | https://github.com/Kilo-Org/kilocode | 2026-08-15 | [github] |
| GitHub LICENSE (MIT) | https://github.com/Kilo-Org/kilocode/blob/main/LICENSE | 2026-08-15 | [github] |
| GitHub API repo metadata (license/description) | https://api.github.com/repos/Kilo-Org/kilocode | 2026-08-15 | [github] |
| Docs GitHub repo | https://github.com/Kilo-Org/docs | 2026-08-15 | [github] |
| AGENTS.md support | https://kilo.ai/docs/customize/agents-md | 2026-08-15 | [official] |
| Custom Instructions / Custom Rules index | https://kilo.ai/docs/customize | 2026-08-15 | [official] |
| Custom Modes | https://kilo.ai/docs/customize/custom-modes | 2026-08-15 | [official] |
| Custom Subagents | https://kilo.ai/docs/customize/custom-subagents | 2026-08-15 | [official] |
| Marketplace / Skills | https://kilo.ai/docs/customize/marketplace | 2026-08-15 | [official] |
| Automate index | https://kilo.ai/docs/automate | 2026-08-15 | [official] |
| Agent Manager (setup scripts) | https://kilo.ai/docs/automate/agent-manager | 2026-08-15 | [official] |
| Plugins (Plugin API / lifecycle hooks) | https://kilo.ai/docs/automate/extending/plugins | 2026-08-15 | [official] |
| Tools list | https://kilo.ai/docs/automate/tools | 2026-08-15 | [official] |
| MCP overview | https://kilo.ai/docs/automate/mcp/overview | 2026-08-15 | [official] |
| MCP: Using MCP in Kilo Code (config schema) | https://kilo.ai/docs/automate/mcp/using-in-kilo-code | 2026-08-15 | [official] |
| MCP server transports (STDIO/SSE) | https://kilo.ai/docs/automate/mcp/server-transports | 2026-08-15 | [official] |
| CLI command reference | https://kilo.ai/docs/code-with-ai/platforms/cli-reference | 2026-08-15 | [official] |
| Config precedence / merge rules (`kilo-config.md` skill file, in-repo) | https://github.com/Kilo-Org/kilocode/blob/main/packages/opencode/src/kilocode/skills/kilo-config.md | 2026-08-15 | [github] |
