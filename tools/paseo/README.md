# Paseo

> Self-hosted orchestrator that runs *other* coding agents (Claude Code, Codex, Copilot, OpenCode, Pi, + a large ACP catalog) in parallel from desktop, mobile, web, and CLI.

**Vendor:** Paseo (`getpaseo`, Mo Boudra) | **License:** Apache-2.0 | **Runtime:** Node.js/TypeScript (`@getpaseo/cli`), Electron desktop app, Expo mobile app, Docker image

> **Not a coding agent.** Paseo ships no model of its own and no built-in `Read`/`Write`/`Bash` tools. It is a **daemon + client** harness that launches, supervises, and multiplexes the agent CLIs you already have installed. It appears in this almanac because it has its own config file, lifecycle hook system, MCP server, and skills — the same surfaces the agent pages document. Version documented: **v0.8.0** (2026-09-10).

## Links

- Website: https://paseo.sh
- Docs: https://paseo.sh/docs
- GitHub: https://github.com/getpaseo/paseo
- Releases / Changelog: https://github.com/getpaseo/paseo/releases
- Plugin reference: https://paseo.sh/docs/plugins
- MCP tool reference: https://paseo.sh/docs/mcp
- SDK: https://paseo.sh/docs/sdk/quickstart (`@getpaseo/client`)

---

## Installation

```sh
# CLI / headless (also installs the daemon)
npm install -g @getpaseo/cli
paseo
```

```sh
# Desktop app (daemon starts automatically) — https://paseo.sh/download
```

```sh
# Docker (daemon + self-hosted web UI on :6767)
docker run -d --name paseo \
  -p 6767:6767 \
  -e PASEO_PASSWORD=change-me \
  -v "$PWD/paseo-home:/home/paseo" \
  -v "$PWD:/workspace" \
  ghcr.io/getpaseo/paseo:latest
```

The base Docker image contains no agent CLIs — extend it with the ones you use and supply their credentials via env vars or the persistent `/home/paseo` volume.

---

## Architecture

A local **daemon** owns agent processes, workspaces/worktrees, terminals, schedules, and the MCP server. **Clients** (desktop, iOS/Android, web, CLI, VS Code extension, `@getpaseo/client` SDK) connect over a WebSocket at `ws://127.0.0.1:6767/ws`, directly over TCP/Tailscale, or through an end-to-end-encrypted relay for device pairing. Relay is **off** for new installations.

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.paseo/config.json` | Global (daemon) | Single config file: `daemon.listen`, `daemon.hostnames`, `daemon.mcp`, `agents.providers`, `worktrees.root`, `features.*`, logging, voice |
| `$PASEO_HOME/worktrees/` | Global | Default root for Paseo-managed git worktrees |
| `$PASEO_HOME/daemon.log` | Global | Daemon log (`trace` and above by default) |

Home directory is `~/.paseo`, overridable with `PASEO_HOME` or `paseo daemon start --home`. Config precedence: defaults → `config.json` → environment variables → CLI flags; list fields (`hostnames`, `cors.allowedOrigins`) **append** across sources rather than replacing.

Schema: `https://paseo.sh/schemas/paseo.config.v1.json`

```json
{
  "$schema": "https://paseo.sh/schemas/paseo.config.v1.json",
  "version": 1,
  "daemon": {
    "listen": "127.0.0.1:6767",
    "hostnames": ["localhost", ".localhost"],
    "mcp": { "enabled": true, "injectIntoAgents": true }
  }
}
```

`paseo reload` validates the whole file and applies runtime-safe changes (relay toggle, MCP settings, browser tools, hostnames, CORS, provider/agent profiles, plugin switch), then reports paths that still need `paseo daemon restart` (listen addresses, auth, TLS, worktree allocation, web UI, logging, voice, credentials).

### Common environment variables

| Variable | Purpose |
|----------|---------|
| `PASEO_HOME` | Paseo home directory |
| `PASEO_HOST` | Daemon target for CLI commands (equivalent to `--host`) |
| `PASEO_PASSWORD` | Daemon password to require / CLI password to connect |
| `PASEO_LISTEN` | Override `daemon.listen` |
| `PASEO_RELAY_ENABLED` | Enable/disable the outbound relay for one daemon launch |
| `PASEO_HOSTNAMES` | Override/extend `daemon.hostnames` (`PASEO_ALLOWED_HOSTS` is a deprecated alias) |
| `PASEO_WEB_UI_ENABLED` | Serve the browser web client from the daemon |
| `PASEO_TRUSTED_PROXIES` | Trusted reverse-proxy ranges for `X-Forwarded-*` |
| `PASEO_LOG_*` | Console/file log level, path, rotation |

