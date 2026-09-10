# MiMo Code

> Xiaomi's TypeScript CLI agent built around co-evolving models and agents, with mode-locked primary agents, 14+ builtin skills, and cross-agent skill directory discovery.

**Vendor:** Xiaomi (MiMo) | **License:** MIT | **Runtime:** Node.js (npm) or install script

## Links

- Docs / site: https://mimo.xiaomi.com/mimocode
- GitHub: https://github.com/XiaomiMiMo/MiMo-Code

---

## Installation

```sh
curl -fsSL https://mimo.xiaomi.com/install | bash
# or
npm install -g @mimo-ai/cli
```

**Live-verified on st3ve (Ubuntu 24.04, 2026-09-10):** `npm install -g
@mimo-ai/cli` installed v0.1.14 (5 packages) and provides the `mimo` command.
First invocation runs a one-time SQLite database migration
(`sqlite-migration:done`).

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.config/mimocode/` | Global | Config root; holds a managed `node_modules/` for plugins |
| `.mimocode/skills/<name>/SKILL.md` | Project | Project skills (override builtins by name) |
| `~/.claude/skills/`, `~/.opencode/skills/`, … | Global | Personal skill dirs discovered from *other* agents |
| `checkpoint.md` | Project | Session checkpoint, maintained by the checkpoint-writer subagent |
| `AGENTS.md` | Project | Instruction file |

**Directory layout confirmed live** — `~/.config/mimocode/` contained a
`.gitignore` and a full `node_modules/` tree including `@mimo-ai`, i.e. plugins
are installed as real npm modules into the config root.

## Instruction File

`AGENTS.md`. A `checkpoint.md` session checkpoint holds structured state
snapshots, written automatically by a dedicated checkpoint-writer subagent.

## Hooks

❌ **No declarative shell-hook system.** MiMo Code extends through npm plugins
(`mimo plugin <module>`), not a lifecycle hook table with an exit-code contract.

The builtin `evolve` skill is described as "total self-modification — rewrite any
layer of the agent: tools, behavior hooks, knowledge, workflows, even the UI",
which implies internal behavior hooks, but no external hook configuration
surface is documented.

`--pure` runs without external plugins — useful as a known-clean baseline.

## Built-in Tools

Observed live via `mimo export <session-id>` on st3ve:

| Tool | Type | Verified |
|------|------|----------|
| `exec` | shell | ✅ |
| `exec_command` | shell (command form) | ✅ |
| `apply_patch` | file write (patch/diff apply) | ✅ |

❓ No published enumerated tool table. The three names above are what a
five-part task (list dir, grep, read, write, shell) actually emitted — notably,
**MiMo routes file reads and searches through shell execution and does its
writes through `apply_patch`**, rather than exposing separate `read`/`grep`
tools the way most harnesses in this repo do.

## MCP Support

✅ Full.

```
mimo mcp add            add an MCP server
mimo mcp list           list MCP servers and their status        [aliases: ls]
mimo mcp auth [name]    authenticate with an OAuth-enabled MCP server
mimo mcp logout [name]  remove OAuth credentials for an MCP server
mimo mcp debug <name>   debug OAuth connection for an MCP server
```

OAuth-enabled MCP servers are first-class, with dedicated auth, logout, and
connection-debug subcommands — a more complete OAuth story than most entries in
this repo.

## Tool Substitution

❓ Not live-verified. Environment variables can disable builtin *skills* (see
below) but no equivalent for disabling builtin tools was found.

## Skills / Commands

Skills are reusable instruction sets. For a new task MiMoCode searches available
non-Compose skills by **exact name, localized alias, and BM25 relevance**;
high-confidence matches load automatically and uncertain ones are ranked for the
agent to assess. In the TUI, `/` browses autocomplete or invokes directly;
mentioning two or more skills in one message auto-loads them and injects a
multi-skill orchestration plan.

Builtin skills include `deep-research` (cited multi-source reports with parallel
subagents and web tools), `evolve` (self-modification), `skill-creator`,
`compose-next`, office/media skills (`docx-official`, `pdf-official`,
`pptx-official`, `xlsx-official`, `html-to-video-pipeline`), and `claude-code` /
`codex` — the last two exposed **only when the `claude` and `codex` executables
are installed**.

**Overriding a builtin:** create a skill with the same `name` in
`.mimocode/skills/<name>/SKILL.md` or a personal skill directory. User skills
discovered later in the scan order override builtins.

| Env var | Effect |
|---------|--------|
| `MIMOCODE_DISABLE_BUILTIN_SKILLS=true` | Disable all builtin skills |
| `MIMOCODE_DISABLE_OFFICIAL_SKILLS=true` | Disable only the office/media skills |
| `MIMOCODE_DISABLE_SLASH_SKILLS=true` | Hide from TUI `/` autocomplete without disabling |

The first two remove skills from the agent's list entirely — not in context, not
invocable. The third affects autocomplete only.

## Agent / Subagent Configuration

`mimo agent create` / `mimo agent list` manage agents. Three primary agents,
switched with `Tab`:

| Mode | Purpose |
|------|---------|
| **build** | Default implementation agent |
| **plan** | Planning |
| **compose** | Orchestration for specs-driven development and skill-driven workflows |

> ⚠️ **Mode locks after the first message.** Build and Plan can still switch
> between each other; **Compose is isolated once entered.** The docs are explicit
> that keeping the skill/tool set fixed from session start "significantly
> improves tool-call reliability" — a deliberate constraint worth knowing before
> scripting around it.

The primary agent creates subagents on demand. Subagents share the current
session context and run in parallel, with lifecycle tracking, cancellation, and
background execution.

For frontier models the recommended path is the **build** agent with the
`/compose-next` skill — one self-contained contract covering grill → workspace →
spec → implement → verify → review → finalize → finish, with feature docs at
`docs/compose/spec/<feature>.md`. The legacy **compose agent** orchestrates
fourteen builtin skills step by step and remains useful for weaker models.

## Notes

- **Context-window control**: per-model window overrides accept a token count
  (`"272K"`), or a percentage of the window (`"50%"`), always clamped to what the
  provider actually accepts — it can only lower the window. `mimo models
  <provider>` prints the resolved window per model. The docs call out OpenAI's
  cost tier where GPT-5.6 prompts above 272K input bill at 2× input / 1.5×
  output.
- **Providers**: catalog providers by API key, OAuth where supported (e.g.
  xAI/Grok), Codex via OpenAI OAuth (ChatGPT Pro/Plus), or any OpenAI-compatible
  API added as a Custom Provider in the TUI.
- **Voice input**: `/voice` streams via TenVAD + MiMo ASR for logged-in MiMo
  users; requires `sox`.
- Also ships `mimo serve` (headless server), `mimo attach <url>`, `mimo acp`
  (Agent Client Protocol), `mimo github` (GitHub agent), `mimo pr <number>`,
  `mimo llm-server` (mint credentials letting a task reach this instance's
  models), `mimo stats`, and `mimo db`.

## Live Verification (st3ve, 2026-09-10)

Ubuntu 24.04.4, MiMo Code 0.1.14, OpenAI `gpt-5.4-mini`.

```sh
mimo run --model openai/gpt-5.4-mini "<task>"
```

- ✅ Zero configuration beyond `OPENAI_API_KEY` in the environment — no login,
  no provider setup, no config file written by hand.
- ✅ Simple task (write `hello.py`, run it, report `4`) passed.
- ✅ Full five-part task passed, including creating `summary.md` and reporting
  `Python 3.12.3` from a real shell call.
- ✅ `mimo run` is genuinely headless — no TTY, no approval prompts.
- Tool names observed via `mimo export <sid>`: `exec`, `exec_command`,
  `apply_patch`.
- ⚠️ `mimo export` with **no** session id blocks waiting for input rather than
  erroring; pass the `ses_…` id from `mimo session list` explicitly in scripts.
- Runtime banner reported the active agent and model as `build · gpt-5.4-mini`.

## Sources (Official)

| Topic | URL |
|-------|-----|
| Docs | https://mimo.xiaomi.com/mimocode |
| GitHub repo | https://github.com/XiaomiMiMo/MiMo-Code |
