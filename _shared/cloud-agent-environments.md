# Cloud Agent Environments

Hosted platforms that run a coding agent in a VM you never log into: what runs
at boot, and **which hook config the cloud agent actually loads**.

The tool pages answer *how do I hook agent X on my laptop*. This page answers
**"the agent runs on someone else's VM — can my hook still block a tool call
there, and where does the config have to live?"**

> **Read the verification column before relying on a row.** Only Replicas was
> driven on the real platform. Four rows are the platform's open-source or CLI
> agent run in a fresh VM stand-in, which proves the *agent's* behaviour but not
> the hosted harness around it. Everything else is doc-only, fetched 2026-09-16.

Measured 2026-09-16/17. Stand-in VM: `python:3.12-slim` container on the st3ve
sandbox (Ubuntu 24.04), non-root user, setup and agent phases run as separate
shell sessions. Models via the OpenAI API except the Claude Code row.

---

## The two findings

**1. A boot script is universal.** Every platform below lets the customer run
arbitrary shell before the agent's first prompt. Installing a binary is never
the problem.

**2. The home directory is usually the wrong place for hook config.** The three
largest platforms document loading hooks *only from the cloned repo* (or from
org-managed settings). A `~/.claude/settings.json` that a setup script writes is
not documented as honored anywhere except on platforms that run a stock agent
CLI with a normal home (Replicas, and by the look of their docs Terragon and
Warp Oz).

The consequence for anyone shipping a hook tool: the hook command that lands in
a committed file cannot contain an interpreter path, a shim path, or a workspace
path. It has to find its binary at run time. The pattern that worked across
every agent tested here:

```json
{
  "type": "command",
  "command": "sh -c 'p=$(command -v mytool || echo \"$HOME/.local/bin/mytool\"); [ -x \"$p\" ] || { echo \"mytool not installed: tool call not screened\" >&2; [ -n \"$MYTOOL_REQUIRED\" ] && exit 2; exit 0; }; exec \"$p\" hook --agent claude'"
}
```

Two-step recipe: **the setup script installs the binary, the repo carries the
hook config.** Missing binary warns and allows, so a teammate without the tool
is not locked out; an environment variable set on the cloud environment flips
that to a block (exit 2), so a failed setup script does not silently mean an
unscreened agent.

---

## Platform matrix

