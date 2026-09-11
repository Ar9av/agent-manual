# Framework Governance Claims — Live Verification

Most framework catalogs rate Responsible-AI / security capabilities from docs. This
page does the opposite: it **installs the framework and tries to falsify the rating**.

The claim set and the four-column taxonomy come from
[`prayagupa/agent-frameworks`](https://github.com/prayagupa/agent-frameworks), whose
own legend says the ratings are "a best-effort snapshot — features move fast, so
verify against each project's current docs." This page is that verification, done
against running code rather than docs.

## The four columns, as defined upstream

| Column | Meaning |
|---|---|
| **Guardrails** | Native input/output validation, content/safety filters, policy checks, structured-output validation, or human-in-the-loop gates |
| **Identity** | A first-class agent identity or principal (stable agent ID, credentials, auth, RBAC) beyond a display name/role |
| **Audit** | Built-in tracing/observability of agent steps (events, callbacks, OpenTelemetry) |
| **Sandbox** | Isolated execution of agent-generated code or tools (containers, microVMs, or a remote sandbox) |

Ratings: ✅ built-in / first-class · 🟡 partial, basic, or via an official plugin · — not built-in.

## Method

Each framework was installed into a **fresh throwaway venv** on the st3ve sandbox
(Ubuntu 24.04.4, Python 3.12.3) and driven against the **OpenAI API** with
`gpt-5.4-mini`. Each venv was destroyed before the next install — the box had
~1.5 GB free, not enough for concurrent trees.

For each claim the test tries to **break** it:

- **Guardrails** — write a guardrail that must reject, then send input that should
  trip it. A ✅ only survives if the run is actually blocked or corrected.
- **Audit** — attach a handler / read the tracer and require real emitted events or
  a real trace id, not just the presence of a class.
- **Sandbox** — attempt an escape (`import os`) under the default configuration.
- **Identity** — inspect for an actual credential/auth surface.

Verified 2026-09-10.

## Results

| Framework | Version | Guardrails | Identity | Audit | Sandbox |
|---|---|:---:|:---:|:---:|:---:|
| OpenAI Agents SDK | 0.22.2 | ✅ **confirmed** | — (not tested) | ✅ **confirmed** | 🟡 → **✅ understated** |
| CrewAI | 1.15.21 | ✅ **confirmed** | — | ✅ holds, **stale link** | 🟡 confirmed fair |
| smolagents | 1.26.0 | 🟡 fair | — | ✅ confirmed | ✅ **strongly confirmed** |
| Pydantic AI | 2.42.0 | 🟡 fair | — | ✅ confirmed | — correct |
| Google ADK | 2.9.0 | ✅ **confirmed** | 🟡 → **arguably ✅** | ✅ confirmed | ✅ confirmed |
| LangGraph | 1.2.11 | 🟡 fair | — | ✅ confirmed | — correct |

Six of 24 ratings needed no change, and **three should be revised upward**. No
rating was found to be *overstated* — every ✅ tested held up under a real attempt
to break it, which is a better result than doc-sourced matrices usually earn.

---

## Corrections

### 1. OpenAI Agents SDK — Sandbox `🟡` should be `✅`

The upstream table rates this 🟡. Version 0.22.2 exports a whole isolation surface
from the top-level package:

```
sandbox, CodeInterpreterTool,
ShellToolContainerAutoEnvironment, ShellToolContainerReferenceEnvironment,
ShellToolContainerSkill,
ShellToolContainerNetworkPolicy,
ShellToolContainerNetworkPolicyAllowlist,
ShellToolContainerNetworkPolicyDisabled,
ShellToolContainerNetworkPolicyDomainSecret
```

A dedicated `sandbox` module, a container-backed shell tool, and **egress control
with three network-policy modes** (allowlist, disabled, domain-secret) is
first-class isolation by the upstream legend's own definition, not "partial".

### 2. CrewAI — Audit `✅` holds, but the supporting link is dead

The upstream Audit cell links to `crewai.utilities.events`. **That module does not
exist in CrewAI 1.15.21** — importing it raises
`ModuleNotFoundError: No module named 'crewai.utilities.events'`.

Tracing is still there, but it moved and now has a **preference gate**. First run
printed:

```
╭─── Tracing Preference Saved ───╮
│  Info: Tracing has been disabled.                             │
│  To enable tracing later, do any one of these:                │
│  • Set tracing=True in your Crew/Flow code                    │
│  • Set CREWAI_TRACING_ENABLED=true in your project's .env     │
│  • Run: crewai traces enable                                  │
╰───────────────────────────────────────────────────────────────╯
```

Two things worth flagging for anyone relying on CrewAI for audit:

- **Tracing defaults to disabled** in this path — an audit capability you have to
  opt into is materially different from one that is on.
- The preference is **decided at first run**. In a non-interactive context it
  silently resolved to "disabled" and persisted that choice.

### 3. Google ADK — Identity `🟡` is arguably `✅`

ADK 2.9.0 ships a complete auth package, not a partial one:

```
google.adk.auth
google.adk.auth.auth_credential
google.adk.auth.auth_handler
google.adk.auth.auth_schemes
google.adk.auth.auth_provider_registry
google.adk.auth.auth_preprocessor
google.adk.auth.auth_tool
google.adk.auth._auth_headers
```

A credential model, a handler, a scheme registry, and a provider registry is a
first-class credential surface. It is fair to argue 🟡 if the bar is "stable agent
principal with RBAC" specifically, but the cell deserves a footnote either way.

### 4. Google ADK — an unlisted first-class guardrail

Not mentioned upstream: ADK ships
**`google.adk.integrations.model_armor._plugin`** — an integration with Google's
Model Armor content-safety service — alongside
`google.adk.evaluation.safety_evaluator`. That is a dedicated safety product
wired in as a plugin, which strengthens the already-✅ Guardrails rating.

---

## What each test actually did

### OpenAI Agents SDK 0.22.2 — ✅ Guardrails, ✅ Audit

An `@input_guardrail` with a tripwire on the word "homework":

```
BASELINE: BASELINE_OK
GUARDRAIL: tripwire raised -> InputGuardrailTripwireTriggered
AUDIT: trace provider present -> DefaultTraceProvider
AUDIT: trace_id -> trace_0daaa442321e4cd09229c062aae9b12b
```

The offending request raised rather than reaching the model — a real block, and a
real trace id, not just an importable class.

### CrewAI 1.15.21 — ✅ Guardrails, genuinely enforced

`Task(guardrail=...)` demanded uppercase output; the task description explicitly
asked for lowercase. Result:

```
GUARDRAIL: Task.guardrail field present -> True
GUARDRAIL RESULT: 'HELLO WORLD'
```

The guardrail did not merely reject — it **forced a correction loop** until the
output satisfied the constraint. This is the strongest guardrail behaviour observed
in the six.

### smolagents 1.26.0 — ✅ Sandbox, on by default

Executors available: `LocalPythonExecutor` (**the default**), `DockerExecutor`,
`E2BExecutor`, `ModalExecutor`, `BlaxelExecutor`.

The escape attempt, run against the default local executor:

```
Code execution failed at line 'import os' due to:
InterpreterError: Import of os is not allowed. Authorized imports are:
['re','collections','time','datetime','itertools','random','stat','queue',
 'statistics','unicodedata','math']
```

Notable because **the isolation is on without opting in**. Most frameworks rated
✅ for sandbox require you to select a container/remote executor first; smolagents
enforces an import allowlist even in its default in-process mode.

### Pydantic AI 2.42.0 — 🟡 Guardrails (fair), ✅ Audit

A `field_validator` requiring uppercase, against a prompt asking for lowercase:

```
OUTPUT VALIDATION: shout='HELLO THERE'
GUARDRAIL symbols: ['FilteredToolset']
AUDIT symbols: ['InstrumentationSettings', '_instrumentation']
```

Validation forced a retry, so it works — but this is structured-output validation
plus a `FilteredToolset`, not a safety/content-filter layer. 🟡 is the right call.

### Google ADK 2.9.0 — ✅ Guardrails, live-blocked

`LlmAgent` exposes eight callback hooks:

```
before_agent_callback   after_agent_callback
before_model_callback   after_model_callback
before_tool_callback    after_tool_callback
on_model_error_callback on_tool_error_callback
```

A `before_model_callback` returning an `LlmResponse` short-circuits the model call.
Driven through `LiteLlm(model="openai/gpt-5.4-mini")`:

```
BASELINE:  OK
GUARDRAIL: BLOCKED_BY_CALLBACK
BLOCKED callbacks fired: ['before_model']
```

The model was never reached for the flagged input. Telemetry side:
`google.adk.telemetry` with OTel semconv, a metric exporter, and — unusually — a
`_hallucination` module.

> ⚠️ Install gotcha: the LiteLLM bridge is **not** `google-adk[litellm]` (pip warns
> `does not provide the extra 'litellm'` and installs nothing useful). It is
> `pip install "google-adk[extensions]"`. Importing `google.adk.models.lite_llm`
> without it raises `ImportError: LiteLLM support requires: pip install
> google-adk[extensions]`.

### LangGraph 1.2.11 — 🟡 Guardrails (fair), ✅ Audit

The human-in-the-loop gate, which is what earns the 🟡, works end to end:

```
GUARDRAIL: run paused at interrupt -> True
GUARDRAIL: after deny  -> HALTED_BY_GATE
GUARDRAIL: after allow -> OK
AUDIT: callback events -> ['llm_start', 'llm_end']
SANDBOX modules: NONE
```

`interrupt()` genuinely suspends the graph mid-run behind a checkpointer, and
`Command(resume=...)` routes to halt or proceed. Audit confirmed by real
`BaseCallbackHandler` events, not by class presence. Sandbox `—` is correct: no
isolation modules exist in the package.

---

## Caveats

- **Identity was only spot-checked.** Five of six frameworks are rated `—` upstream
  and were not probed; only ADK's was inspected.
- **Static-surface checks are not behavioural.** The OpenAI Agents SDK and ADK
  sandbox findings come from inspecting the shipped API surface. The
  *smolagents* sandbox result is behavioural (a real escape attempt) and is the
  stronger kind of evidence.
- **Versions move.** Every rating here is pinned to the version in the results
  table. The CrewAI finding exists precisely because an upstream link outlived the
  module it pointed at.
- The upstream list has **64 entries**; six were tested. This page does not claim
  to validate the rest.

## Source

| Topic | URL |
|---|---|
| Upstream claim set | https://github.com/prayagupa/agent-frameworks |
| OpenAI Agents SDK | https://github.com/openai/openai-agents-python |
| CrewAI | https://github.com/crewAIInc/crewAI |
| smolagents | https://github.com/huggingface/smolagents |
| Pydantic AI | https://github.com/pydantic/pydantic-ai |
| Google ADK | https://github.com/google/adk-python |
| LangGraph | https://github.com/langchain-ai/langgraph |
