# Muse Code

> Meta Superintelligence Labs' terminal coding agent, co-trained with the Muse Spark 1.2 model for long-horizon, multi-step software engineering across large repositories.

**Vendor:** Meta (Meta Superintelligence Labs) | **License:** Proprietary (closed-source native binary; no public source repo found) | **Runtime:** Native binary (macOS, Linux) — terminal CLI, no Node.js runtime required

## Links

- Product page: https://developer.meta.com/ai/products/muse-code/
- Docs (landing): https://dev.meta.ai/docs/muse-code
- Announcement blog: https://research.meta.ai/blog/introducing-muse-code-and-muse-spark-1-2
- Dev blog: https://developer.meta.com/ai/resources/blog/build-with-muse-code/
- GitHub: ❓ No official Muse Code source/binary repo found. `github.com/meta-models` hosts only cookbook/recipe repos (`meta-model-cookbook`, `meta-oss-cookbook`) for the Meta Model API, not Muse Code itself.
- Changelog: ❓ Not found on official docs (product is in beta as of Aug 2026; no changelog page located).

---

## Installation

**macOS / Linux (beta only — no Windows support documented):**
```sh
curl -fsSL https://dev.meta.ai/install.sh | bash
```

Verify install:
```sh
muse --version
```

Sign in via browser, or set `META_API_KEY` for non-interactive/CI/headless environments.

Scaffold a project instruction file:
```sh
muse init
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.config/muse/settings.json` | Global (user) | Model defaults, terminal UI preferences, tool/MCP config, `hooks` block, `managed_hooks_path` pointer, `runtime_capabilities` map, telemetry options. Requires a `"schema_version": 1` field — missing it fails startup. |
| `AGENTS.md` | Project (preferred) | Natural-language project instructions |
| `CLAUDE.md` | Project (fallback) | Confirmed: Muse Code "prefers `AGENTS.md` over `CLAUDE.md` when both exist" |
| `.agents/memory/MEMORY.md` + topic files | Project (committed) | Persistent project memory system — index file plus individual Markdown topic files. Confirmed: three memory scopes exist — personal project memory (default, machine-local), project memory (`.agents/memory/`, repo-committed), and personal memory (machine-wide, cross-project) |
| `.muse/worktrees/` | Project (runtime) | Isolated git worktrees created for parallel subagent fan-out |
| `.agents/plans/` | Project | Output location for `/plan`-generated, approval-gated plans |

