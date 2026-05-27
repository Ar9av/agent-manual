# Google Antigravity

> Google's AI coding agent. Docs at antigravity.google/docs (requires auth/invite access as of May 2026).

**Vendor:** Google | **Status:** Limited / invite-only public access

## Links

- Docs (auth required): https://antigravity.google/docs/home
- Community forum: https://discuss.ai.google.dev/t/does-antigravity-support-hooks-similar-to-the-hook-functionality-in-windsurf/121062
- Related: [Gemini CLI](../gemini-cli/README.md) — Google's public terminal agent

---

## Extensibility: Workflows & Rules

Antigravity does **not** have a `PreToolUse`/`PostToolUse` hook system like Claude Code or Copilot. Instead it uses a **workflows + rules** model confirmed via the Google AI developer forum:

- **Workflows** — defined in `.agent/workflows/` directory. Multi-step automated processes (e.g., run tests after refactoring, generate docs after schema changes).
- **Rules** — defined in `.agent/rules/` directory. Behavioral triggers that fire when specific conditions are met (e.g., validate before modifying a directory).

These are declarative/config-driven rather than imperative shell commands with exit code blocking.

> Source: [Google AI dev forum thread](https://discuss.ai.google.dev/t/does-antigravity-support-hooks-similar-to-the-hook-functionality-in-windsurf/121062) — the official reference cited in the forum is `antigravity.google/docs/rules-workflows` (auth required).

### Directory Structure

| Directory | Purpose |
|-----------|---------|
| `.agent/workflows/` | Multi-step automated workflows |
| `.agent/rules/` | Behavioral rules / triggers |
| `.agents/skills/` | Skills (shared convention with Codex CLI) |

## Skills

From graphify's `_PLATFORM_CONFIG` (a third-party installer that documents confirmed paths):

```python
"antigravity": {
    "skill_file": "skill.md",
    "skill_dst": ".agents/skills/graphify/SKILL.md",
},
"antigravity-windows": {
    "skill_file": "skill-windows.md",   # PowerShell variant
    "skill_dst": ".agents/skills/graphify/SKILL.md",
},
```

| Scope | Path |
|-------|------|
| Global | `~/.agents/skills/` |
| Project | `.agents/skills/` |

Same `skill.md` format as Claude Code (YAML frontmatter: `name`, `description`, `trigger`). Windows variant uses PowerShell.

## Known Information

| Aspect | Status |
|--------|--------|
| Public docs | ❌ Gated — returns empty HTML to unauthenticated requests |
| Workflows system | ✅ Confirmed — `.agent/workflows/` directory |
| Rules system | ✅ Confirmed — `.agent/rules/` directory |
| Skill system | ✅ Confirmed — `.agents/skills/` directory, Markdown format |
| Native PreToolUse/PostToolUse hooks | ❌ Not present — uses workflows/rules instead |
| MCP support | ❓ Unknown — docs not publicly accessible |
| Windows support | ✅ Confirmed — separate PowerShell skill variant |

## Notes

- `.agents/skills/` is shared with Codex CLI — likely an intentional convergence.
- The docs site (`antigravity.google/docs/*`) returns only a title tag to unauthenticated requests.
- If you have invite access, contribute MCP and workflow schema details here.

## Sources (Official)

| Topic | URL |
|-------|-----|
| Docs home (auth required) | https://antigravity.google/docs/home |
| Rules & workflows (auth required) | https://antigravity.google/docs/rules-workflows |
| Community forum (hooks discussion) | https://discuss.ai.google.dev/t/does-antigravity-support-hooks-similar-to-the-hook-functionality-in-windsurf/121062 |
| graphify `_PLATFORM_CONFIG` (source of skill path data) | https://github.com/safishamsi/graphify |