Paseo also sets `PASEO_AGENT_ID` and `PASEO_AGENT_CWD` in every launched agent's environment — that is how a nested `paseo run` knows to attach the new agent as a subagent of the caller.

## Instruction File

None of its own. Each launched agent reads **its own** instruction file (`CLAUDE.md`, `AGENTS.md`, …) from the workspace directory. Paseo can inject a `systemPrompt` per agent through `agents.providers` config, agent profiles, or the `agent.create` hook.

## Hooks

Paseo's hook system lives in **TypeScript plugins**, not in a config-file matcher table. A plugin's `index.server.ts` exports `contribute(server)` and registers callbacks with `server.on()` (events, observe-only) and `server.before()` (gates, can rewrite the request). Hooks run **on the daemon** whenever the plugin is enabled — no client needs to be connected.

Crucially, these are **agent-lifecycle** hooks, not per-tool-call hooks: Paseo sees turns, permission requests, and agent/workspace creation, but individual `Bash`/`Read` tool calls stay inside the underlying agent CLI and must be intercepted with *that* tool's own hook system.

### Supported Events — `server.on(name, cb)` (8)

| Event | Payload | Trigger | Can Block? |
|-------|---------|---------|-----------|
| `agent.created` | `agent` | Ordinary creation finishes (excludes import/resume) | ❌ |
| `agent.turn_started` | `agent`, `turnId` | Live turn starts | ❌ |
| `agent.turn_ended` | `agent`, `turnId`, `outcome`, `timeline` | Turn completes, fails, or is canceled | ❌ |
| `agent.permission_requested` | `agent`, `request` | Permission or question becomes pending | ❌ (but can be *answered* via SDK) |
| `agent.permission_resolved` | `agent`, `requestId`, `resolution` | Pending request answered or cleared | ❌ |
| `agent.archived` | `agent`, `archivedAt` | Archive state saved | ❌ |
| `workspace.created` | `workspace` | Record created, directory available | ❌ |
| `workspace.archived` | `workspace` | Archive state saved | ❌ |

`outcome` is `{ kind: "completed" }`, `{ kind: "failed", error }`, or `{ kind: "canceled", reason }`. Agent events exclude internal utility agents. Delivery is live and best-effort — no replay, persistence, or retry, and concurrent events may overlap.

### Before Hooks — `server.before(name, cb)` (3)

| Name | Request fields | Editable | Can Block? |
|------|----------------|----------|-----------|
| `agent.create` | `config`, optional `env` | Public agent config except `cwd`/`internal`; `env` | ✅ (throw → operation fails) |
| `agent.session_open` | `agentId`, `workspaceId`, `provider`, `cwd`, `reason`, `purpose`, `env` | `env` only | ✅ (throw) |
| `workspace.create` | `source`, optional `title`, `firstAgentContext` | Entire request | ✅ (throw) |

Eleven hooks total. Return a modified request to change it, `undefined` to leave it alone. There is **no deep merge** — later callbacks overwrite earlier values.

### Ordering

```text
Creation request
  → agent.create hooks (plugin-ID order; registration order within a plugin)
  → resolve defaults and validate provider configuration
  → derive launch configuration with Paseo runtime tools and daemon prompt
  → agent.session_open hooks (same ordering; env only)
  → set PASEO_AGENT_ID and PASEO_AGENT_CWD
  → open provider session and save agent configuration
```

### Failure Behavior

| Condition | Result |
|-----------|--------|
| Before hook throws or returns invalid data | Operation fails; later callbacks do not run |
| Event handler throws | Logged against the plugin; original operation continues |
| Hook timeout | 30 s; `context.signal` aborts. Before hook → pending operation fails; event handler → error logged |
| Reload / disable / removal / shutdown | Remaining registrations removed, subprocess stopped, pending RPCs rejected |
| Unknown hook name | Registration fails |

### Example: inject an MCP server and force a Codex sandbox mode

```ts
import type { PluginServerContext } from "@getpaseo/plugin/server";

export default function contribute(server: PluginServerContext) {
  server.before("agent.create", ({ request }) => {
    if (request.config.provider !== "codex") return request;
    return {
      ...request,
      config: {
        ...request.config,
        providerOptions: {
          ...request.config.providerOptions,
          sandbox_mode: "workspace-write",
          approval_policy: "on-request",
        },
        mcpServers: {
          ...request.config.mcpServers,
          company: { type: "http", url: "https://tools.example.com/mcp" },
        },
      },
    };
  });

  return () => {};
}
```

### Example: auto-answer a permission request

