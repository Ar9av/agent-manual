# DeepSeek Harness

> DeepSeek's open-source, plugin-first agent harness ("dsh") — everything (models, tools, skills, sessions, sandboxes, storage, loops, orchestration, UI) is a Cordis plugin. Released as a v0.1 developer preview on/around 2026-08-13.

**Vendor:** DeepSeek | **License:** MIT | **Runtime:** Node.js ^22.19 or ≥24 (monorepo also contains a Python test surface — ❓ whether a standalone Python runtime/SDK ships in v0.1)

## Links

- Official landing page: https://deepseek.com/harness/en/
- GitHub (official): https://github.com/deepseek-ai/deepseek-harness
- Docs site (official, GitHub Pages): https://deepseek-harness.github.io/deepseek-harness/en/guide/quickstart
- Architecture reference: https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/architecture.md
- Config catalog: https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/config-catalog.md
- Changelog: ❓ still not located — no dedicated CHANGELOG.md/release-notes page found in the repo root, docs site nav, or reference index as of 2026-08-15 (re-checked)

---

## Installation

Quick start (npx, no clone):
```sh
npx @deepseek-ai/dsh web
```
Serves a local Web UI at `http://127.0.0.1:3080` by default.

From source:
```sh
git clone https://github.com/deepseek-ai/deepseek-harness.git
cd deepseek-harness
pnpm install
pnpm run build
pnpm dsh web
```

Headless / CLI demo:
```sh
pnpm dsh --profile headless "prompt"
```

Requirements: Node.js ^22.19 or ≥24 (CI also validates Node 26); corepack-enabled pnpm pinned to v11.7.0; `DEEPSEEK_API_KEY` (and optional `DEEPSEEK_BASE_URL`) env var or `.env` file for model access.

❓ No OS package manager (brew/apt/winget) or standalone binary installer was found — npx/npm and source-build appear to be the only documented v0.1 install paths.

---

## Configuration Files

DeepSeek Harness uses a layered **Cordis "profile → bundle → patch"** composition model rather than a single settings file.

| File / Path | Scope | Purpose |
|------|-------|---------|
| `package.json` (`dsh` field) | Package | `dsh.profile` lists a profile's bundles; `dsh.bundle` points at a bundle's Cordis patch file |
| `<profile>/cordis.patch.yml` | Profile (local) | User patches layered on top of a profile's bundle stack |
| `$DSH_HOME/cordis.patch.yml` (default `~/.dsh/cordis.patch.yml`) | User (global) | Home-level overrides applied on top of profile patches |
| `$DSH_HOME/AGENTS.md` (default `~/.dsh/AGENTS.md`) | User (global) | Fixed user-global instruction file, loaded into every session |
| `.dsh/skills/`, `.agents/skills/` | Project | Filesystem-discovered project Skills (SKILL.md) roots |
| `.dsh/mcp.json` ❓ | Project | Referenced by a **community** bridge plugin (`dsh-project-mcp-bridge`) for per-project MCP loading — ❓ not confirmed as a first-party/official config path; official MCP wiring is via `cordis.yml`/patch rows for the `mcp-client` plugin |
| CLI `--patch` flag | Session | One-off Cordis config overlay for a single run |

View the fully composed config with:
```sh
dsh --profile web --dump-config
```

---

## Instruction File

The official `agent-instructions` plugin's loader "loads AGENTS.md and CLAUDE.md-compatible project files" — it reads the fixed user-global `$DSH_HOME/AGENTS.md` (default `~/.dsh/AGENTS.md`) plus the project directory hierarchy, and re-checks nested instruction files after first-party file operations. Automatic loading and its byte budget are configurable, or can be disabled (`false`). ❓ Exact precedence/merge order between user-global, project-root, and nested instruction files was not confirmed from primary sources beyond "loads the user-global file and the project hierarchy."

---

## Hooks

DeepSeek Harness's **native** extension surface is Cordis's typed plugin-event system, not a Claude-Code-style external command-hook config file. Any plugin can listen on these events (`agent/*`, `tools/pre-execute`, `tools/execute`, `tools/post-execute`, `session/event`, `turn/start`, `turn/end`, `step/start`, `subagent/start`, `subagent/end`, etc.) and must call `next()` in waterfall listeners to delegate. This requires writing a Cordis plugin (TypeScript), not dropping in a shell script.

For teams migrating existing external hook configs, the official monorepo ships **bridge plugins** under `packages/hooks/` that translate other tools' hook wire-formats onto these native interception points:

