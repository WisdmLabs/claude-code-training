# Module 3: Commands — Built-in & Custom

**Duration**: 45 minutes | **Level**: Intermediate

## TL;DR

Commands are slash-invoked entry points. Claude Code has 75+ built-in commands and you can create custom ones. Commands run in the main conversation context and are always user-initiated.

---

## 3.1 What Are Commands?

Commands are actions triggered with `/` in the Claude Code prompt. They are:

- **User-initiated** — never auto-invoked by Claude
- **Run in shared context** — they see the full conversation
- **Entry points** — they orchestrate work, delegating to agents and skills

Think of commands as the "front door" to structured workflows.

## 3.2 Built-in Commands by Category

### Session Management (8)

| Command | Purpose |
|---------|---------|
| `/clear` | Clear conversation history |
| `/compact` | Compress conversation to save context |
| `/continue` | Continue last conversation |
| `/resume` | Resume a specific conversation |
| `/undo` | Undo the last file change |
| `/status` | Show session info |
| `/logout` | Sign out |
| `/exit` | Exit Claude Code |

### Model & Configuration (5 + 11)

| Command | Purpose |
|---------|---------|
| `/model` | Switch model (opus, sonnet, haiku) |
| `/fast` | Toggle fast mode (Opus with faster output) |
| `/config` | Open configuration |
| `/permissions` | Manage permissions |
| `/cost` | Show token usage and costs |

### Context & Memory (7)

| Command | Purpose |
|---------|---------|
| `/add-dir` | Add directory to context |
| `/init` | Initialize CLAUDE.md for a project |
| `/memory` | View/edit memory files |
| `/review` | Review current changes |
| `/pr-review` | Review a pull request |

### Remote & Background (9)

| Command | Purpose |
|---------|---------|
| `/remote` | Run on remote machine |
| `/remote-control` | Control remote sessions |
| `/teleport` | Transfer session to another device |
| `/loop` | Schedule recurring tasks |
| `/schedule` | Create scheduled agents |

### Extensions (7)

| Command | Purpose |
|---------|---------|
| `/install` | Install MCP servers |
| `/mcp` | Manage MCP servers |
| `/powerup` | Interactive lessons |

### Debug (6)

| Command | Purpose |
|---------|---------|
| `/doctor` | Diagnose issues |
| `/bug` | Report a bug |
| `/terminal-setup` | Configure terminal |

## 3.3 Key Built-in Commands in Detail

### `/compact` — Save Your Context

When your conversation gets long, context degrades. `/compact` compresses the history:

```
You: /compact
Claude: Compressed conversation from 45,000 tokens to 8,000 tokens.
```

Use this when:
- Claude starts forgetting earlier instructions
- You get "context window approaching limit" warnings
- You're switching to a different subtask in the same session

### `/init` — Bootstrap a Project

```
You: /init
Claude: I'll analyze your project and create a CLAUDE.md file...
```

This generates a starter CLAUDE.md based on your project structure, package.json, and common patterns.

### `/review` — Code Review

```
You: /review
Claude: I'll review the uncommitted changes...
```

Reviews your git diff for bugs, style issues, and improvements.

### `/pr-review` — Pull Request Review

```
You: /pr-review 123
```

Reviews a GitHub PR by number, analyzing all changes.

## 3.4 Custom Commands

Create your own commands by adding markdown files to `.claude/commands/`.

### File structure

```
.claude/
└── commands/
    └── my-command.md
```

### Frontmatter fields

```yaml
---
name: deploy-check          # Command name (used as /deploy-check)
description: Pre-deployment validation checks
arguments:
  - name: environment       # Named argument
    description: Target environment (staging/production)
    required: true
  - name: skip-tests
    description: Skip test suite
    required: false
    default: "false"
allowed-tools:              # Restrict available tools
  - Read
  - Bash
  - Agent
model: opus                 # Model override
permission-mode: default    # Permission mode
user-invocable: true        # User can call directly
---
```

