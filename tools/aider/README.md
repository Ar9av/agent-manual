# Aider

> AI pair programming in your terminal — chat with LLMs to edit code in your local Git repository.

**Vendor:** Aider-AI | **License:** Apache 2.0 | **Runtime:** Python (LiteLLM)

## Links

- Docs: https://aider.chat/
- Configuration Guide: https://aider.chat/docs/config.html
- YAML Config Reference: https://aider.chat/docs/config/aider_conf.html
- Git Integration: https://aider.chat/docs/git.html
- GitHub: https://github.com/Aider-AI/aider

---

## Installation

Recommended methods (in order of preference):

```sh
# Option 1: aider-install (recommended — handles Python env isolation)
python -m pip install aider-install
aider-install

# Option 2: pipx
pipx install aider-chat

# Option 3: uv
uv tool install --force --python python3.12 --with pip aider-chat@latest

# Option 4: One-liner (Mac/Linux)
curl -LsSf https://aider.chat/install.sh | sh

# Option 5: pip (still works, but not recommended for isolation)
pip install -U --upgrade-strategy only-if-needed aider-chat
```

Supported Python versions: 3.8–3.13 (aider-install/one-liners); 3.8–3.13 (uv — provisions its own Python 3.12); 3.9–3.12 (pipx/pip).

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.aider.conf.yml` | Global | User-level settings |
| `.aider.conf.yml` | Project | Project-level settings |
| `.env` | Project | Environment variables and API keys |
| `CONVENTIONS.md` | Project | Standing instructions and coding style conventions (any name, loaded via `--read`) |

## Instruction File

Aider reads persistent, natural-language instructions from a conventions file. The official docs use `CONVENTIONS.md` as the example name, but any file can be used via `--read`. The name `.aider.conventions.md` is **not documented officially** — it is an unofficial convention used in some community guides.

### Loading Conventions

1. **Automatically via Configuration:**
   Add it to your `.aider.conf.yml` file:
   ```yaml
   read: CONVENTIONS.md
   # or multiple files:
   read: [CONVENTIONS.md, anotherfile.txt]
   ```
2. **On Session Start:**
   ```sh
   aider --read CONVENTIONS.md
   ```
3. **During Chat Session:**
   ```text
   /read-only CONVENTIONS.md
   ```

Loading via `--read` / `/read-only` marks the file as read-only and enables prompt caching.

## Hooks

Aider does not feature a native pre/post tool use hook system like Claude Code or Cursor.

### Git Hooks Integration

- **Skip pre-commit:** Aider runs with `--no-verify` by default to avoid triggering slow pre-commit git hooks during auto-commits.
- **Commit messages:** Automatically generates descriptive commit messages based on the diffs (similar to `commit-msg` hooks).

### Functional Quality Gates (Lint/Test)

Aider uses functional quality gates via CLI flags or YAML config. After editing files, Aider runs these commands automatically, intercepts non-zero exit codes, feeds errors back into context, and attempts to self-correct.

```yaml
# .aider.conf.yml
lint-cmd: "eslint"          # filenames are passed as arguments by aider; no {files} placeholder
test-cmd: "npm test"        # run without arguments
auto-lint: true             # default: true
auto-test: false            # default: false; enable with --auto-test flag
```

For language-specific linters, use the `language: cmd` prefix format:
```yaml
lint-cmd:
  - "python: flake8 --select=E9,F63,F7,F82"
  - "javascript: eslint"
```

> **Note on `{files}`:** The `{files}` placeholder is **not documented** in the official lint-cmd reference. Filenames are passed as arguments directly to the lint command. Avoid using `{files}` in `lint-cmd` as its behavior is unverified.

> **Note on formatters:** Formatters that return non-zero exit codes when they make changes (e.g. `black`) can confuse aider. Workaround: run the formatter twice — the second run confirms actual errors vs. formatting changes.

### Script-Level Workaround

To run arbitrary scripts before/after Aider sessions, wrap the execution in a shell script:

```sh
#!/bin/bash
pre_aider() {
  echo "Enforcing pre-session policies..."
}
post_aider() {
  echo "Auditing changes post-session..."
}