- `@deepseek-ai/dsh-hooks-claude-code` — bridges an existing Claude Code `hooks.json` onto the harness
- `@deepseek-ai/dsh-hooks-codex` — bridges Codex-style hooks onto the same interception points as the Claude Code bridge (confirmed via its official README, re-fetched 2026-08-15)
- `dsh-hook-protocol` — shared wire-protocol library the two bridges are built on

### Supported Events (via the `hooks-claude-code` bridge only — 7 of Claude Code's ~30 hook events)

| Claude Code event | Harness interception point | Outcome mapping | Can Block? |
|-------|--------------|-----------|-----------|
| `SessionStart` | `agent/session-start` | Emits `additionalContext` injected into the new session | ❌ |
| `UserPromptSubmit` | `agent/pre-step` | `deny` → prompt rejected; otherwise delegates via `next()` | ✅ |
| `PreToolUse` | `tools/pre-execute` | `deny`/`ask` → typed `PreToolDecision` | ✅ |
| `PostToolUse` | `tools/post-execute` | `deny` → blocks; context prepended to decision | ❌ (tool already ran) |
| `Stop` | `agent/turn-stopping` | Blocking reason fed through `steer()` | ✅ |
| `SubagentStart` | `subagent/start` | Context injected into in-process child only | ❌ |
| `SubagentStop` | `subagent/end` | Observe-only emission | ❌ |

The other 23 Claude Code hook events (e.g. `PermissionRequest`, `PreCompact`, `Notification`, etc.) are **not** bridged in v0.1. Only shell-form `type: "command"` hooks are executed by the bridge; `http`/`mcp_tool`/`prompt`/`agent`-type hook entries are parsed and skipped with a warning.

### `hooks-codex` bridge event mapping (confirmed 2026-08-15)

The `@deepseek-ai/dsh-hooks-codex` package's own README (re-fetched) confirms a narrower mapping — 5 Codex hook events, no subagent events:

| Codex event | Harness interception point | Outcome mapping |
|-------|--------------|-----------|
| `SessionStart` | `agent/session-start` (emit) | Plain stdout → `additionalContext` via `agent.inject()` |
| `UserPromptSubmit` | `agent/pre-step` (waterfall) | Exit 2 → reject; otherwise context appended downstream |
| `PreToolUse` | `tools/pre-execute` (waterfall) | Exit 2 → deny tool execution |
| `PostToolUse` | `tools/post-execute` (waterfall) | Exit 2 → block with feedback; context prepended |
| `Stop` | `agent/turn-stopping` (serial) | Blocking reason fed via `steer()`, forcing another step |

Config: `configPath` (required, points at `.codex/hooks.json`), `model` (optional, stamped on every payload), `defaultTimeoutMs` (default 600000), `stderrSummaryMaxChars` (default 500). Config loads once at startup; relative paths resolve against the process launch directory, not per-session.

### Hook Input (bridge behavior)

The bridge reuses Claude-Code-shaped per-event stdin JSON payloads and substitutes `${CLAUDE_PLUGIN_ROOT}` / `${CLAUDE_PROJECT_DIR}` env vars, so an existing CC hook script largely works unmodified. ❓ Exact JSON schema for native (non-bridged) Cordis event payloads was not confirmed.

### Exit Code Behavior (confirmed 2026-08-15)

Both bridges share the `dsh-hook-protocol` codec, whose README (re-fetched) states the mapping used by `parseHookOutput`: **exit 0** → success, hook output parsed normally; **exit 2** → blocks the operation, stderr reported as the blocking reason; **any other non-zero exit** → non-blocking failure (operation continues). This matches Claude Code's exit-2-blocks convention. A hook-specific permission decision in the parsed JSON output overrides this legacy exit-code-only behavior when present.

### Example Config (bridge plugin mount, Cordis-style)

```yaml
- id: hooks-cc
  name: '@deepseek-ai/dsh-hooks-claude-code'
  config:
    configPath: '/path/to/hooks.json'   # required — an existing Claude Code hooks.json
    pluginRoot: '/path/to/plugin'       # optional
    projectDir: '/path/to/project'      # optional
    defaultTimeoutMs: 600000            # optional
    stderrSummaryMaxChars: 500          # optional
```

---

## Built-in Tools

Tools are Cordis plugin packages registered on `ctx.tools` and organized under `packages/<group>/<pkg>/`. Confirmed capability families (24 tool packages per community/aggregator reporting — ❓ exact count not verified against an official manifest):