```ts
server.on("agent.permission_requested", async (event, { paseo }) => {
  const command = shellCommand(event.request);          // helper from plugin-examples/lifecycle-actions
  const agent = await paseo.agents.get(event.agent.id);
  if (/^rm\s+-rf/.test(command ?? "")) {
    await agent.respondToPermission({ requestId: event.request.id, response: { behavior: "deny" } });
  } else if (command?.trim() === "git status") {
    await agent.respondToPermission({ requestId: event.request.id, response: { behavior: "allow" } });
  }
  // anything else stays pending for the human
});
```

Reference plugins: [`lifecycle-logger`](https://github.com/getpaseo/paseo/tree/main/plugin-examples/lifecycle-logger) (all eleven hooks, redacted JSON logs), [`lifecycle-actions`](https://github.com/getpaseo/paseo/tree/main/plugin-examples/lifecycle-actions), [`agent-configuration`](https://github.com/getpaseo/paseo/tree/main/plugin-examples/agent-configuration). Read output with `paseo plugin logs <id>` or the host's `daemon.log`.

### Plugin management

```sh
paseo plugin init /abs/path        # scaffold
paseo plugin install /abs/path     # local directory
paseo plugin add owner/repo        # git; :subdir for monorepos, --ref for a branch
paseo plugin ls | update | reload | logs | disable | enable | remove
```

> ⚠️ Plugin server code and git preparation commands run **unsandboxed as the daemon user** on the daemon host; client contributions run inside the Paseo app. Installing a plugin is a trust decision covering its dependencies and future updates.

## Built-in Tools

Paseo has no file/shell tool surface of its own — those belong to the launched agent. What it exposes instead is an **orchestration** tool catalog, served over MCP (see below) and, for some providers, through their native tool interface.

## MCP Support

Two distinct roles:

**1. Paseo as an MCP *server*.** The daemon runs an MCP server (`daemon.mcp.enabled`, default `true`) exposing the orchestration catalog. Set `daemon.mcp.injectIntoAgents: true` (default `false`) to hand that catalog to every agent Paseo launches — this is what lets an agent spawn and drive other agents.

| Group | Tools |
|-------|-------|
| Agents | `create_agent`, `send_agent_prompt`, `get_agent_status`, `list_agents`, `cancel_agent`, `archive_agent`, `kill_agent`, `update_agent`, `get_agent_activity`, `set_agent_mode` |
| Workspaces | `create_workspace`, `list_workspaces`, `rename_workspace`, `archive_workspace` |
| Workspace scripts | `list_workspace_scripts`, `start_workspace_script`, `stop_workspace_script` |
| Terminals | `list_terminals`, `create_terminal`, `kill_terminal`, `capture_terminal`, `send_terminal_keys` |
| Schedules | `create_schedule`, `list_schedules`, `inspect_schedule`, `pause_schedule`, `resume_schedule`, `update_schedule`, `schedule_logs`, `run_schedule_once`, `delete_schedule`, `create_heartbeat`, `delete_heartbeat` |
| Profiles / providers | `list_profiles`, `list_providers`, `list_models`, `inspect_provider` |
| Permissions | `list_pending_permissions`, `respond_to_permission` |
| Voice | `speak` (voice-enabled sessions only) |

Per-provider narrowing via `agents.providers.<id>.paseoTools`:

```json
{
  "agents": {
    "providers": {
      "codex-worker": {
        "extends": "codex",
        "paseoTools": { "disabledTools": ["create_agent", "send_agent_prompt", "kill_agent"] }
      },
      "codex-isolated": { "extends": "codex", "paseoTools": { "enabled": false } }
    }
  }
}
```

Custom profiles do **not** inherit `paseoTools` from `extends` — configure each ID. A running session keeps the catalog it received at launch, so start or reload an agent after changing injection.

**2. Paseo as an MCP *config injector*.** `agent.create.config.mcpServers` (set in config, agent profiles, or a `before` hook) writes MCP servers into the launched agent's own configuration, so a plugin can add an MCP server to every Codex or Claude Code session on the host.

## Tool Substitution

Paseo has no native tool surface to substitute — substitution semantics are whatever the launched agent enforces (see `_shared/mcp-tool-substitution.md` for the per-agent behavior). What Paseo controls is the *layer above*:

- **Server trust**: MCP servers come from Paseo's own config or a plugin's `agent.create` hook, both of which are daemon-side and already trusted; there is no per-project auto-load path in Paseo itself.
- **Native tool disablement**: ❌ for the agent's own built-ins. ✅ for Paseo's injected catalog (`paseoTools.enabled: false` or `disabledTools`), and `toolPolicy` in `AgentSessionConfig` carries exact-tool preapprovals into the agent.
- **Headless behaviour**: a pending permission request does not fail — it sits until answered. `paseo permissions` / `list_pending_permissions` / `respond_to_permission` or an `agent.permission_requested` hook can answer it programmatically, which is the practical way to run non-interactive.

## Skills / Commands

Orchestration skills, installed onto the host where agents run (`npx skills add getpaseo/paseo`, or **Settings → host → Agents → Orchestration skills**). They are ordinary agent skills that teach the *agent* to use Paseo's tools:

| Skill | Purpose |
|-------|---------|
| `/paseo` | Reference for managing agents, workspaces, schedules, heartbeats |
| `/paseo-handoff` | Transfer a task + briefing to another agent (e.g. plan with Claude, implement with Codex) |
| `/paseo-committee` | Two contrasting agents analyze independently; main agent synthesizes |
| `/paseo-advisor` | Single second-opinion agent that does not edit files |

Source: [`skills/`](https://github.com/getpaseo/paseo/tree/main/skills) in the repo. Selected skills refresh on host startup.

## Agent / Subagent Configuration

Providers with native support (CLI must be installed and authenticated): **Claude Code, Codex, GitHub Copilot, OpenCode, Pi**. Anything speaking **ACP** can be added from the in-app catalog — 35+ entries including Amp, Auggie, Cline, Cursor, Factory Droid, Gemini CLI, goose, Grok, Hermes, Junie, Kilo Code, Kimi Code, Mistral Vibe, Qwen Code, TRAE CLI. Custom providers go under `agents.providers` (`extends`, `label`, custom binaries, `additionalModels`, Anthropic-compatible endpoints).

Subagents are implicit: when an existing Paseo agent runs `paseo run`, `PASEO_AGENT_ID` makes the new agent its child in the same workspace unless `--workspace` places it elsewhere.

```sh
paseo run --provider claude/opus-4.6 "implement user authentication"
paseo run --provider codex/gpt-5.5 --new-workspace worktree --worktree-mode branch-off --new-branch feature/x "implement feature X"
paseo run --background "run the focused test suite"
paseo run --output-schema schema.json "extract release notes"   # JSON-only output; not with --background
paseo ls | attach <id> | send <id> "..." | logs <id> | stop <id>
paseo --host workstation.local:6767 run --cwd /workspace "run the full test suite"
```

Cron **schedules** start a fresh agent per run; **heartbeats** send a recurring prompt into an existing agent.

## SDK

`@getpaseo/client` (TypeScript) drives the same daemon API used by the apps:

```ts
import { createPaseoClient } from "@getpaseo/client";

const client = createPaseoClient({ url: "ws://127.0.0.1:6767/ws" });
await client.connect();
const agent = await client.agents.create({
  config: { provider: "codex/gpt-5.5" },
  cwd: "/Users/me/dev/storefront",
  prompt: "Review the current diff and name the riskiest change.",
});
console.log((await agent.waitForFinish()).lastMessage);
await client.close();
```

Plugins get a pre-connected instance of the same SDK as `context.paseo`.

## Security Notes

- Daemon listens on `127.0.0.1:6767` by default; `daemon.hostnames` provides DNS-rebinding protection and `PASEO_PASSWORD` adds password auth (hashed at startup).
- Relay pairing is end-to-end encrypted and **disabled for new installations**; in non-interactive/JSON mode a disabled relay returns `RELAY_DISABLED` rather than silently enabling.
- No telemetry, tracking, or forced login.
- Plugins are unsandboxed daemon-side code — see the warning above.

## Notes

- Doc-only pass (2026-09-10) — sourced from official docs and the repo at `v0.8.0`; not sandbox live-verified like `tools/claude-code/` or `tools/openclaw/`.
- Plugin API is versioned and still moving: `/docs/plugins/v0.7` and `/docs/plugins/v0.8` are separate references with a migration guide. Hook names/payloads above are v0.8.
- Related: [`getpaseo/paseo-relay`](https://github.com/getpaseo/paseo-relay) (Elixir relay), [paseo-vscode](https://marketplace.visualstudio.com/items?itemName=hinnes.paseo-vscode) (third-party VS Code extension).

## Sources (Official)

| Topic | URL | Label |
|-------|-----|-------|
| Docs home | https://paseo.sh/docs | [official] |
| Configuration | https://paseo.sh/docs/configuration | [official] |
| Plugin reference (hooks) | https://paseo.sh/docs/plugins/v0.8 | [official] |
| MCP tool reference | https://paseo.sh/docs/mcp | [official] |
| Orchestration skills | https://paseo.sh/docs/skills | [official] |
| Supported providers | https://paseo.sh/docs/supported-providers | [official] |
| CLI reference | https://paseo.sh/docs/cli | [official] |
| SDK | https://paseo.sh/docs/sdk/quickstart | [official] |
| Security | https://paseo.sh/docs/security | [official] |
| GitHub repo | https://github.com/getpaseo/paseo | [github] |
| Docker | https://github.com/getpaseo/paseo/blob/main/docs/docker.md | [github] |
