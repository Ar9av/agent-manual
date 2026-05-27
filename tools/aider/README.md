# Aider

> AI pair programming in your terminal — chat with LLMs to edit code in your local repo.

**Vendor:** Aider-AI | **License:** Apache 2.0 | **Runtime:** Python

## Links

- Docs: https://aider.chat/docs/
- Configuration: https://aider.chat/docs/config.html
- Options reference: https://aider.chat/docs/config/options.html
- YAML config: https://aider.chat/docs/config/aider_conf.html
- Git integration: https://aider.chat/docs/git.html
- GitHub: https://github.com/Aider-AI/aider

---

## Installation

```sh
pip install aider-chat
aider
```

## Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.aider.conf.yml` | Global | User settings |
| `.aider.conf.yml` | Project | Project settings |
| `.env` | Project | Environment variables |
| `CONVENTIONS.md` | Project | Coding conventions loaded into context |

## Hooks

> **Note:** Aider does not have a native pre/post tool use hook system as of 2026. A hook feature has been requested (issue #2045) but is not yet implemented.

### Git Hooks (Available)

Aider integrates with standard Git hooks:

- **`pre-commit`**: Aider can run linters/formatters via `--lint-cmd` before committing.
- **`commit-msg`**: Aider generates AI commit messages automatically.

```yaml
# .aider.conf.yml
lint-cmd: "flake8 {files}"
format-cmd: "black {files}"
auto-commits: true
dirty-commits: false
```

### Script-Level Workarounds

Since Aider lacks native hooks, common patterns:

```sh
#!/bin/bash
# Wrapper script for pre/post logic
pre_aider_hook() { echo "Running pre-hook..." }
post_aider_hook() { echo "Running post-hook..." }

pre_aider_hook
aider "$@"
post_aider_hook
```

## Configuration Options

```yaml
# .aider.conf.yml
model: claude-opus-4
editor-model: claude-haiku-4-5
auto-commits: true
lint-cmd: "eslint --fix {files}"
test-cmd: "npm test"
watch-files: true
```

## Key CLI Flags

| Flag | Description |
|------|-------------|
| `--model` | LLM model to use |
| `--auto-commits` | Auto-commit changes |
| `--lint-cmd` | Linter to run before commits |
| `--test-cmd` | Test command to verify changes |
| `--watch-files` | Watch for file changes |
| `--read` | Read-only files (context only) |
| `--map-tokens` | Repo map token budget |

## Built-in Capabilities

Aider operates differently from tool-calling agents — it works through:

1. **Chat**: Natural language edits to files
2. **Repo map**: Automatically builds a map of your codebase
3. **Git integration**: All edits committed to git automatically
4. **Multi-file edits**: Edit multiple files in one prompt

## MCP Support

Aider does not natively support MCP servers as of 2026.

## Supported Models

Aider supports 100+ LLMs via LiteLLM integration:
- Claude Opus/Sonnet/Haiku (Anthropic)
- GPT-4o, o1, o3 (OpenAI)
- Gemini (Google)
- Local models via Ollama

## Notes

- Aider's architecture (diff-based edits + git commits) differs fundamentally from tool-calling agents.
- Pre/post hook support is a highly requested feature — track issue #2045.
- Use `--read` for files that should be in context but not edited.

## Sources (Official)

| Topic | URL |
|-------|-----|
| Main docs | https://aider.chat/docs/ |
| Configuration | https://aider.chat/docs/config.html |
| Options reference | https://aider.chat/docs/config/options.html |
| YAML config file | https://aider.chat/docs/config/aider_conf.html |
| Git integration | https://aider.chat/docs/git.html |
| Hooks feature request | https://github.com/Aider-AI/aider/issues/2045 |
| GitHub repo | https://github.com/Aider-AI/aider |