| Tool / Family | Description |
|------|-------------|
| `fs` (read/write/edit/glob) | Filesystem capability: local implementation, model-facing file tools, bash-backed discovery |
| `shell` / `bash` | Bash executor seam + model-facing bash tool |
| `pwsh` | PowerShell variant (local + sandboxed configs found in config catalog: `pwsh-local`, `pwsh-sandbox`) |
| `terminal` | Persistent PTY sessions, owner-scoped, with model-facing tools |
| `lsp` | Language Server Protocol navigation tool |
| `web` | Web search/fetch providers + model-facing web tools |
| `code-runtime` | Sandboxed code execution via worker-thread provider |
| `subagent` | Delegation tool for spawning subagents |
| `skill` | Skill registry, local provider, model-facing catalog/loader tool |
| `workflow` | Workflow engine (worker-thread) + model-facing workflow tools |
| `mcp` (`mcp-client`) | Registers external MCP server tools into `ctx.tools` |

Two named **operating modes** affect which tools are present:
- **Standard** — full coding agent: filesystem tools, shell access, web, subagents, plan mode.
- **Minimal** — two tools only, `bash` and `str_replace_editor`, intended for benchmarking.
- **Code Mode** — tools exposed via an SDK for TypeScript-based orchestration rather than direct model tool-calls.
- **Creator** — runtime inspection / custom preset development mode.

**Update (confirmed 2026-08-15):** the official docs site now publishes a generated tool-schema catalog at `en/reference/tool-catalog`, which enumerates exact model-facing tool names, including:

- Filesystem: `read`, `write`, `edit`, `read_image`, `str_replace_editor`, `glob`, `grep`
- Shell/terminal: `bash`, `pwsh`, `terminal_open`, `terminal_close`, `terminal_list`, `terminal_read`, `terminal_send`, `terminal_signal`
- Web: `web_search`, `web_fetch`
- Code: `run_code`, `lsp`
- Jobs: `job_list`, `job_output`, `job_kill`
- Agents/delegation: `subagent`, `subagent_fork`, `send_message`, `interrupt_agent`, `list_agents`, `report`
- Workflow: `workflow`, `ralph`
- Session: `session_search`, `session_trace`, `session_event_read`, `session_event_search`, `session_event_trace`
- Goals/tasks: `create_goal`, `get_goal`, `update_goal`, `todo_write`
- Scheduling: `schedule_create`, `schedule_delete`, `schedule_list`
- Skills: `skill`
- Interaction: `ask_user_question`, `exit_plan_mode`
- Dynamic/creator-mode: `cordis_define`, `cordis_inspect_list`, `cordis_inspect_query`, `cordis_inspect_self`, `cordis_run`, `cordis_stop`, `cordis_undefine`

This list was reconstructed by an AI fetch summary of that page rather than a byte-for-byte transcription, so treat exact spelling/completeness as high-confidence but not hand-verified against the raw markdown.

---

## MCP Support