### Example: Deploy Check Command

File: `.claude/commands/deploy-check.md`

```markdown
---
name: deploy-check
description: Run pre-deployment checks
arguments:
  - name: environment
    description: staging or production
    required: true
---

# Deploy Check: {{environment}}

Run these checks before deploying to {{environment}}:

1. Run the test suite: `pnpm test`
2. Run the linter: `pnpm lint`
3. Check for TypeScript errors: `pnpm tsc --noEmit`
4. Verify no console.log statements in production code
5. Check that all environment variables are documented
6. Review the git diff for any debug code

Report results as a checklist with pass/fail for each item.
```

Usage:
```
You: /deploy-check production
```

### Example: Feature Scaffold Command

File: `.claude/commands/new-feature.md`

```markdown
---
name: new-feature
description: Scaffold a new feature with all required files
arguments:
  - name: feature-name
    description: Name of the feature in kebab-case
    required: true
---

# New Feature: {{feature-name}}

Create the following files for the {{feature-name}} feature:

1. `src/features/{{feature-name}}/index.ts` — Feature entry point
2. `src/features/{{feature-name}}/types.ts` — TypeScript types
3. `src/features/{{feature-name}}/hooks.ts` — React hooks
4. `src/features/{{feature-name}}/components/` — Component directory
5. `src/features/{{feature-name}}/__tests__/` — Test directory

Follow the patterns established in existing features. Check existing features
in `src/features/` for reference before creating files.
```

## 3.5 Command vs Skill vs Agent

Understanding when to use each:

| Property | Command | Skill | Agent |
|----------|---------|-------|-------|
| Invocation | User types `/name` | Auto or `/name` | Spawned by Claude |
| Context | Shared (main session) | Shared (inline) | Isolated (separate) |
| Purpose | Entry point | Reusable procedure | Autonomous worker |
| Auto-invoke | Never | Yes (intent-based) | Yes (by Claude) |
| Best for | Workflows, orchestration | Repeated tasks | Research, parallel work |

**Rule of thumb**: Commands are the "what", skills are the "how", agents are the "who".

## 3.6 Mid-Response Commands

You can invoke some commands while Claude is still responding:

- `Escape` to stop, then type a command
- Some commands like `/compact` can be used between tool calls

---

## Exercise 3.1: Explore Built-in Commands

1. Start Claude Code and type `/` — browse the available commands
2. Try `/status` to see your session info
3. Try `/cost` to see token usage
4. Try `/model` to see and switch models

## Exercise 3.2: Create a Custom Command

1. Create `.claude/commands/code-review.md`:
```markdown
---
name: code-review
description: Review code in a specific file
arguments:
  - name: file
    description: Path to the file to review
    required: true
---

Review {{file}} for:
1. Bugs and logic errors
2. Security vulnerabilities
3. Performance issues
4. Code style and readability

Provide specific line numbers and concrete suggestions.
```

2. Restart Claude Code
3. Run: `/code-review src/index.ts`

## Exercise 3.3: Orchestration Command

1. Create a command that coordinates multiple steps:
   - Runs tests
   - Checks linting
   - Reviews git diff
   - Generates a summary
2. Test it on your project

---

## Common Pitfalls

- **Too many custom commands**: Start with 3-5 that you use daily. Don't create commands for one-off tasks.
- **Commands that are too vague**: "Review the code" is too vague. Specify what to check and how to report.
- **Not using arguments**: Hard-coding values makes commands inflexible. Use `{{argument}}` placeholders.
- **Putting business logic in commands**: Commands orchestrate. Put reusable logic in skills.

## Key Takeaways

1. 75+ built-in commands cover sessions, models, context, debugging, and extensions
2. `/compact` and `/clear` are essential for context management
3. Custom commands live in `.claude/commands/*.md` with YAML frontmatter
4. Commands are user-initiated entry points that orchestrate work
5. Use arguments with `{{name}}` syntax for flexibility
