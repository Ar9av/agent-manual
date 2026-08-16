# QM (Quartermaster)

> Y Combinator's MIT-licensed "multiplayer agent harness for work" — a cloud-hosted, self-deployed system where every person and every room (Slack channel or web project) gets a scoped agent workspace, running on top of Pi, OpenCode, Codex, or Claude Code as pluggable model harnesses.

**Vendor:** Y Combinator (yc-software) | **License:** MIT (with exceptions noted in-repo) | **Runtime:** Node.js/TypeScript service (Fastify), Postgres, deployed to Fly.io or AWS — not a local CLI

## Links

- GitHub: https://github.com/yc-software/qm
- Announcement/homepage: https://qm.ycombinator.com
- Getting started: https://github.com/yc-software/qm/blob/main/docs/getting-started.md
- Deploy directory contract: https://github.com/yc-software/qm/blob/main/docs/deploy-directory.md
- Security model: https://github.com/yc-software/qm/blob/main/SECURITY.md
- Contact: labs@ycombinator.com

---

## Installation

QM is not installed as a personal CLI tool — it is a service an organization deploys and self-hosts. There is no single-user "run it in your terminal" install path.

```sh
npm exec --yes --package=@yc-software/qm@latest -- qm init . --org <slug> --target <fly-or-aws>
npm install
```

`qm init` materializes a deployment directory from the published npm package (no source checkout of the QM repo required) and walks the operator through infrastructure choice (Fly.io or AWS), authentication (built-in email one-time-link auth broker, requiring a Resend key or SMTP config, or an external identity provider), connector credentials, and optional Slack app setup. Deployment requires a cloud account and a Postgres database. Confirmed from `package.json`: `engines` pins `node >= 24.15.0` and `npm >= 11.10.0`; `.node-version` in-repo currently pins `24.18.0`. No explicit minimum Postgres server version is stated in docs — local dev tooling (`scripts/dev/lib/postgres.ts`) defaults to a `postgres:16-alpine` image, but that's a dev-instance default, not a documented production minimum.

---

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `qm.config.jsonc` | Deployment | Main deployment configuration |
| `.env` / `.env.example` | Deployment | Secrets and runtime settings (API keys, `HARNESS`, `HARNESS_SECURITY_POSTURE`, `ORG_ID`, Slack tokens, rate-limit/budget vars, signing secrets) — `.env.example` documents names only, `.env` supplies real local values and is never committed |
| `slack-app-manifest.yml` | Deployment | Generated Slack app manifest when Socket Mode bot is enabled |
| `deploy/layers/<org>/` | Org | Org-owned customization layer: config, sandbox tool/skill overrides, provider (Fly/AWS) infra templates, kept isolated so a private fork can track upstream |
| `sandbox/tools/<id>/tool.json` | Org (sandbox) | Declares a sandbox CLI tool (`id`, `label`, `advertise`, `hints`, `auth.credentialPaths`, `egress`, `approvals`) |
| `sandbox/skills/<id>/SKILL.md` | Org (sandbox) | Org-defined skill, `name`/`description` frontmatter required |
| `package.json` (in deployment dir) | Deployment | Pins the exact `@yc-software/qm` package version in use |

There is no per-user personal config file analogous to `~/.claude/settings.json` — configuration lives at the org/deployment level, since QM is a shared, multi-tenant service rather than a local agent.

---

## Instruction File

❓ No confirmed equivalent of a repo-root `AGENTS.md`/`CLAUDE.md` "always-loaded instructions" file for the deployed agent's own behavior was found in official docs. (Note: the QM *source repository itself* still contains top-level `AGENTS.md` and `CLAUDE.md` files, confirmed still present, but these appear to be contributor-facing notes for coding agents working on QM's own codebase, not a feature QM exposes to end-organizations for instructing their deployed agents.) Org- and scope-level customization instead happens through `qm.config.jsonc`, the `deploy/layers/<org>/` directory, and a per-scope **SOUL**.

SOUL is **not a file** — it is a versioned, audited, per-scope string of standing instructions stored server-side (Postgres-backed `soul_configs`/`soul_history` artifact maps) and folded into the agent's system prompt. Confirmed from source: it's readable/writable by the scope's own agent via the agent API (`GET /v1/soul`, `POST /v1/soul`), and by an org admin via the admin API (`GET /v1/admin/scopes/<scopeId>` returns it under `soul`/`soulVersion`; `PUT /v1/admin/scopes/<scopeId>/soul` updates it). The onboarding skill (`plugins/onboarding/skills/onboarding/SKILL.md`) instructs agents to keep SOUL "compact" and "first-person operating" guidance, read-preserve-then-update rather than overwrite.

---

## Hooks

QM does **not** document a Claude-Code-style hook system (no `PreToolUse`/`PostToolUse` lifecycle events with matcher-based command/HTTP/MCP-tool handlers were found in the repo or docs). Instead, QM's own `SECURITY.md` describes its interception layer as **security postures + command policy + content screening**, which function as the closest workaround:

- **Security postures** (set via `HARNESS_SECURITY_POSTURE` in `.env`): `strict` (human approval required for tool calls), `auto` (default — screens external/provenance-labelled content before it reaches the model), and `dangerous` (no content screening).
- **Command policy / approvals**: declared per sandbox tool in `sandbox/tools/<id>/tool.json` via an `approvals` field "appended to the command-policy floor." Per `SECURITY.md`: "It classifies shell text and catches configured or common dangerous forms, but obfuscation, encoding, or writing and then executing a script can evade it." Browser-runner actions explicitly do **not** re-enter command policy or human-in-the-loop approval.
- **Approval is human-only by design**: "Approving a gated command is a human judgment made on the approver's own turn" — this is deliberately kept out of the agent's own self-API to block prompt-injection from self-approving.
- **Content screening gaps** (self-disclosed): command/background-process output, opaque or multimodal tool results, and raw webhook payloads are "not all covered" by screening.

Because this is not an event/exit-code hook API, the template's Supported Events / Hook Input JSON / Exit Code Behavior / Example Config subsections don't apply as-is — there is no user-authored hook script contract analogous to Claude Code's `hooks` key. The repo does have a `plugins/` directory (`admin`, `auth`, `chassis`, `onboarding`, `portal`, `web-ui`) confirmed in-repo, but per the root README these are "optional plugins over the core's HTTP API" — separate processes (e.g. the admin plugin is "a separate process that talks to the core only over the admin governance API; the core has zero dependency on it") rather than an in-process middleware/hook point that fires around individual tool calls. ❓ Whether a lower-level extension point (e.g. custom middleware in the orchestrator, as opposed to these standalone HTTP-API plugin processes) exists for organizations forking the deployment layer still was not confirmed from docs.

---

## Built-in Tools

| Tool | Description |
|------|-------------|
| `execute` | The one core, fixed tool: runs commands inside the scope's isolated, durable sandbox. Per the repo's own description, QM deliberately ships a "small, fixed tool surface" rather than a large enumerated toolset like single-user CLI agents. |

Additional capability is added via **sandbox tools** (`sandbox/tools/<id>/tool.json`, optionally with a bundled binary) that get advertised into the agent's installed-CLI list, and via **connectors**/skills (Slack, Google Drive/Sheets, Dropbox, GitHub/GitLab, Linear, cloud CLIs, etc. — seeded under `skills-seed/`) rather than a fixed Read/Write/Edit/Grep tool family.

---

## MCP Support

Confirmed via `src/mcp/mcp-client.ts`, `src/mcp/mcp-tool-service.ts`, `src/mcp/mcp-server-store.ts`, and admin routes at `src/api/routes/admin/mcp-servers.ts`: QM has first-class **MCP client** support, administered org-wide (not per-user).

- MCP servers are registered by an **org admin only**, since a registered server becomes an outbound HTTP destination reachable by all of that org's agents.
- Registration fields (from `McpServer` type): `id` (2–40 chars, lowercase/digits/hyphens), `name` (≤80 chars), `url` (HTTPS or HTTP, no embedded credentials/query/fragment), `auth` mode (`"none"`, `"bearer"`, or `"client-credentials"`) with `bearerToken` or `clientId`/`clientSecret`, plus `readOnly` and `enabled` flags. Secrets are redacted in API responses (returned as presence booleans).
- Harness adapters (`src/harness/claude-harness.ts`, `codex-harness.ts`, `opencode-harness.ts`, `pi-harness.ts`) plug MCP tools into whichever underlying model harness (Pi, OpenCode, Codex, Claude Code) is driving a given deployment.
- Confirmed REST-only: `GET /v1/admin/mcp-servers`, `PUT /v1/admin/mcp-servers/:id` (create/update), and a delete handler, all in `src/api/routes/admin/mcp-servers.ts` / wired in `src/api/routes/admin.ts`. No dedicated CLI subcommand exists — `cli/src/commands/` contains only `check`, `conformance`, `infra`, `init`, `outputs`, `sandbox`, `setup` — and the admin web-UI plugin's documented tabs (Metrics, History, Files, Live, Errors, Audit, Skills, Crons, Deployments, Volumes, Retention, Users — per `plugins/admin/README.md`) don't list an MCP-servers panel either. ❓ So it remains unconfirmed whether an admin-facing UI exists for this at all; today it looks like a REST-API-only surface (presumably intended for internal/future UI use).

---

## Skills / Commands

