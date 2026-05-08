# Community Tips & Insights

Best tips sourced from GitHub, Reddit, and Hacker News -- curated from thousands of developer discussions and validated by community consensus.

---

## CLAUDE.md Mastery

### Keep It Under 60 Lines

Boris from the Claude Code team recommends keeping CLAUDE.md under 1,000 tokens. The community has found that beyond ~60-80 lines, Claude starts ignoring parts. For every line, ask: "Would Claude make a mistake without this?"

The Anthropic team updates theirs multiple times weekly and **removes** entries as newer models require less guidance.

### Compliance Numbers That Matter

Research from the community shows:
- Specific, concrete rules ("use pnpm, not npm") -- **~89% compliance**
- Detailed, well-formatted files -- **~95% compliance**
- Vague instructions ("write clean code") -- **~35% compliance**
- Single-rule files without structure -- **~50% compliance**

The practical limit is **150-200 instructions** before compliance degrades significantly.

### Progressive Disclosure

Don't stuff everything into one root file. Use a table-of-contents approach:

```markdown
# CLAUDE.md (root)
## Architecture
See [docs/architecture.md](docs/architecture.md) for system design.

## API Conventions
See [docs/api-conventions.md](docs/api-conventions.md) for endpoint patterns.
```

Claude automatically pulls in CLAUDE.md files from subdirectories when accessing those directories.

### Effective Rule Patterns

```markdown
# GOOD: Pair "never" with alternatives
Never use `var`. Use `const` by default, `let` only when reassignment is needed.

# GOOD: Conditional formatting
If writing a React component, use functional components with hooks. Never use class components.

# GOOD: Emphasis keywords that work
IMPORTANT: All API responses must include a `requestId` field for tracing.
YOU MUST run `pnpm check` before committing.

# BAD: Performative urgency
CRITICAL (PRIORITY 0)!!!! NEVER EVER DO THIS!!!!
# (Excessive markers are ignored. Calm, clear directives work better.)
```

### Token-Saving Output Rules

Add these to CLAUDE.md to reduce token waste:

```markdown
- Answer first, reasoning after. Never lead with explanation.
- No sycophantic openers ("Great question!", "Absolutely!")
- Don't repeat information already established in this session.
```

---

## The Planning-First Paradigm

### Plan Mode Multiplies Results 2-3x

Boris from the Anthropic team says using Plan mode (Shift+Tab) before complex tasks multiplies results 2-3x. The workflow:

1. Enter plan mode (Shift+Tab twice)
2. Describe what you want at a high level
3. Go back and forth until the plan looks right
4. Let Claude execute

Plans are automatically written to files, so you can review and edit before execution.

### The Clarifying Questions Method

Ask Claude to generate clarifying questions about what you want to implement. Write those into a `planning.md`. Then have Claude answer them. One developer noted: "Letting Claude answer sometimes gives you surprising answers that broaden your vision on the problem."

### POC-Driven Development

1. Use Claude to understand the problem deeply
2. Code the core solution yourself (even uncompiled)
3. Fire up Claude: "I need to do X, uncommitted we have a POC -- create implementation plan"

This leverages Claude's strength in reading/explaining code while you handle architecture.

### Spec Investment Pays Off

One developer spent 2 hours writing a 12-step implementation plan, which Claude executed step-by-step, saving 6-10 hours of manual coding. The investment in specification always pays off.

---

## Context & Cache Management

### Don't Bust the Cache

- **Never manually edit files mid-session** -- this invalidates the prompt cache
- Cache expires after **5-15 minutes** of inactivity
- Avoid reopening old sessions if the cache has expired
- Disable format-on-save in your editor to prevent token burn from failed diffs

### The Double-Escape Checkpoint

Build context by putting relevant codebase into the window. At each logical stopping point, hit **double-escape** to rewind to the context-filled checkpoint without re-spending those tokens.

### Smart Compaction

`/compact` can lose important context. Use it strategically:

```
/compact Focus on the auth module and current test failures
```

Steering what gets retained makes compaction dramatically more useful.

When `/compact` fails (it does ~50% of the time with exhausted context), close Claude and restart with `claude --continue` to recover.

### How Compaction Actually Works

