# Plugins, Tools & Ecosystem Directory

A curated directory of the best Claude Code plugins, MCP servers, community tools, and resources -- sourced from GitHub, Reddit, and Hacker News.

*Last updated: May 2026*

---

## Official Resources

| Resource | Description |
|----------|-------------|
| [claude-plugins-official](https://github.com/anthropics/claude-plugins-official) | Anthropic's curated plugin directory |
| [claude-code-action](https://github.com/anthropics/claude-code-action) | Official GitHub Action for PR review and issue implementation |
| [claude-code-security-review](https://github.com/anthropics/claude-code-security-review) | AI-powered security review GitHub Action |
| [Official Hook Examples](https://github.com/anthropics/claude-code/tree/main/examples/hooks) | Reference hook implementations from Anthropic |

---

## Awesome Lists

Start here to explore the ecosystem:

| List | Focus | Scale |
|------|-------|-------|
| [awesome-claude-code-toolkit](https://github.com/rohitg00/awesome-claude-code-toolkit) | Everything | 135 agents, 176+ plugins, 42 commands, 20 hooks |
| [awesome-claude-code](https://github.com/jqueryscript/awesome-claude-code) | Tools & integrations | Curated quality picks |
| [awesome-claude-plugins](https://github.com/quemsah/awesome-claude-plugins) | Plugin metrics | 43 marketplaces, 834+ plugins tracked |
| [awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills) | Skills only | Focused on the skills standard |

---

## Plugins & Marketplaces

### Essential Plugins

| Plugin | What It Does | Install |
|--------|-------------|---------|
| **Context7** | Fetches up-to-date library docs instead of relying on training data | Official marketplace |
| **Playwright** | Browser automation for testing and visual verification | Official marketplace |
| **GitHub** | Full GitHub integration (PRs, issues, repos) | Official marketplace |
| **Security Guidance** | Security best practices during code review | Official marketplace |

### Community Plugin Collections

| Collection | Scale | Notable Feature |
|------------|-------|-----------------|
| [claude-code-plugins-plus-skills](https://github.com/jeremylongshore/claude-code-plugins-plus-skills) | 425 plugins, 2,810 skills, 200 agents | `ccpi` package manager |
| [compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) | 37 skills, 51 agents | The compound loop workflow |
| [minimal-claude](https://github.com/KenKaiii/minimal-claude) | Smart defaults | Auto-configures linting and typechecking |

### Plugin Management

| Tool | Purpose |
|------|---------|
| **McPick** | CLI/TUI for managing MCP servers -- simpler than built-in config |
| **MCP-tidy** | Audits which MCP servers you actually use vs. which waste context |
| **cclint** | Linter for Claude Code project files (agents, commands, settings) |

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

### Research & Knowledge

| Server | Purpose |
|--------|---------|
| **Almanac MCP** | Turns Claude Code into a deep research agent |
| **Context7** | Real-time library documentation lookup |

---

## Hooks Collections

| Collection | Notable Hooks |
|------------|---------------|
| [claude-code-hooks-mastery](https://github.com/disler/claude-code-hooks-mastery) | Standalone Python scripts with embedded dependencies |
| [claude-code-hooks](https://github.com/karanb192/claude-code-hooks) | Copy-paste-customize collection |
| [claude-hooks](https://github.com/decider/claude-hooks) | Hierarchical config per directory, quality validation |
| [claude-code-quality-pipeline](https://github.com/jseldess/claude-code-quality-pipeline) | Full pipeline: auto-format, security scan, testing, docs |

### Must-Have Hook Patterns

```jsonc
// Auto-format on every edit
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "command": "prettier --write \"$CLAUDE_FILE_PATH\""
    }]
  }
}
```

```jsonc
// Block dangerous commands
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "command": "echo \"$CLAUDE_TOOL_INPUT\" | grep -qE 'rm -rf|DROP TABLE|force push' && exit 1 || exit 0"
    }]
  }
}
```

```jsonc
// Notify when Claude needs input (macOS)
{
  "hooks": {
    "Notification": [{
      "matcher": "",
      "command": "osascript -e 'display notification \"Claude needs your attention\" with title \"Claude Code\"'"
    }]
  }
}
```

---

## Skills & Commands

| Collection | Scale | Highlights |
|------------|-------|------------|
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

---

## Worktree Tools

| Tool | Purpose |
|------|---------|
| **Harness** | Manage parallel Claude Code agents across git worktrees |
| **Gwt-Claude** | Purpose-built for parallel Claude Code + worktree workflows |
| **wt / Wtx** | Lightweight worktree management |
| **FleetCode** | Fleet management for parallel agents |
| **Branchlet** | Lightweight worktree orchestration |
| **Agentastic.dev** | Web-based parallel agent management |

---

## Cost & Usage Monitoring

| Tool | Type | Features |
|------|------|----------|
| [claude-usage](https://github.com/phuryn/claude-usage) | Web dashboard | Token usage, costs, session history, progress bar for Max |
| [claude-code-dashboard](https://github.com/Stargx/claude-code-dashboard) | Real-time monitor | Multi-session tracking, active tools, subagent status |
| [token-dashboard](https://github.com/nateherkai/token-dashboard) | Analytics | Per-prompt costs, tool heatmaps, cache analytics |
| [Claude-Code-Usage-Monitor](https://github.com/Maciek-roboblog/Claude-Code-Usage-Monitor) | Terminal TUI | ML-based predictions, session limit warnings |
| [codeburn](https://github.com/getagentseal/codeburn) | Multi-tool TUI | Claude Code + Codex + Cursor cost tracking |
| [claude-code-otel](https://github.com/ColeMurray/claude-code-otel) | OpenTelemetry | Enterprise observability for usage and performance |
| [vscode-claude-status](https://github.com/long-910/vscode-claude-status) | VS Code extension | Token usage in status bar |

---

## Status Line Tools

| Tool | Features |
|------|----------|
| [ccstatusline](https://github.com/sirmalloc/ccstatusline) | Powerline themes, multi-line, interactive TUI, zero-config |
| [claude-code-statusline](https://github.com/levz0r/claude-code-statusline) | Real-time tokens, cost calculation, git integration |

**Tip:** Claude Code has a built-in `/statusline` command. Describe what you want (e.g., "show model name and context percentage with a progress bar") and it generates the script automatically.

---

## Configuration References

| Resource | What It Provides |
|----------|-----------------|
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
| [claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice) | The definitive community guide (20k+ stars) |
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

- **4,200+** skills available
- **770+** MCP servers
- **2,500+** marketplace entries
- **834+** tracked plugins across 43 marketplaces
