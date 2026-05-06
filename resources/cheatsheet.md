# Claude Code Cheat Sheet

## Quick Reference Card

### Startup

```bash
claude                        # Start interactive session
claude --continue             # Continue last session
claude --resume               # Pick a session to resume
claude --model opus           # Use specific model
claude --permission-mode plan # Require plan approval
claude --bare                 # Minimal startup (no MCP)
claude --print "prompt"       # One-shot, non-interactive
claude --add-dir /path        # Add extra directory context
claude --agent "task"         # Fully autonomous headless mode
```

### Essential Commands

```
/help                 Show available commands
/compact              Compress context (use often!)
/clear                Reset conversation
/model                Switch model
/fast                 Toggle fast output mode
/cost                 Show token usage
/status               Show session info
/undo                 Undo last file change
/review               Review uncommitted changes
/pr-review <num>      Review a GitHub PR
/init                 Generate CLAUDE.md for project
/doctor               Diagnose issues
/mcp                  Manage MCP servers
```

### Power Commands

```
/teleport             Transfer session to another device
/loop 5m <task>       Run task every 5 minutes
/schedule <task>      Schedule a remote agent
/branch <name>        Quick branch management
/batch <task> <glob>  Process multiple files
/btw <note>           Side-note for later
/voice                Voice input mode
/ultrareview          Multi-agent cloud code review
```

### Keyboard Shortcuts

```
Ctrl+C                Cancel current operation
Escape                Exit Claude Code (or stop response)
Enter                 Submit prompt
Shift+Enter           New line in prompt (multi-line)
```

---

## File Structure

```
~/.claude/
├── CLAUDE.md                   # Global instructions
├── settings.json               # Global settings & permissions
└── keybindings.json            # Custom keyboard shortcuts

<project>/
├── CLAUDE.md                   # Project instructions (always loaded)
├── .mcp.json                   # MCP server config
└── .claude/
    ├── settings.json           # Project settings
    ├── settings.local.json     # Local overrides (gitignored)
    ├── commands/               # Custom commands
    │   └── my-command.md
    ├── skills/                 # Custom skills
    │   └── my-skill.md
    └── agents/                 # Custom agents
        └── my-agent.md
```

---

## Extension Frontmatter

### Command (`/name`)

```yaml
---
name: my-command
description: What it does
arguments:
  - name: arg1
    description: First argument
    required: true
allowed-tools: [Read, Bash]
model: opus
user-invocable: true
---
```

### Skill (auto-invoke or `/name`)

```yaml
---
name: my-skill
description: >
  What it does. Use when...
context:
  - path: config.ts
user-invocable: true       # false for agent-preloaded skills
---
```

### Agent (spawned by Claude)

```yaml
---
name: my-agent
description: >
  What it does.
  PROACTIVE: Use when user mentions X.
tools: [Read, Bash, Edit]
model: sonnet
maxTurns: 10
skills: [preloaded-skill]
isolation: worktree
---
```

---

## Architecture Pattern

```
User types /command
    ↓
Command orchestrates workflow
    ↓
Command spawns Agent(s)
    ↓
Agent uses preloaded Skill(s)
    ↓
Results bubble up to Command
    ↓
Command presents output
```

**Preference**: Skill (lightest) → Agent → Command (heaviest)

---

## Settings.json

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Bash(git *)",
      "Bash(pnpm *)",
      "mcp__context7__*"
    ],
    "deny": [
      "Bash(rm -rf *)"
    ]
  },
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "command": "prettier --write \"$CLAUDE_FILE_PATH\""
    }]
  },
  "env": {
    "CLAUDE_MODEL": "opus"
  }
}
```

---

## MCP Config (.mcp.json)

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    },
    "playwright": {
      "command": "npx",
      "args": ["@anthropic-ai/mcp-playwright@latest"]
    }
  }
}
```

---

## Golden Rules

1. **One task per session** — start fresh for new tasks
2. **Be specific** — file paths, line numbers, clear constraints
3. **Use `/compact` proactively** — don't wait for degradation
4. **Subagents for research** — protect main context
5. **Read the diff** — always review before approving
6. **Match model to task** — Opus for complex, Sonnet for standard, Haiku for simple
7. **Spec first for big features** — write the spec, then implement from it
