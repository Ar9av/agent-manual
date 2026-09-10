# Tool Name → Normalized Event Taxonomy

Every agent in this repo names its built-in tools differently (`Bash` vs `run_shell_command` vs `execute_bash` vs `terminal`...), but the *actions* those tools perform collapse into a small, stable set of event kinds. This page is the Rosetta Stone: it maps each agent's native tool identifiers to that canonical taxonomy, so a monitoring/interception layer (hook adapter, audit log, policy engine, etc.) only has to reason about ~8 event kinds instead of 300+ tool names.

Modeled on the normalized event schema used by adapters like `_normalize_claude` / `_normalize_codex` / `_normalize_gemini` (payload → `{type, ...fields}`).

## Canonical types

| Type | Meaning |
|------|---------|
| `prompt` | User (or sub-agent) prompt entering the loop |
| `shell` | Arbitrary command / process execution |
| `file_read` | Read file/dir/image/notebook content |
| `file_write` | Create, overwrite, or edit a file (includes patch/diff apply) |
| `network` | Outbound HTTP — web fetch or web search |
| `memory` | Persist/recall info across sessions (not the transcript itself) |
| `subagent_spawn` | Launch a child/delegate agent |
| `other` | Doesn't collapse cleanly — planning/todo, browser/computer-use, LSP, cloud-provider API calls, MCP passthrough, media generation, messaging, etc. (sub-tagged below) |

Tools whose *only* function is control-flow inside the harness (ask-user, plan-mode toggles, background-job polling) are still listed under `other` for completeness, tagged with what they actually do, since a security/audit layer usually still wants visibility into them even though they aren't file/shell/network primitives.

**Adapter status** — of the 41 tools tracked in this repo, Prismor's `runtime/hooks.py` currently ships live adapters for **Claude Code, Codex CLI, Gemini CLI, GitHub Copilot, Cursor, OpenClaw, and Hermes** (7/41 — plus Windsurf and Grok CLI, which aren't tracked in this repo). The other 34 rows below are unmapped in Prismor today; this table is the input for scoping those adapters the way PR #247 scoped Gemini.

---

## Claude Code — adapter: ✅ `_normalize_claude`

| Native tool | Type | Notes |
|---|---|---|
| `Bash` | `shell` | |
| `Read` | `file_read` | text/image/PDF/Jupyter |
| `Write` | `file_write` | |
| `Edit` | `file_write` | |
| `MultiEdit` | `file_write` | |
| `Glob` | `other:search` | filename pattern match, no content read |
| `Grep` | `other:search` | content search |
| `LS` | `other:search` | directory listing |
| `WebFetch` | `network` | |
| `WebSearch` | `network` | |
| `NotebookRead` | `file_read` | |
| `NotebookEdit` | `file_write` | |
| `TodoRead` / `TodoWrite` | `other:todo` | |
| `AskUserQuestion` | `other:elicitation` | |
| `Agent` | `subagent_spawn` | |
| `Skill` | `other:skill_invoke` | |

## Codex CLI — adapter: ✅ `_normalize_codex`

| Native tool | Type | Notes |
|---|---|---|
| `shell` | `shell` | |
| `read_file` | `file_read` | |
| `write_file` | `file_write` | |
| `apply_patch` | `file_write` | unified diff |
| `exec_command` / `write_stdin` | `shell` | long-lived PTY session |
| `update_plan` | `other:todo` | |
| `multi_tool_use` / `multi_tool_use.parallel` | — | wrapper, not itself an event |
| `web_search` | `network` | |
| `browser` | `other:browser` | |
| `image_generation` | `other:media_gen` | |

## GitHub Copilot CLI / VS Code — adapter: ✅ `_normalize_copilot`

| Native tool | Type | Notes |
|---|---|---|
| `run_in_terminal` | `shell` | |
| `read_file` | `file_read` | |
| `create_file` / `insert_edit_into_file` | `file_write` | |
| `list_directory` | `other:search` | |
| `get_errors` | `other:diagnostics` | editor/LSP |
| `run_tests` | `shell` | invoked as a process |
| `web_search` | `network` | |

## Gemini CLI — adapter: ✅ `_normalize_gemini` (PR #247)

| Native tool | Type | Notes |
|---|---|---|
| `run_shell_command` | `shell` | |
| `read_file` / `read_many_files` | `file_read` | |
| `write_file` | `file_write` | |
| `replace` | `file_write` | in-place text replace |
| `list_directory` / `glob` / `grep_search` | `other:search` | |
| `web_fetch` | `network` | |
| `google_web_search` | `network` | mapped from `query` |
| `save_memory` | `memory` | |
| `ask_user` | `other:elicitation` | |
| `invoke_agent` | `subagent_spawn` | |
| `activate_skill` | `other:skill_invoke` | |
| `enter_plan_mode` | `other:planning` | |
| `update_topic` | `other:session_meta` | |
| `list_background_processes` / `read_background_output` | `other:job_control` | |
| `SessionStart` (hook input, scans `GEMINI.md`/`AGENTS.md`) | `memory` | not a tool call — hook-level normalization |
| `BeforeAgent` / `AfterAgent` | `prompt` | |

## Cursor — adapter: ✅ `_normalize_cursor`

| Native tool | Type | Notes |
|---|---|---|
| `Run shell commands` | `shell` | |
| `Read files` | `file_read` | text + images |
| `Edit files` | `file_write` | |
| `Semantic search` / `Search files and folders` | `other:search` | |
| `Web` | `network` | |
| `Fetch Rules` | `other:context_inject` | pulls `.cursor/rules/*.mdc` |
| `Browser` | `other:browser` | screenshots, navigate, interact |
| `Image generation` | `other:media_gen` | |
| `Ask questions` | `other:elicitation` | |

## OpenClaw — adapter: ✅ `_normalize_openclaw`

| Native tool | Type | Notes |
|---|---|---|
| `bash` | `shell` | |
| `read_file` | `file_read` | |
| `write_file` | `file_write` | |
| `edit_file` | `file_write` | targeted edits |
| `apply_patch` | `file_write` | multi-file unified diff |
| `search` | `other:search` | ripgrep-backed |
| `web_fetch` | `network` | |

## Hermes Agent — adapter: ✅ `_normalize_hermes`

Hermes ships 50+ tools; only the categories that map cleanly are enumerated — the rest are `other`, sub-tagged by category.

