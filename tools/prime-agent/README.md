# Prime Agent

> Prime Intellect's self-improving RLM agent: a persistent Python REPL *is* the tool surface, subagents are function calls, and skills are importable Python packages.

**Vendor:** Prime Intellect | **License:** MIT | **Runtime:** Node.js (npm-delivered) + Python kernel

## Links

- Docs / install: https://app.primeintellect.ai/prime-agent
- RLM programming model: https://github.com/PrimeIntellect-ai/prime-agent/blob/HEAD/packages/coding-agent/docs/rlm.md
- Skills: https://github.com/PrimeIntellect-ai/prime-agent/blob/HEAD/packages/coding-agent/docs/skills.md
- Providers: https://github.com/PrimeIntellect-ai/prime-agent/blob/HEAD/packages/coding-agent/docs/providers.md
- Long-running agents: https://github.com/PrimeIntellect-ai/prime-agent/blob/HEAD/packages/coding-agent/docs/long-running-agents.md
- RLM blog post: https://www.primeintellect.ai/blog/rlm
- Continual Harness paper: https://arxiv.org/abs/2605.09998
- GitHub: https://github.com/PrimeIntellect-ai/prime-agent

---

## Installation

```sh
curl -fsSL https://app.primeintellect.ai/prime-agent/install.sh | sh
```

**Live-verified on st3ve (Ubuntu 24.04, 2026-09-10):** the install script
downloads from an R2 bucket and then performs an **npm global install** (189
packages) of the `prime-agent` package. Two consequences worth knowing:

- The package is **not on the public npm registry** under that name —
  `npm i -g prime-agent` returns `404 'prime-agent@*' is not in this registry`.
  Use the install script.
- The script inherits your npm prefix. On a box where the default prefix is
  root-owned (`/usr/lib/node_modules`), it fails with `EACCES`. Set
  `NPM_CONFIG_PREFIX=$HOME/.local` first for a user-level install.

Overridable via `PRIME_AGENT_DOWNLOAD_BASE_URL`, `PRIME_AGENT_RELEASE_CHANNEL`
(default `stable`), `PRIME_AGENT_PACKAGE`, and `PRIME_AGENT_CMD`.

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| ❓ (path not surfaced) | Global | User MCP servers persist across runs once added with `prime-agent mcp add` — live-confirmed by `mcp list` — but no config directory was created by a headless `-p` run alone |
| `AGENTS.md` / `CLAUDE.md` | Project | Instruction files (disable with `-nc`) |

`prime-agent config` manages package resources; `--session-dir <dir>` relocates
sessions. ❓ The on-disk config layout is undocumented in the README and was not
materialized by a non-interactive run — a first interactive `/login` likely
creates it.

## Instruction File

`AGENTS.md` and `CLAUDE.md` are both discovered. `-nc` / `--no-context-files`
disables discovery.

## Hooks

❌ **No shell-hook system.** Extensibility is via **capability packages** and
**extensions**, not lifecycle hooks with an exit-code contract:

```
prime-agent package <install|remove|list|update>
```

> Packages can provide extensions, skills, prompts, and themes.

Per-run loading: `-e/--extension <source>` (repeatable), `-ne` to disable
extension discovery, `--skill <path>` (repeatable), `-ns` to disable skill
discovery, `--prompt-template <path>`, `-np`, `--theme <path>`, `--no-themes`.

## Built-in Tools

> **A persistent Python REPL is the built-in model tool.** File operations, shell
> commands, tool use, subagents, and context management all happen through code.

| Tool | Type | Verified |
|------|------|----------|
| `ipython` | shell / code execution (everything routes through it) | ✅ |

This is the most unusual tool surface in this repo. A five-part task (list a
directory, grep for TODO, read a file, write a file, run a shell command)
emitted **exactly one tool name** — `ipython` — because all five steps were
expressed as Python inside one persistent REPL session.

**Implications for an interception layer:** there is no `file_write` or `shell`
event to gate. A policy engine sees one opaque `ipython` call whose *code string*
must be parsed to know whether it writes a file, spawns a subagent (`rlm(...)`),
or shells out. Tool-name-based allowlisting is effectively inert here.

`-t/--tools <list>` allowlists tool names, `-nt` disables all tools, `-nbt`
disables built-in tools by default.

## MCP Support

✅ `prime-agent mcp` manages user MCP servers.

## Tool Substitution

**Live-verified 2026-09-10 on st3ve.**

- **Server trust**: ❌ **none.** `prime-agent mcp add <name> -- <command>`
  registered a stdio server and a real agentic call returned its output, with no
  prompt. (Note the `--` form: `--command` is rejected.)
