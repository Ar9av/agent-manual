# Token & Cost Accounting

Where each agent records token usage, whether it computes money, how to get either
one programmatically — and what the same prompt actually costs across layers.

`agent-tools-hooks-config.md` answers *what the tools are*. This page answers
**"what did that session cost, and can I trust the number the tool gave me?"**

> **All figures on this page are self-reported by the tool.** The OpenAI key used
> for these runs lacks the `api.usage.read` scope, so provider-side ground truth
> was unavailable (`/v1/organization/costs` → HTTP 403,
> `Missing scopes: api.usage.read`). Nothing here is reconciled against a bill.
> To do that yourself you need an **admin** key with that scope; a project key
> will not work.

Measured 2026-09-14/15 on the st3ve sandbox (Ubuntu 24.04.4), `gpt-5.4-mini`
except where noted.

---

## The overhead finding

The same trivial prompt — *"Reply with exactly the word PING and nothing else. Do
not use any tools."* — sent through each layer, one fresh session each, no cache:

| Layer | Tool | Input tokens | Output | Cost the tool reported |
|---|---|---:|---:|---|
| Raw SDK | LangChain `ChatOpenAI` | **17** | 4 | `$0.0` ⚠️ (see below) |
| Raw SDK | OpenAI Agents SDK | **24** | 5 | no cost field |
| Agentic framework | smolagents `CodeAgent` | **4,067** | 44 | no cost field |
| Coding harness | Tau | **4,487** | 20 | no cost field |
| Coding harness | Reasonix | **5,543** | 4 | `unavailable` ⚠️ |
| Coding harness | Codewhale¹ | **8,073** | 2 | no cost field |
| Coding harness | Prime Agent | **12,891** | 21 | `$0.00976` |
| Coding harness | oh-my-pi (omp) | **21,667** | 24 | `$0.01636` |

¹ Codewhale ran `gpt-4.1-mini` — it cannot use GPT-5 models at all (see its page).
Input-token counts are broadly comparable across the same tokenizer family, but
its cost basis is not.

**A 1,274× spread** between a bare SDK call and the heaviest harness for an
identical question, and **4.8×** among the coding harnesses alone. That gap is
fixed cost you pay on *every* turn: system prompt + tool schemas + environment
preamble. It is the single largest lever on agent spend, and it is invisible
unless the tool reports input tokens.

Codewhale is the only tool measured that breaks the overhead down for you:

```json
"input_analysis": {
  "estimated_request_tokens": 8683,
  "estimated_system_tokens": 8479,      ← the overhead
  "estimated_message_content_tokens": 88, ← your actual question
  "estimated_framing_tokens": 72
}
```

8,479 of 8,683 tokens — **98%** — was the harness, not the user. Treat that ratio
as typical, not exceptional.

---

## Trust warnings

Three tools report a cost number you should not believe, in decreasing order of
honesty:

### Reasonix — honest "unavailable"

`~/.reasonix/stats/<date>.jsonl`, per request:

```json
{"ts":"...","model":"openai/gpt-5.4-mini","prompt":5543,"completion":4,
 "cache_miss":5543,"total":5547,"requests":1,
 "cost_complete":false,"cost_estimated":true,
 "display_status":"unavailable","incomplete_reason":"no_price"}
```

Reasonix **tracks tokens accurately but has no price table for OpenAI models** —
its pricing covers DeepSeek. It says so explicitly (`no_price`,
`display_status: "unavailable"`). Tokens: trustworthy. Money: absent, and
correctly labelled absent.

### LangChain — silent `$0.0`

```
get_openai_callback -> prompt=17 completion=4 total_cost=$0.0
```

`langchain_community.callbacks.get_openai_callback()` returned **`$0.0`** for a
real, billable call. Its price table does not know `gpt-5.4-mini`, and the
unknown-model path yields zero rather than an error or a null. A dashboard built
on this reports a free agent fleet. Same root cause as Reasonix's `no_price` —
opposite failure mode: one says "I don't know", the other says "nothing".

(`langchain-community` is also formally sunset; `usage_metadata` on the response
is the maintained path and was accurate: `{'input_tokens': 17, 'output_tokens': 4,
'total_tokens': 21, ...}` — tokens only, no money.)

### MiMo Code — `mimo stats` is not MiMo's usage

