# jcode

> Rust-native, RAM-efficient AI coding agent harness with swarm coordination, a semantic memory graph, remote/persistent server sessions, and a TypeScript SDK.

**Vendor:** Solo Systems (solo founder Jeremy Huang / GitHub `1jehuang`; YC-backed) | **License:** MIT | **Runtime:** Native Rust binary (single-binary CLI/TUI, no Node.js/Python runtime required)

## Links

- Website: https://jcode.sh
- Docs: https://jcode.sh/docs (marked "under construction" as of fetch date — several subpages are stubs)
- GitHub: https://github.com/1jehuang/jcode
- Releases / Changelog: https://github.com/1jehuang/jcode/releases
- SDK: https://jcode.sh/sdk
- Benchmarks: https://jcode.sh/bench

---

## Installation

```sh
# macOS & Linux
curl -fsSL https://jcode.sh/install | bash
```

```powershell
# Windows 11 x64/ARM64 (PowerShell 5.1+)
irm https://jcode.sh/install.ps1 | iex
```

```sh
# Homebrew (macOS)
brew tap 1jehuang/jcode
brew install jcode
```

```sh
# From source
git clone https://github.com/1jehuang/jcode.git
cd jcode
cargo build --release
scripts/install_release.sh
```

Termux (Android) is supported with `pkg install glibc patchelf` before running the install script. Platform support: Linux (x86_64/aarch64), macOS (Apple Silicon & Intel), Windows x86_64 (native + WSL2), Termux (aarch64/x86_64).

Uninstall (keeps config/auth/sessions): `curl -fsSL https://raw.githubusercontent.com/1jehuang/jcode/master/scripts/uninstall.sh | bash -s -- --yes`. Add `--purge` for a full wipe.

