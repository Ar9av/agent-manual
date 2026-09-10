# oh-my-pi (omp)

> A Pi fork rebuilt as a coding-first surface: 60+ providers, an ~80k-line Rust core, LSP/DAP wired into the tool surface, and first-class subagents.

**Vendor:** can1357 (community) | **License:** MIT | **Runtime:** Bun/TypeScript + prebuilt native core

## Links

- Docs: https://omp.sh
- Hooks: https://github.com/can1357/oh-my-pi/blob/HEAD/docs/hooks.md
- Extensions: https://github.com/can1357/oh-my-pi/blob/HEAD/docs/extensions.md
- MCP config: https://github.com/can1357/oh-my-pi/blob/HEAD/docs/mcp-config.md
- Skills: https://github.com/can1357/oh-my-pi/blob/HEAD/docs/skills.md
- GitHub: https://github.com/can1357/oh-my-pi

---

## Installation

```sh
# macOS / Linux
curl -fsSL https://omp.sh/install | sh

# Homebrew
brew install can1357/tap/omp
```

**Live-verified on st3ve (Ubuntu 24.04, 2026-09-10):** the install script fetched
`omp-linux-x64` v18.1.16 and dropped a single binary into `~/.local/bin/omp`. No
Node/Bun toolchain was needed on the host.

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.omp/agent/config.yml` | Global | Default model selector and agent settings (YAML) |
| `~/.omp/agent/models.yml` | Global | Custom provider/model declarations |
| `~/.omp/agent/agent.db` | Global | Session and state store (SQLite) |
| `~/.omp/agent/sessions/` | Global | Session transcripts |
| `~/.omp/logs/` | Global | Per-process logs plus a `*-audit.json` audit sidecar |
| `~/.omp/natives/<version>/` | Global | Versioned native core |
| `.omp/hooks/pre/*.ts` | Project | Hook/extension factories (see Hooks) |

Config format is YAML. `--config <file>` layers an extra `config.yml`-shaped
overlay for a single run (repeatable), and `--profile <name>` gives a run an
isolated auth/session/settings/cache tree.

**Directory layout confirmed live** — after a first non-interactive run, `~/.omp`
contained `agent/` (with `models.db`, `agent.db`, `sessions/`), `logs/`,
`natives/18.1.16/`, `run/daemons/`, and `gpu_cache.json`.

## Instruction File

omp reads what is already on disk rather than requiring a migration. On first run
it inherits rules, skills, and MCP servers from `.claude`, `.cursor`,
`.windsurf`, `.gemini`, `.codex`, `.cline`, `.github/copilot`, and `.vscode`,
and it parses eight instruction formats natively (Cursor MDC, Cline
`.clinerules`, Codex `AGENTS.md`, Copilot `applyTo`, and the rest). Workspace
walking discovers `AGENTS.md` in the same pass as gitignore handling.

## Hooks

omp's hook subsystem has converged onto the **extension runner**: `--hook` is an
alias for `--extension`, and JS/TS hook factories discovered through
`hookCapability` (for example `.omp/hooks/pre/*.ts`) are loaded as extension
modules so their `pi.on(...)` handlers bind to the runtime event bus. Tools are
wrapped by `ExtensionToolWrapper` rather than the legacy `HookToolWrapper`.

Hooks are **in-process TypeScript modules, not shell commands** — this is the
main structural difference from the Claude Code / Codex hook families, and it
means there is no stdin-JSON / exit-code contract to normalize against.

### Hook Module Shape

A hook module default-exports a factory receiving a `HookAPI`:

```ts
import type { HookAPI } from "@oh-my-pi/pi-coding-agent/extensibility/hooks";

export default function hook(pi: HookAPI): void {
  pi.on("tool_call", async (event, ctx) => {
    if (
      event.toolName === "bash" &&
      String(event.input.command ?? "").includes("rm -rf")
    ) {
      return { block: true, reason: "blocked by policy" };
    }
  });
}
```

### Can Block?

✅ — a `tool_call` handler returning `{ block: true, reason }` stops the call and
surfaces the reason. Blocking is expressed as a **returned object**, not an exit
code.

The factory can also `pi.sendMessage(...)` (persistent custom messages),
`pi.appendEntry(...)` (non-LLM state), `pi.registerCommand(...)` (slash
commands), `pi.registerMessageRenderer(...)`, `pi.exec(...)`, and log through
`pi.logger`. Schema builders are injected (`pi.zod`, native `pi.arktype`, legacy
`pi.typebox`).

### Discovery Order

`discoverExtensionPaths(configuredPaths, cwd)`:

1. Native extension modules from the capability registry
2. Importable `.ts`/`.js` hook factories from the hook capability registry
3. Plugin extension entry points
4. Explicitly configured paths

❓ The docs describe the loader and the factory contract but do not publish a
closed list of event names; `tool_call` is the documented and verified one.

## Built-in Tools

The README advertises **31 built-in tools, 14 LSP ops, and 28 DAP ops**. Names
below marked ✅ were observed in a live `--mode json` run on st3ve; the rest are
from the documented tool listing.

| Tool | Description | Verified |
|------|-------------|----------|
| `bash` | Shell execution (PTY-based interactive by default; `--no-pty` disables) | ✅ |
| `read` | Read files — and any of 16 internal schemes (`pr://`, `issue://`, `agent://`, `skill://`, `ssh://`, …) | ✅ |
| `write` | Create/overwrite a file | ✅ |
| `grep` | Content search; also walks a diff like a directory | ✅ |
| `todo` | Plan/todo list management | ✅ |
| `task` | Fan out subagents in parallel, optionally workspace-isolated | doc |
| `web_search` | One query across configured providers, returns answer plus citations | doc |
| `learn` | Capture a reusable lesson; optionally promote it into a managed skill | doc |
| `manage_skill` | Create, update, or delete an isolated managed skill | doc |
| `orchestrate` | Run independent work through parallel subagents, verifying each phase | doc |
| `workflowz` | Build a deterministic multi-subagent workflow over the `task` tool | doc |

`--tools=<list>` allowlists specific tools, `--no-tools` disables all built-ins,
and `--no-lsp` drops the LSP tools, formatting, and diagnostics.

## MCP Support

MCP servers are inherited from other agents' configs on first run (see
Instruction File). Dedicated docs cover config, protocol transports, runtime
lifecycle, and server/tool authoring. The `exa` web-search provider falls back to
a public MCP endpoint when no `EXA_API_KEY` is set.

## Tool Substitution

- **Native tool disablement**: ✅ `--no-tools` turns off every built-in and
  `--tools=<list>` restricts to an explicit allowlist, so MCP-only operation is
  reachable rather than strictly additive.
- **Server trust**: ❓ not live-verified. Inheriting `.claude`/`.cursor`/etc. MCP
  declarations on first run means a project-shared config from another agent can
  introduce servers — worth auditing before trusting a fresh clone.
- **MCP tool naming + permissioning**: ❓ not verified.
- **Headless behaviour**: `-p/--print` completed a write+bash task
  non-interactively with no approval prompt on st3ve.

## Skills / Commands

- Skills: documented at `docs/skills.md`, plus a `docs/skills/` directory.
  Managed skills are created and edited by the agent itself through
  `manage_skill`, and `learn` can promote a lesson into one.
- Slash commands can be registered from hook/extension modules via
  `pi.registerCommand(...)`.

## Agent / Subagent Configuration

First-class subagents. `task` fans out parallel children, optionally
workspace-isolated. `Alt+A` opens the **Agent Hub**: a roster with live activity
and per-subagent usage, live transcripts, steering messages, and revive/kill
controls that do not abort the parent session. `/review` spawns dedicated
reviewer subagents that rank issues P0–P3 with confidence scores.

Nine model roles route work by intent: `default`, `smol` (cheap subagent
fan-out), `slow` (deep reasoning), `plan`, `commit`, `vision`, `task`,
`advisor`, `tiny`. Override at launch with `--smol` / `--slow` / `--plan`, cycle
the active role's models with `Ctrl+P`, or swap mid-session with `/model`.

## Notes

- **Provider breadth is the headline feature**: 60+ providers, ~1000 models.
  Custom providers speaking `openai-completions`, `openai-responses`,
  `openai-codex-responses`, `azure-openai-responses`, `anthropic-messages`,
  `bedrock-converse-stream`, `google-generative-ai`, `google-gemini-cli`, or
  `google-vertex` can be declared in `~/.omp/agent/models.yml`.
- **`--prewalk` / `--plan-yolo`** are cost-control routing flags: swap to a
  cheap model at the first edit after a plan's todo list exists, or run plan mode
  read-only and auto-approve into a cheaper implementation model.
- **`--advisor`** runs a passive reviewer over each turn and injects notes.
- Sessions can be imported from Claude Code (`--from-claude`) and Codex
  (`--from-codex`).
- Lineage: a fork of [Pi](https://github.com/badlogic/pi-mono) by Mario Zechner,
  tracked separately in this repo at [`tools/pi-agent/`](../pi-agent/).

## Live Verification (st3ve, 2026-09-10)

Ubuntu 24.04.4, omp 18.1.16, OpenAI `gpt-5.4-mini` via `OPENAI_API_KEY`.

```sh
omp -p --model openai/gpt-5.4-mini --mode json \
  "Create a file hello.py that prints the sum of 2+2, then run it with python3 and tell me the exact output."
```

- ✅ Zero configuration beyond `OPENAI_API_KEY` in the environment — no setup
  wizard, no config file, no login.
- ✅ Wrote `hello.py`, ran it, reported `4`.
- ✅ A five-part task (list dir, search TODO, read a file, write a summary, run a
  shell command) completed end to end.
- ✅ `--mode json` emits a full structured event stream: `message_update` /
  `toolcall_delta` / `tool_execution_start` / `tool_execution_end` / `turn_end`,
  with per-turn `usage` including cache reads and a computed `cost` object.
- Tool names observed: `todo`, `read`, `grep`, `bash`, `write`.
- API surface reported as `openai-responses`.

## Sources (Official)

| Topic | URL |
|-------|-----|
| Hooks | https://github.com/can1357/oh-my-pi/blob/HEAD/docs/hooks.md |
| Extensions | https://github.com/can1357/oh-my-pi/blob/HEAD/docs/extensions.md |
| MCP config | https://github.com/can1357/oh-my-pi/blob/HEAD/docs/mcp-config.md |
| Skills | https://github.com/can1357/oh-my-pi/blob/HEAD/docs/skills.md |
| LSP config | https://github.com/can1357/oh-my-pi/blob/HEAD/docs/lsp-config.md |
| GitHub repo | https://github.com/can1357/oh-my-pi |
