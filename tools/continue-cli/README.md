# Continue CLI (`cn`)

> Continue Dev's standalone terminal coding agent — distinct from the Continue VS Code/JetBrains extension, though it shares config format and MCP conventions with them.

**Vendor:** Continue Dev, Inc. (acquired by Cursor/Anysphere, June 18 2026 — see Notes) | **License:** Apache 2.0 | **Runtime:** Node.js 20+ (the shell installer bundles its own runtime)

## Links

- Docs (quickstart): https://docs.continue.dev/cli/quickstart
- Docs (install): https://docs.continue.dev/cli/install
- Docs (configuration): https://docs.continue.dev/cli/configuration
- Docs (tool permissions): https://docs.continue.dev/cli/tool-permissions
- Docs (MCP, shared with IDE extensions): https://docs.continue.dev/customize/deep-dives/mcp
- GitHub repo: https://github.com/continuedev/continue (`extensions/cli`) — now **read-only**, see Notes
- npm package: https://www.npmjs.com/package/@continuedev/cli

---

## Installation

**macOS / Linux (shell installer, bundles its own Node runtime):**
```sh
curl -fsSL https://raw.githubusercontent.com/continuedev/continue/main/extensions/cli/scripts/install.sh | bash
```

**npm (requires Node.js 20+):**
```sh
npm i -g @continuedev/cli
```

**Verify:**
```sh
cn --version
```

**First run / auth** — on first launch `cn` prompts to either log in with a Continue account (`cn login`, browser-based OAuth) or paste an Anthropic API key directly. For headless/CI use, set an API key as an environment variable:
```sh
export CONTINUE_API_KEY=your-key-here
cn -p "your prompt"
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.continue/config.yaml` | Global | Default configuration (same schema as the IDE extensions) |
| Custom path via `--config ./my-config.yaml` or an org config slug | Project/launch | Overrides default config for a session |
| `.continue/mcpServers/*.yaml` (or `.json`) | Project | Standalone MCP server "block" files |
| `.continue/skills/<name>/SKILL.md` | Project | Agent Skills (see below) |
| `~/.claude/settings.json`, `~/.continue/settings.json` | Global | Hooks config (lowest precedence) |
| `.claude/settings.json`, `.continue/settings.json` | Project | Hooks config |
| `.claude/settings.local.json`, `.continue/settings.local.json` | Project (local) | Hooks config, highest precedence |

Config resolution order at launch: `--config` flag → last-used/saved config → default `~/.continue/config.yaml`. Within a TUI session, `/config` switches between available configs and persists the choice. Secrets should be stored as environment variables and referenced with `${{ secrets.MY_API_KEY }}` syntax rather than committed to config files. ❓ The exact on-disk layout of a project-level `.continue/config.yaml` (vs. global-only) was not explicitly documented on the pages fetched — treat project-level config as inferred-by-convention rather than confirmed.

## Instruction File

Launch-time flags let you inject rules without editing config: `--rule ./rules/style.md` or `--rule "inline rule text"` (repeatable), and `--agent my-org/pr-reviewer` (repeatable) to attach reusable custom agents/prompts. ❓ Whether `cn` also auto-reads a standalone convention file such as `AGENTS.md` or `CLAUDE.md` from the project root (the way many other CLIs do) is not confirmed on the fetched docs pages — the CLI's own `extensions/cli/AGENTS.md` in the GitHub repo is contributor-facing engineering documentation, not evidence of end-user auto-loading behavior.

## Hooks