Claude Code uses three compaction types internally:
1. **Full compaction** -- API summarizes old messages
2. **Session memory compaction** -- cheaper alternative
3. **Microcompaction** -- clears old tool results after 1+ hour idle, keeping only the 5 most recent

---

## Thinking Budget Control

Magic keywords trigger progressively larger thinking token budgets:

| Keyword | Budget | Use When |
|---------|--------|----------|
| *(default)* | Standard | Routine coding tasks |
| "think" | Extended | Multi-step problems |
| "think hard" | Large | Complex refactoring, debugging |
| "ultrathink" | Maximum | Architecture decisions, gnarly bugs |

Use "ultrathink" for genuinely complex problems. Default mode for routine work saves significant cost.

---

## The Slot Machine Pattern

*From Anthropic's internal teams:*

> "Save state before letting Claude work, let it run for 30 minutes, then either accept the result or start fresh rather than trying to wrestle with corrections."

Key observations:
- Claude reaches **70-80% completion** on most tasks
- The remaining work requires human intervention
- **Attempting fixes often underperforms versus restarting fresh**
- Multiple concurrent attempts increase success rates compared to sequential refinement

Practical workflow: commit before starting, let Claude run autonomously, evaluate the result, accept or `git reset` and try again with a different prompt.

---

## Git Worktrees: The Power User Unlock

The most consistently praised pattern across all platforms. Use `git worktree` to create multiple working directories from the same repo, each on its own branch.

```bash
git worktree add ../feature-auth -b feature/auth
git worktree add ../feature-api -b feature/api
git worktree add ../bugfix-login -b fix/login

# Run separate Claude instances in each
cd ../feature-auth && claude
cd ../feature-api && claude
cd ../bugfix-login && claude
```

### Why It Works

- No stashing or context switching -- each worktree has its own files
- Each agent's work becomes a focused, manageable code review
- Anthropic's team reports running **10-15 parallel sessions**
- Practical sweet spot: **3-5 concurrent worktrees**

### Worktree Tooling

| Tool | Purpose |
|------|---------|
| **Harness** | Manage parallel agents across worktrees |
| **Gwt-Claude** | Parallel sessions with git worktrees |
| **wt / Wtx** | Worktree management utilities |
| **FleetCode** | Fleet management for parallel agents |
| **Branchlet** | Lightweight worktree orchestration |

### Tips

- Automate worktree creation with scripts that copy `.env` files and remap container ports
- Scope worktrees by **module**, not by task
- Each worktree should have a clear, focused plan before Claude starts

---

## Verification Loops

### Give Claude a Way to Check Its Work

Boris recommends this as the **highest-leverage practice**. Puppeteer MCP or similar tools let Claude verify changes in a browser, providing a 2-3x improvement in reliability.

```
Implement the login form. After each change, use Playwright to verify:
1. The form renders correctly
2. Validation messages appear for empty fields
3. Successful login redirects to /dashboard

Run the tests after implementing. Do not modify the test files.
```

### Self-Review Pattern

Ask Claude to review its own output before you do:

```
Review the code you just wrote. Look for:
- Logic errors
- Missing edge cases
- Performance issues
- Security vulnerabilities
```

One HN user noted: "It immediately dumps on its own last output, talking both of us out of it, usually with good reasons."

### Automated Quality Gates

Complement Claude with deterministic tools:
- **husky + lint-staged + commitlint** for pre-commit enforcement
- **CodeRabbit / Sourcery** for automated code review
- **Hooks** for formatting and quality checks on every edit

---

## CLIs Over MCP

A recurring theme from senior developers: convert stateless services to simple CLI wrappers rather than MCP servers.

```bash
# Instead of an MCP server for Jira:
jira-cli list --project MYPROJ --status "In Progress"

# Instead of an MCP server for AWS:
aws s3 ls s3://my-bucket/
```

**Why:** CLIs are more intuitive for Claude than tool abstractions. Write guidance directly into CLI help text so Claude discovers how to use them naturally.

**When MCP is better:** Authentication-heavy services, real-time data streams, tools that need complex state management.

**Audit unused MCPs:** Use `MCP-tidy` to check which MCP servers you actually use. Unused ones still consume context with their tool descriptions.

---

## The Four-Question Prompt Framework

Every effective Claude Code prompt answers four questions:

1. **What** specifically needs to change?
2. **Where** in the codebase? (file paths, line numbers)
3. **Why** -- the constraint or requirement driving the change?
4. **How** should Claude verify the result? (tests, linting, expected output)

```
# Example applying all four:
The /api/users endpoint (src/routes/users.ts:42) returns 500 when
email is empty. Add validation to return 400 with a clear error
message. Run `pnpm test:api` to verify -- the existing test for
empty email should pass.
```

### Constraint Placement

Put constraints at the **beginning**, not the end. When context gets long, the end of the prompt gets less attention. Critical constraints should also live in CLAUDE.md for persistence.

---

## Language & Task Suitability

### Where Claude Excels

- **Python, JavaScript/TypeScript, Go** -- abundant training data, simpler structures
- Boilerplate/scaffolding and CRUD apps
- Refactoring to modern patterns
- SQL generation and web app prototyping
- Targeted refactoring of existing code

### Where to Exercise Caution

- **Rust** -- complex ownership semantics
- **Elasticsearch queries** -- subtle DSL issues
- **Complex SQL** -- operator precedence subtleties
- Race conditions and concurrency bugs
- Complex multi-file changes across large codebases
- Greenfield exploration without clear specs

---

## Security Awareness

### Untrusted Repositories

A disclosed vulnerability: if you start Claude Code in an attacker-controlled repository with a crafted `.claude/settings.json` that sets `ANTHROPIC_BASE_URL` to an attacker endpoint, Claude Code sends API requests (including API keys) before showing the trust prompt.

**Rule:** Always review `.claude/settings.json` before trusting a cloned project.

### AI-Generated Code Review

Claude's code often looks correct but may contain:
- Placeholder comments masking incomplete logic
- Subtle bugs in error handling paths
- Over-engineered abstractions
- Tests that assert the implementation rather than the requirement

**Always review the diff.** Treat generated tests as scaffolding, not protection.

---

## Session Knowledge Extraction

At the end of productive sessions, route learnings to the right place:

| Learning Type | Destination |
|---------------|-------------|
| General project concepts | CLAUDE.md |
| Good practices / conventions | Rules files (`~/.claude/rules/`) |
| Technical procedures | Skill files (`.claude/skills/`) |
| Architectural choices | ADRs (Architecture Decision Records) |

Ask Claude: "What did you learn during this session that should be persisted?" Then curate the output.

---

## Sources

### GitHub
- [awesome-claude-code-toolkit](https://github.com/rohitg00/awesome-claude-code-toolkit) -- 135 agents, 176+ plugins, 42 commands
- [claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice) -- 20k+ stars
- [claude-code-tips](https://github.com/ykdojo/claude-code-tips) -- 45 tips from basics to advanced
- [claude-code-ultimate-guide](https://github.com/FlorianBruniaux/claude-code-ultimate-guide)
- [awesome-claude-plugins](https://github.com/quemsah/awesome-claude-plugins) -- 834+ plugins tracked
- [VILA-Lab/Dive-into-Claude-Code](https://github.com/VILA-Lab/Dive-into-Claude-Code) -- Architecture deep dive

### Reddit & Blogs
- [50 Claude Code Tips and Best Practices - Builder.io](https://www.builder.io/blog/claude-code-tips-best-practices)
- [Claude Code Tips I Wish I Had From Day One - Marmelab](https://marmelab.com/blog/2026/04/24/claude-code-tips-i-wish-id-had-from-day-one.html)
- [32 Claude Code Tips - YK Sugi](https://agenticcoding.substack.com/p/32-claude-code-tips-from-basics-to)
- [How to Hyper-Optimise Claude Code - DEV Community](https://dev.to/andrei_nita/how-to-hyper-optimise-claude-code-the-complete-engineering-guide-1eh3)

### Hacker News
- [Ask HN: How Do You Actually Use Claude Code Effectively?](https://news.ycombinator.com/item?id=44362244)
- [Ask HN: What are your best practices for Claude Code?](https://news.ycombinator.com/item?id=44776941)
- [Boris from Claude Code team tips](https://news.ycombinator.com/item?id=46256606)
- [How Anthropic teams use Claude Code](https://news.ycombinator.com/item?id=44678535)
- [Collation of Claude Code Best Practices v2](https://news.ycombinator.com/item?id=45830267)
