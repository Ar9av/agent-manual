# Tau

> Hugging Face's Python port of Pi's minimalist coding agent — a small, provider-neutral harness with durable JSONL sessions.

**Vendor:** Hugging Face | **License:** MIT | **Runtime:** Python (pipx / uv / pip)

## Links

- Docs / site: http://twotimespi.dev/
- Providers guide: https://twotimespi.dev/guides/providers-and-models/
- GitHub: https://github.com/huggingface/tau

---

## Installation

```sh
curl -LsSf https://twotimespi.dev/install.sh | sh
# or
uv tool install tau-ai
pipx install tau-ai
python -m pip install tau-ai
```

**Live-verified on st3ve (Ubuntu 24.04, 2026-09-10):** `pipx install tau-ai`
installed tau-ai 0.4.2 on Python 3.12.3 and exposed the `tau` command. The
cleanest install of the six harnesses tested in this pass — a plain Python
package with no native binary fetch, no npm prefix problem, and no post-install
migration.

## Architecture

The README describes a deliberate split:

- `tau_ai` — translates model providers into Tau's **provider-neutral stream**.
- the harness proper — provider config, project instructions, skills, and
  on-disk sessions.

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.tau/catalog.toml` | Global | Custom provider/model catalog (same schema as the built-in one) |
| `~/.tau/sessions/<workspace-slug>/` | Global | Durable JSONL sessions with resume and branching |
| `~/.tau/state/extensions/` | Global | Installed extensions |
| `~/.tau/cache/` | Global | Update-check cache |
| `.tau/` | Project | Project resources |
| `.agents/` | Project | Project resources |
| `AGENTS.md` | Project | Instruction file |

**Directory layout confirmed live** — after one headless run, `~/.tau` contained
`state/extensions/`, `cache/update-check.json`, and
`sessions/home-harness-lab-ws-tau-05eeae/` holding a per-session `.jsonl`, an
`index.jsonl`, and a lock file.

## Instruction File

Project instructions come from `AGENTS.md`, plus `.tau/` and `.agents/`
resources.

## Providers

Tau ships support for OpenAI, Anthropic, OpenAI Codex subscription auth,
OpenRouter, Hugging Face, and custom OpenAI-compatible endpoints including local
models. Connect one interactively with `/login`:

```
/login openai
/login openai-codex
```

Or non-interactively via flags — `--base-url` defaults to
`https://api.openai.com/v1` and `--api-key-env` defaults to `OPENAI_API_KEY`,
both consumed by `tau setup`. `tau providers` lists what is configured.

Extend the catalog by dropping a `~/.tau/catalog.toml` with the same schema as
the built-in one.

## Hooks

❌ **No hook system shipped.** Extensibility is through **extensions**:

```sh
tau install SOURCE [--force]   # Install a trusted local or Git extension
```

Note the word *trusted* — extensions are installed from a local path or Git
source with no documented sandbox or review gate.

## Built-in Tools

Observed live in session JSONL on st3ve:

| Tool | Type | Verified |
|------|------|----------|
| `bash` | shell | ✅ |
| `read` | file read | ✅ |
| `write` | file write | ✅ |

A minimal, Pi-lineage surface — the same `bash`/`read`/`write` core as omp
without omp's `grep`/`todo`/`task` layer. Session records carry both `"name"`
and `"toolName"` spellings for the same call, so a normalizer should read either.

`-t/--thinking <level>` sets the initial thinking level per run (`off`,
`minimal`, `low`, `medium`, `high`, `xhigh`, `max`) and overrides remembered
defaults without persisting.

## MCP Support

❓ Not documented in the README and not surfaced in `tau --help`. No `tau mcp`
subcommand exists in 0.4.2.

## Tool Substitution

❌ / ❓ — with no MCP surface and no documented tool-disablement flag, tool
substitution does not appear to be reachable in 0.4.2.

## Skills / Commands

User skills, prompt templates, and custom TUI themes are supported. Skills are
discovered from project `.tau/` and `.agents/` resources.

## Agent / Subagent Configuration

❓ No subagent surface documented or exposed in the CLI. Consistent with the
"minimalist" framing — Tau is the smallest harness in this pass.

## CLI Surface

```
tau install SOURCE [--force]  Install a trusted local or Git extension
tau update                    Upgrade Tau
tau sessions                  List indexed sessions
tau export REF [DEST]         Export a session as HTML or JSONL
tau providers                 List configured model providers
tau setup                     Configure an OpenAI-compatible provider
```

Plus per-run flags: `-p/--print` (non-interactive), `--provider`, `-m/--model`,
`-t/--thinking`, `--base-url`, `--api-key-env`, `--timeout-seconds`.

## Notes

- **Provider-neutral event rendering** for Rich, plain text, JSON, transcripts,
  and sessions — the rendering layer is decoupled from the provider stream.
- Sessions are durable JSONL with **resume and branching**, and
  `tau export REF [DEST]` writes HTML or JSONL.
- Lineage: a Python port of [Pi](https://github.com/badlogic/pi-mono), tracked
  in this repo at [`tools/pi-agent/`](../pi-agent/). The other Pi descendant in
  this pass is [`tools/oh-my-pi/`](../oh-my-pi/), which took the opposite path —
  maximal surface area rather than minimal.

## Live Verification (st3ve, 2026-09-10)

Ubuntu 24.04.4, tau 0.4.2, OpenAI `gpt-5.4-mini`.

```sh
tau -p --provider openai --model gpt-5.4-mini "<task>"
```

- ✅ Zero configuration beyond `OPENAI_API_KEY` in the environment — no
  `tau setup`, no `/login`, no config file.
- ✅ Simple task (write `hello.py`, run it) passed, reporting `4`.
- ✅ Full five-part task passed; `summary.md` was created.
- ✅ Clean non-interactive behaviour with `-p` — no TTY required, no approval
  prompts.
- Tool names observed in session JSONL: `bash`, `read`, `write`.
- ⚠️ `tau install --help` is not accepted (`Unknown option for tau install:
  --help`); subcommand help is not uniformly wired in 0.4.2.

## Sources (Official)

| Topic | URL |
|-------|-----|
| Docs | http://twotimespi.dev/ |
| Providers and models | https://twotimespi.dev/guides/providers-and-models/ |
| GitHub repo | https://github.com/huggingface/tau |
