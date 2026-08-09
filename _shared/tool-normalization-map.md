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

**Adapter status** — of the 26 tools tracked in this repo, Prismor's `runtime/hooks.py` currently ships live adapters for **Claude Code, Codex CLI, Gemini CLI, GitHub Copilot, Cursor, OpenClaw, and Hermes** (7/26 — plus Windsurf and Grok CLI, which aren't tracked in this repo). The other 19 rows below are unmapped in Prismor today; this table is the input for scoping those adapters the way PR #247 scoped Gemini.

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

---

## Cross-agent frequency (how common is each `other:*` sub-tag)

Rough count of distinct tools falling into each `other:*` bucket across all 26 agents — useful for prioritizing which sub-categories deserve their own top-level normalized type if this taxonomy gets extended:

| Sub-tag | Agents where it appears |
|---|---|
| `other:search` (glob/grep/list/semantic search) | nearly all 26 — the single most common non-canonical bucket |
| `other:todo` | Claude Code, Codex, Gemini CLI, Factory Droid, Kiro, Kimi Code, Amazon Q, Amp, Goose, OpenHands, Crush, Continue CLI, Qwen Code, Warp |
| `other:mcp_passthrough` | Factory Droid, Kiro, Amp, OpenHands, Crush, Auggie, Qwen Code, Warp |
| `other:browser` | Codex, Cursor, Hermes, Goose (Computer Controller), OpenHands, Warp |
| `other:elicitation` (ask-user) | Claude Code, Gemini CLI, Cursor, Hermes, Kimi Code, Crush, Continue CLI |
| `other:job_control` (background tasks) | Kiro, Kimi Code, Crush, Continue CLI |
| `other:diagnostics` (LSP/editor) | GitHub Copilot, Kiro, Amp, Crush |
| `other:skill_invoke` | Claude Code, Gemini CLI, Devin CLI, Kimi Code (via `Agent`), Warp |
| `other:cloud_api` (AWS etc.) | Kiro, Amazon Q |
| `other:media_gen` | Codex, Cursor, Hermes |
| `other:reasoning` | Kiro, Kimi Code, Amazon Q |
| `other:vcs` | Aider, Continue CLI |
| `other:messaging` | Hermes, Amazon Q, Amp, Continue CLI, Kimi Code |
| `other:home_automation` | Hermes only |
| `other:third_party_api` (Spotify/Kanban/Yuanbao) | Hermes only |

`other:search` and `other:todo` are the strongest candidates for promotion to first-class canonical types if the taxonomy is revisited — they show up in almost every agent and currently get flattened into the catch-all bucket.

---

## Sources

Built-in tool tables are pulled from each tool's `tools/<name>/README.md` and `_shared/agent-tools-hooks-config.md` in this repo (see those files' own source citations for `[official]`/`[github]`/inferred labeling per tool). Entries marked ❓ above indicate the underlying README itself flagged the tool list as inferred or incomplete — re-verify against the live CLI (`<tool> tools list` where available) before relying on exact names for an integration.

**2026-08-08 sweep of the 19 previously doc-only agents on st3ve:** real installs + static/live inspection (no paid API keys used) were attempted for all 19. Results: **Factory Droid, Amazon Q Dev CLI, OpenHands** fully confirmed with zero corrections needed; **Auggie** and **Google Antigravity** had substantial corrections/additions (Auggie's entire tool list was wrong — see its section); **Amp, Kimi Code, Qwen Code, Aider, Pi Coding Agent, Goose** mostly confirmed with one or two flagged discrepancies each; **OpenCode, Crush, Continue CLI** were installed and probed but yielded no static tool-name list to confirm/deny against (left as-is); **Devin CLI, Kiro CLI** installed cleanly but their compiled binaries don't embed a client-side tool schema, so only partial signal was recovered before an authenticated session would have been required; **Trae, Trae CN, Warp** have no freely downloadable headless CLI at all and were left doc-only per this sweep's scope.