| Platform | Agent that runs | Boot mechanism | Where it lives | Hook config the cloud agent loads | Verification |
|---|---|---|---|---|---|
| [Replicas](https://docs.replicas.dev/features/environments.md) | Claude Code, Codex, Cursor, OpenCode, Gemini CLI, Pi, Kimi, others | Start hook (every boot) + warm hook (pool build) | App UI per environment, or `replicas.json` `startHook` in repo | **Home and repo both.** Stock CLI, normal `$HOME` | **Live on platform** |
| [Claude Code on the web](https://code.claude.com/docs/en/claude-code-on-the-web) | Claude Code | Setup script, runs as root, result snapshotted ~7 days | Environment dialog at claude.ai/code | **Repo `.claude/settings.json` and server-managed settings only.** "User-level settings stay on your machine" | Agent verified in stand-in; platform doc-only |
| [Codex cloud](https://learn.chatgpt.com/docs/environments/cloud-environment) | Codex | Setup script + maintenance script, cached ≤12 h | Codex settings → Environments (UI only) | **Undocumented.** Cloud docs mention only `AGENTS.md`. See the Codex finding below | Agent verified in stand-in; platform doc-only |
| [Cursor Cloud Agents](https://cursor.com/docs/cloud-agent/setup) | Cursor agent | `install` (build), `start` (every boot), `terminals` | `.cursor/environment.json` in repo | **Repo `.cursor/hooks.json`** plus dashboard team hooks. Not run during early read-only exploration turns | Doc-only |
| [GitHub Copilot coding agent](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/customize-the-agent-environment) | Copilot | Job `copilot-setup-steps` | `.github/workflows/copilot-setup-steps.yml` | **Repo `.github/hooks/*.json` only.** `ask` is treated as deny | Doc-only |
| [OpenHands Cloud](https://docs.openhands.dev/openhands/usage/customization/repository) | OpenHands | `.openhands/setup.sh`, every conversation | Repo | **Repo `.openhands/hooks.json`** (no documented global path) | Agent verified in stand-in; platform doc-only |
| [Devin](https://docs.devin.ai/onboard-devin/environment/blueprint-reference.md) | Devin | Blueprint `initialize` / `maintenance`, at snapshot build, not per session | `.devin/blueprint.yaml`, org or repo level | Hooks are documented for the Devin **CLI**; nothing says they apply to cloud sessions | Doc-only |
| [Factory Droid computers](https://docs.factory.ai/droid-computers/cloud-templates.md) | Droid | Template setup script, once at template build | Template creation modal | `.factory/hooks.json` in repo is the likely vector; cloud behaviour undocumented | Doc-only |
| [Ona](https://ona.com/docs/ona/configuration/tasks-and-services/overview) (ex-Gitpod) | Codex harness; or your own agent | Tasks with `triggeredBy: postEnvironmentStart` | `.ona/config.yaml`, devcontainer | Native guardrail is a command deny-list, not hooks. A self-installed Claude Code should behave as local | Doc-only |
| [Warp Oz](https://docs.warp.dev/platform/environments/configuring-environments/) | Warp Agent, Claude Code, or Codex | `--setup-command` (repeatable) or custom glibc Docker image | `oz environment create` | Undocumented; plausible for the `claude` harness | Doc-only |
| [Jules](https://jules.google/docs/environment/) | Jules | Setup script, then snapshot | Repo → Configuration in web UI | **None. Jules has no hook mechanism** | Doc-only |
| [Terragon](https://docs.terragonlabs.com/docs/configuration/environment-setup/setup-scripts) | Claude Code | Setup script, every sandbox start, `~/.bashrc` sourced | Repo or per-user | Likely home and repo (stock Claude Code) | Doc-only (search summary) |
| [Amp orbs](https://ampcode.com/notes/putting-an-agent-in-an-orb) | Amp | `.agents/setup` on each fresh orb | Repo | Amp has a plugin API, no declarative hooks | Doc-only (search summary) |
| [Kilo Cloud Agent](https://kilo.ai/docs/code-with-ai/platforms/cloud-agent) | Kilo Code | Setup Commands (UI or profile); `.kilo/setup-script` is **not** auto-run | UI | Plugin API, no exit-code hook contract | Doc-only (search summary) |

### Network at boot vs. during the agent phase

A hook that phones home (a hosted judge, telemetry) can work at setup and fail
once the agent starts.

| Platform | Setup phase | Agent phase |
|---|---|---|
| Codex cloud | Always has internet | **Off by default.** Opt-in allowlist: None / Common dependencies (includes PyPI) / All; optional GET/HEAD/OPTIONS only |
| Claude Code on the web | Same level as the session | None / Trusted (default, includes PyPI) / Full / Custom |
| Copilot coding agent | Unrestricted ("firewall only applies to processes started by the agent") | Default allowlist includes package registries |
| Cursor Cloud Agents | Undocumented whether `install` is exempt | Allow all / Default + allowlist / Allowlist only |
| Devin | Unrestricted unless a Security Profile is pinned | Same |
| Jules | "VM with internet access", no allowlist documented | Same |
| Replicas | Unrestricted; optional static egress IP | Same |

### Snapshots change where install steps belong

Codex cloud, Claude Code on the web, Devin, Factory, Jules and Replicas warm
pools all run the script **once** and start sessions from the snapshot. Install
binaries there. Anything that mints an identity (device enrollment, a session
token) must run per boot or it is cloned into every VM from that snapshot.

---

## Live verification

### Replicas — on the real platform (2026-09-16)

Global environment start hook installed a hook tool into a venv and ran its
non-interactive setup for Claude Code and Codex. The **Test** button runs the
hook in a throwaway sandbox in about ten seconds.

| Step | Result |
|---|---|
| `pipx install` / `pip install --user` | ❌ No `pipx`; system pip is PEP 668 locked. A venv under `$HOME` works |
| Hook config written to `~/.claude/settings.json` by the start hook | ✅ Honored by the Claude Code session |
| Claude Code (Opus 5): `touch /tmp/f && chmod u+s /tmp/f` | ✅ Blocked at `PreToolUse`, block text returned as the tool result |
| Claude Code: `cp ~/.claude/settings.json /tmp/x.json` | ✅ Blocked |
| Claude Code: `echo hello` | ✅ Ran |
| Binary on the agent's `PATH` | ❌ `export PATH` in the hook and a `~/.bashrc` line both fail to reach the agent's non-interactive Bash. Use absolute paths in hook commands |
| Codex hooks | ⚠️ Installed but untrusted, so never run (see below) |

Working directory of the start hook is `$HOME`, so a tool that installs "project
scope" hooks from there writes them to the same directory as global scope and
every call dispatches twice.

### Committed portable hook in a fresh VM — deterministic (2026-09-17)

Repo generated on a "developer" container, then mounted into fresh VMs. No
machine path appeared in any committed file (`grep` for the dev paths: none).
The committed `PreToolUse` command was fed a real payload on stdin.

| VM | setuid `chmod` payload | `echo hello` payload |
|---|---|---|
| No setup script ran | exit 0, stderr `not installed: tool call not screened` | exit 0 |
| No setup script, required-env set | **exit 2** | **exit 2** |
| Setup script ran in a separate session first | **exit 2**, rule text on stderr | exit 0 |

### Codex CLI 0.154.0 in the fresh VM, OpenAI API (2026-09-17)

Same repo, committed `.codex/hooks.json` with the portable command, `codex exec`.

| Variant | Command executed? |
|---|---|
| Repo hooks only | **Yes — unscreened** |
| + `[features] hooks = true` in `~/.codex/config.toml` (project already `trust_level = "trusted"`) | **Yes — unscreened** |
| + `--dangerously-bypass-hook-trust` | **No — blocked by the hook** |

**The Codex finding.** Project trust and the hooks feature flag are not enough.
Codex runs a hook only when `~/.codex/config.toml` holds a per-hook trust record,
`[hooks.state."<abs path to hooks.json>:<event>:0:0"]` with a `trusted_hash`,
and those are written by accepting an interactive prompt. Nobody opens a TUI in
a cloud VM, and the customer does not control the `codex` invocation there. So
unless Codex cloud's own harness bypasses hook trust — undocumented, and not
testable without a connected cloud environment — **a hook cannot block Codex in
any hosted environment today**, including Codex sessions on Replicas.

### OpenHands CLI 1.16.0, headless, `gpt-4o-mini` (2026-09-17)

Isolated fresh `$HOME`, committed `.openhands/hooks.json` with the portable
command, `openhands --headless --override-with-envs`.

| VM | "Hooks loaded" | setuid `chmod` |
|---|---|---|
| Binary not installed | ✅ | **Ran.** The warning goes to stderr and is not surfaced to the agent or the transcript |
| After the setup script | ✅ | **Blocked**; agent reported "blocked due to security restrictions against privilege escalation" |

### Claude Code, project settings only (2026-09-17)

`claude -p ... --setting-sources project` in a repo whose only hook config is
the committed portable `.claude/settings.json`. This is the shape Claude Code on
the web documents: repo hooks load, user-level settings do not.

| Step | Result |
|---|---|
| setuid `chmod` | ✅ Blocked by the committed hook; agent declined to work around it |

> A first run of this test *allowed* the command. The hook fired and matched, but
> the hook tool on that machine was in a paused/observe state and exited 0. When
> a committed hook "does nothing", check the tool's local state before blaming
> the agent.

---

## Not verified

| Platform | Why |
|---|---|
| Codex cloud | Needs a GitHub repo connected to a ChatGPT workspace. The one open question (does the hosted harness honor hook trust?) cannot be answered from a CLI |
| Claude Code on the web | Not reachable from the automation browser used here |
| Cursor Cloud Agents | Needs a Cursor account; `cursor-agent` on the sandbox is logged out |
| Copilot coding agent | Not enabled on the test GitHub account |
| Jules, Devin, Factory, Ona, Warp Oz, Terragon, Amp, Kilo | No account. An OpenAI key does not help: these run the vendor's own agent or their own credentials |

## Sources

| Claim | Source | Date | Type |
|---|---|---|---|
| Replicas environments, hooks, plugins | https://docs.replicas.dev/llms.txt and linked pages | 2026-09-16 | [official] + [live-verified] |
| Codex cloud environments, internet access | https://learn.chatgpt.com/docs/environments/cloud-environment, https://learn.chatgpt.com/docs/cloud/internet-access | 2026-09-16 | [official] |
| Codex hook trust records | `~/.codex/config.toml` on a machine with accepted hooks; `codex exec --help` | 2026-09-17 | [live-verified] |
| Claude Code on the web | https://code.claude.com/docs/en/cloud-environments, https://code.claude.com/docs/en/claude-code-on-the-web | 2026-09-16 | [official] |
| Cursor cloud agents | https://cursor.com/docs/cloud-agent/setup, https://cursor.com/docs/cloud-agent/network-access, https://cursor.com/docs/agent/hooks | 2026-09-16 | [official] |
| Copilot coding agent | https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/customize-the-agent-environment, https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/customize-the-agent-firewall, https://docs.github.com/en/copilot/reference/hooks-configuration | 2026-09-16 | [official] |
| Jules, Devin, Factory, Ona, Warp Oz | URLs in the matrix | 2026-09-16 | [official] |
| Terragon, Amp, Kilo, OpenHands Cloud | URLs in the matrix | 2026-09-16 | [official, via search summary — page not fetched] |