`cn` ships a **hooks system explicitly modeled on / compatible with Claude Code's** (per the CLI's own `AGENTS.md` and multiple "add hooks docs" GitHub issues opened against `docs.continue.dev`). As of this writing there is **no dedicated hooks page on docs.continue.dev** — the feature (merged via PR #11029, ~March 2026) is documented only in the CLI source tree (`extensions/cli/src/hooks/types.ts`, `extensions/cli/AGENTS.md`) and tracked as a docs gap in GitHub issues #11536, #11678, #11758. The table below is sourced from those files, not from published end-user docs — verify against source before relying on it in production.

> ⚠️ **Live-tested 2026-07-23 and root-caused further — hooks are wholesale non-functional in headless (`-p`) mode, across every event type and config location tried.** On Ubuntu 24.04 with `cn` v1.5.47, systematically isolated the variables:
> - **Event type:** tested both `PreToolUse` (should fire before a `Bash` tool call) and `UserPromptSubmit` (should fire the instant a prompt is submitted, with no tool call involved at all) — **neither ever fired**.
> - **Config file location:** tried a custom `--config ./continue-config.yaml`, the default project path `.continue/config.yaml`, hooks in project-scope `.continue/settings.json`, and hooks in global-scope `~/.continue/settings.json` — **none caused the hook to fire**.
> - **Flags:** tried `--auto` and `--allow Bash` — no difference.
> - In every case the underlying task itself completed successfully (the shell command ran, the prompt got a reply) — this isn't a case of the CLI failing outright, just the hooks layer silently doing nothing.
>
> Traced this to `extensions/cli/src/commands/chat.ts`'s `runHeadlessMode()`, which *does* call `initializeServices({ headless: true, ... })` (which in turn registers and eagerly initializes the `HOOKS` service per `extensions/cli/src/services/index.ts`) before running the prompt — so on paper, hooks should be ready in time. The practical symptom nonetheless matches a real code-level caveat in `extensions/cli/src/hooks/fireHook.ts`: hook-firing calls are explicitly designed to "be safe to call even if the HookService is not yet initialized (they return no-op results)." Given `UserPromptSubmit` — the very first hook event of any session — never fires either, the failure isn't tool-specific or timing-marginal; something in the headless/`-p` path is either never actually reaching `isHookServiceReady()`-gated firing code, or the service reports itself ready without having loaded any hook config. This wasn't traced further into the service container internals, but the finding is now solid: **treat hooks as effectively non-functional in `cn -p`/headless mode in this version — do not rely on them for anything security-sensitive in CI or scripted use, regardless of how you configure them.** (Not re-tested in interactive mode; it's possible hooks work there and the bug is `-p`-specific.)

### Supported Events (17 total)

| Event | When (best available description) | Can Block? |
|-------|------|-----------|
| `PreToolUse` | Before a tool executes | ✅ via `permissionDecision: deny` or exit 2 |
| `PostToolUse` | After a tool completes successfully | ❌ (post-execution; can inject `additionalContext`/feedback) |
| `PostToolUseFailure` | After a tool call fails | ❌ |
| `PermissionRequest` | When a permission decision is needed for a tool call | ✅ via structured `behavior: allow/deny` |
| `UserPromptSubmit` | When the user submits a message | ✅ (exit 2 blocks the prompt) |
| `SessionStart` | Session begins (`source`: startup/resume/clear/compact) | ❌ |
| `SessionEnd` | Session ends | ❌ |
| `Stop` | Agent wants to stop / finish turn | ✅ |
| `Notification` | CLI emits a notification | ❌ |
| `SubagentStart` | A subagent starts | ❌ |
| `SubagentStop` | A subagent finishes | ✅ |
| `PreCompact` | Before context compaction | ❌ |
| `ConfigChange` | Configuration is changed mid-session | ❌ |
| `TeammateIdle` | A teammate/agent goes idle (team mode) | ❌ |
| `TaskCompleted` | A task completes (team mode) | ❌ |
| `WorktreeCreate` | A git worktree is created | ❌ |
| `WorktreeRemove` | A git worktree is removed | ❌ |

❓ Block-ability for events not explicitly documented as blocking (`PostToolUse`, `SessionStart/End`, `Notification`, `SubagentStart`, `PreCompact`, `ConfigChange`, `TeammateIdle`, `TaskCompleted`, `WorktreeCreate/Remove`) is inferred from event semantics (lifecycle/notification-only events are typically non-blocking in this hook-model family), not from an explicit statement per event — treat as unverified.

### Hook Handler Types

Four handler types share common optional fields `timeout`, `statusMessage`, `once`:

| Type | Behavior |
|------|----------|
| `command` | Runs a shell command; stdin/stdout JSON, supports async execution |
| `http` | Sends a POST request to an endpoint, with header/env-var interpolation |
| `prompt` | Single-turn LLM evaluation of the event (per `AGENTS.md`, listed as "not yet implemented" even though a `PromptHookHandler` type exists in source — inconsistency noted) |
| `agent` | Multi-turn subagent handler with its own tool access and model selection (same "not yet implemented" caveat applies per `AGENTS.md`) |

Handlers are grouped by regex `matcher` (e.g. to filter by tool name) into arrays under each event key — this mirrors Claude Code's `matcher` + `hooks[]` grouping structure.

### Hook Input (stdin JSON) — common fields

```json
{
  "session_id": "...",
  "transcript_path": "...",
  "cwd": "...",
  "permission_mode": "..."
}
```