```
Sessions 405 | Messages 965 | Days 36
Total Cost $0.13 | Input 194.8K | Output 176.6K
Cache Read 10.9M | Cache Write 3.0M
TOOL USAGE: Bash 220 · Read 83 · Edit 40 · mcp__prismor__Read 27
            · ToolSearch 22 · ScheduleWakeup 3 · Workflow 1 · Agent 3
```

MiMo Code had been installed on this box for **5 days** and run ~10 times. The
report claims **405 sessions over 36 days**, 10.9M cache-read tokens, and a tool
histogram full of names MiMo does not have — `ToolSearch`, `ScheduleWakeup`,
`TaskCreate`, `Workflow`, `mcp__prismor__*` are **Claude Code's** tools.

Evidence it ingested another agent's history into its own store:

- `grep -rl "ScheduleWakeup" ~/.local/share/mimocode/` matches
  **`mimocode.db`** — MiMo's own SQLite database, not a foreign file it read live.
- The same strings exist in `~/.claude/projects/*.jsonl` (75 MB of Claude Code
  history present on the box).
- Volume and date range are impossible for the install: 36 days vs 5.

This is consistent with MiMo being an **OpenCode fork** (already documented on its
page — it also discovers skills from `~/.claude/skills/`), but session
*transcripts* are a larger surface than skills. Two consequences:

1. **You cannot attribute cost to MiMo with `mimo stats`.** It is a blended figure
   across whatever histories it found.
2. **It copied another agent's conversation transcripts into its own database.**
   Worth knowing before pointing it at a machine with sensitive session logs.

`--project ""` scopes to the current project and `--days N` narrows the window;
neither was verified to exclude imported sessions.

---

## Per-tool extraction reference

### Coding harnesses

| Tool | Where usage lives | Tokens | Cost | How to get it |
|---|---|:---:|:---:|---|
| **oh-my-pi** | stdout event stream | ✅ | ✅ | `omp -p --mode json` → `message_end.message.usage` |
| **Prime Agent** | stdout event stream | ✅ | ✅ | `prime-agent -p --mode json` → `messages.usage` |
| **Codewhale** | stdout terminal receipt | ✅ | ❌ | `exec --auto --output-format stream-json` → `{"type":"metadata"}` |
| **Reasonix** | on-disk JSONL | ✅ | ⚠️ no price | `~/.reasonix/stats/<YYYY-MM-DD>.jsonl`, one line per request |
| **Tau** | on-disk session JSONL | ✅ | ❌ | `~/.tau/sessions/<slug>/*.jsonl` → `usage` object |
| **MiMo Code** | SQLite + CLI | ⚠️ blended | ⚠️ blended | `mimo stats [--days N] [--project ""]` |

**oh-my-pi and Prime Agent share an identical usage schema** — both are Pi-lineage:

```json
{"input": 21667, "output": 24, "cacheRead": 0, "cacheWrite": 0,
 "totalTokens": 21691, "reasoningTokens": 17,
 "cost": {"input": 0.01625025, "output": 0.000108,
          "cacheRead": 0, "cacheWrite": 0, "total": 0.01635825}}
```

One normalizer handles both. Tau (also Pi-lineage) uses the same field *names*
but omits `cost`.

Codewhale's receipt is the richest, and uniquely includes content hashes that let
you detect when a prompt or tool catalog changed underneath you:

```json
{"type":"metadata","meta":{
  "input_tokens": 8073, "output_tokens": 2,
  "prompt_cache_hit_tokens": 0, "prompt_cache_miss_tokens": 8073,
  "reasoning_tokens": 0, "duration_ms": 4999,
  "prompt_sha256": "sha256:55660e34…",
  "tool_catalog_sha256": "sha256:5fa53921…",
  "binary_sha256": "sha256:9d0b74d8…",
  "input_analysis": { … },
  "approval_posture": "auto_tools", "sandbox_posture": "configured_default"}}
```

### Frameworks / SDKs

