# Example Configuration Files

Copy these files into your project or global config as a starting point.

## Files

| File | Copy To | Purpose |
|------|---------|---------|
| [`settings.json`](settings.json) | `~/.claude/settings.json` or `.claude/settings.json` | Permissions, deny list, and auto-format hook |
| [`CLAUDE.md`](CLAUDE.md) | Project root `CLAUDE.md` | Project instructions template |
| [`.mcp.json`](.mcp.json) | Project root `.mcp.json` | Essential MCP servers (Context7 + Playwright) |
| [`.claudeignore`](.claudeignore) | Project root `.claudeignore` | Exclude build artifacts and binaries from context |
| [`hooks-settings.json`](hooks-settings.json) | Merge into `settings.json` | Advanced hooks: format, guardrails, logging, notifications |

## Quick Start

```bash
# 1. Set up global settings (permissions + basic hook)
cp examples/settings.json ~/.claude/settings.json

# 2. Set up project config
cp examples/CLAUDE.md ./CLAUDE.md        # Edit with your tech stack
cp examples/.mcp.json ./.mcp.json        # MCP servers
cp examples/.claudeignore ./.claudeignore # Context exclusions

# 3. Start Claude Code
claude
```

## Customizing

- **settings.json**: Add your package manager (`pnpm`, `yarn`, `bun`) to the allow list. Remove commands you don't use.
- **CLAUDE.md**: Replace the tech stack, commands, and conventions with your own. Keep it under 60 lines.
- **.mcp.json**: Add servers for your specific needs (PostgreSQL, Sentry, etc.).
- **.claudeignore**: Add project-specific patterns (e.g., `data/`, `fixtures/`).
- **hooks-settings.json**: Cherry-pick the hooks you need and merge into your `settings.json`.
