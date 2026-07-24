# Goose

> Open source, extensible AI agent (formerly by Block, now stewarded by the Agentic AI Foundation) that runs on your machine and connects to any LLM via MCP.

**Vendor:** Originally built by Block, Inc.; donated in 2026 to the **Agentic AI Foundation (AAIF)**, a Linux Foundation project (alongside MCP and AGENTS.md) | **License:** Apache 2.0 | **Runtime:** Native binary (Rust core), CLI + Desktop app; also embeddable as a library

## Links

- Docs (current): https://goose-docs.ai/
- Docs (legacy, redirects to goose-docs.ai): https://block.github.io/goose/
- GitHub repo: https://github.com/aaif-goose/goose (redirected from https://github.com/block/goose)
- Hooks announcement (blog): https://goose-docs.ai/blog/2026/05/14/goose-hooks/
- AAIF project page: https://aaif.io/projects/goose

> ❓ **Naming/hosting note:** Goose was created inside Block and open-sourced under Apache 2.0. In 2026 Block donated the project to the newly formed Agentic AI Foundation (AAIF) under the Linux Foundation. As a result, the canonical GitHub org moved from `block/goose` to `aaif-goose/goose`, and the docs site moved from `block.github.io/goose` to `goose-docs.ai`. Binary/package names (e.g. `block-goose-cli` on Homebrew, `download_cli.sh` URLs under the `aaif-goose` org) still reference "Block" for historical/compatibility reasons.

---

## Installation

**macOS / Linux (install script):**
```sh
curl -fsSL https://github.com/aaif-goose/goose/releases/download/stable/download_cli.sh | bash

# Non-interactive (skip the `goose configure` prompt):
curl -fsSL https://github.com/aaif-goose/goose/releases/download/stable/download_cli.sh | CONFIGURE=false bash
```

**macOS (Homebrew):**
```sh
brew install block-goose-cli          # CLI
brew install --cask block-goose       # Desktop app
```

**Windows (Git Bash / MSYS2):**
```sh
curl -fsSL https://github.com/aaif-goose/goose/releases/download/stable/download_cli.sh | bash
```

**Windows (PowerShell):**
```powershell
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/aaif-goose/goose/main/download_cli.ps1" -OutFile "download_cli.ps1"
.\download_cli.ps1
```

**Windows (WSL):** install WSL, then follow the Linux install script above.

**Cargo:**
```sh
cargo install goose-cli
```

**Update:**
```sh
goose update
```

**Desktop app:** downloadable per-platform builds (macOS Silicon/Intel `.zip`, Linux `.deb`/`.rpm`/Flatpak, Windows `.zip`) from the GitHub releases page.