pre_aider
aider "$@"
post_aider
```

## Built-in Tools

Instead of tool-calling loops, Aider implements standard in-chat slash commands to manage files, context, and environment actions. The full set of 43 documented commands is listed below.

| Command | Description |
|---------|-------------|
| `/add <file>` | Add files to the chat so aider can edit them or review them in detail |
| `/architect` | Enter architect/editor mode using 2 different models |
| `/ask <query>` | Ask questions about the code base without editing any files |
| `/chat-mode` | Switch to a new chat mode |
| `/clear` | Clear the chat history |
| `/code` | Ask for changes to your code |
| `/commit` | Commit edits to the repo made outside the chat |
| `/context` | Enter context mode to see surrounding code context |
| `/copy` | Copy the last assistant message to the clipboard |
| `/copy-context` | Copy the current chat context as markdown |
| `/diff` | Display the diff of changes since the last message |
| `/drop <file>` | Remove files from the chat session to free up context space |
| `/edit` | Alias for `/editor`: open an editor to write a prompt |
| `/editor` | Open an editor to write a prompt |
| `/editor-model` | Switch the Editor Model to a new LLM |
| `/exit` | Exit the application |
| `/git <cmd>` | Run a git command (output excluded from chat) |
| `/help` | Ask questions about aider |
| `/lint` | Lint and fix in-chat files or all dirty files |
| `/load <file>` | Load and execute commands from a file |
| `/ls` | List all known files and indicate which are included in chat |
| `/map` | Print out the current repository map |
| `/map-refresh` | Force a refresh of the repository map |
| `/model <model>` | Switch the Main Model to a new LLM |
| `/models <query>` | Search the list of available models |
| `/multiline-mode` | Toggle multiline mode |
| `/ok` | Alias for `/code Ok, please go ahead and make those changes (any args are appended)` |
| `/paste` | Paste image/text from the clipboard into the chat |
| `/quit` | Exit the application |
| `/read-only <file>` | Add files to the chat that are for reference only (read-only; enables prompt caching) |
| `/reasoning-effort` | Set the reasoning effort level |
| `/report` | Report a problem by opening a GitHub Issue |
| `/reset` | Drop all files and clear the chat history |
| `/run <cmd>` | Run a shell command and optionally add output to chat (alias: `!`) |
| `/save <file>` | Save commands to a file that can reconstruct the chat session |
| `/settings` | Print out the current settings |
| `/test <cmd>` | Run a shell command and add output on non-zero exit |
| `/think-tokens` | Set the thinking token budget |
| `/tokens` | Report on the number of tokens used |
| `/undo` | Undo the last git commit if done by aider |
| `/voice` | Record and transcribe voice input |
| `/weak-model` | Switch the Weak Model to a new LLM |
| `/web <url>` | Scrape a webpage, convert to markdown and send in a message |

> **Note:** The previous version of this file listed `/read` — the correct command name is `/read-only`.

## MCP Support

Aider does not natively support Model Context Protocol (MCP) servers.

*   **Workaround:** Community wrappers (such as `aider-mcp-server`) or external frontends (like `AiderDesk`) can wrap Aider as an MCP client or service.

## Skills / Commands

Aider does not support custom programmable commands or skills directory paths. Standing skills and behavioral workflows are instead defined using declarative Markdown guidelines loaded via `--read` (e.g. a `CONVENTIONS.md` file).

## Agent / Subagent Configuration

Aider is designed as a single-agent pair programmer and does not support autonomous subagent routing, parallel agent spawning, or tree planning.

## Notes

- **Git-Centric Workflow:** Aider operates directly on Git diffs. Every successful model-proposed edit is automatically committed unless `--no-auto-commits` is specified.
- **LiteLLM Integration:** Supports over 100 LLMs (Claude, GPT, Gemini, Ollama, DeepSeek) through LiteLLM.
- **Prompt Caching:** Highly optimized for prompt caching (e.g. Anthropic's prompt caching) to keep large contexts affordable. `/read-only` is ideal for loading static references.

## Sources

| Topic | URL | Fetched | Label |
|-------|-----|---------|-------|
| Main documentation | https://aider.chat/docs/ | 2026-06-13 | [official] |
| Installation | https://aider.chat/docs/install.html | 2026-06-13 | [official] |
| Slash commands reference | https://aider.chat/docs/usage/commands.html | 2026-06-13 | [official] |
| Lint and test | https://aider.chat/docs/usage/lint-test.html | 2026-06-13 | [official] |
| Coding conventions | https://aider.chat/docs/usage/conventions.html | 2026-06-13 | [official] |
| Configuration options | https://aider.chat/docs/config/options.html | 2026-06-13 | [official] |
| YAML configuration reference | https://aider.chat/docs/config/aider_conf.html | 2026-06-13 | [official] |
| Git integration & hooks | https://aider.chat/docs/git.html | 2026-06-13 | [official] |
| Hooks request issue | https://github.com/Aider-AI/aider/issues/2045 | — | [github] |
| lint-cmd examples request | https://github.com/Aider-AI/aider/issues/909 | — | [github] |
