# Plugins, Tools & Ecosystem Directory

A curated directory of the best Claude Code plugins, MCP servers, community tools, and resources -- sourced from GitHub, Reddit, and Hacker News.

*Last updated: May 2026*

---

## Table of Contents

1. [Understanding the Ecosystem](#understanding-the-ecosystem)
2. [Official Resources](#official-resources)
3. [Awesome Lists](#awesome-lists)
4. [Plugins & Marketplaces](#plugins--marketplaces)
5. [MCP Servers](#mcp-servers)
6. [IDE Extensions & Integrations](#ide-extensions--integrations)
7. [Hooks Collections](#hooks-collections)
8. [Skills & Commands](#skills--commands)
9. [Multi-Agent & Orchestration](#multi-agent--orchestration)
10. [Worktree Tools](#worktree-tools)
11. [Cost & Usage Monitoring](#cost--usage-monitoring)
12. [Status Line Tools](#status-line-tools)
13. [Configuration References](#configuration-references)
14. [Learning & Deep Dives](#learning--deep-dives)
15. [Complementary Quality Tools](#complementary-quality-tools)
16. [Ecosystem Scale](#ecosystem-scale-may-2026)

---

## Understanding the Ecosystem

Claude Code's extensibility has two layers:

| Layer | What It Is | How to Add |
|-------|-----------|------------|
| **Plugins** | Bundles of skills, agents, hooks, and MCP configs | `/plugin install <name>@<marketplace>` |
| **MCP Servers** | Model Context Protocol services that provide tools | Configure in `.mcp.json` |

**Plugins** are the distribution mechanism -- they package skills, agents, hooks, and MCP server configurations together. **MCP Servers** are the underlying protocol connecting Claude to external tools and data sources. A plugin often wraps one or more MCP servers.

---

## Official Resources

| Resource | Description |
|----------|-------------|
| [claude-plugins-official](https://github.com/anthropics/claude-plugins-official) | Anthropic's official plugin marketplace (auto-available via `/plugin install`) |
| [claude-code-action](https://github.com/anthropics/claude-code-action) | Official GitHub Action for PR review and issue implementation |
| [Claude Code Docs](https://docs.anthropic.com/en/docs/claude-code) | Official documentation and guides |

---

## Awesome Lists

Start here to explore the ecosystem:

| List | Focus | Scale |
|------|-------|-------|
| [awesome-claude-code-toolkit](https://github.com/rohitg00/awesome-claude-code-toolkit) | Comprehensive | 135 agents, 176+ plugins, 42 commands, 20 hooks |
| [awesome-claude-code](https://github.com/jqueryscript/awesome-claude-code) | Tools & integrations | Curated quality picks |
| [awesome-claude-plugins](https://github.com/quemsah/awesome-claude-plugins) | Plugin metrics | 43 marketplaces, 834+ plugins tracked |
| [awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills) | Skills only | Focused on the skills standard |

> **Note:** Scale numbers change rapidly. Check the repos directly for current counts.

---

## Plugins & Marketplaces

### Essential Plugins

| Plugin | Purpose | Install |
|--------|---------|---------|
| **Context7** | Fetches up-to-date library docs instead of relying on training data | `/plugin install context7@claude-plugins-official` |
| **Playwright** | Browser automation for testing and visual verification | `/plugin install playwright@claude-plugins-official` |
| **GitHub** | Full GitHub integration (PRs, issues, repos) | `/plugin install github@claude-plugins-official` |

> Browse the full official marketplace: run `/plugin search` inside Claude Code.

### Community Plugin Collections

| Collection | Scale | Highlight |
|------------|-------|-----------|
| [claude-code-plugins-plus-skills](https://github.com/jeremylongshore/claude-code-plugins-plus-skills) | 425 plugins, 2,810 skills, 200 agents | `ccpi` package manager |
| [compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) | 37 skills, 51 agents | The compound loop workflow |
| [minimal-claude](https://github.com/KenKaiii/minimal-claude) | Smart defaults | Auto-configures linting and typechecking |

**Choosing a Collection:**
- Just want sensible defaults? Start with **minimal-claude**
- Want a full ecosystem with its own package manager? Try **claude-code-plugins-plus-skills**
- Want the compound engineering workflow (plan > work > review > learn)? Use **compound-engineering-plugin**

---

## MCP Servers

### Productivity

| Server | Purpose |
|--------|---------|
| [claude-context](https://github.com/zilliztech/claude-context) | Semantic code search -- makes your entire codebase the context |
| [Context Mode MCP](https://news.ycombinator.com/item?id=47193064) | 98% context reduction via SQLite FTS5 indexing |
| [claude-code-mcp](https://github.com/steipete/claude-code-mcp) | Claude Code as an MCP server for agent-in-agent workflows |
| [github-mcp-server](https://github.com/github/github-mcp-server) | GitHub's official MCP for repo management |
| [n8n-mcp](https://github.com/czlonkowski/n8n-mcp) | Build n8n workflows from Claude Code |

### Database & Infrastructure

Many database MCP servers are available through the [MCP Servers Directory](https://github.com/modelcontextprotocol/servers). Common setups:

| Server | Purpose |
|--------|---------|
| PostgreSQL MCP | Query databases, inspect schemas |
| SQLite MCP | Local database operations |
| Supabase MCP | Supabase project management |
| Redis MCP | Key-value store operations |

> **Finding MCP servers:** Search [mcp.so](https://mcp.so/) or the [MCP Servers repo](https://github.com/modelcontextprotocol/servers) for servers matching your stack.

### Cloud & Communication

| Server | Purpose |
|--------|---------|
| Vercel MCP | Interact with Vercel deployments, logs, and projects |
| Sentry MCP | Error tracking and performance monitoring |
| Linear MCP | Issue tracking and project management |
| Slack MCP | Channel messaging and notifications |

> These servers are available through their respective providers or community packages. Search the [MCP Servers Directory](https://github.com/modelcontextprotocol/servers) for install instructions.

---

## IDE Extensions & Integrations

Claude Code works across multiple surfaces:

| Surface | Details |
|---------|---------|
| **CLI** | The core experience -- `claude` in any terminal |
| **VS Code** | Official extension with inline chat and terminal integration |
| **JetBrains** | Official extension for IntelliJ, WebStorm, PyCharm, etc. |
| **Desktop App** | Native app for macOS and Windows |
| **Web App** | Browser-based at claude.ai/code |

All surfaces share the same Claude Code engine, configuration, and plugin system.

---

## Hooks Collections

| Collection | Highlight |
|------------|-----------|
| [claude-code-hooks-mastery](https://github.com/disler/claude-code-hooks-mastery) | Standalone Python scripts with embedded dependencies |
| [claude-code-hooks](https://github.com/karanb192/claude-code-hooks) | Copy-paste-customize collection |
| [claude-hooks](https://github.com/decider/claude-hooks) | Hierarchical config per directory, quality validation |
| [claude-code-quality-pipeline](https://github.com/jseldess/claude-code-quality-pipeline) | Full pipeline: auto-format, security scan, testing, docs |

### Must-Have Hook Patterns

> **Important:** Claude Code uses **exit 2** to block actions (not exit 1). Exit 1 is treated as a non-blocking error and the action proceeds anyway.

```jsonc
// Auto-format on every edit
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "command": "prettier --write \"$CLAUDE_FILE_PATH\" 2>/dev/null || true",
      "description": "Auto-format files after Claude edits them",
      "timeout": 5000
    }]
  }
}
```

```jsonc
// Block dangerous commands (exit 2 = block, exit 0 = allow)
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "command": "echo \"$CLAUDE_TOOL_INPUT\" | grep -qE 'rm -rf|DROP TABLE|--force|--no-verify' && echo 'BLOCKED: dangerous command' >&2 && exit 2 || exit 0",
      "description": "Block dangerous bash commands",
      "timeout": 5000
    }]
  }
}
```

```jsonc
// Notify when Claude needs input (macOS + Linux)
{
  "hooks": {
    "Notification": [{
      "matcher": "",
      "command": "osascript -e 'display notification \"Claude needs your attention\" with title \"Claude Code\"' 2>/dev/null || notify-send 'Claude Code' 'Claude needs your attention' 2>/dev/null || true",
      "description": "Desktop notification when Claude needs input"
    }]
  }
}
```

---

## Skills & Commands

| Collection | Scale | Highlight |
|------------|-------|-----------|
| [Claude-Command-Suite](https://github.com/qdhenry/Claude-Command-Suite) | 216+ commands, 12 skills, 54 agents | Code review, testing, deployment |
| [Claude-Skills](https://github.com/borghei/Claude-Skills) | 245 skills, 32 agents | Multi-agent coding tools |
| [claude-code-skill-factory](https://github.com/alirezarezvani/claude-code-skill-factory) | Framework | Build production skills at scale |
| [commands](https://github.com/wshobson/commands) | Production-ready | Intelligent automation commands |

---

## Multi-Agent & Orchestration

| Tool | Pattern |
|------|---------|
| [parallel-worktrees](https://github.com/spillwavesolutions/parallel-worktrees) | Git worktrees + background subagents with status tracking |
| [ccswarm](https://github.com/nwiizo/ccswarm) | Multi-agent with worktree isolation |
| [claude-code-workflow-orchestration](https://github.com/barkain/claude-code-workflow-orchestration) | Auto task decomposition + parallel agents |
| [metaswarm](https://github.com/dsifry/metaswarm) | 18 agents, TDD enforcement, quality gates |
| [pro-workflow](https://github.com/rohitg00/pro-workflow) | Self-correcting memory that compounds over 50+ sessions |

**Choosing an orchestration tool:** Start with Claude Code's built-in subagent system (Module 5). Add a tool from this list when you need persistent multi-session coordination, fleet management across worktrees, or enforced quality gates.

---

## Worktree Tools

Tools for managing parallel Claude Code agents across git worktrees. Search GitHub for current options:

| Tool | Purpose |
|------|---------|
| [parallel-worktrees](https://github.com/spillwavesolutions/parallel-worktrees) | Purpose-built for parallel Claude Code + worktree workflows |

> **Tip:** Claude Code has built-in worktree support via the `--worktree` flag. Third-party tools add fleet management, status dashboards, and orchestration on top. Search GitHub for "claude code worktree" for the latest options.

---

## Cost & Usage Monitoring

| Tool | Type | Purpose |
|------|------|---------|
| [claude-usage](https://github.com/phuryn/claude-usage) | Web dashboard | Token usage, costs, session history, progress bar for Max |
| [claude-code-dashboard](https://github.com/Stargx/claude-code-dashboard) | Real-time monitor | Multi-session tracking, active tools, subagent status |
| [token-dashboard](https://github.com/nateherkai/token-dashboard) | Analytics | Per-prompt costs, tool heatmaps, cache analytics |
| [Claude-Code-Usage-Monitor](https://github.com/Maciek-roboblog/Claude-Code-Usage-Monitor) | Terminal TUI | ML-based predictions, session limit warnings |
| [codeburn](https://github.com/getagentseal/codeburn) | Multi-tool TUI | Claude Code + Codex + Cursor cost tracking |
| [claude-code-otel](https://github.com/ColeMurray/claude-code-otel) | OpenTelemetry | Enterprise observability for usage and performance |
| [vscode-claude-status](https://github.com/long-910/vscode-claude-status) | VS Code extension | Token usage in status bar |

**Choosing a monitor:** For personal use, **claude-usage** gives the best overview. For team/enterprise, **claude-code-otel** integrates with existing observability stacks. For multi-tool shops, **codeburn** tracks costs across AI coding tools.

---

## Status Line Tools

| Tool | Purpose |
|------|---------|
| [ccstatusline](https://github.com/sirmalloc/ccstatusline) | Powerline themes, multi-line, interactive TUI, zero-config |
| [claude-code-statusline](https://github.com/levz0r/claude-code-statusline) | Real-time tokens, cost calculation, git integration |

**Tip:** Claude Code has a built-in `/statusline` command. Describe what you want (e.g., "show model name and context percentage with a progress bar") and it generates the script automatically.

---

## Configuration References

| Resource | Purpose |
|----------|---------|
| [claude-md-templates](https://github.com/abhishekray07/claude-md-templates) | CLAUDE.md best practices with good/bad examples |
| [claude-code-showcase](https://github.com/ChrisWiles/claude-code-showcase) | Complete project config: hooks, skills, agents, commands, CI |
| [settings.json reference gist](https://gist.github.com/mculp/c082bd1e5a439410158974de90c89db7) | 125+ settings keys documented |
| [claude-code-settings](https://github.com/feiskyer/claude-code-settings) | Pre-configured settings for various model providers |
| [dot-claude](https://github.com/evantahler/dot-claude) | Shareable dotfiles with one-line install |

---

## Learning & Deep Dives

| Resource | Focus |
|----------|-------|
| [Dive-into-Claude-Code](https://github.com/VILA-Lab/Dive-into-Claude-Code) | Architecture analysis of Claude Code internals (~512K lines) |
| [claude-code-system-prompts](https://github.com/Piebald-AI/claude-code-system-prompts) | All system prompt parts, updated per version |
| [claude-code-reverse](https://github.com/Yuyz0112/claude-code-reverse) | Visualize Claude Code's LLM interactions from logs |
| [claude-code-internals](https://github.com/eran-broder/claude-code-internals) | 72 sections of undocumented internals |
| [claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice) | The definitive community guide (51k+ stars) |
| [claude-code-ultimate-guide](https://github.com/FlorianBruniaux/claude-code-ultimate-guide) | Beginner to power user with quizzes |
| [Collation of Best Practices v2](https://news.ycombinator.com/item?id=45830267) | Community-maintained at rosmur.github.io |

---

## Complementary Quality Tools

Use alongside Claude Code for automated quality enforcement:

| Tool | Purpose |
|------|---------|
| **CodeRabbit** | AI code review bot for PRs |
| **Sourcery** | Automated code quality suggestions |
| **CodeScene** | Behavioral code analysis |
| **husky + lint-staged** | Pre-commit hooks for formatting and linting |
| **commitlint** | Enforce conventional commit messages |

---

## Ecosystem Scale (May 2026)

Approximate numbers from community trackers ([awesome-claude-plugins](https://github.com/quemsah/awesome-claude-plugins), [awesome-claude-code-toolkit](https://github.com/rohitg00/awesome-claude-code-toolkit)):

- **4,200+** skills available
- **770+** MCP servers
- **2,500+** marketplace entries
- **834+** tracked plugins across 43 marketplaces