After install, run `goose configure` to pick an LLM provider (OpenAI, Anthropic, Gemini, Azure OpenAI, AWS Bedrock, Databricks, Ollama for local models, etc.) and set up extensions.

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.config/goose/config.yaml` (macOS/Linux); `%APPDATA%\Block\goose\config\config.yaml` (Windows) | Global | Active provider/model, global settings (`GOOSE_MODE`, `GOOSE_TEMPERATURE`, `GOOSE_MAX_TOKENS`, `GOOSE_PLANNER_PROVIDER`/`GOOSE_PLANNER_MODEL`, `GOOSE_TOOLSHIM`, `GOOSE_SEARCH_PATHS`), `extensions` (MCP servers), `slash_commands` |
| `~/.config/goose/permission.yaml` | Global | Tool permission levels (managed via `goose configure`) |
| `~/.config/goose/secrets.yaml` | Global | API keys, only used with file-based secret storage (goose otherwise prefers the OS keyring / env vars — API keys are **not** read from `config.yaml` itself) |
| `~/.config/goose/permissions/tool_permissions.json` | Global | Auto-managed runtime permission decisions |
| `~/.config/goose/prompts/` | Global | Customized prompt templates |
| `~/.config/goose/.goosehints` | Global | Natural-language hints applied to every session |
| `.goosehints` (project root / nested dirs) | Project | Project- or directory-scoped hints, loaded hierarchically |
| `.agents/plugins/<name>/hooks/hooks.json` | Project or `~/.agents/plugins/<name>/` (Global) | Lifecycle hooks (Open Plugins spec) |
| `recipe.yaml` (anywhere, referenced via `--recipe`) | Project/shared | Packaged workflow: prompt/instructions, extensions, parameters, settings |

## Instruction File

Goose reads natural-language project context from **`.goosehints`** (global at `~/.config/goose/.goosehints`, local per-directory in the project). It also searches for **`AGENTS.md`** by default, and the set of recognized context filenames (e.g. to also pick up `.cursorrules`, `CLAUDE.md`) can be extended via the `CONTEXT_FILE_NAMES` environment variable. Hints support `@relative/path.md` syntax to pull a file's content directly into context; a bare (non-`@`) reference just points goose at an optional/large file. When both global and local hints exist, local hints win on conflicts, and hints are loaded hierarchically as goose accesses nested directories.

## Hooks

Goose has a real hook system, added via the **Open Plugins spec**: shell scripts wired to lifecycle events, auto-discovered from plugin directories.

**Plugin discovery locations:**
- User scope: `~/.agents/plugins/<name>/`
- Project scope: `<project>/.agents/plugins/<name>/`

Any directory containing a `hooks/hooks.json` is picked up automatically at startup.

### Supported Events

| Event | When | Matcher target | Can Block? |
|-------|------|-----------------|-----------|
| `SessionStart` | Session begins | — | ❌ |
| `SessionEnd` | Session ends | — | ❌ |
| `Stop` | A turn completes / stop event received | — | ✅ |
| `UserPromptSubmit` | User submits a prompt | Prompt text | ❌ |
| `PreToolUse` | Before a tool executes | Tool name | ✅ |
| `PostToolUse` | After a tool succeeds | Tool name | ❌ |
| `PostToolUseFailure` | After a tool fails | Tool name | ❌ |
| `BeforeReadFile` | Before a file read | File path | ❌ |
| `AfterFileEdit` | After a successful file edit | File path | ❌ |
| `BeforeShellExecution` | Before a shell command runs | Shell command | ❌ |
| `AfterShellExecution` | After a successful shell command | Shell command | ❌ |

Only `PreToolUse` and `Stop` can actually block/deny the action; all other events are observation-only.

### Hook Input (stdin JSON)

```json
{
  "event": "PreToolUse",
  "session_id": "identifier",
  "tool_name": "developer__shell",
  "tool_input": { "command": "..." },
  "working_dir": "/path/to/directory"
}
```

### Blocking a `PreToolUse`/`Stop` hook

Either:
- **Exit code `2`**, with the denial reason written to stderr, or
- **stdout JSON** (must start with `{`), regardless of exit code:
```json
{ "decision": "block", "reason": "explanation" }
```

When blocked, goose surfaces: *"Tool call denied by policy hook `<plugin>`: `<reason>`. Do not retry; this is a policy denial, not a transient failure."*

Failed or timed-out hooks are logged but do not crash goose (fail-open for non-blocking events).

### Example `hooks/hooks.json`

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "developer__shell",
        "hooks": [
          { "type": "command", "command": "${PLUGIN_ROOT}/scripts/block-sudo.sh", "timeout": 10 }
        ]
      }
    ],
    "AfterFileEdit": [
      {
        "matcher": "\\.(ts|tsx|js|jsx)$",
        "hooks": [
          { "type": "command", "command": "${PLUGIN_ROOT}/scripts/prettier.sh", "timeout": 30 }
        ]
      }
    ],
    "PostToolUseFailure": [
      { "hooks": [ { "type": "command", "command": "${PLUGIN_ROOT}/scripts/notify.sh" } ] }
    ]
  }
}
```

Notes: `matcher` is a **regular expression** (not a glob) tested against the tool name / file path / shell command depending on event; omit it (or use `".*"`) to match everything for that event. `${PLUGIN_ROOT}` (also exposed as the `PLUGIN_ROOT` env var) resolves to the plugin's own directory so scripts don't hardcode absolute paths.

> ❓ Because the hooks feature and the Open Plugins spec are recent additions (blog post dated 2026-05-14), some edge-case behavior (e.g. cross-platform script execution, full list of blockable events beyond `PreToolUse`/`Stop`) may still evolve — verify against the live docs for the version installed.

## Built-in Tools

Goose ships a built-in **Developer** MCP extension (enabled by default) that provides the core coding tools:

| Tool | Description |
|------|-------------|
| `shell` | Execute shell commands (tests, package installs, git operations, etc.) |
| `write` | Create or overwrite files |
| `edit` | Replace exact text within a file (targeted edits/refactors) |
| `tree` | List a directory tree with line counts, to understand project structure before reading files |
| `read_image` | Read a local or remote image for model inspection (screenshots, diagrams) |

Other first-party built-in extensions (bundled but not always enabled by default) include:

| Extension | Description |
|-----------|-------------|
| Computer Controller | Web scraping and file/desktop automation |
| Memory | Retains preferences/facts across sessions |
| Chat Recall | Searches prior conversation history |
| Todo | Task tracking within a session |

## MCP Support

Goose is **MCP-native** — nearly everything beyond the Developer extension is added as an MCP server, called an "**extension**" in goose terminology. Extensions can connect over stdio, and goose runs automatic malware/safety scanning before activating an external extension.

**Add via CLI:**
```sh
goose configure
# → "Add Extension" → choose type (stdio command, e.g.:)
#   npx -y @modelcontextprotocol/server-memory
#   uvx mcp-wiki
#   jbang -Dspring.profiles.active=dev org.example:spring-data-mcp:1.0.0
```

**Config location:** `~/.config/goose/config.yaml`, under the `extensions` key:

```yaml
extensions:
  github:
    name: GitHub
    cmd: npx
    args: [-y @modelcontextprotocol/server-github]
    enabled: true
    envs: { "GITHUB_PERSONAL_ACCESS_TOKEN": "<YOUR_TOKEN>" }
    type: stdio
    timeout: 300
```

Goose can also itself be run as an MCP-compatible server (`goose mcp <name>`) or as an **Agent Client Protocol (ACP)** server over stdio (`goose acp`) for integration with ACP clients like Zed.

## Recipes

**Recipes** are goose's reusable-workflow / "agent config as code" mechanism: a YAML file bundling a prompt/instructions, the extensions (MCP servers) it needs, parameters, and model/provider settings, so a workflow can be shared and rerun reproducibly. Recipes can be composed of **subrecipes** — each subrecipe is effectively an independent goose agent (its own extensions, plugins, provider, system prompt) that runs in an isolated session with no shared conversation history, invoked sequentially or in parallel by the parent recipe, functioning as goose's subagent mechanism.

**Example `recipe.yaml`:**
```yaml
version: "1.0.0"
title: "Your Recipe Title"
description: "What this recipe does"
instructions: "Step-by-step guidance for the agent"
prompt: "Primary task instruction"

parameters:
  - key: param_name
    input_type: string
    requirement: required
    description: "Parameter explanation"

extensions:
  - type: stdio
    name: extension_name
    cmd: uvx
    args:
      - package@latest
    timeout: 300

settings:
  goose_provider: "anthropic"
  goose_model: "claude-sonnet-4-20250514"
  temperature: 0.7
```

At least one of `instructions` or `prompt` is required. Recipes can be validated, turned into shareable deeplinks, listed, or opened in the Desktop app via `goose recipe`, and run directly with `goose run --recipe <path> --params key=value`. Custom **slash commands** can map to a recipe for quick reuse inside a chat session (configured under `slash_commands` in `config.yaml`).

## Skills / Commands

Goose has "skills for custom context" referenced alongside subagents/recipes in official materials, and supports custom slash commands (`slash_commands` in `config.yaml`) that trigger a recipe. ❓ A dedicated `SKILL.md`-style skills file format/location (comparable to Claude Code's `.claude/skills/`) was not confirmed in the docs pages fetched for this entry — treat "skills" here as recipe-driven context rather than a separate first-class file format until confirmed against `goose-docs.ai`.

## Agent / Subagent Configuration

Parallel/independent task execution is done via **subrecipes** (see Recipes above) rather than a separate "subagent profile" file format. Each subrecipe runs as its own isolated goose agent process with its own extensions/provider/system prompt, and the parent recipe's agent decides whether to run subrecipes sequentially or in parallel and chains their outputs back into the main context.

## Notes

- Goose defaults to an **autonomous permission mode**: out of the box it can run shell commands and edit any accessible file without per-action approval, because the Developer extension's `shell`/`edit`/`write` tools are enabled by default. Permission levels (`GOOSE_MODE`: `auto`, `approve`, `chat`, `smart_approve`) are configurable in `config.yaml` / via `goose configure`.
- `goose info` shows the active version, config file location, session storage path, and logs.
- Sessions can be listed, resumed, forked, edited, and exported (`goose session …`, markdown/JSON/YAML export).
- `goose term` sets up terminal-integrated shell aliases (`@goose`, `@g`) for direct prompting from a shell.

## Sources

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| Docs home (redirect notice: block.github.io → goose-docs.ai) | https://block.github.io/goose/ | 2026-07-23 | [official] |
| Installation | https://goose-docs.ai/docs/getting-started/installation/ | 2026-07-23 | [official] |
| Config files | https://goose-docs.ai/docs/guides/config-files/ | 2026-07-23 | [official] |
| .goosehints / context engineering | https://goose-docs.ai/docs/guides/context-engineering/using-goosehints/ | 2026-07-23 | [official] |
| Extensions (MCP) | https://goose-docs.ai/docs/getting-started/using-extensions/ | 2026-07-23 | [official] |
| Developer extension tools | https://goose-docs.ai/docs/mcp/developer-mcp/ | 2026-07-23 | [official] |
| Recipes overview | https://goose-docs.ai/docs/guides/recipes/ | 2026-07-23 | [official] |
| Recipe reference (schema) | https://goose-docs.ai/docs/guides/recipes/recipe-reference/ | 2026-07-23 | [official] |
| Hooks blog announcement | https://goose-docs.ai/blog/2026/05/14/goose-hooks/ | 2026-07-23 | [official] |
| Hooks reference (events, blocking) | https://goose-docs.ai/docs/guides/context-engineering/hooks | 2026-07-23 | [official] |
| Plugins (Open Plugins spec) | https://goose-docs.ai/docs/guides/context-engineering/plugins/ | 2026-07-23 | [official] |
| CLI commands reference | https://goose-docs.ai/docs/guides/goose-cli-commands/ | 2026-07-23 | [official] |
| GitHub repo (current org, post-AAIF move) | https://github.com/aaif-goose/goose | 2026-07-23 | [official] |
| AAIF formation / Block donates goose | https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation | 2026-07-23 | [official] |
| "goose has a new home" (AAIF move announcement) | https://goose-docs.ai/blog/2026/04/07/goose-moves-to-aaif/ | 2026-07-23 | [official] |
| AAIF project page for goose | https://aaif.io/projects/goose | 2026-07-23 | [official] |