- Skills location: `sandbox/skills/<id>/SKILL.md` (org-level, deployed) — mirrors `skills-seed/<name>/SKILL.md` in the source repo, which ships 19 seed skills (`admin`, `browse`, `cloud-cli`, `connect-apps`, `dropbox`, `email-draft-in-voice`, `email-voice-profile`, `github-gitlab`, `google-drive-sheets`, `google-workspace`, `interactive-login`, `linear`, `memory`, `morning-digest`, `popular-web-designs`, `publish`, `slack-drafts`, `taste-skill`, `use-shared-credential`).
- Format: Markdown file with YAML frontmatter requiring `name` and `description` (same shape as Claude Code's `SKILL.md`); text assets ship alongside the skill and are "delivered through the deployment-layer API" — binaries must live in the sandbox image instead.
- Skills are **scope-owned and shareable** (a person's or room's skill can be granted to others), with **org-admin promotion** to make a skill available org-wide.
- Two maintenance skills, `update-qm` and `upstream-pr`, are provided specifically to help an org's private fork stay in sync with upstream QM releases.
- ❓ No separate "slash command" mechanism distinct from skills was confirmed (unlike Claude Code's Skill-vs-slash-command split).

---

## Agent / Subagent Configuration

QM's unit of "agent" is a **scope** — a person or a room (Slack channel/DM or web project) — each with its own memory, files, keychain view, permissions, crons, web apps, and durable sandbox. Multiple scopes can collaborate in the same Slack channel or web project simultaneously ("multi-player projects"), which is the headline architectural difference from the single-user CLI tools elsewhere in this catalog: QM is designed for **multiple humans and multiple agent scopes sharing one live session**, not one human driving one agent.

- Which underlying model harness runs a scope is configurable per deployment via `HARNESS` in `.env` (`pi`, and per-repo harness adapters also cover `codex`, `opencode`, and `claude` — i.e., QM can run Anthropic's Claude Code, OpenAI Codex, or OpenCode as its execution engine, in addition to YC's own "Pi" harness).
- An `admin` skill (seeded, `skills-seed/admin/SKILL.md`) lets an org admin act org-wide from chat — inspecting/changing any scope's config, memory, transcripts, files, roster, audit logs, egress — but only on turns the admin personally started; it's explicitly refused on autonomous runs (crons/webhooks), and admin promotion/revocation itself is portal-only, not agent-token-controlled.
- Background automation exists via **crons and webhook triggers** per scope (`src/triggers/provenance.ts` and related), separate from the human-in-the-loop chat turns.
- **Nested subagents within a single scope are confirmed**, at the harness-adapter level (not documented in prose, but present in source and tests): `src/harness/claude-harness.ts` gates child-agent spawning through an allowlist of `subagent_type` values (tests confirm `research`, `code`, and `consult` are permitted, `general-purpose` is denied); `src/harness/opencode-harness.ts` spawns child runs with `mode: "subagent"`; `src/harness/codex-harness.ts` derives a subagent title by pattern-matching "You are the X subagent" out of the prompt. So a single scope's agent can spawn child agents mid-turn, gated per underlying harness — this is separate from, and nested one level below, the multi-scope "multiplayer" collaboration described above.

---

## Notes

- **Multiplayer is the core differentiator.** Every other tool in this catalog is a single-user CLI/IDE agent; QM is architected so a Slack channel or web project is itself a shared agent workspace multiple humans and the agent(s) act within together, with per-person and per-room scoping of memory/files/permissions layered on top. That changes the trust model (org-admin-gated MCP registration, human-only approval turns, egress allowlisting) compared to a solo developer's local hook/permission config.
- QM is **self-hosted infrastructure**, not a downloadable app — deployment targets are Fly.io or AWS, backed by Postgres, and it explicitly assumes a platform engineer is available.
- Released under YC's own `yc-software` GitHub org on 2026-07-29 (repo creation) / announced 2026-07-31; MIT licensed with noted exceptions. YC states it uses QM internally across accounting, legal, events, and engineering, including using QM to build QM itself.
- This is a **doc-only** pass based on official repository files, docs, and GitHub API metadata — no live deployment, installation, or hands-on testing was performed.

---

## Sources (Official)

| Topic | URL | Fetched (date) | Label |
|-------|-----|-----------------|-------|
| GitHub repo (README, overview, license) | https://github.com/yc-software/qm | 2026-08-15 | [github] |
| Repo metadata (description, license, stars, created date) | https://api.github.com/repos/yc-software/qm | 2026-08-15 | [github] |
| Homepage / announcement | https://qm.ycombinator.com | 2026-08-15 | [official] |
| Getting started doc | https://github.com/yc-software/qm/blob/main/docs/getting-started.md | 2026-08-15 | [github] |
| Deploy directory contract (config files, sandbox tools, skills) | https://github.com/yc-software/qm/blob/main/docs/deploy-directory.md | 2026-08-15 | [github] |
| Security model (postures, command policy, screening) | https://github.com/yc-software/qm/blob/main/SECURITY.md | 2026-08-15 | [github] |
| `.env.example` (config/env vars) | https://github.com/yc-software/qm/blob/main/.env.example | 2026-08-15 | [github] |
| MCP client/admin routes (source) | https://github.com/yc-software/qm/blob/main/src/api/routes/admin/mcp-servers.ts | 2026-08-15 | [github] |
| Admin skill (SKILL.md source) | https://github.com/yc-software/qm/blob/main/skills-seed/admin/SKILL.md | 2026-08-15 | [github] |
| Skills-seed directory listing | https://github.com/yc-software/qm/tree/main/skills-seed | 2026-08-15 | [github] |