---

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.jcode/config.toml` | Global | Main config: providers, display, hooks, terminal spawn, safety rules, etc. |
| `~/.jcode/mcp.json` | Global | MCP server definitions |
| `.jcode/mcp.json` | Project | Project-local MCP server definitions |
| `~/.jcode/auth.json`, `~/.jcode/openai-auth.json`, `~/.jcode/gemini_oauth.json` | Global | Provider OAuth credentials |
| `~/.config/jcode/*.env` | Global | Per-provider API key / endpoint env files (e.g. `openai-compatible.env`) |
| `~/.jcode/pending-login/` | Global | In-flight scriptable login state |
| `~/.jcode/skills/` | Global | Skill markdown packs — one directory per skill containing a `SKILL.md` file (confirmed on jcode.sh/docs); ❓ exact frontmatter schema still not published |
| `.jcode/swarm-prompt.md`, `~/.jcode/swarm-prompt.md` | Project / Global | Swarm routing policy prompts — docs describe these as controlling "model routing and structure policy"; ❓ full schema still not confirmed in primary docs |

jcode also **reads (does not copy) Claude Code's and Codex CLI's config live** for compatibility:
- `~/.claude.json` (top-level `mcpServers`, plus `projects.<abs_path>.mcpServers`)
- `.mcp.json` (repo root) and `.claude/mcp.json` (legacy fallback)
- One-time import from `~/.codex/config.toml` into `~/.jcode/mcp.json` if the latter doesn't already exist (imported file becomes jcode-owned; later Codex changes are not synced)

## Instruction File

`AGENTS.md` (per-repo, loaded from the working directory) and `~/AGENTS.md` (global, loaded every session). `CLAUDE.md` is also read for compatibility. The base system prompt is small (~671 tokens, o200k_base) and is dynamically extended by these instruction files plus self-dev-mode context.

## Hooks

jcode has a genuine, config-driven **lifecycle hook system** (documented in `docs/HOOKS.md`), separate from a **spawn hook** for controlling where headed terminal sessions open (`docs/SPAWN_HOOK.md`). This is a smaller, config-file-driven mechanism than Claude Code's hook system — five lifecycle events (one of which can block), plus a separate window/terminal-routing hook. Hooks run **external commands directly** (not through a shell) and communicate via environment variables + a JSON payload, not via stdin JSON matchers/arrays like Claude Code.

### Supported Events

| Event | When | Can Block? | Type |
|-------|------|-----------|------|
| `session_start` | Agent session becomes active (`create`/`attach`/`resume`) | ❌ | Observer (detached, fire-and-forget) |
| `session_end` | Session closes normally | ❌ | Observer |
| `turn_end` | An agent turn completes | ❌ | Observer |
| `post_tool` | After every tool call | ❌ | Observer |
| `pre_tool` | Before every tool call | ✅ (exit 2 blocks) | Gate (synchronous) |

There is also a separate **spawn hook** (`[terminal] spawn_hook` in config, or `JCODE_SPAWN_HOOK` env var) that intercepts headed-window spawns (swarm agent spawn, resume-in-new-terminal, self-dev sessions, restart, jade relay) and lets an external program decide where the window/pane appears (tmux, kitty, zellij, etc.). It does not gate agent behavior — it only redirects window placement.

### Hook Input

Not stdin JSON for lifecycle hooks (unlike Claude Code) — instead environment variables, with a JSON mirror capped at 16 KB:

```
JCODE_HOOK_EVENT=pre_tool
JCODE_HOOK_SESSION_ID=<session id>
JCODE_HOOK_CWD=<session working directory>
JCODE_HOOK_PAYLOAD=<JSON object mirroring all fields, capped at 16KB>
JCODE_HOOKS_DISABLED=1   # always set on hook-spawned processes (recursion guard)
```

`pre_tool` additionally receives `JCODE_HOOK_TOOL_NAME` plus the full tool-input JSON on **stdin** (and a 16 KB-truncated copy in `JCODE_HOOK_TOOL_INPUT`). `turn_end` adds `JCODE_HOOK_STATUS`, `JCODE_HOOK_DURATION_MS`, `JCODE_HOOK_MODEL`, `JCODE_HOOK_LAST_ASSISTANT_TEXT` (first 4000 chars), `JCODE_HOOK_ERROR`. `post_tool` adds `JCODE_HOOK_TOOL_NAME`, `JCODE_HOOK_STATUS`, `JCODE_HOOK_DURATION_MS`, `JCODE_HOOK_OUTPUT_BYTES`, `JCODE_HOOK_ERROR`. `session_start`/`session_end` add `JCODE_HOOK_SOURCE` (`create`/`attach`/`resume` for start; `close` for end).

### Hook Output

`pre_tool` (the only blocking hook) communicates via **exit code + stderr**, not stdout JSON:
- Blocking a call returns the hook's stderr (trimmed, capped at 2000 chars) to the model as the tool error.
- Observer hooks (`turn_end`, `session_start`, `session_end`, `post_tool`) produce no structured output — they are fire-and-forget; failures are only logged.

### Exit Code Behavior

| Code | Meaning |
|------|---------|
| `0` | Allow the tool call (`pre_tool`) |
| `2` | Block the tool call (`pre_tool` only); stderr returned to the model as the tool error |
| Other / timeout / missing binary | **Fails open** — treated as "no policy," logged as a warning (deliberate design choice, not fail-closed) |

### Example Config

```toml
# ~/.jcode/config.toml
[hooks]
turn_end      = "~/bin/jcode-turn-notify"     # observer
session_start = ""                            # observer (disabled)
session_end   = ""                            # observer (disabled)
pre_tool      = "~/bin/jcode-tool-policy"     # gate
post_tool     = ""                            # observer (disabled)
pre_tool_timeout_ms = 5000
```

```bash
#!/usr/bin/env bash
# ~/bin/jcode-tool-policy — pre_tool gate example
input=$(cat)   # tool input JSON on stdin
case "$JCODE_HOOK_TOOL_NAME" in
  bash)
    if grep -qE 'rm -rf /([^a-zA-Z]|$)|mkfs|dd if=' <<<"$input"; then
      echo "blocked: destructive shell command" >&2
      exit 2
    fi
    ;;
esac
exit 0
```

Env var overrides (always win over config; empty value disables): `JCODE_HOOK_TURN_END`, `JCODE_HOOK_SESSION_START`, `JCODE_HOOK_SESSION_END`, `JCODE_HOOK_PRE_TOOL`, `JCODE_HOOK_POST_TOOL`, `JCODE_HOOK_PRE_TOOL_TIMEOUT_MS`.

Separately, jcode ships an unrelated **"Safety System"** (status: *Design*, per `docs/SAFETY_SYSTEM.md`) for ambient/unattended mode — a two-tier (auto-allowed / requires-permission) action classifier with a `request_permission` tool, review queue, and notification dispatch (email/SMS/desktop/webhook/TUI). It is not a general hook mechanism and its consumer is currently only ambient mode; custom classification rules are configured under `[safety.rules]` in `config.toml`. ❓ Because docs mark this "Design" status, live behavior/completeness is unconfirmed.

## Built-in Tools

| Tool | Description |
|------|-------------|
| File read/write/edit | Standard file operations |
| `bash`/shell execution | Run shell commands |
| Agent grep | Custom grep that adds file-structure info (function list, displacement) to results, with adaptive truncation to save context |
| Todo management | Task tracking with confidence scoring / "hill-climbability" assessment |
| `run_in_background` | Long-running background process execution with progress monitoring |
| `browser` | Built-in browser-automation tool (Firefox via Firefox Agent Bridge); actions: `status`, `setup`, `open`, `snapshot`, `get_content`, `interactables`, `click`, `type`, `fill_form`, `select`, `wait`, `screenshot`, `eval`, `scroll`, `upload`, `press` |
| Memory tools | Explicit store/retrieve/search of the semantic memory graph, plus session search (RAG over past sessions) |
| Swarm tool | Spawn and coordinate child agents (workers), send DMs/broadcasts, message channels |
| `request_permission` | Ambient-mode safety-system tool for requesting human approval on Tier-2 actions |
| Image generation / rendering | Native terminal image support |
| LaTeX rendering | Mathematical notation rendering |
| Mermaid diagram rendering | Inline diagram rendering (custom renderer, no browser/TypeScript dependency) |

❓ No single canonical "full tool list" reference page was found; this table is assembled from README and docs mentions across multiple official pages, and may be incomplete.

## MCP Support

Config is separate from `config.toml`:
- `~/.jcode/mcp.json` — global MCP servers
- `.jcode/mcp.json` — project-local MCP servers
- Both the canonical `mcpServers` key and jcode's historical `servers` key are accepted
- **stdio (command-based) servers only** — `"type": "http"` / `"sse"` entries are recognized but skipped (logged, not connected) as of the fetched version
- MCP tool schemas are cached on-disk and advertised at session startup (no cache-miss wait); servers connect in the background so a slow MCP server doesn't block session start or invalidate the prompt cache
- Also reads Claude Code's live MCP config (`~/.claude.json`, `.mcp.json`, `.claude/mcp.json`) for compatibility, and does a one-time import from Codex CLI's `~/.codex/config.toml`

Example (`~/.jcode/mcp.json` or `.jcode/mcp.json`):

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "/path/to/mcp-server",
      "args": ["--root", "/workspace"],
      "env": {},
      "shared": true
    }
  }
}
```

## Skills / Commands

- Skills location: `~/.jcode/skills/` (per jcode.sh homepage copy)
- Format: One directory per skill containing a `SKILL.md` file (confirmed on jcode.sh/docs); ❓ exact frontmatter schema and naming convention still not documented in the primary sources fetched
- Invocation: Skills are **not all loaded at startup**. The conversation is embedded as a semantic vector; a skill is auto-injected on an embedding hit (similar to the memory-recall mechanism). Agents also have an explicit skill tool to activate a skill manually, and skills can be invoked via slash commands (e.g. `/skills`)
- Other slash commands mentioned: `/model`, `/resume`, `/commit`, `/memory`, `/alignment`, `/account`

## Agent / Subagent Configuration

jcode's multi-agent model is **swarm-based**, not a static subagent-definition file:

- **Modes:** ad hoc swarm and "light-swarm" mode are one-level fan-out — only the root session may spawn workers, and workers cannot spawn further. **`swarm-deep` mode** allows recursive spawning at arbitrary depth by descendants, subject to a configurable live-worker budget and an absolute swarm member cap (❓ exact config keys/CLI flags for these budgets/caps not found in the fetched docs — `jcode.sh/swarm` is marked "unfinished" as of fetch date).
- **Roles:** Coordinator (owns the shared swarm-level plan; assigns scopes; approves plan updates — root-session role), Worktree Manager (owns integration for one git-worktree scope), Agents/workers (execute tasks, can DM or broadcast to each other, propose plan updates).
- **Lifecycle states:** spawned → ready → running → blocked → completed / failed / stopped / crashed.
- Server automatically notifies an agent when another agent edits a file it has read (conflict awareness), rather than using file locks.
- Swarm plan is server-side, scoped by `swarm_id`, not stored in a repo file; distribution is out-of-band.
- Agents can spawn their own sub-swarms autonomously via a swarm tool, turning the spawning agent into a coordinator.
- Session resume is supported across harnesses: Codex, Claude Code, OpenCode, and pi sessions can be resumed from jcode.

## Notes

- **RAM/perf claims are vendor-published benchmarks** (from the project's own README, using `jcode-bench`), not independently verified here: e.g. ~27.8 MB PSS single-session (local embedding off) vs. reported 386.6 MB for Claude Code 2.1.86, and ~14.0 ms time-to-first-frame vs. reported 3436.9 ms for Claude Code, on the vendor's own Linux test machine. Treat as vendor marketing figures pending independent reproduction.
- **Memory system:** each turn/response is embedded as a semantic vector; queries a memory graph via cosine similarity to auto-inject relevant context (no explicit tool call needed). Memories are periodically extracted (semantic drift, K-turns-since-last-extraction, session end) via a "memory sideagent," and consolidated periodically ("ambient mode") to check staleness/conflicts.
- **Self-dev mode:** agents can be told to modify jcode's own Rust source, build, test, and reload the binary mid-session. Vendor explicitly recommends only frontier models for this due to codebase complexity.
- **40+ provider integrations** with built-in OAuth login flows (`jcode login --provider <name>`) for Claude, OpenAI, Gemini, GitHub Copilot, Azure OpenAI, Alibaba Cloud, Fireworks, MiniMax, Meta Model API, LM Studio, Ollama, OpenRouter, DeepSeek, and more; supports multi-account switching (`/account`).
- Persistent server/client architecture: `jcode serve` + `jcode connect` for remote/multi-client sessions; `jcode dictate` for voice input via a configured STT command.
- v0.75.0 shipped Aug 10, 2026 (GitHub release timestamp); v0.75.1–v0.75.5 and v0.76.0 followed within days, indicating a very high release cadence.
- ❓ Could not confirm from official sources: exact `swarm-prompt.md` schema, `~/.jcode/skills/` frontmatter schema (directory-per-skill with a `SKILL.md` file is now confirmed, but field-level schema is not), and the specific config keys for swarm-deep's live-worker budget / member cap (docs pages exist but are marked incomplete/under construction as of the fetch date — `jcode.sh/swarm` is still explicitly marked "unfinished" on re-check).

## Sources (Official)

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| Homepage / overview / memory / MCP summary | https://jcode.sh/ | 2026-08-15 | [official] |
| Docs index | https://jcode.sh/docs | 2026-08-15 | [official] |
| GitHub repo root (confirms MIT license, description, star count) | https://github.com/1jehuang/jcode | 2026-08-15 | [github] |
| README (install, performance, memory, swarm, providers, MCP, browser) | https://raw.githubusercontent.com/1jehuang/jcode/master/README.md | 2026-08-15 | [github] |
| Lifecycle Hooks reference | https://raw.githubusercontent.com/1jehuang/jcode/master/docs/HOOKS.md | 2026-08-15 | [github] |
| Spawn Hook reference | https://raw.githubusercontent.com/1jehuang/jcode/master/docs/SPAWN_HOOK.md | 2026-08-15 | [github] |
| Safety System design doc | https://raw.githubusercontent.com/1jehuang/jcode/master/docs/SAFETY_SYSTEM.md | 2026-08-15 | [github] |
| Swarm Architecture doc | https://raw.githubusercontent.com/1jehuang/jcode/master/docs/SWARM_ARCHITECTURE.md | 2026-08-15 | [github] |
| Repo root file listing (confirms AGENTS.md, docs/, sdk/, crates/) | https://api.github.com/repos/1jehuang/jcode/contents | 2026-08-15 | [github] |
| Releases (v0.75.0 date confirmation) | https://github.com/1jehuang/jcode/releases | 2026-08-15 | [github] |
| Swarm marketing/docs page (marked unfinished) | https://jcode.sh/swarm | 2026-08-15 | [official] |