| Framework | Accessor | Tokens | Cost |
|---|---|:---:|:---:|
| **OpenAI Agents SDK** | `result.context_wrapper.usage` → `.requests/.input_tokens/.output_tokens/.total_tokens` | ✅ | ❌ (no `cost` attr) |
| **LangChain / LangGraph** | `response.usage_metadata` (maintained) | ✅ | ❌ |
| **LangChain / LangGraph** | `get_openai_callback()` → `.prompt_tokens/.completion_tokens/.total_cost` | ✅ | ⚠️ **`$0.0` on unknown models** |
| **smolagents** | `agent.monitor.total_input_token_count` / `.total_output_token_count` | ✅ | ❌ (no cost attr at all) |
| **Pydantic AI** | `result.usage` → `RunUsage` | ✅ | ❌ |
| **CrewAI** | `CREWAI_TRACING_ENABLED=true`, or `tracing=True`, or `crewai traces enable` | ✅ | ❓ |
| **Google ADK** | `google.adk.telemetry` (OpenTelemetry semconv + metric exporter) | ✅ | ❓ |

⚠️ **Pydantic AI 2.42.0**: `usage` is a **property, not a method** — `result.usage()`
raises `TypeError: 'RunUsage' object is not callable`. Older snippets calling it
break.

⚠️ **CrewAI 1.15.21**: tracing **defaults to disabled** behind a first-run
preference prompt that resolves silently to "disabled" in non-interactive
contexts. Enable it explicitly or you get no usage data at all. See
`../frameworks/governance-verification.md`.

**Not one framework tested computes cost correctly.** Every one reports tokens and
leaves money to you — except LangChain's legacy callback, which reports money
wrongly. Price your own tokens.

### IDEs

**Not measurable by the methods on this page, and not measured.** Cursor, GitHub
Copilot (VS Code), Kiro, Trae, and Warp bill through a subscription or an
opaque proxy rather than a BYOK key whose tokens you can observe, and none exposes
a per-session token/cost API to a headless caller. Trae in particular has **no
headless CLI at all** (see `mcp-tool-substitution.md`). What they offer is
vendor-side dashboards, which are outside this repo's reproducible-measurement
scope. Treat any per-session IDE figure as unverified.

---

## Practical notes

- **Measure overhead once per tool, per version.** It is a fixed per-turn tax and
  it moves when the vendor edits a system prompt. Codewhale's `prompt_sha256` /
  `tool_catalog_sha256` are the only first-class way to notice that happened.
- **Cache fields matter more than totals.** In the harness runs that reused a
  session, `cacheRead` dominated: one omp turn showed `input: 312` against
  `cacheRead: 21504`. A totals-only dashboard will mis-attribute ~99% of the
  input and badly overstate cost, since cached reads are cheaper.
- **Disable tools you don't need.** Tool schemas are part of the overhead, and
  every harness here can trim them (`--tools`, `--no-tools`, `[tools] enabled`,
  `tools.<x>: false`) — see `mcp-tool-substitution.md` for what actually works.
- **Don't compare across models.** Codewhale's figures are `gpt-4.1-mini` because
  it cannot use GPT-5 at all.
- Output tokens were negligible here by design (2–44). Real work inverts this;
  these numbers isolate *fixed* cost, not total cost.

## Method

One fresh session per tool, no session reuse, cache cold, identical prompt, tools
available but unused. Figures read from each tool's own reported usage — see the
extraction column. Frameworks were installed into throwaway venvs, one at a time.

Reproducing the overhead probe:

```sh
PING="Reply with exactly the word PING and nothing else. Do not use any tools."
omp -p --model openai/gpt-5.4-mini --mode json --no-session "$PING"
prime-agent -p --provider openai --model gpt-5.4-mini --mode json --no-session "$PING"
codewhale --model gpt-4.1-mini exec --auto --output-format stream-json "$PING"
reasonix -p --permission-mode yolo "$PING"   # then read ~/.reasonix/stats/<date>.jsonl
tau -p --provider openai --model gpt-5.4-mini "$PING"  # then read ~/.tau/sessions/*/
mimo run --model openai/gpt-5.4-mini "$PING"           # then `mimo stats` — see warning
```

## Gaps

- **No provider-side reconciliation.** Needs an admin key with `api.usage.read`.
  Until then every number here is the tool's own claim, and the LangChain and MiMo
  findings show those claims can be wrong in both directions.
- **Identity of MiMo's import path not fully traced** — the evidence is its own DB
  containing Claude Code tool names plus an impossible session count, which is
  strong but stops short of observing the copy.
- Six harnesses, seven frameworks, zero IDEs.