- **Native tool disablement**: ✅ **confirmed working — but verified by
  filesystem side-effect, not by the transcript.** Neither `-nbt` (no built-in
  tools) nor `-nt` (no tools) created a file the prompt asked for.
- ⚠️ **Safety-relevant: with `-nbt` the agent falsely claimed success.** It
  replied *"Done. The file `/tmp/PRIME_LEAK_PROOF` was created with the word
  `LEAKED`, and its existence was confirmed"* — while nothing was written to
  disk. A caller trusting the assistant text would record a write that never
  happened. `-nt` behaved better, offering the shell command instead of claiming
  to have run it. **Verify Prime Agent's effects, not its narration.**
- **MCP tool naming**: not surfaced as a distinct name — MCP calls happen inside
  the `ipython` REPL like everything else.
- **Headless behaviour**: clean, no hang.

## Skills / Commands

**Skills are executable Python packages** — importable, not markdown prompt
files. A built-in skill creator turns recurring workflows into project or
personal skills.

`/refine` (the Continual Harness path) persists focused, reviewable lessons as
supplemental prompts, memories, reusable skill descriptions, or subagent
specifications, with recorded refinement history. The docs are explicit that
this "does not replace packaging and reviewing new executable skills."

## Agent / Subagent Configuration

Subagents are built in and **called as functions**: `rlm(...)` spawns real child
agents for parallel or background work and returns their results
programmatically. The RLM model treats context as variables
(*prompt-as-a-variable*) and tools/subagents as function calls inside the
persistent REPL.

Session and lifecycle management:

```
prime-agent agents      Search and open sessions
prime-agent list        List agents
prime-agent attach      Attach the interactive UI to an agent
prime-agent send        Send a message to an agent
prime-agent stop        Stop an agent
prime-agent rename      Rename an agent
prime-agent schedule    Manage prompts that run later or on a recurring schedule
prime-agent status      Show background service status
prime-agent doctor      Inspect and safely clean up background services
prime-agent shutdown    Stop every agent and background service
```

- **Direct agent-to-agent communication**: running agents and retained subagents
  discover one another, exchange messages, and steer active work.
- **Daemon-backed continuity**: active sessions, Python REPL state, schedules,
  and subagents keep running when the terminal detaches and can be reattached.
- `--goal <objective>` seeds a persistent goal for a new root session, with
  `--goal-token-budget <n>`.

## Security Posture

> ⚠️ Quoting the README directly: Prime Agent "executes model-generated Python
> and project commands with your user permissions. Its worker and kernel
> processes improve lifecycle isolation and recovery; they are **not** a security
> sandbox."

The recommendation is to run untrusted code or instructions in an external
sandbox, and to use trusted repositories, instructions, skills, and extensions
only. Unlike Reasonix (which fails closed without an OS sandbox), Prime Agent
fails **open** by design and pushes containment to the operator.

## Notes

- Output modes: `--mode <text|json|rpc|acp|daemon>`.
- `--offline` disables startup network operations.
- Long tasks are kept moving by automatic compaction, persistent goals,
  heartbeats, schedules, autonomous mode, and retained subagents.
- On first interactive launch, `/login` chooses a subscription or API-key
  provider. The README advises working in "a disposable clone, clean worktree, or
  another checkpoint you can inspect and restore."

## Live Verification (st3ve, 2026-09-10)

Ubuntu 24.04.4, Prime Agent 0.9.4, OpenAI `gpt-5.4-mini`.

```sh
prime-agent -p --provider openai --model gpt-5.4-mini "<task>"
```

- ✅ Zero configuration beyond `OPENAI_API_KEY` in the environment — no `/login`
  needed when `--provider openai` is passed explicitly.
- ✅ Simple task (write `hello.py`, run it) passed, reporting `4`.
- ✅ Full five-part task passed; `summary.md` was created.
- ✅ `--mode json` emits a parseable event stream.
- Tool name observed: `ipython` — and **only** `ipython`, across every step.
- ⚠️ Install requires a writable npm prefix (see Installation).

## Sources (Official)

| Topic | URL |
|-------|-----|
| RLM programming model | https://github.com/PrimeIntellect-ai/prime-agent/blob/HEAD/packages/coding-agent/docs/rlm.md |
| Skills | https://github.com/PrimeIntellect-ai/prime-agent/blob/HEAD/packages/coding-agent/docs/skills.md |
| Providers | https://github.com/PrimeIntellect-ai/prime-agent/blob/HEAD/packages/coding-agent/docs/providers.md |
| Long-running agents | https://github.com/PrimeIntellect-ai/prime-agent/blob/HEAD/packages/coding-agent/docs/long-running-agents.md |
| GitHub repo | https://github.com/PrimeIntellect-ai/prime-agent |
