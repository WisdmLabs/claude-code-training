# Claude Code Configuration Best Practices

A practical guide derived from a production-grade Claude Code setup. This document explains each configuration layer, what the current setup does well, and recommendations for teams adopting similar patterns.

---

## Table of Contents

1. [Configuration Hierarchy](#configuration-hierarchy)
2. [Settings (settings.json)](#settings-settingsjson)
3. [Permissions Strategy](#permissions-strategy)
4. [Plugins & Marketplaces](#plugins--marketplaces)
5. [Rules Files](#rules-files)
6. [CLAUDE.md (Global Instructions)](#claudemd-global-instructions)
7. [Project-Level Configuration](#project-level-configuration)
8. [Memory System](#memory-system)
9. [Environment Variables](#environment-variables)
10. [Status Line](#status-line)
11. [Security Considerations](#security-considerations)
12. [Recommendations Summary](#recommendations-summary)

---

## Configuration Hierarchy

Claude Code uses a layered configuration system. Understanding the priority order is critical:

```
User Global (~/.claude/settings.json)        ← applies to ALL projects
  └── User Local (~/.claude/settings.local.json)  ← personal overrides, not committed
      └── Project (.claude/settings.json)         ← shared with team via git
          └── Project Local (.claude/settings.local.json) ← personal project overrides
```

**Best Practice:** Keep team-shared configuration in project-level `settings.json`. Keep personal tools, paths, and experimental features in `settings.local.json` files (which should be gitignored).

---

## Settings (settings.json)

### Current Configuration Breakdown

```jsonc
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"  // Enables multi-agent teams
  },
  "permissions": { ... },       // Granular tool allowlists
  "statusLine": { ... },        // Custom status bar
  "enabledPlugins": { ... },    // Plugin enable/disable map
  "extraKnownMarketplaces": {}, // Custom plugin sources
  "skipAutoPermissionPrompt": true,  // Suppresses auto-permission popup
  "disableAutoMode": "disable"       // Disables auto-mode toggling
}
```

### Key Decisions Explained

| Setting | Value | Why |
|---------|-------|-----|
| `skipAutoPermissionPrompt` | `true` | Prevents interruptions — you've already defined an explicit allowlist |
| `disableAutoMode` | `"disable"` | Forces consistent permission behavior instead of letting Claude auto-escalate |
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` | `"1"` | Enables the Agent Teams feature for multi-agent orchestration |

### Best Practices for Settings

1. **Be explicit about defaults.** Setting `defaultMode: "auto"` in permissions makes the intent clear even though it's the default.
2. **Use env vars for experimental features.** This keeps experimental toggles visible and easy to revert.
3. **Don't over-configure.** Only set values that differ from defaults or that you want to be explicit about for team clarity.

---

## Permissions Strategy

The current setup uses a comprehensive allowlist approach. This is the **most secure and recommended** pattern for production use.

### Structure of Permission Rules

```
"allow": [
  "ToolName",                    // Allow tool entirely
  "Bash(command prefix:*)",      // Allow specific command patterns
  "Read(//path/pattern/**)",     // Allow reads to specific paths
  "mcp__server__tool_name"       // Allow specific MCP tool calls
]
```

### Current Allowlist Categories

| Category | Examples | Purpose |
|----------|----------|---------|
| **Core Tools** | `Read`, `Edit`, `Write`, `Glob`, `Grep` | File operations — always safe to allow |
| **Git Operations** | `Bash(git status:*)`, `Bash(git commit:*)` | Full git workflow without prompts |
| **Package Managers** | `Bash(npm run:*)`, `Bash(composer:*)`, `Bash(pnpm:*)` | Build & dependency management |
| **PHP Toolchain** | `Bash(phpcs:*)`, `Bash(phpunit:*)`, `Bash(phpstan:*)` | WordPress/PHP quality tools |
| **WP-CLI** | `Bash(wp plugin:*)`, `Bash(wp db:*)`, etc. | WordPress management |
| **Docker** | `Bash(docker ps:*)`, `Bash(docker logs:*)` | Container management |
| **System Utils** | `Bash(ls:*)`, `Bash(find:*)`, `Bash(grep:*)` | Read-only exploration |
| **Network** | `Bash(curl:*)`, `Bash(ping:*)`, `Bash(ssh:*)` | External connectivity |
| **GitHub CLI** | `Bash(gh pr:*)`, `Bash(gh issue:*)` | GitHub workflow |
| **CodeGraph MCP** | `mcp__codegraph__*` | Semantic code navigation |

### Best Practices for Permissions

1. **Start restrictive, expand as needed.** Only add commands you actually use.
2. **Use command prefixes, not wildcards.** `Bash(git commit:*)` is safer than `Bash(git:*)` because it prevents `git push --force` without explicit approval.
3. **Separate read-only from write operations.** The current config allows `git reset` and `git restore` — consider whether your team needs these auto-approved.
4. **Group by workflow.** Organize permissions by what you do (git, PHP, Docker) for easier auditing.
5. **Document dangerous permissions.** Commands like `Bash(ssh:*)` and `Bash(docker exec:*)` give significant access — ensure your team understands the implications.

### Security Recommendations

Commands to **consider removing** from auto-allow (require manual approval instead):

```jsonc
// These can cause data loss or affect remote systems:
"Bash(git reset:*)",    // Can discard commits
"Bash(git restore:*)",  // Can discard uncommitted changes
"Bash(ssh:*)",          // Remote system access
"Bash(docker exec:*)",  // Container command execution
"Bash(rm:*)",           // Not present (good!) — never auto-allow
```

---

## Plugins & Marketplaces

### Currently Enabled Plugins

| Plugin | Source | Purpose |
|--------|--------|---------|
| **wisdmlabs-engineering** | Custom marketplace | 100+ specialized agents for WordPress, Moodle, Shopify, testing, architecture |
| **context7** | Official | Live documentation lookup for libraries and frameworks |
| **claude-mem** | thedotmack | Persistent memory system with MCP search |
| **frontend-design** | Official | UI/UX design assistance |
| **vercel** | Official | Vercel deployment and configuration |
| **vercel-plugin** | Custom (local) | Extended Vercel features |
| **figma** | Official | Figma design integration |
| **php-lsp** | Official | PHP language server for code intelligence |
| **typescript-lsp** | Official | TypeScript language server |
| **vscode-langservers** | Piebald-AI | Additional language servers (HTML, CSS, JSON) |
| **github** | Official | GitHub integration |
| **security-guidance** | Official | Security best practices |
| **prompt-improver** | severity1 | Auto-improves vague prompts |
| **rust-analyzer-lsp** | Official | Rust language support |

### Disabled Plugins

| Plugin | Why Disabled |
|--------|-------------|
| `phpactor` | Conflicts with php-lsp or not needed alongside it |
| `pyright` | Not working on primary Python projects |
| `vtsls` | TypeScript LSP already covered by typescript-lsp |

### Best Practices for Plugins

1. **Disable unused LSPs.** Each active LSP consumes resources. Only enable languages you actively develop in.
2. **Use official marketplace first.** Third-party marketplaces (`extraKnownMarketplaces`) should be from trusted sources with pinned versions.
3. **Audit marketplace sources.** The current config references:
   - `anthropics/claude-plugins-official` (trusted)
   - `aruneshwisdm/wisdmlabs-engineering-plugin` (custom team plugin)
   - `Piebald-AI/claude-code-lsps` (community)
   - `thedotmack/claude-mem` (community)
   - `severity1/severity1-marketplace` (community)
   - `vercel/vercel-plugin` (local directory install)
4. **Review plugin updates.** Plugins auto-update — check changelogs periodically for breaking changes.
5. **One LSP per language.** Don't enable both `phpactor` and `php-lsp`, or both `vtsls` and `typescript-lsp`.

---

## Rules Files

Rules in `~/.claude/rules/` provide behavioral instructions that apply to every session. They're the best place for team-wide engineering practices.

### Current Rules

#### Rule 1: Team Learnings First (`wisdm-01-team-learnings.md`)

**Pattern:** Search collective knowledge before investigating independently.

```
Search → Match → Record Usage → (or) Solve → Extract Learning
```

**Why this works:** Prevents duplicate debugging effort across teams. The `record_usage` step creates a feedback loop that surfaces the most valuable learnings.

#### Rule 2: Compound Engineering Loop (`wisdm-02-compound-loop.md`)

**Pattern:** Plan → Work → Review → Compound

**Why this works:** Forces knowledge extraction after every non-trivial task. The "Compound" step (extracting learnings, updating team knowledge) is what differentiates this from a simple checklist.

**When to apply:** Tasks touching 3+ files, investigations, architectural decisions.
**When to skip:** Single-line fixes, docs-only changes, config updates.

#### Rule 3: Specialized Agent Selection (`wisdm-03-agent-selection.md`)

**Pattern:** Route tasks to domain-specific agents instead of using general-purpose approaches.

**Why this works:** Specialized agents carry domain-specific knowledge (WordPress hooks, Moodle standards, Shopify patterns) that a general agent would need to re-discover each time.

### Best Practices for Rules

1. **Number your rules.** The `wisdm-01`, `wisdm-02` prefix makes ordering explicit and aids discovery.
2. **Include "When to Skip" criteria.** Rules without escape hatches get ignored when they feel heavy for trivial work.
3. **Keep rules actionable.** Each rule should describe a clear workflow, not just a philosophy.
4. **Limit to 3-5 rules.** More than that and they compete for attention. The current set of 3 is ideal.
5. **Rules vs. CLAUDE.md:** Use rules for behavioral instructions (how to work). Use CLAUDE.md for tool/capability instructions (what tools to use and how).

---

## CLAUDE.md (Global Instructions)

The global `~/.claude/CLAUDE.md` is loaded into every conversation. It should contain instructions that are always relevant regardless of project.

### Current Content: CodeGraph Integration

The current CLAUDE.md focuses exclusively on CodeGraph — a semantic code navigation tool. This is a good example of a globally-relevant tool instruction:

- If `.codegraph/` exists → use codegraph tools for exploration
- If not → offer to initialize it

### Best Practices for Global CLAUDE.md

1. **Keep it short.** Every token here is loaded into every conversation. Current size (~30 lines) is appropriate.
2. **Only include always-relevant instructions.** CodeGraph applies to any codebase — good fit for global.
3. **Don't duplicate rules.** Rules files handle behavioral patterns; CLAUDE.md handles tool awareness.
4. **Use conditional patterns.** "If X exists, do Y; otherwise, do Z" makes the instruction adaptive.
5. **Avoid project-specific details.** Those belong in project-level `CLAUDE.md` files.

---

## Project-Level Configuration

### settings.local.json (Per-Project)

```json
{
  "permissions": {
    "allow": [
      "Read(//home/arunesh/.claude/**)",
      "Read(//home/arunesh/.claude/plugins/**)",
      "Bash(gh auth *)"
    ]
  }
}
```

This grants the project permission to read Claude's own configuration — useful for self-documentation tasks (like generating this document).

### Best Practices

1. **Project settings.json → team config.** Commit this to git for shared team behavior.
2. **Project settings.local.json → personal overrides.** Add to `.gitignore`. Use for local paths, personal tools.
3. **Create project-level CLAUDE.md** for repo-specific conventions (architecture, testing commands, deployment workflows).
4. **Keep agent-usage.json.** It tracks which specialized agents have been used — useful for understanding team patterns.

---

## Memory System

The memory system (`~/.claude/projects/<path>/memory/`) persists context across sessions.

### Current Memory Organization

The setup uses project-scoped memory with several patterns observed:

- `MEMORY.md` — Index file per project
- `user_*.md` — User context (role, preferences)
- `project_*.md` — Project decisions and state
- `feedback_*.md` — Behavioral corrections
- `reference_*.md` — External system pointers

### Best Practices

1. **One memory per concept.** Don't combine unrelated information.
2. **Use frontmatter.** Each memory file should have `name`, `description`, and `type` fields for searchability.
3. **Keep MEMORY.md under 200 lines.** It's loaded every session — keep entries to one-line summaries.
4. **Prune stale memories.** Project decisions change; review memories periodically.
5. **Don't store code patterns.** Those belong in the code itself, not memory. Memory is for non-obvious context.

---

## Environment Variables

```json
"env": {
  "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
}
```

### Best Practices

1. **Use env for feature flags.** Experimental features should be explicitly opted into via env vars.
2. **Document what each var does.** Future you won't remember what `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` enables.
3. **Don't store secrets in settings.json.** Use `settings.local.json` (gitignored) or system environment variables.
4. **Check for deprecation.** Experimental flags may become defaults or be removed — review periodically.

---

## Status Line

```json
"statusLine": {
  "type": "command",
  "command": "npx -y ccstatusline@latest",
  "padding": 0
}
```

This runs a custom status line tool that shows contextual information in the Claude Code interface.

### Best Practices

1. **Use `@latest` carefully.** Convenient but can break if the package introduces breaking changes. Consider pinning for stability.
2. **Keep padding minimal.** `padding: 0` maximizes screen real estate.
3. **Ensure the command is fast.** Status line commands run frequently — avoid expensive operations.

---

## Security Considerations

### What This Config Does Well

- Explicit allowlist instead of permissive defaults
- Command-prefix matching (not blanket `Bash(*)`)
- Local settings separated from shared settings
- No secrets in committed files
- MCP tools individually allowlisted

### Areas to Review

| Concern | Current State | Recommendation |
|---------|--------------|----------------|
| `Bash(ssh:*)` | Auto-allowed | Consider requiring approval for remote access |
| `Bash(docker exec:*)` | Auto-allowed | Container breakout risk — review if needed |
| `Bash(git reset:*)` | Auto-allowed | Can cause data loss — consider requiring approval |
| `enableAllProjectMcpServers` | `false` (good) | Prevents untrusted projects from auto-loading MCP |
| Plugin sources | Mix of official + community | Audit community plugins periodically |
| `Bash(curl:*)` / `Bash(wget:*)` | Auto-allowed | Can exfiltrate data — acceptable if you trust the model |

---

## Recommendations Summary

### For Individual Developers

1. Start with the official plugins only, then add community plugins as needed
2. Use `settings.local.json` for personal preferences
3. Create project-level CLAUDE.md files for every repo you work in regularly
4. Keep 3-5 rules maximum in your rules directory
5. Review and prune memories quarterly

### For Teams

1. Share project-level `settings.json` and `CLAUDE.md` via git
2. Standardize on rules files for engineering practices
3. Use the compound loop pattern to build collective knowledge
4. Establish a shared plugin marketplace for team-specific agents
5. Document which permissions are needed and why in your repo's README or CONTRIBUTING.md

### For Production Workflows

1. Remove `ssh`, `docker exec`, and `git reset` from auto-allow in shared configs
2. Pin plugin versions instead of using `@latest`
3. Audit `extraKnownMarketplaces` sources regularly
4. Keep `enableAllProjectMcpServers: false` — never auto-trust project MCP configs
5. Use `disableAutoMode: "disable"` to maintain consistent permission behavior

---

## Quick Reference: File Locations

| File | Scope | Purpose |
|------|-------|---------|
| `~/.claude/settings.json` | Global | Shared settings, permissions, plugins |
| `~/.claude/settings.local.json` | Global (personal) | Personal overrides, not synced |
| `~/.claude/CLAUDE.md` | Global | Always-loaded instructions |
| `~/.claude/rules/*.md` | Global | Behavioral rules for all projects |
| `.claude/settings.json` | Project (team) | Team-shared project config |
| `.claude/settings.local.json` | Project (personal) | Personal project overrides |
| `CLAUDE.md` (project root) | Project | Repo-specific instructions |
| `~/.claude/projects/<path>/memory/` | Project | Persistent cross-session context |