Instruction-file loading walks from the workspace root to the nearest `.git` boundary; deeper files override shallower ones, and project rules override user rules. Project rules load only **after** the workspace is explicitly trusted (a first-run trust prompt, similar in spirit to other terminal agents' workspace-trust gates).

## Instruction File

The agent reads natural-language instructions from `AGENTS.md` (preferred), falling back to `CLAUDE.md` if present — now confirmed directly ("prefers `AGENTS.md` over `CLAUDE.md` when both exist"). `muse init` creates the file in the current directory. ❓ Exact supported frontmatter/import syntax (e.g., `@file` includes) is still not documented on the reachable pages.

## Hooks

Muse Code documents a first-class hook system under "Extending and automating" in its docs nav (confirmed live at `https://dev.meta.ai/docs/muse-code/extending`), described as lifecycle hooks that **bind a shell command to a lifecycle event**. Hooks, MCP, skills, sandboxing, and worktree parallelism are described as shipping complete at the initial beta release (day one), rather than being added later.

### Supported Events

| Event | When | Can Block? |
|-------|------|-----------|
| `SessionStart` | Session begins | ❓ Not stated explicitly |
| `UserPromptSubmit` | User submits a prompt | ❓ Not stated explicitly |
| `PreToolUse` | Before a tool call runs | Likely — pre-action event |
| `PermissionRequest` | An approval/permission check fires | Likely — pre-action event |
| `PostToolUse` | After a tool call completes | ❓ Not stated explicitly |
| `PreLLMCall` | Before a model call | Likely — pre-action event |
| `PostLLMCall` | After a model call | ❓ Not stated explicitly |
| `PreCompact` | Before context compaction | Likely — pre-action event |
| `PostCompact` | After context compaction | ❓ Not stated explicitly |
| `SubagentStart` | A subagent starts | ❓ Not stated explicitly |
| `SubagentStop` | A subagent stops | ❓ Not stated explicitly |
| `Stop` | Session/turn stops | ❓ Not stated explicitly |

12 named events total. Docs describe hooks as able to "enforce a check, format code, or block an action before it happens" — implying the `Pre*`/`PermissionRequest` events support blocking, but the page does not give an explicit per-event table of which ones can block or what exit code signals a block.

### Hook Input JSON

Each hook fixture has the shape:
```json
{ "event": "PreToolUse", "stdin": {} }
```
i.e. an `"event"` field matching the hook's event name, plus a `"stdin"` object carrying the event-specific payload. The exact payload schema per event was not given on the page.

### Exit Code Behavior

Documented for `muse exec` (not necessarily identical for hook scripts, but the closest confirmed exit-code semantics):
- `0` — turn completes (note: "An agent can finish its turn and still report that the tests fail, and it exits `0`")
- `1` — fails or is cancelled (including hitting `--max-model-steps`)
- `2` — usage error
- `130` / `143` — on `SIGINT` / `SIGTERM`

### Example Config

Hooks are sourced from three places: committed at `<project-root>/.muse/hooks.json`, machine-wide hooks defined in `~/.config/muse/settings.json` (the `hooks` block), or a centrally managed file pointed to by `managed_hooks_path`. No full example `hooks.json`/settings `hooks` block was given verbatim on the page — only the binding description above.

## Built-in Tools

Official pages describe capabilities rather than an enumerated tool table. Confirmed/implied tools:

| Tool | Description |
|------|-------------|
| File edit/write | Implied by "plans, edits, and runs commands" — exact tool name(s) ❓ |
| Shell/command execution | Runs commands under an OS-enforced sandbox (Seatbelt on macOS, bubblewrap on Linux) restricting filesystem writes to the workspace/temp dirs; three approval modes confirmed — `on-request` (default, only flagged dangerous commands like `rm -rf`/`sudo` need review), `untrusted` (any unrecognized shell stage needs approval), and `never` (sandbox-only, no prompts). `--yolo` removes both layers; `--disable-approval`/`--disable-sandbox` disable them individually |
| Search grounding | "Give your model live access to the web" (listed among Muse Code/Model API capabilities) |
| Computer use | Listed as a capability ("enabling agent control of software building") — ❓ scope/limits not detailed |
| Subagent fan-out | Spawns parallel subagents into isolated git worktrees under `.muse/worktrees/` |

❓ No official page enumerates a discrete tool list (e.g., equivalent of Claude Code's Read/Write/Edit/Grep/Glob) the way other agents' docs do — treat the above as inferred from prose, not a verified API surface.

## MCP Support

Muse Code's docs nav lists MCP among "Extending and automating" features alongside hooks, skills, and headless runs, and the product page states it ships with MCP support "complete on day one." Confirmed from the live `extending` sub-page:

- **Config location/key:** Declare servers in the `mcp_servers` block of `~/.config/muse/settings.json`:
  ```json
  { "mcp_servers": {
      "my-tools": { "transport": "stdio", "command": "my-mcp-server", "args": [] }
  } }
  ```
- **Transport types supported:** `"stdio"` (with `command`, `args`, `env`) and `"streamable_http"` (with `url`, `headers`) — no `sse` transport documented.
- **Other fields:** `enabled`, `mode` (defaults to `"required"`; set `"optional"` for a skippable server), and an optional `framing` field.
- **Project vs. user scope:** The docs do not distinguish a separate project-level MCP config from the user-level `settings.json` — as documented, MCP servers appear to be a single (user-level) settings block, not a repo-committed file. ❓ Worth re-checking if a project-level override surfaces later.

Separately, Meta's **Model API** (the underlying model-serving product, distinct from the Muse Code harness) is documented as drop-in compatible with the OpenAI SDK, Anthropic SDK, and other agent CLIs (OpenCode, Claude Code) — this is an API-compatibility feature, not Muse Code's own MCP client config.

## Skills / Commands

Confirmed from the live `extending` sub-page. Skills load from three sources:

- **Built-in:** skills that ship with Muse Code.
- **User:** `$XDG_CONFIG_HOME/muse/skills` (and `~/.agents/skills`).
- **Project:** `<repo>/.agents/skills/<skill-id>/SKILL.md`; Muse Code also scans repo-local `.codex/skills` and `.claude/skills` for interop with other tools' skill directories.

Format: each skill is "a set of instructions, and optionally tools and files," with `SKILL.md` as the core file (same filename convention as other agent CLIs). No further frontmatter/schema detail was given.

Built-in slash commands confirmed from official sources:

| Command | Purpose |
|---------|---------|
| `/plan` | Turns a task into an approval-gated plan, grounded in actual files, saved to `.agents/plans/` |
| `/grill` (also documented as `/grilling`) | Stress-tests a plan / decision-forcing interview format until it holds up |
| `/grill-with-docs` | Same interview format, additionally documents decisions as durable records |
| `/goal` | Works toward successful completion of a specified objective |
| `/taste` | "Anti-slop" filter applying visual defaults for UI generation |
| `/effort` | Dials reasoning depth up or down |
| `/login` | Interactive re-authentication |

Headless/CI usage: `muse exec "<prompt>"`.

## Agent / Subagent Configuration

Muse Code is described as **"multi-agent by default"** — "workers in parallel, reviewers in the background." Two distinct subagent mechanisms are documented across official/press-adjacent sources:

1. **Persistent background agents** — specialized agents that stay active for the whole session (rather than being spawned fresh per task), accumulating context over time. ❓ The exact config surface for defining/naming/customizing these persistent background agents (e.g., a dedicated agent-definition file, similar to other tools' subagent `.md` files) was not found in official docs reachable during this pass.
2. **Parallel subagent fan-out with git worktree isolation** — for large jobs, Muse Code fans out write-capable subagents into isolated git worktrees (under `.muse/worktrees/`, in detached-HEAD state), so the developer's primary working copy is never touched mid-run. Invoked via the `--subagent-worktree-isolation` flag (confirmed exact form: `muse --subagent-worktree-isolation`). Related commands: `subagent_status`, `subagent_wait`, `muse resume`. Newly confirmed details: the runtime owns one git worktree per child; default parallelism cap scales with the machine ("roughly core count minus two, clamped between 2 and 16"); children run **one level deep only** — a child cannot spawn its own children; a cancelled child that never reaches a checkpoint keeps running in the background.

An append-only local event log records every model call, tool run, approval, and edit, described by Meta as "replay-exact and restart-safe" — enabling crash recovery and resumption of long-running (including background) tasks from the exact point of interruption, without lost work or re-prompting.

## Notes

- This is a **doc-only research pass** — Muse Code was not installed or live-tested; nothing here reflects hands-on verification.
- Muse Code entered public beta August 5, 2026 (macOS/Linux only); Windows support is not mentioned in any official source found.
- Underlying model: `muse-spark-1.2` (default). A rate-limited `muse-spark-1.2-contributor` tier is also referenced, available in select countries.
- Pricing (Model API, which Muse Code runs on), standard tier: $0.15 / 1M cached input tokens, $1.25 / 1M input tokens, $4.25 / 1M output tokens; zero-data-retention available on request via Meta sales. Contributor tier (data used "to improve our products"), re-confirmed on the product page: $0.002 / 1M cached input, $0.10 / 1M input, $0.20 / 1M output — press coverage (VentureBeat) separately describes this tier as "$0.30 per million total tokens," likely a blended/rounded figure rather than the per-type breakdown above.
- Meta positions Muse Code directly against Anthropic's Claude Code and OpenAI's Codex CLI.
- No dedicated end-user license/EULA text was found separate from Meta's general Facebook policy-center ToS/Privacy links surfaced on the product page — this is unusual for a dev tool and worth re-checking once the product matures past beta.
- **Re-verified 2026-08-15:** this pass successfully reached the previously-404ing "Extending and automating" sub-page (`https://dev.meta.ai/docs/muse-code/extending.md`) and the "Configuration and context" sub-page (`https://dev.meta.ai/docs/muse-code/configuration.md`), plus the "Permissions and safety" sub-page (`https://dev.meta.ai/docs/muse-code/permissions.md`). Most hooks/MCP ❓ fields above are now confirmed; remaining ❓s are narrower (e.g. per-event blocking behavior, exact hook payload schema per event, project-vs-user MCP scoping, persistent-background-agent config surface, skill frontmatter syntax). Sub-pages require the `.md` suffix (or a search-assisted route) to fetch reliably — plain `/docs/muse-code/extending` (no `.md`) had previously 404'd on automated fetch.

## Sources (Official)

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| Product overview | https://developer.meta.com/ai/products/muse-code/ | 2026-08-15 | [official] |
| Announcement blog | https://research.meta.ai/blog/introducing-muse-code-and-muse-spark-1-2 | 2026-08-15 | [official] |
| Dev blog ("Build with Muse Code") | https://developer.meta.com/ai/resources/blog/build-with-muse-code/ | 2026-08-15 | [official] |
| Docs landing (installation, auth, nav structure) | https://dev.meta.ai/docs/muse-code | 2026-08-15 | [official] |
| Configuration and context (settings.json, AGENTS.md/CLAUDE.md, memory system) | https://dev.meta.ai/docs/muse-code/configuration.md | 2026-08-15 | [official] |
| Extending and automating (hooks, MCP, skills, subagents) — **now confirmed reachable** | https://dev.meta.ai/docs/muse-code/extending.md | 2026-08-15 | [official] |
| Permissions and safety (approval modes, sandboxing) — **now confirmed reachable** | https://dev.meta.ai/docs/muse-code/permissions.md | 2026-08-15 | [official] |
| GitHub org (cookbook repos only, not Muse Code source) | https://github.com/meta-models | 2026-08-15 | [github] |
| Meta ToS (policy center, linked from product page) | https://www.facebook.com/policies_center/ | 2026-08-15 | [official] |
| Beta launch coverage / persistent background agents detail | https://venturebeat.com/orchestration/meta-enters-the-ai-coding-wars-with-muse-spark-1-2-and-muse-code-with-persistent-async-background-agents | 2026-08-15 | [press] (403/429 bot-blocked on re-fetch; existence/content reconfirmed via search snippet) |
| Launch coverage | https://techcrunch.com/2026/08/05/meta-launches-muse-code-an-ai-agent-for-large-code-bases/ | 2026-08-15 | [press] |
| Pricing/competitive framing coverage | https://www.cnbc.com/2026/08/05/meta-debuts-muse-code-to-take-on-anthropic-and-openai-.html | 2026-08-15 | [press] (403 bot-blocked on re-fetch; existence/content reconfirmed via search snippet) |