| Native tool(s) | Type | Notes |
|---|---|---|
| `terminal`, `process` | `shell` | |
| `read_file`, `search_files` | `file_read` | |
| `write_file`, `patch` | `file_write` | |
| `web_search`, `web_extract`, `x_search` | `network` | |
| `memory` | `memory` | |
| `delegate_task` | `subagent_spawn` | |
| `clarify` | `other:elicitation` | |
| `todo` | `other:todo` | |
| `session_search` | `other:search` | prior-session recall, not filesystem |
| `skill_manage`, `skills_list` | `other:skill_invoke` | |
| `cronjob` | `other:scheduling` | |
| `execute_code` | `shell` | sandboxed code exec |
| `browser_*` (10 tools) | `other:browser` | navigate/click/type/scroll/vision/cdp/console/dialog/get_images |
| `image_generate`, `video_generate`, `video_analyze`, `text_to_speech`, `vision_analyze` | `other:media_gen` | |
| `send_message`, `discord`, `discord_admin`, `feishu_*` | `other:messaging` | |
| `ha_call_service`, `ha_get_state`, `ha_list_entities`, `ha_list_services` | `other:home_automation` | |
| `computer_use` | `other:browser` | full desktop control |
| `spotify_*` (7), `kanban_*` (9), `yb_*` (5) | `other:third_party_api` | |
| `mixture_of_agents` | `subagent_spawn` | fan-out to multiple models |

## Devin CLI — adapter: ❌

No single canonical built-in-tools table is published (docs focus on hooks); inferred from hook/skill docs and general Devin tool behavior. Mark entries below as inferred, not `[official]`.

| Native tool (inferred) | Type | Notes |
|---|---|---|
| shell/terminal exec | `shell` | name not confirmed in public docs |
| file read | `file_read` | |
| file write/edit | `file_write` | |
| skills (`~/.config/devin/skills/`, `.devin/skills/`) | `other:skill_invoke` | |
| subagent profiles (`.devin/agents/`) | `subagent_spawn` | |

**[attempted live-verify st3ib, inconclusive]** `curl -fsSL https://cli.devin.ai/install.sh | bash` installs cleanly with no account needed (installed v3000.3.27 to `~/.local/bin/devin`, 4KB — a small native ELF, not a Node/Python bundle). `devin --help` confirms a `sandbox` subcommand described as "Process sandboxing for the exec tool," which at least confirms the shell tool is internally called **`exec`**, not a bare "shell/terminal" name — worth updating the inferred row above. Actually starting a session requires login (`Error: Login canceled` with no credentials), and per this task's rules no account was created. `strings` on the binary found no `"exec"`/`"read_file"`/`"write_file"`/`"skill"`/`"subagent"` literal matches at all — the binary appears to negotiate its tool schema server-side rather than embedding it client-side, so static inspection couldn't confirm more than the `exec` name above. Rest of the table remains inferred/unconfirmed.

## Factory Droid — adapter: ❌

| Native tool | Type | Notes |
|---|---|---|
| `Execute` | `shell` | [live-verified st3ve] |
| `Read` | `file_read` | [live-verified st3ve] |
| `LS` | `other:search` | [live-verified st3ve] |
| `Grep` | `other:search` | [live-verified st3ve] |
| `Glob` | `other:search` | [live-verified st3ve] |
| `Create` | `file_write` | [live-verified st3ve] |
| `Edit` | `file_write` | [live-verified st3ve] |
| `ApplyPatch` | `file_write` | auto-included with `Edit` for OpenAI models — [live-verified st3ve] |
| `WebSearch` | `network` | [live-verified st3ve] |
| `FetchUrl` | `network` | [live-verified st3ve] |
| `TodoWrite` | `other:todo` | auto-included for all droids — [live-verified st3ve] |
| `mcp__<server>__<tool>` | `other:mcp_passthrough` | dynamic |

**[live-verified st3ve]** Installed via `npm install -g droid` (pulls the real `@factory/cli-linux-x64` native binary). No `tools list`/`--help` subcommand exposes tool names, but every name above was confirmed present as an exact-case string literal via `strings` on the bundled (not-stripped) ELF binary, at counts consistent with real usage (`Execute`×24, `Read`×19, `LS`×12, `Grep`×10, `Glob`×10, `Create`×16, `Edit`×19, `ApplyPatch`×9, `WebSearch`×6, `FetchUrl`×6, `TodoWrite`×12) — the whole table is confirmed accurate as-is.

## Kiro IDE / CLI — adapter: ❌

| Native tool | Type | Notes |
|---|---|---|
| `read` | `file_read` | files, folders, images |
| `write` | `file_write` | |
| `shell` | `shell` | |
| `glob` / `grep` | `other:search` | |
| `web_search` | `network` | |
| `web_fetch` | `network` | |
| `aws` | `other:cloud_api` | AWS CLI calls |
| `code` | `other:diagnostics` | symbol search + LSP |
| `tool_search` | `other:mcp_passthrough` | find/load MCP tools on demand |
| `delegate` | `subagent_spawn` | background async agents |
| `subagent` | `subagent_spawn` | parallel specialized subagents |
| `knowledge` | `memory` | semantic store, cross-session |
| `thinking` | `other:reasoning` | |
| `todo` | `other:todo` | |
| `session` | `other:session_meta` | |
| `introspect` | `other:self_query` | |
| `report` | `other:messaging` | files GitHub issues |
| Hook matcher names: `fs_read`, `fs_write`, `execute_bash`, `use_aws` | (same as above) | internal names differ from the CLI's user-facing tool names |

**[attempted live-verify st3ve, mostly inconclusive]** `curl -fsSL https://cli.kiro.dev/install | bash` installs `kiro-cli` cleanly with no AWS Builder ID needed for the install itself (only for `kiro-cli chat`). `kiro-cli --help`/`--help-all` exposes no `tools list` equivalent. `strings` on the ~109MB binary found only `"name":"aws"` as a confirmed literal tool-name match — the rest of the table could not be confirmed or refuted statically (the binary likely negotiates its tool schema against the AWS backend at runtime rather than embedding it client-side). Left as doc-only/inferred pending an authenticated session, which this task's scope excludes.

## Kimi Code CLI — adapter: ❌

| Native tool | Type | Notes |
|---|---|---|
| `Shell` | `shell` | |
| `ReadFile` | `file_read` | |
| `ReadMediaFile` | `file_read` | images/video, ≤100MB |
| `WriteFile` | `file_write` | |
| `StrReplaceFile` | `file_write` | |
| `Glob` / `Grep` | `other:search` | |
| `SearchWeb` | `network` | |
| `FetchURL` | `network` | |
| `Agent` | `subagent_spawn` | `coder`/`explore`/`plan` subagent types |
| `AskUserQuestion` | `other:elicitation` | |
| `SetTodoList` | `other:todo` | |
| `Think` | `other:reasoning` | |
| `SendDMail` | `other:messaging` | delayed/checkpoint messages |
| `EnterPlanMode` / `ExitPlanMode` | `other:planning` | |
| `TaskList` / `TaskOutput` / `TaskStop` | `other:job_control` | background task management |