MCP is officially supported via the `@deepseek-ai/dsh-mcp-client` plugin. One plugin instance = one MCP server, mounted as a row in `cordis.yml`/patch (not a simple top-level `mcpServers` block like Claude Code's `.mcp.json`).

Transports: `stdio` (local subprocess) and `streamable-http` (remote endpoint).

Discovered external tools are namespaced `mcp__<serverName>__<rawName>` (matches Claude Code's convention) and pass through the same `tools/pre-execute` / `tools/post-execute` pipeline as native tools.

Example (stdio):
```yaml
- id: mcp-github
  name: '@deepseek-ai/dsh-mcp-client'
  config:
    serverName: github
    transport: stdio
    command: npx
    args: ['-y', '@modelcontextprotocol/server-github']
    env:
      GITHUB_TOKEN: !!js process.env.GITHUB_TOKEN
```

Example (remote):
```yaml
- id: mcp-web
  name: '@deepseek-ai/dsh-mcp-client'
  config:
    serverName: web
    transport: streamable-http
    url: http://localhost:3000/mcp
```

Config fields include `transport`, `serverName` (1–32 chars, alphanumeric/`_`/`-`), `command`/`args`/`env` (stdio) or `url` (http), `toolCallTimeoutMs` (default 60000), `failOnStartupError` (default `false`).

**Re-checked 2026-08-15:** the official `docs/config-catalog.md` (re-fetched from GitHub) still lists only `cordis.yml`, `.credentials.yaml`, and `settings.yaml` under `$DSH_HOME` as config paths — no `.dsh/mcp.json` is mentioned anywhere in official docs. A project-scoped `.dsh/mcp.json` convenience file remains attested only in the third-party community bridge plugin (`dsh-project-mcp-bridge`) — do not treat it as first-party.

---

## Skills / Commands

- Skills location: `.dsh/skills/` and `.agents/skills/` (project-scoped, filesystem-discovered), plus configured user and bundled roots — per the official `skill` package (registry, local provider, model-facing catalog/loader).
- Format: SKILL.md-based — the harness's skill package exposes a catalog/loader tool to the model. DeepSeek Harness is reported (via search aggregation, not fetched official docs) to be compatible with the open Agent Skills SKILL.md standard used by Claude Code, Codex, Cursor, and others.
- **Confirmed 2026-08-15** via the official docs site's Skills subsystem reference (`en/reference/subsystems/skills`): the local skill provider recognizes kebab-case frontmatter keys `disable-model-invocation` and `user-invocable` (both default `true`), which normalize into a `SkillInvocationPolicy` controlling whether a skill appears in model-facing vs. human-facing catalogs. Skills are discovered, ranked by priority, from `.dsh/skills` (100), `.agents/skills` (200), config-specified custom dirs (300), `<dshHome>/skills` (400), `<agentsHome>/skills` (500), and an optional bundled root (600). Accepted layouts are `<name>/SKILL.md` directory bundles or flat `<name>.md` files; nested/recursive `**/SKILL.md` discovery is **not** supported. Names must match `^[a-z0-9]+(?:-[a-z0-9]+)*$`. Whether `name`/`description` fields are present and semantically identical to Claude Code's SKILL.md was still not spelled out on this page — treat that narrower point as likely-compatible, not verified-identical.
- No slash-command file format was documented in official sources; the Skills subsystem reference page specifically does not mention slash-command support. ❓ whether `dsh` has a Claude-Code-style `commands/*.md` convention remains unconfirmed (re-checked 2026-08-15, still absent from official docs).

---

## Agent / Subagent Configuration

Subagent delegation is a first-class capability seam: `subagent` package provides a provider-registry contract plus a model-facing delegation tool. Related plugin packages referenced in search results (not independently fetched from official source): `@deepseek-ai/dsh-subagent`, `@deepseek-ai/dsh-subagent-spawn-in-process`, `@deepseek-ai/dsh-subagent-fork-in-process`, `@deepseek-ai/dsh-tool-subagent-control`, `@deepseek-ai/dsh-tool-subagent-report`.

Subagents are spawned in-process (per the `subagent-*-in-process` package names) and can receive hook-bridge context injection on `SubagentStart`/`subagent/start` (see Hooks section).

**Confirmed 2026-08-15** via the official docs site's Subagent subsystem reference (`en/reference/subsystems/subagent`): providers register by name on `ctx.subagents` and advertise a static `SubagentCapabilities` descriptor (`outputSchema`, `depthLimit`/`maxDepth`, `toolFilter`, `persona`) checked before a run starts — an unsupported capability rejects the start with a typed `SubagentError` rather than silently degrading. A `SubagentStartRequest` carries `prompt`, `parent` (workspace/lineage/depth source), `signal`, and optional `outputSchema`, `maxDepth`, `toolFilter`, `persona`. Two operational modes exist: one-shot runs (`start()` → disposable `SubagentRun` with a terminal `result` promise) and continuable/durable background sessions (optional `prepareContinuable()`). Included providers: spawn, fork, ACP, Codex, Claude Code, SDK — fork uniquely inherits the parent's context; the others start fresh. `listChildren()`/`listDescendants()` enumerate session-backed subagents. Per-subagent model override was not spelled out as a distinct top-level field on this page (persona/toolFilter are the confirmed scoping knobs); treat model-override support as still unconfirmed.

---

## Notes

- Core architectural idea: **"everything is a plugin"** — models, tools, skills, sessions, sandboxes, storage, loops, scheduling, and the UI are all Cordis plugins with "no privileged core."
- Sandboxing: a `sandbox` package exists with **bwrap / Landlock / Seatbelt** backends (Linux: bwrap then falls back to a native Landlock launcher; macOS: Seatbelt; Windows: ACL restricted-token) for process confinement, supplied by `dsh-sandbox-local` and consumed by the bash/PowerShell sandbox packages. **Confirmed 2026-08-15** via the official docs site's Sandboxing subsystem reference: three file-effect policy modes exist — `read-only` (denies writes except required sinks like `/dev/null`), `workspace-write` (writes confined to the workspace root plus a backend-defined temp area), and `danger-full-access` (bypasses confinement entirely; consumers spawning this mode skip the sandbox service). Only the first two are "confined" and reach the sandbox provider. Backends self-report enforcement completeness as `full` or `partial` (e.g. older Landlock kernel ABIs, Windows hard-link limits); callers requiring absolute guarantees must reject `partial`. Workspace-root resolution precedence: explicit mode override → session's immutable working directory → deployment-configured fallback root. The page did not state a single global default mode (it's resolved per capability call from approval/session context), so treat "default" as **per-session/per-call**, not a fixed harness-wide setting.
- Session log: "everything the model sees is recorded in an append-only session log," enabling run inspection, forking, search, and replay (`session-query` package: bounded reads, semantic filtering, full-text search).
- Extensions package exists for "agent runtime self-modification capabilities" — ❓ scope/safety model not confirmed.
- This is a **v0.1 developer preview**: the project explicitly warns of compatibility-breaking changes and says it's suitable for experimenting/prototyping/contributing but not yet for critical workflows.
- Positioned by press coverage as an open-source (MIT), plugin-first rival to Claude Code, built to run DeepSeek's V4-series models as autonomous coding agents.
- This page is a **doc-only research pass** based on official/GitHub sources plus limited press coverage where official docs were thin; it does not reflect a live install/test of the tool.

---

## Sources (Official)

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| Official product landing page | https://deepseek.com/harness/en/ | 2026-08-15 | [official] |
| GitHub repo (root) | https://github.com/deepseek-ai/deepseek-harness | 2026-08-15 | [github] |
| README | https://github.com/deepseek-ai/deepseek-harness/blob/master/README.md | 2026-08-15 | [github] |
| AGENTS.md (repo conventions) | https://github.com/deepseek-ai/deepseek-harness/blob/master/AGENTS.md | 2026-08-15 | [github] |
| Architecture doc | https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/architecture.md | 2026-08-15 | [github] |
| Development guide | https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/development.md | 2026-08-15 | [github] |
| Config catalog | https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/config-catalog.md | 2026-08-15 | [github] |
| Packages directory listing | https://github.com/deepseek-ai/deepseek-harness/tree/master/packages | 2026-08-15 | [github] |
| Hooks package directory | https://github.com/deepseek-ai/deepseek-harness/tree/master/packages/hooks | 2026-08-15 | [github] |
| hooks-claude-code bridge README | https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/hooks/hooks-claude-code/README.md | 2026-08-15 | [github] |
| Docs site — quickstart | https://deepseek-harness.github.io/deepseek-harness/en/guide/quickstart | 2026-08-15 (re-verified) | [official] |
| Docs site — reference/architecture | https://deepseek-harness.github.io/deepseek-harness/en/reference/ | 2026-08-15 (re-verified) | [official] |
| Docs site — reference/tool-catalog | https://deepseek-harness.github.io/deepseek-harness/en/reference/tool-catalog | 2026-08-15 | [official] |
| Docs site — subsystems/sandbox | https://deepseek-harness.github.io/deepseek-harness/en/reference/subsystems/sandbox | 2026-08-15 | [official] |
| Docs site — subsystems/subagent | https://deepseek-harness.github.io/deepseek-harness/en/reference/subsystems/subagent | 2026-08-15 | [official] |
| Docs site — subsystems/skills | https://deepseek-harness.github.io/deepseek-harness/en/reference/subsystems/skills | 2026-08-15 | [official] |
| hooks-codex bridge README | https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/hooks/hooks-codex/README.md | 2026-08-15 | [github] |
| hook-protocol README (exit-code mapping) | https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/hooks/hook-protocol/README.md | 2026-08-15 | [github] |
| DeepSeek official announcement (X/Twitter) | https://x.com/deepseek_ai/status/2087887408440164663 | 2026-08-15 (re-check failed — HTTP 402, see Sources note) | [official, unverified] |
| Third-party MCP-in-DSH writeup (mirrors official mcp-client behavior; not deepseek.com) | https://deepseekdocs.com/en/docs/features/mcp | 2026-08-15 (re-verified) | [third-party] |
| Press coverage — release summary | https://cryptobriefing.com/deepseek-harness-open-source-developer-preview/ | 2026-08-15 (re-verified) | [press] |

**Re-verification note (2026-08-15):** all URLs above except the X/Twitter announcement were re-fetched and still load and support their attributed claims — no dead links, no content that contradicts the claims cited from it, no vendor/license/repo-identity changes. The X/Twitter post returned an HTTP 402 (Payment Required) on this re-check; this is a WebFetch access limitation on X, not confirmed evidence the post itself was removed. Treat that single citation as unverified rather than dead, and do not rely on it alone — its claims (v0.1 release announcement) are independently corroborated by the GitHub repo and press coverage. No vendor, license, MCP config path/format, or install-command claims changed in this pass; several previously-❓ items (tool-name enumeration, sandbox policy modes, subagent capability schema, hooks-codex mapping, hook exit-code table, SKILL.md frontmatter fields) were newly confirmed from official docs-site pages not previously cited and have been added to this table.
