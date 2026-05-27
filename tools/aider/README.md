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

```sh
pip install aider-chat
# Run aider in any git repo
aider
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.aider.conf.yml` | Global | User-level settings |
| `.aider.conf.yml` | Project | Project-level settings |
| `.env` | Project | Environment variables and API keys |
| `CONVENTIONS.md` | Project | Standing instructions and coding style conventions |

## Instruction File

Aider reads persistent, natural-language instructions from a conventions file:
- **`CONVENTIONS.md`** (or `.aider.conventions.md`): Loaded into the context to enforce coding standards, preferred libraries, or style guides.

### Loading Conventions

1. **Automatically via Configuration:**
   Add it to your `.aider.conf.yml` file:
   ```yaml
   read:
     - CONVENTIONS.md
   ```
2. **On Session Start:**
   ```sh
   aider --read CONVENTIONS.md
   ```
3. **During Chat Session:**
   ```text
   /read CONVENTIONS.md
   ```

## Hooks

Aider does not feature a native pre/post tool use hook system like Claude Code or Cursor.

### Git Hooks Integration

- **Skip pre-commit:** Aider runs with `--no-verify` by default to avoid triggering slow pre-commit git hooks during auto-commits.
- **Commit messages:** Automatically generates descriptive commit messages based on the diffs (similar to `commit-msg` hooks).

### Functional Quality Gates (Lint/Test Hooks)

Aider uses functional quality gates via CLI flags/YAML options. When specified, Aider runs these commands automatically after editing files, intercepts any non-zero exit codes, feeds the errors back into the context, and attempts to self-correct the issues.

```yaml
# .aider.conf.yml
lint-cmd: "eslint --fix {files}"
test-cmd: "npm test"
auto-commits: true
```

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

Instead of tool-calling loops, Aider implements standard in-chat slash commands to manage files, context, and environment actions.

| Command | Description |
|---------|-------------|
| `/add <file>` | Add files to the chat session for editing |
| `/drop <file>` | Remove files from the chat session |
| `/read <file>` | Load a file as read-only context (prevents editing, uses prompt caching) |
| `/ls` | List all files in the project and see what's in the chat |
| `/diff` | Show active changes since last commit |
| `/commit` | Manually commit active changes to Git |
| `/run <cmd>` | Run shell command (alias `!`; can optionally send stdout to chat) |
| `/test <cmd>` | Run command, add output to chat only if command fails |
| `/lint` | Run the linter on active files and fix errors |
| `/model <model>` | Switch to another language model |
| `/models <query>`| Search or list available models |
| `/ask <query>` | Ask questions without Aider attempting to edit any files |
| `/clear` | Clear the active chat history |
| `/exit` | Exit the Aider session |

## MCP Support

Aider does not natively support Model Context Protocol (MCP) servers.

*   **Workaround:** Community wrappers (such as `aider-mcp-server`) or external frontends (like `AiderDesk`) can wrap Aider as an MCP client or service.

## Skills / Commands

Aider does not support custom programmable commands or skills directory paths. Standing skills and behavioral workflows are instead defined using declarative Markdown guidelines in the `CONVENTIONS.md` file.

## Agent / Subagent Configuration

Aider is designed as a single-agent pair programmer and does not support autonomous subagent routing, parallel agent spawning, or tree planning.

## Notes

- **Git-Centric Workflow:** Aider operates directly on Git diffs. Every successful model-proposed edit is automatically committed unless `--no-auto-commits` is specified.
- **LiteLLM Integration:** Supports over 100 LLMs (Claude, GPT, Gemini, Ollama, DeepSeek) through LiteLLM.
- **Prompt Caching:** Highly optimized for prompt caching (e.g. Anthropic's prompt caching) to keep large contexts affordable. `/read` is perfect for loading static references.

## Sources (Official)

| Topic | URL |
|-------|-----|
| Main documentation | https://aider.chat/docs/ |
| Configuration options | https://aider.chat/docs/config.html |
| Options reference | https://aider.chat/docs/config/options.html |
| YAML configuration reference | https://aider.chat/docs/config/aider_conf.html |
| Git integration & hooks | https://aider.chat/docs/git.html |
| Hooks request issue | https://github.com/Aider-AI/aider/issues/2045 |