Event-specific additions include: `tool_name`, `tool_input`, `tool_use_id` (tool events; `PostToolUse` adds `tool_response`), `source` (`SessionStart`), `agent_id`/`agent_type`/`agent_transcript_path` (subagent events), `task_id`/`task_subject`/`task_description`/`teammate_name`/`team_name` (task/team events).

### Hook Output (stdout JSON, optional) — common fields

```json
{
  "continue": true,
  "suppressOutput": false,
  "decision": "approve",
  "reason": "explanation shown to the agent/user"
}
```

Event-specific output extensions: `PreToolUse` → `permissionDecision` (`allow`/`deny`/`ask`), `updatedInput`, `additionalContext`; `PermissionRequest` → structured decision object with `behavior` (`allow`/`deny`) plus optional params; `PostToolUse` → `updatedMCPToolOutput`. A `hookSpecificOutput` wrapper carries this fine-grained, per-event data.

### Exit Code Behavior

| Code | Meaning |
|------|---------|
| `0` | Proceed normally (stdout parsed as JSON if present) |
| `2` | Block the action; stderr becomes feedback |
| Other | Non-blocking error; execution continues |

### Config Merging

Hooks config is merged from **both** `.continue/` and `.claude/` directories (project and global, with `.local.json` variants taking highest precedence), which is how `cn` claims Claude Code hook-config compatibility — existing Claude Code `settings.json` hook blocks can be picked up directly.

## Built-in Tools

| Tool | Description | Default permission |
|------|-------------|---------------------|
| `Read` | Read file contents | allow |
| `List` | List directory contents | allow |
| `Search` | Search codebase | allow |
| `Fetch` | Fetch a URL | allow |
| `Diff` | Show a diff | allow |
| `AskQuestion` | Ask the user a clarifying question | allow |
| `Checklist` | Manage a task checklist | allow |
| `Status` | Report status | allow |
| `CheckBackgroundJob` | Poll a background job's status | allow |
| `ReportFailure` | Report a failure | allow |
| `UploadArtifact` | Upload an artifact | allow |
| `Edit` | Apply an edit to a file | ask |
| `MultiEdit` | Apply multiple edits | ask |
| `Write` | Write/create a file | ask |
| `Bash` | Execute a shell command | ask |

Permission modes: `--allow <tool>`, `--ask <tool>`, `--exclude <tool>` override defaults per-launch; `cn --auto` allows all tools with no prompting; `cn --readonly` restricts to read-only tools (plan mode, no writes) and ignores other flags.

## MCP Support

**Yes.** Config lives in `.continue/mcpServers/` as standalone block files (YAML or JSON), or inline under `mcpServers:` in `config.yaml`. JSON MCP configs copied from Claude Desktop/Cursor/Cline are auto-detected and picked up directly if dropped into `.continue/mcpServers/`.

**Standalone block file** (`.continue/mcpServers/playwright.yaml`):
```yaml
name: Playwright mcpServer
version: 0.0.1
schema: v1
mcpServers:
  - name: Browser search
    command: npx
    args:
      - "@playwright/mcp@latest"
```

**Inline in `config.yaml`:**
```yaml
mcpServers:
  - name: SQLite MCP
    command: npx
    args:
      - "-y"
      - "mcp-sqlite"
      - "/path/to/your/database.db"
```

**Remote transport (SSE):**
```yaml
mcpServers:
  - name: Name
    type: sse
    url: https://....
```

Supported transports: `stdio` (`command`/`args`), `sse`, and `streamable-http` (both via `url`).

## Skills

Built-in "Skills" tool for task-specific agent instructions, added per GitHub PR #9696 (tracked as needing a dedicated `docs/cli/skills.mdx` page in issue #11758 — not yet published as of this writing). Skill files follow the `SKILL.md` convention and are read from:

| Location | Scope |
|----------|-------|
| `.continue/skills/<skill-name>/SKILL.md` | Project |
| `.claude/skills/` | Project (Claude Code compatibility) |
| `$CONTINUE_HOME/skills/` | Global (custom home dir) |

❓ Exact `SKILL.md` frontmatter schema (fields, required vs. optional) is not documented on a published docs.continue.dev page — inferred to be compatible with the general SKILL.md convention used elsewhere in the ecosystem, unconfirmed for `cn` specifically.

## Agent / Subagent Configuration

Reusable custom agents/prompts can be attached at launch with `--agent my-org/pr-reviewer` (repeatable). The hooks system separately supports `SubagentStart`/`SubagentStop` events and an `agent` hook-handler type for spawning subagents as hook responders. ❓ Beyond these launch-flag and hook-handler mechanics, no dedicated subagent-authoring doc page was found.