**[live-verified st3ve]** Installed via `npm install -g @moonshot-ai/kimi-code`; no `tools list` command, but grepping the bundled `dist/main.mjs` confirmed exact-case literal matches for `Shell`, `ReadFile`, `WriteFile`, `Glob`, `Grep`, `FetchURL`, `Agent`, `AskUserQuestion`, `SetTodoList`, `Think`, `EnterPlanMode`, `ExitPlanMode`, `TaskList`, `TaskOutput`, `TaskStop`. Two rows are ❓ pending further verification: `SearchWeb` was only found internally as `SEARCH_WEB` (different casing — the row's name may be wrong), and `StrReplaceFile` / `SendDMail` were not found under any casing searched (may have been renamed or are gated behind a provider-specific schema not present in the bundled JS).

## Aider — adapter: ❌ (no tool-calling loop)

Aider doesn't have an LLM tool-calling loop at all — it's driven by in-chat slash commands, so there's no `tool_name`/`tool_input` payload to normalize. Listed here for completeness against the taxonomy anyway, since a wrapper script could still intercept these:

| Native command | Type | Notes |
|---|---|---|
| `/run <cmd>` (alias `!`) | `shell` | |
| `/test <cmd>` | `shell` | |
| `/lint` | `shell` | |
| `/read-only <file>` | `file_read` | read-only context — [live-verified st3ve] corrected from `/read`, which is not a real aider command |
| `/add <file>` / `/drop <file>` | `other:context_manage` | not a read/write itself |
| `/commit` | `other:vcs` | git commit |
| `/diff` | `other:vcs` | |
| `/ls` | `other:search` | |
| `/ask <query>` | `prompt` | no-edit query mode |
| `/model <model>` | `other:session_meta` | |

**[live-verified st3ve]** Installed via `pip3 install --user aider-chat`. Grepping `aider/commands.py` for `def cmd_*` confirms `run`, `test`, `lint`, `add`, `drop`, `commit`, `diff`, `ls`, `ask`, `model` all exist as real commands, matching the table. One correction: the read-only-context command's real name is **`/read-only`**, not `/read` — no `cmd_read` function exists, only `cmd_read_only`.

## Pi Coding Agent — adapter: ❌

No published built-in-tools table (docs focus on the `pi-yaml-hooks` package and TS extensions). Hook matcher pattern `tool.before.*` / `tool.after.*` glob-matches on tool name, implying a standard file/shell/network toolset consistent with peers, but names are unconfirmed — mark as ❓ until sourced from `pi.dev` tool docs directly.

**[partial live-verify st3ve]** Installed via `npm install -g --ignore-scripts @earendil-works/pi-coding-agent`. The CLI's own `pi --help` banner literally states: *"pi - AI coding assistant with **read, bash, edit, write** tools"* — confirming a fixed core toolset of at least those four names (lowercase, matching the `tool.before.*`/`tool.after.*` matcher convention). No `tools list` subcommand exists to enumerate beyond this; the rest of the toolset (network/memory/subagent tools, if any) remains unconfirmed.

## OpenCode — adapter: ❌

Docs describe the plugin/hook surface (`tool.execute.before/after`) but not a canonical built-in tool name list — OpenCode's toolset is largely defined via the `tool()` factory + Zod schemas, so tool names are project-specific/dynamic rather than a fixed vendor set. Treat any `tool.execute.*` event's `tool` field as needing per-deployment mapping.

**[attempted live-verify st3ve, inconclusive]** Installed via `npm install -g opencode-ai` (pulls the real `opencode-linux-x64` Bun-compiled binary). `opencode --help` / `opencode debug --help` expose no tool-listing subcommand — `opencode debug v2` ("debug v2 catalog and built-in plugins") printed only an internal Effect-runtime dump, not a tool schema. `strings` on the compiled binary found weak, ambiguous matches for lowercase `bash`/`read`/`write`/`glob`/`list`/`patch` substrings, not reliable enough to assert as confirmed tool names. This live attempt corroborates the doc's own claim that there's no fixed, statically-discoverable built-in tool list — leaving this entry as-is.

## Trae / Trae CN — adapter: ❌ (no native hook system)

No native hook or tool-name API — extensibility is MCP-only, so there's no first-party payload to normalize. Any Prismor integration would have to go through the MCP server layer instead of a hooks adapter (see `_shared/mcp-support.md`).

**Could not install/inspect on st3ve.** Trae ships only as an Electron IDE desktop download from `https://www.trae.ai` (Mac/Windows/Linux `.deb`/`.rpm`) — there is no headless CLI binary, npm package, or curl installer script documented anywhere, and installing a full Electron GUI app just to grep its resources wasn't attempted given the disk budget and the GUI-first nature of the product. Trae CN shares the same codebase/distribution model. Left as doc-only for both.

## Google Antigravity — adapter: ❌

| Native tool | Type | Notes |
|---|---|---|
| `run_command` | `shell` | [live-verified st3ve] |
| `view_file` | `file_read` | [live-verified st3ve] |
| `replace_file_content` | `file_write` | contiguous single-block edit — [live-verified st3ve] |
| `multi_replace_file_content` | `file_write` | non-contiguous multi-block edit — ❓ not found as a literal string; the error message `"file edit must have at least one replacement chunk"` suggests multi-chunk edits may now be handled *inside* `replace_file_content` itself rather than as a separate tool |
| `write_to_file` | `file_write` | new files — [live-verified st3ve] |
| `list_dir` / `grep_search` | `other:search` | [live-verified st3ve] |
| `search_web` | `network` | [live-verified st3ve] (embedded as protobuf field `search_web`) |
| `read_url_content` | `network` | fetch + HTML→markdown — [live-verified st3ve] (found as literal path `read_url_content.proto`) |
| `find_by_name` | `other:search` | undocumented — [live-verified st3ve] found alongside `grep_search`/`view_file`/`list_dir` in a bundled prompt string listing "read-only tools" |
| `codebase_search` | `other:search` | undocumented — [live-verified st3ve] semantic/embedding-based code search, distinct from `grep_search` |
| `delete_knowledge` | `memory` | undocumented — [live-verified st3ve] found in a bundled prompt listing tools as `list_dir`, `view_file`, `write_to_file`, `replace_file_content`, `delete_knowledge` |

**[live-verified st3ve]** The real installed CLI binary is named **`agy`**, not `antigravity`/`gemini-antigravity` — worth noting since the map/README don't currently state the actual binary name. Installed via `curl -fsSL https://antigravity.google/cli/install.sh | bash`; the binary embeds full example tool-call JSON and prompt fragments (found via `strings`), which is how the corrections/additions above were sourced — much stronger evidence than a typical compiled-binary string scan. One ambiguous finding not folded into the table: an embedded example also shows `{"name": "edit_file", "arguments": {...}}` with a `TargetFile`/`CodeEdit` schema resembling `replace_file_content`'s — this may be a legacy/alternate name for the same tool from an older prompt version bundled alongside the current one, not a distinct live tool.

## Amazon Q Developer CLI — adapter: ❌ (unmaintained upstream)

| Native tool | Type | Notes |
|---|---|---|
| `execute_bash` | `shell` | [live-verified st3ve] |
| `fs_read` | `file_read` | files, dirs, images; trusted by default — [live-verified st3ve] |
| `fs_write` | `file_write` | [live-verified st3ve] |
| `use_aws` | `other:cloud_api` | AWS CLI calls — [live-verified st3ve] |
| `knowledge` | `memory` | persistent semantic store — [live-verified st3ve] |
| `delegate` | `subagent_spawn` | background sub-agent tasks — [live-verified st3ve] |
| `todo_list` | `other:todo` | [live-verified st3ve] |
| `thinking` | `other:reasoning` | [live-verified st3ve] |
| `introspect` | `other:self_query` | trusted by default — [live-verified st3ve] |
| `report_issue` | `other:messaging` | opens a browser GitHub issue — [live-verified st3ve] |

**[live-verified st3ve]** Downloaded the headless Linux zip (`q-x86_64-linux.zip`, no account/dpkg needed just to install) and ran `strings` against the extracted `qchat` binary — every single tool name above was found as an exact-case literal match with realistic occurrence counts (5-10 each). Full table confirmed accurate with no corrections needed.

## Amp — adapter: ❌

| Native tool | Type | Notes |
|---|---|---|
| `Bash` | `shell` | [live-verified st3ve] |
| `Read` | `file_read` | [live-verified st3ve] |
| `create_file` | `file_write` | [live-verified st3ve] |
| `edit_file` | `file_write` | [live-verified st3ve] |
| `undo_edit` | `file_write` | reverts an edit — ❓ not found as a literal string in the current binary; may have been removed/renamed since this was documented |
| `Grep` / `glob` / `finder` | `other:search` | `finder` = multi-step codebase search — [live-verified st3ve] |
| `read_web_page` | `network` | [live-verified st3ve] |
| `web_search` | `network` | [live-verified st3ve] |
| `read_mcp_resource` | `other:mcp_passthrough` | [live-verified st3ve] |
| `Task` | `subagent_spawn` | [live-verified st3ve] |
| `oracle` | `other:review` | code review / design feedback tool — [live-verified st3ve] |
| `todo_read` / `todo_write` | `other:todo` | [live-verified st3ve] |
| `get_diagnostics` | `other:diagnostics` | [live-verified st3ve] — no longer unconfirmed |

**[live-verified st3ve]** Installed via `curl -fsSL https://ampcode.com/install.sh | bash` (real native binary, not stripped). `strings` confirmed every row above except `undo_edit`, which had zero matches — worth re-checking against current Amp release notes since it may have been folded into `edit_file`'s own history/undo mechanism.

## Goose — adapter: ❌

| Native tool | Type | Notes |
|---|---|---|
| `shell` | `shell` | [live-verified — see `tools/goose/README.md`'s 2026-07-23 live hook test: confirmed the real `tool_name` payload is bare `shell`, not the docs' own stale `developer__shell`] |
| `write` | `file_write` | |
| `edit` | `file_write` | targeted text replace |
| `tree` | `other:search` | directory tree w/ line counts |
| `read_image` | `file_read` | screenshots/diagrams for model inspection |
| Computer Controller extension | `other:browser` | web scraping + desktop automation |
| Memory extension | `memory` | |
| Chat Recall extension | `other:search` | searches prior conversation history |
| Todo extension | `other:todo` | |

## OpenHands — adapter: ❌

| Native tool | Type | Notes |
|---|---|---|
| `TerminalTool` | `shell` | [live-verified st3ve] |
| `FileEditorTool` | `file_read` + `file_write` | single tool covers read, write, and edit — [live-verified st3ve] |
| `TaskTrackerTool` | `other:todo` | [live-verified st3ve] |
| `BrowserToolSet` | `other:browser` | navigate/click/fill/extract — [live-verified st3ve] |
| MCP tools (`mcp_config`) | `other:mcp_passthrough` | auto-discovered |

**[live-verified st3ve]** Installed via `uv tool install openhands --python 3.12`. `grep -rl 'class (TerminalTool|FileEditorTool|TaskTrackerTool|BrowserToolSet)'` against the installed `openhands` package found each class defined in its own module (`openhands/tools/terminal/definition.py`, `openhands/tools/file_editor/definition.py`, `openhands/tools/task_tracker/definition.py`, `openhands/tools/browser_use/definition.py`) — full table confirmed accurate.

## Crush — adapter: ❌

| Native tool | Type | Notes |
|---|---|---|
| `bash` | `shell` | background job support |
| `view` | `file_read` | |
| `write` | `file_write` | |
| `edit` / `multiedit` | `file_write` | |
| `ls` / `glob` / `grep` (`rg`) | `other:search` | |
| `fetch` | `network` | |
| `download` | `network` | file download |
| `sourcegraph` | `network` | public code search via Sourcegraph API |
| `question` | `other:elicitation` | |
| `todos` | `other:todo` | |
| `job_kill` / `job_output` | `other:job_control` | manage background `bash` jobs |
| `diagnostics` / `references` / `lsp_*` (6 tools) | `other:diagnostics` | LSP-powered code intelligence |
| `list_mcp_resources` / `read_mcp_resource` | `other:mcp_passthrough` | |
| `crush_info` / `crush_logs` | `other:self_query` | |

**[partial live-verify st3ve]** Installed via `npm install -g @charmland/crush` (pulls the real Go binary from GitHub releases). No `tools list` subcommand exists — `crush --help` only shows top-level commands (`run`, `session`, `models`, `logs`, `server`, etc.), and `crush schema` dumps the *config* JSON Schema, not a tool/action schema. Ran out of time to grep the compiled Go binary for tool-name literals; no contradiction found in what was checked, so the table is left as-is.

## Continue CLI — adapter: ❌

| Native tool | Type | Notes |
|---|---|---|
| `Bash` | `shell` | |
| `Read` | `file_read` | |
| `List` | `other:search` | |
| `Search` | `other:search` | |
| `Edit` / `MultiEdit` | `file_write` | |
| `Write` | `file_write` | |
| `Fetch` | `network` | |
| `Diff` | `other:vcs` | |
| `AskQuestion` | `other:elicitation` | |
| `Checklist` | `other:todo` | |
| `Status` | `other:session_meta` | |
| `CheckBackgroundJob` | `other:job_control` | |
| `ReportFailure` | `other:messaging` | |
| `UploadArtifact` | `other:messaging` | |

**[partial live-verify st3ve]** Installed via `npm i -g @continuedev/cli` (binary `cn`). `cn --help` confirms `--readonly` (plan/read-only mode) and `--auto` (all-tools mode) flags matching the general permission model implied by the table, but exposes no `tools list`/tool-name enumeration. Ran out of time to grep the bundled JS for the exact tool-name literals (`Bash`, `Read`, `List`, etc.); no contradiction found in what was checked, table left as-is.

## Auggie CLI — adapter: ❌

**[live-verified st3ve, table corrected]** `auggie tools list` is a real, working, no-auth-required command (`npm install -g @augmentcode/auggie`, no login needed to list — only warns about missing MCP server config). It returned this authoritative list of 20 built-in tools, which is materially different from the doc-sourced guess previously in this row — in particular there is **no tool literally named `terminal`, `read`, or `edit`**:

| Native tool | Type | Notes |
|---|---|---|
| `remove-files` | `file_write` | [live-verified st3ve] delete |
| `save-file` | `file_write` | [live-verified st3ve] |
| `apply_patch` | `file_write` | [live-verified st3ve] |
| `str-replace-editor` | `file_write` | [live-verified st3ve] — this is the real name; the doc's `edit` was a guess |
| `view` | `file_read` | [live-verified st3ve] — this is the real name; the doc's `read` was a guess |
| `launch-process` / `kill-process` / `read-process` / `write-process` / `list-processes` | `shell` | [live-verified st3ve] process management group — no tool is literally named `terminal` |
| `web-fetch` | `network` | [live-verified st3ve] |
| `view-session` | `other:session_meta` | [live-verified st3ve] undocumented — views a prior session's output |
| `codebase-retrieval-raw` | `other:search` | [live-verified st3ve] undocumented — semantic codebase search |
| `view_tasklist` / `reorganize_tasklist` / `update_tasks` / `add_tasks` | `other:todo` | [live-verified st3ve] undocumented task-management group |
| `grep-search` | `other:search` | [live-verified st3ve] |
| `view-range-untruncated` / `search-untruncated` | `other:search` | [live-verified st3ve] undocumented — untruncated variants of view/search |
| `{tool}_{server}` (MCP) | `other:mcp_passthrough` | not in the base 20; dynamic per configured MCP server |

Note: **no `web-search` tool appeared** in the live list (only `web-fetch`) — the doc's `web-search` row may be stale or gated behind a flag/plan not enabled in this environment.

## Qwen Code — adapter: ❌

Docs describe categories rather than exhaustive tool ids; individual `read_file`/`write_file`/`edit_file` names are inferred from hook-matcher examples, not a canonical table.

| Native tool | Type | Notes |
|---|---|---|
| `read_file` / `write_file` | `file_read` / `file_write` | rootDirectory-scoped — [live-verified st3ve] |
| edit tool | `file_write` | ❓ corrected — `edit_file` was **not** found anywhere in the installed package; a bundled chunk file is literally named `edit-ITMZYFKA.js` and contains only the bare string `"edit"` as the operative tool-ish identifier, suggesting the real name may just be `edit`, not `edit_file`. Not confirmed with full certainty (schema-level `name` field wasn't isolated), so treat as a lead, not a final answer |
| `read_many_files` | `file_read` | [live-verified st3ve] |
| `run_shell_command` | `shell` | [live-verified st3ve] |
| `web_fetch` | `network` | fetches + processes with a model — [live-verified st3ve] |
| `web_search` | `network` | [live-verified st3ve] |
| `save_memory` | `memory` | [live-verified st3ve] |
| `todo_write` | `other:todo` | [live-verified st3ve] (also appears as its own bundled chunk `todoWrite-JGV6VLOX.js`) |
| MCP-provided tools | `other:mcp_passthrough` | |

**[live-verified st3ve]** Installed via `npm install -g @qwen-code/qwen-code@latest`. Grepped the installed package's `chunks/*.js` for each tool-name literal; all rows above except the edit tool were confirmed as exact string matches across 3-6 separate chunk files each (consistent with real, actively-referenced tool names). `edit_file` itself had zero matches anywhere in the package — see the correction above.

## Warp (Agent Mode) — adapter: ❌ (no hook system; permissions/rules instead)

| Native capability | Type | Notes |
|---|---|---|
| Terminal command execution | `shell` | |
| Full Terminal Use | `shell` | drives interactive PTY apps (psql, vim, REPLs, `git rebase -i`) |
| Computer Use | `other:browser` | desktop GUI control |
| File search / code search | `other:search` | grep/glob-style |
| Codebase Context | `other:search` | semantic index over Git-tracked files |
| Web Search | `network` | |
| Planning / Task Lists | `other:planning` | |
| Skills | `other:skill_invoke` | |
| Slash Commands | `other:session_meta` | |
| MCP tools | `other:mcp_passthrough` | |

**Could not fully install/inspect on st3ve.** Warp is a GUI-first Electron-style desktop app (`brew install --cask warp` on macOS; Linux `.deb`/`.rpm`/AppImage per the docs, but no anonymous downloadable Linux artifact was found from the box). The headless `oz` CLI is documented as "bundled alongside the Warp app install" or via `brew install --cask oz` on macOS, with only a vague "Linux: apt/yum/pacman packages available" and no actual repo URL or package name given — `apt-cache search warp`/`which oz` on Ubuntu 24.04 found nothing installable. Left as doc-only; this matches the task's expectation that Warp is one of the GUI-first/enterprise products without a freely reachable headless CLI path.

## Cline — adapter: ❌

Two naming generations coexist: the classic XML-style extension tools, and the newer `ClineCore` SDK names. Both map to the same canonical types.

| Native tool (classic) | SDK equivalent | Type | Notes |
|---|---|---|---|
| `execute_command` | `bash` | `shell` | |
| `read_file` | `read_files` | `file_read` | |
| `write_to_file` | `apply_patch` | `file_write` | create/overwrite |
| `replace_in_file` | `apply_patch` | `file_write` | diff-style edit — docs say `apply_patch` supersedes both classic write tools |
| — | `editor` | `file_write` | SDK-only; no documented classic equivalent |
| `search_files` | `search` | `other:search` | regex across files |
| `list_files` | ❓ (`search`?) | `other:search` | |
| `list_code_definition_names` | ❓ | `other:search` | top-level symbol listing |
| — | `fetch_web` | `network` | no classic XML equivalent documented |
| `use_mcp_tool` / `access_mcp_resource` | — | `other:mcp_passthrough` | |
| `ask_followup_question` | `ask_question` | `other:elicitation` | |
| `use_skill` | — | `other:skill_invoke` | |
| `use_subagents` | — | `subagent_spawn` | parallel read-only research agents |

❓ The docs confirm `read_files`/`apply_patch`/`bash` supersede the classic names, but never publish a full 1:1 map for the search/list family — a normalizer should accept both spellings.

## Junie — adapter: ❌

Claude-Code-shaped names (assembled from the subagent tool-groups reference, not a single canonical table).

| Native tool | Type | Notes |
|---|---|---|
| `Bash` | `shell` | approval-gated by the Action Allowlist |
| `Read` | `file_read` | |
| `Write` | `file_write` | |
| `Edit` | `file_write` | search/replace + patches |
| `Glob` | `other:search` | |
| `Grep` | `other:search` | |
| `WebSearch` | `network` | |
| `AskUserQuestion` | `other:elicitation` | |
| MCP server tools | `other:mcp_passthrough` | |

❓ No official enumerated tool table; names above are inferred from the tool-groups docs.

## Grok Build — adapter: ❌

No enumerated tool names published — only capability categories. Map by category until names are sourced.

| Native capability | Type | Notes |
|---|---|---|
| Terminal / shell execution | `shell` | |
| File edit (read/write/patch) | `file_read` + `file_write` | |
| Search | `other:search` | |
| Workspace ops (filesystem, git, checkpoints) | `other:vcs` | |
| MCP tools (`<server>__<tool>`) | `other:mcp_passthrough` | double-underscore namespacing is the reliable discriminator |

❓ Whether the built-ins are literally named `Read`/`Edit`/`Bash` (Claude-Code style) is unconfirmed.

## jcode — adapter: ❌

| Native tool | Type | Notes |
|---|---|---|
| `bash` / shell execution | `shell` | |
| `run_in_background` | `other:job_control` | long-running processes with progress monitoring |
| File read/write/edit | `file_read` + `file_write` | exact tool names ❓ |
| Agent grep | `other:search` | grep + file-structure info, adaptive truncation |
| Todo management | `other:todo` | includes confidence / "hill-climbability" scoring |
| `browser` | `other:browser` | Firefox Agent Bridge; `open`/`click`/`type`/`fill_form`/`screenshot`/`eval`/… |
| Memory tools | `memory` | store/retrieve/search over the semantic memory graph |
| Session search (RAG over past sessions) | `other:search` | |
| Swarm tool | `subagent_spawn` | spawn workers + DM/broadcast/channels — also `other:messaging` |
| `request_permission` | `other:elicitation` | ambient-mode Tier-2 approval |
| Image generation / rendering | `other:media_gen` | |
| LaTeX / Mermaid rendering | `other:media_gen` | display-only, no side effects |

❓ No canonical tool-list page exists; table assembled from README + scattered docs pages.

## Muse Code — adapter: ❌

Capability-level only — Meta's docs describe behavior, not a tool schema.

| Native capability | Type | Notes |
|---|---|---|
| Shell/command execution | `shell` | sandboxed (Seatbelt/bubblewrap); approval modes `on-request`/`untrusted`/`never` |
| File edit/write | `file_write` | exact name(s) ❓ |
| Search grounding | `network` | live web access |
| Computer use | `other:browser` | scope ❓ |
| Subagent fan-out | `subagent_spawn` | isolated git worktrees under `.muse/worktrees/` |
| MCP tools (`mcp_servers` block) | `other:mcp_passthrough` | |

❓ No page enumerates a discrete tool list — treat every row as inferred from prose.

## DeepSeek Harness — adapter: ❌

The only agent here that publishes a generated tool-schema catalog (`en/reference/tool-catalog`), so names are high-confidence.

| Native tool | Type | Notes |
|---|---|---|
| `bash`, `pwsh` | `shell` | PowerShell variant has local + sandboxed configs |
| `terminal_open` / `terminal_close` / `terminal_list` / `terminal_read` / `terminal_send` / `terminal_signal` | `shell` | persistent PTY sessions, owner-scoped |
| `read`, `read_image` | `file_read` | |
| `write`, `edit`, `str_replace_editor` | `file_write` | |
| `glob`, `grep` | `other:search` | |
| `web_search`, `web_fetch` | `network` | |
| `run_code` | `shell` | sandboxed worker-thread execution |
| `lsp` | `other:diagnostics` | |
| `job_list` / `job_output` / `job_kill` | `other:job_control` | |
| `subagent`, `subagent_fork` | `subagent_spawn` | |
| `send_message`, `interrupt_agent`, `list_agents`, `report` | `other:messaging` | multi-agent coordination |
| `workflow`, `ralph` | `other:workflow` | worker-thread workflow engine |
| `session_search` / `session_trace` / `session_event_read` / `session_event_search` / `session_event_trace` | `other:session_meta` | introspects prior sessions |
| `create_goal` / `get_goal` / `update_goal` / `todo_write` | `other:todo` | |
| `schedule_create` / `schedule_delete` / `schedule_list` | `other:scheduling` | |
| `skill` | `other:skill_invoke` | |
| `ask_user_question`, `exit_plan_mode` | `other:elicitation` / `other:planning` | |
| `cordis_define` / `cordis_undefine` / `cordis_run` / `cordis_stop` / `cordis_inspect_*` | `other:dynamic_tooling` | creator-mode: defines/runs new tools at runtime — the highest-risk bucket for a policy layer |
| MCP client tools | `other:mcp_passthrough` | registered into `ctx.tools` |

**Mode matters:** in **Minimal** mode only `bash` + `str_replace_editor` exist; **Code Mode** exposes tools via an SDK rather than model tool-calls, so a hook adapter sees a different (or empty) tool stream.

## Kilo Code — adapter: ❌

| Native tool | Type | Notes |
|---|---|---|
| `bash` | `shell` | configurable timeout/directory |
| `read` | `file_read` | with line numbers |
| `write` | `file_write` | create / full replace |
| `edit` | `file_write` | targeted replacement |
| `apply_patch` | `file_write` | unified diff |
| `glob`, `grep` | `other:search` | |
| `webfetch`, `websearch` | `network` | search via Exa or Parallel |
| `kilo-playwright_browser_navigate` / `_click` / `_type` / `_screenshot` / `_snapshot` | `other:browser` | |
| `question` | `other:elicitation` | selectable response options |
| `task` | `subagent_spawn` | |
| `todowrite` / `todoread` | `other:todo` | |
| `plan` | `other:planning` | |
| `skill` | `other:skill_invoke` | |
| `agent_manager` | `other:session_meta` | VS Code sessions/worktrees |
| `{server}_{tool}` | `other:mcp_passthrough` | single-underscore namespacing |

## QM — adapter: ❌ (no hook system; security postures instead)

| Native tool | Type | Notes |
|---|---|---|
| `execute` | `shell` | **the only fixed tool** — runs commands in the scope's durable sandbox |
| Sandbox tools (`sandbox/tools/<id>/tool.json`) | `shell` | advertised as installed CLIs, invoked *through* `execute` |
| Connectors / skills (Slack, Drive, GitHub, Linear, cloud CLIs…) | `other:third_party_api` | |
| MCP tools (org-admin registered) | `other:mcp_passthrough` | |

QM deliberately collapses the whole toolset into one `shell` event — everything else is a CLI inside the sandbox. For a policy layer this inverts the usual problem: there is nothing to normalize at the tool-name level, and all the signal lives in the *command string* passed to `execute`. Note also that QM runs Claude Code / Codex / OpenCode / Pi as its underlying harness (`HARNESS` in `.env`), so the inner harness's own tool names may surface one layer down.

## oh-my-pi (omp) — adapter: ❌

✅ = observed live on st3ve in a `--mode json` run (2026-09-10); the rest are from the documented tool listing.

| Native tool | Type | Notes |
|---|---|---|
| `bash` | `shell` | ✅ PTY-based interactive by default; `--no-pty` disables |
| `read` | `file_read` | ✅ also resolves 16 internal schemes (`pr://`, `issue://`, `agent://`, `skill://`, `ssh://`, …) through the same FS-shaped tool |
| `write` | `file_write` | ✅ |
| `grep` | `other:search` | ✅ also walks a diff like a directory |
| `todo` | `other:todo` | ✅ |
| `task` | `subagent_spawn` | parallel fan-out, optionally workspace-isolated |
| `web_search` | `network` | one query across configured providers, returns answer plus citations |
| `learn` | `memory` | capture a reusable lesson; can promote it into a managed skill |
| `manage_skill` | `other:skill_invoke` | create/update/delete an isolated managed skill |
| `orchestrate` | `subagent_spawn` | parallel subagents with per-phase verification |
| `workflowz` | `subagent_spawn` | deterministic multi-subagent workflow over `task` |
| LSP ops (14) | `other:lsp` | disabled together with `--no-lsp` |
| DAP ops (28) | `other:debug` | debug-adapter operations — **unique to omp in this repo** |

❓ The README advertises 31 built-in tools but publishes no closed table; the 11 named above plus the LSP/DAP families are what is documented. `--tools=<list>` allowlists, `--no-tools` disables all built-ins.

## Codewhale — adapter: ❌

| Native tool | Type | Notes |
|---|---|---|
| `bash` | `shell` | ✅ observed live in `exec --auto --output-format stream-json` |
| `read` | `file_read` | ✅ |
| `write` | `file_write` | ✅ |
| `exec_shell` | `shell` | named in the hooks doc as the target of the `shell_env` hook and `tool_name` conditions |

❓ **Two spellings for the shell tool.** The hooks documentation gates on `exec_shell`; the live stream-json run reported `bash`. A normalizer should accept both until a canonical tool table is published — and a hook condition written as `{ type = "tool_name", name = "exec_shell" }` may not match what the engine actually emits.

Note for adapter authors: Codewhale hooks fire **only in the interactive TUI**. `codewhale exec`, the CLI dispatcher, app-server, and ACP fire nothing, so a hook-based adapter gets zero coverage of the headless path.

## Reasonix — adapter: ❌

✅ = observed live in session JSONL on st3ve (2026-09-10).

| Native tool | Type | Notes |
|---|---|---|
| `bash` | `shell` | ✅ refused entirely when `[sandbox] bash = "enforce"` and no OS sandbox is available |
| `read_file` | `file_read` | ✅ |
| `write_file` | `file_write` | ✅ |
| `use_capability` | `other:skill_invoke` | ✅ capability/skill dispatch |
| `edit_file` | `file_write` | named in the sandbox docs as part of the file-writer set |
| `multi_edit` | `file_write` | as above |
| `move_file` | `file_write` | as above |
| MCP plugin tools | `other:mcp_passthrough` | `[[plugins]]` entries contribute tools, prompts, and resources |

Permission rules are Claude-Code-shaped (`Tool` / `Tool(specifier)`, e.g. `Bash(go test:*)`, `Edit(src/**)`) with precedence **deny > ask > allow > fallback** — note the rule names are capitalized (`Bash`, `Edit`) while the emitted tool names are snake_case (`bash`, `write_file`).

⚠️ `reasonix run --events-jsonl` did **not** expose tool-name fields in the live run; the redacted event stream omitted what the session JSONL recorded. An adapter should read sessions, not the events stream, for tool identity.

## MiMo Code — adapter: ❌

✅ = observed live via `mimo export <session-id>` on st3ve (2026-09-10).

| Native tool | Type | Notes |
|---|---|---|
| `exec` | `shell` | ✅ |
| `exec_command` | `shell` | ✅ command form |
| `apply_patch` | `file_write` | ✅ patch/diff apply |
| MCP server tools | `other:mcp_passthrough` | OAuth-capable servers via `mimo mcp auth` |

❓ No enumerated tool table is published. Notably, a five-part task (list dir, grep, read, write, shell) emitted **only these three names** — MiMo routes file reads and searches through shell execution and performs writes through `apply_patch`, rather than exposing distinct `read`/`grep` tools. There is no `file_read` event to gate; reads surface as `shell`.

## Prime Agent — adapter: ❌

| Native tool | Type | Notes |
|---|---|---|
| `ipython` | `shell` | ✅ **the only tool** — a persistent Python REPL |

> ⚠️ **This row breaks the premise of the table.** Prime Agent's built-in tool *is* a persistent Python REPL. File operations, shell commands, MCP tool use, subagent spawning (`rlm(...)`), and context management all happen as Python code inside it. A five-part task that in every other harness emitted 4–5 distinct tool names emitted exactly one here.

For an interception layer this means:

- there is no `file_write`, `file_read`, `network`, or `subagent_spawn` event to gate;
- the entire signal lives in the **code string** passed to `ipython`, which must be parsed (not pattern-matched on a tool name) to recover intent;
- tool-name allowlisting is effectively inert — `-t/--tools ipython` permits everything.

This is the same inversion as QM (which collapses everything into `execute`), but at the language level rather than the CLI level, and it is harder: a shell command string is far easier to classify than arbitrary Python. Pair it with the vendor's own warning that the worker/kernel processes are **not** a security sandbox.

## Tau — adapter: ❌

✅ = observed live in session JSONL on st3ve (2026-09-10).

| Native tool | Type | Notes |
|---|---|---|
| `bash` | `shell` | ✅ |
| `read` | `file_read` | ✅ |
| `write` | `file_write` | ✅ |

The minimal Pi-lineage core — the same three primitives as omp without the `grep`/`todo`/`task` layer. ❓ No MCP surface exists in 0.4.2, so there is no `other:mcp_passthrough` row.

⚠️ Session records carry both `"name"` and `"toolName"` spellings for the same call; a normalizer should read either.

---

## Cross-agent frequency (how common is each `other:*` sub-tag)

Counted mechanically from the sections above — how many of the 34 agents have at least one tool in each `other:*` bucket. Useful for prioritizing which sub-categories deserve their own top-level normalized type if this taxonomy gets extended.

| Sub-tag | Agents where it appears |
|---|---|
| `other:search` (glob/grep/list/semantic search) | **24** — Claude Code, GitHub Copilot CLI / VS Code, Gemini CLI, Cursor, OpenClaw, Hermes Agent, Factory Droid, Kiro IDE / CLI, Kimi Code CLI, Aider, Google Antigravity, Amp, Goose, Crush, Continue CLI, Auggie CLI, Warp (Agent Mode), Cline, Junie, Grok Build, jcode, DeepSeek Harness, Kilo Code, oh-my-pi |
| `other:todo` (task lists) | **18** — Claude Code, Codex CLI, Hermes Agent, Factory Droid, Kiro IDE / CLI, Kimi Code CLI, Amazon Q Developer CLI, Amp, Goose, OpenHands, Crush, Continue CLI, Auggie CLI, Qwen Code, jcode, DeepSeek Harness, Kilo Code, oh-my-pi |
| `other:mcp_passthrough` (MCP server tools) | **15** — Factory Droid, Kiro IDE / CLI, Amp, OpenHands, Crush, Auggie CLI, Qwen Code, Warp (Agent Mode), Cline, Junie, Grok Build, Muse Code, DeepSeek Harness, Kilo Code, QM |
| `other:elicitation` (ask-user) | **12** — Claude Code, Gemini CLI, Cursor, Hermes Agent, Kimi Code CLI, Crush, Continue CLI, Cline, Junie, jcode, DeepSeek Harness, Kilo Code |
| `other:browser` (browser / computer use) | **9** — Codex CLI, Cursor, Hermes Agent, Goose, OpenHands, Warp (Agent Mode), jcode, Muse Code, Kilo Code |
| `other:session_meta` (session & workspace introspection) | **8** — Gemini CLI, Kiro IDE / CLI, Aider, Continue CLI, Auggie CLI, Warp (Agent Mode), DeepSeek Harness, Kilo Code |
| `other:skill_invoke` (load/run a skill) | **8** — Claude Code, Gemini CLI, Hermes Agent, Devin CLI, Warp (Agent Mode), Cline, DeepSeek Harness, Kilo Code |
| `other:messaging` (send messages to humans/agents) | **7** — Hermes Agent, Kiro IDE / CLI, Kimi Code CLI, Amazon Q Developer CLI, Continue CLI, jcode, DeepSeek Harness |
| `other:job_control` (background tasks) | **6** — Gemini CLI, Kimi Code CLI, Crush, Continue CLI, jcode, DeepSeek Harness |
| `other:diagnostics` (LSP/editor errors) | **5** — GitHub Copilot CLI / VS Code, Kiro IDE / CLI, Amp, Crush, DeepSeek Harness |
| `other:planning` (plan mode) | **5** — Gemini CLI, Kimi Code CLI, Warp (Agent Mode), DeepSeek Harness, Kilo Code |
| `other:media_gen` (image/video/audio generation) | **4** — Codex CLI, Cursor, Hermes Agent, jcode |
| `other:reasoning` (explicit think/reason tool) | **3** — Kiro IDE / CLI, Kimi Code CLI, Amazon Q Developer CLI |
| `other:self_query` (agent introspects its own config) | **3** — Kiro IDE / CLI, Amazon Q Developer CLI, Crush |
| `other:vcs` (git operations) | **3** — Aider, Continue CLI, Grok Build |
| `other:cloud_api` (AWS etc.) | **2** — Kiro IDE / CLI, Amazon Q Developer CLI |
| `other:scheduling` (cron/scheduled runs) | **2** — Hermes Agent, DeepSeek Harness |
| `other:third_party_api` (Spotify/Kanban/connectors) | **2** — Hermes Agent, QM |
| `other:context_inject` (pulls rules/context files) | **1** — Cursor |
| `other:context_manage` (add/drop files from context) | **1** — Aider |
| `other:dynamic_tooling` (defines new tools at runtime) | **1** — DeepSeek Harness |
| `other:home_automation` (Home Assistant) | **1** — Hermes Agent |
| `other:review` (code review / design feedback) | **1** — Amp |
| `other:workflow` (workflow engine) | **1** — DeepSeek Harness |

`other:search` and `other:todo` are the strongest candidates for promotion to first-class canonical types — they show up in 24 and 18 of the 41 agents respectively and currently get flattened into the catch-all bucket. `other:mcp_passthrough` is a close third, but it's a passthrough wrapper rather than a distinct action, so it arguably belongs as a *flag* on the inner tool's event rather than a type of its own.

Counts exclude agents whose docs publish no enumerated tool list at all (OpenCode, Trae/Trae CN, Pi Agent), so a bucket's real reach is a floor, not a ceiling. The six agents added 2026-09-10 (Codewhale, Reasonix, oh-my-pi, MiMo Code, Prime Agent, Tau) contribute observed-live names rather than published tables, so their rows are a floor too — a task that never needed a given tool never revealed its name.

---

## Sources

Built-in tool tables are pulled from each tool's `tools/<name>/README.md` and `_shared/agent-tools-hooks-config.md` in this repo (see those files' own source citations for `[official]`/`[github]`/inferred labeling per tool). Entries marked ❓ above indicate the underlying README itself flagged the tool list as inferred or incomplete — re-verify against the live CLI (`<tool> tools list` where available) before relying on exact names for an integration.

**2026-08-08 sweep of the 19 previously doc-only agents on st3ve:** real installs + static/live inspection (no paid API keys used) were attempted for all 19. Results: **Factory Droid, Amazon Q Dev CLI, OpenHands** fully confirmed with zero corrections needed; **Auggie** and **Google Antigravity** had substantial corrections/additions (Auggie's entire tool list was wrong — see its section); **Amp, Kimi Code, Qwen Code, Aider, Pi Coding Agent, Goose** mostly confirmed with one or two flagged discrepancies each; **OpenCode, Crush, Continue CLI** were installed and probed but yielded no static tool-name list to confirm/deny against (left as-is); **Devin CLI, Kiro CLI** installed cleanly but their compiled binaries don't embed a client-side tool schema, so only partial signal was recovered before an authenticated session would have been required; **Trae, Trae CN, Warp** have no freely downloadable headless CLI at all and were left doc-only per this sweep's scope.