> **Live-verified 2026-07-23:** The default tool set exposed to the model in a real session (captured from the CLI's own debug log) is `Read, Write, List, Bash, Fetch, Checklist, CheckBackgroundJob, AskQuestion, Search, MultiEdit, Exit, Skills` — **no `Task`/`Agent`/`Delegate`-style tool is present**, so there is no way for the model to spawn a live subagent mid-session on its own initiative (unlike Claude Code's `Task` tool or Goose's `delegate` tool). `--agent <slug>` only swaps which single agent/prompt configuration runs for the *whole* session at launch — it is not runtime delegation. Treat "subagents" here as a **hooks-adjacent, mid-rollout concept** (per the `AGENTS.md` note below), not a working feature you can invoke today.

## Notes

- **Continue CLI vs. Continue IDE extensions**: `cn` shares `config.yaml` schema and MCP conventions with the VS Code/JetBrains extensions but is a separate, standalone terminal binary/npm package (`@continuedev/cli`).
- **Acquisition and maintenance status**: Cursor (Anysphere) acquired Continue, announced June 18, 2026. Continue Dev shipped a **final 2.0.0 release** of the VS Code extension, CLI, and JetBrains plugin (removing anonymous telemetry, dropping the old auth flow, bug fixes), after which the `continuedev/continue` GitHub repository went **read-only** — no new issues, PRs, or roadmap updates from the original maintainers. The code remains Apache 2.0 and forkable, but is not being actively updated upstream. `docs.continue.dev` was still reachable and serving CLI docs as of this research (2026-07-23), but given the acquisition, doc freshness/roadmap continuity going forward is uncertain. ❓ Treat any "current" feature claims here as a snapshot; re-verify before relying on them for new work.
- Hooks documentation is a known, actively tracked gap on the official docs site (three separate GitHub issues requesting it) — this page's hooks section is reconstructed from CLI source (`extensions/cli/src/hooks/types.ts`, `extensions/cli/AGENTS.md`), not from a published doc, and should be re-verified against source if precision matters (e.g. building an integration).
- `AGENTS.md` in the CLI source states `prompt`/`agent` hook handlers are "not yet implemented" while the type system (`types.ts`) already defines `PromptHookHandler`/`AgentHookHandler` — this discrepancy suggests the feature is mid-rollout; do not assume prompt/agent hooks are usable without testing.

## Sources

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| CLI quickstart (install, auth) | https://docs.continue.dev/cli/quickstart | 2026-07-23 | [official] |
| CLI configuration (config.yaml, --rule, --agent, secrets) | https://docs.continue.dev/cli/configuration | 2026-07-23 | [official] |
| CLI tool permissions (built-in tools, --allow/--ask/--exclude, --auto/--readonly) | https://docs.continue.dev/cli/tool-permissions | 2026-07-23 | [official] |
| MCP deep dive (.continue/mcpServers/, transports, examples) | https://docs.continue.dev/customize/deep-dives/mcp | 2026-07-23 | [official] |
| npm package @continuedev/cli (install command, package identity) | https://www.npmjs.com/package/@continuedev/cli | 2026-07-23 | [official] |
| GitHub repo README (license, read-only/archival notice, final 2.0.0 release) | https://github.com/continuedev/continue/blob/main/README.md | 2026-07-23 | [github] |
| Hooks event types & schema (source) | https://github.com/continuedev/continue/blob/main/extensions/cli/src/hooks/types.ts | 2026-07-23 | [github] |
| Hooks config, exit codes, handler types (source) | https://github.com/continuedev/continue/blob/main/extensions/cli/AGENTS.md | 2026-07-23 | [github] |
| Docs gap tracking issue: hooks documentation | https://github.com/continuedev/continue/issues/11536 | 2026-07-23 | [github] |
| Docs gap tracking issue: hooks documentation (dup) | https://github.com/continuedev/continue/issues/11678 | 2026-07-23 | [github] |
| Docs gap tracking issue: hooks, ai-sdk, cn checks, MCP Apps, skills | https://github.com/continuedev/continue/issues/11758 | 2026-07-23 | [github] |
| Repo archival request issue (read-only status confirmation) | https://github.com/continuedev/continue/issues/12629 | 2026-07-23 | [github] |
| Cursor acquisition of Continue (press context) | https://webdeveloper.com/news/cursor-acquires-continue-open-source-agent/ | 2026-07-23 | [press] |
