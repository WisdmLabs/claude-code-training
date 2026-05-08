# Module 11: Tips, Tricks & Optimization

**Duration**: 60 minutes | **Level**: All

## TL;DR

A curated collection of practical tips from the Claude Code community and Anthropic engineers. Organized by topic: prompting, context, performance, debugging, and hidden features.

---

## 11.1 Prompting Tips

### The Four-Question Framework

Every effective Claude Code prompt answers four questions:

1. **What** specifically needs to change?
2. **Where** in the codebase? (file paths, line numbers)
3. **Why** -- the constraint or requirement driving the change?
4. **How** should Claude verify the result? (tests, linting, expected output)

```
# BAD
Fix the bug

# GOOD (answers all four)
The /api/users endpoint (src/routes/users.ts:42) returns 500 when
email is empty. Add validation to return 400 with a clear error
message. Run `pnpm test:api` to verify -- the existing test for
empty email should pass.
```

### Give Claude a Feedback Loop

This is the **single highest-leverage practice** according to community consensus. Include test commands, linter checks, or expected outputs so Claude can verify its own work:

```
Implement the login form. After each change, run `pnpm test:e2e`
to verify. Don't modify the test files.
```

Claude performs dramatically better when it can self-check.

### Give Claude the "why"

```
# BAD
Add a cache to the getUser function

# GOOD
The getUser function hits the database on every call, including repeated
calls in the same request. Add an in-memory cache with a 5-minute TTL
to reduce database load.
```

Claude generalizes better from motivated instructions. Instead of "never use var," say "never use var because we enforce strict ES6+ for consistency."

### Constraint Placement Matters

Put constraints at the **beginning**, not the end. When context gets long, the end of the prompt gets less attention. Critical constraints should also live in CLAUDE.md for persistence.

### Steering mid-conversation

If Claude goes in the wrong direction, **don't continue** -- the wrong response stays in context and pollutes subsequent attempts. Use `/rewind` or `/clear` immediately, then redirect:

```
Stop. Don't refactor the entire module. Only fix the null check on line 15.
```

### The "just" technique

```
Just add input validation to the createUser function. Nothing else.
```

"Just" constrains scope effectively.

## 11.2 Context Management Tips

### One task per session

The single most impactful practice. Don't reuse sessions across unrelated tasks.

### Don't Bust the Cache

- **Never manually edit files mid-session** -- this invalidates the prompt cache
- Cache expires after **5-15 minutes** of inactivity
- Disable format-on-save in your editor to prevent token burn from failed diffs
- Avoid reopening old sessions if the cache has expired

### Smart compaction

Don't wait for degradation, but use `/compact` strategically by steering what gets retained:

```
/compact Focus on the auth module and current test failures
```

When `/compact` fails with exhausted context, close Claude and restart with `claude --continue` to recover.

### The double-escape checkpoint

Build context by putting relevant codebase into the window. At each logical stopping point, hit **double-escape** to rewind to the context-filled checkpoint without re-spending those tokens.

### Use `--continue` for related follow-ups

```bash
# First session: implement the feature
claude

# Later: fix an issue found during testing
claude --continue
```

### Use `--add-dir` for multi-project context

```bash
claude --add-dir /path/to/shared-library
```

This adds another directory to Claude's context without changing your working directory.

### Use subagents for research

Instead of asking Claude to "read all the auth files and explain the flow" (consuming your main context), say:

```
Use an Explore agent to trace the authentication flow from login
to session creation. Report the key files and functions.
```

The research happens in isolated context.

## 11.3 Performance Tips

### Thinking budget keywords

Magic keywords trigger progressively larger thinking token budgets:

| Keyword | Budget | Use When |
|---------|--------|----------|
| *(default)* | Standard | Routine coding tasks |
| "think" | Extended | Multi-step problems |
| "think hard" | Large | Complex refactoring, debugging |
| "ultrathink" | Maximum | Architecture decisions, gnarly bugs |

Use "ultrathink" for genuinely complex problems. Default mode for routine work saves significant cost.

### Pin your model

```bash
claude --model opus
```

Or in environment:
```bash
export CLAUDE_MODEL=opus
```

Consistent model = consistent behavior.

### Use `/fast` for quick tasks

Toggle fast mode for simple questions or small edits:

```
/fast
What does the getUser function return?
```

Fast mode uses the same Opus model with faster output.

### Use `--bare` for minimal startup

```bash
claude --bare
```

Skips loading MCP servers and other extensions. Faster startup for quick tasks.

### Use `--print` for one-shot operations

```bash
claude --print "What's the export from src/utils/index.ts?"
```

No interactive session overhead.

## 11.4 Debugging Tips

### Use `/doctor` first

```
/doctor
```

Diagnoses installation, auth, MCP servers, and config issues.

### Check MCP servers

```
/mcp
```

Shows status of all configured MCP servers.

### Reset when stuck

```
/clear
```

Full context reset. Sometimes a fresh start is faster than debugging a degraded session.

### Use Claude to debug Claude

```
My last request didn't work as expected. What went wrong?
Can you trace through your approach and identify the issue?
```

### Enable verbose logging

```bash
claude --verbose
```

Shows detailed tool call information.

## 11.5 Git Integration Tips

### Review before committing

```
/review
```

Always review before committing. Claude catches bugs you'll miss.

### Branch for experiments

```
Create a new branch called "experiment/new-auth" and try implementing
the auth flow with JWT. If it doesn't work, we'll just delete the branch.
```

### Use Claude for commit messages

```
Look at the staged changes and write a conventional commit message.
```

### PR descriptions

```
Write a PR description for the current branch. Include:
- What changed and why
- How to test
- Any breaking changes
```

## 11.6 The Slot Machine Pattern

*From Anthropic's internal teams:*

> "Save state before letting Claude work, let it run for 30 minutes, then either accept the result or start fresh rather than trying to wrestle with corrections."

Key observations:
- Claude reaches **70-80% completion** on most tasks
- The remaining work requires human intervention
- **Attempting fixes often underperforms versus restarting fresh**
- Multiple concurrent attempts increase success rates compared to sequential refinement

Practical workflow:
1. Commit before starting
2. Let Claude run autonomously
3. Evaluate the result
4. Accept, or `git reset` and try again with a different prompt

### Git worktrees enable the pattern at scale

```bash
git worktree add ../attempt-1 -b attempt/v1
git worktree add ../attempt-2 -b attempt/v2
# Run Claude in each with slightly different prompts
# Pick the best result
```

## 11.7 CLAUDE.md Power Tips

### Keep it under 60 lines

Beyond ~60-80 lines, Claude starts ignoring parts. Be ruthlessly selective. For every line, ask: "Would Claude make a mistake without this?"

### Compliance numbers that matter

| Rule Type | Compliance Rate |
|-----------|----------------|
| Specific, concrete ("use pnpm, not npm") | ~89% |
| Detailed, well-formatted files | ~95% |
| Vague instructions ("write clean code") | ~35% |
| >200 instructions | Significant degradation |

### Pair "NEVER" with alternatives

```markdown
# GOOD
Never use `var`. Use `const` by default, `let` only when reassignment is needed.

# BAD
CRITICAL (PRIORITY 0)!!!! NEVER EVER USE VAR!!!!
```

Calm, clear directives work. Excessive urgency markers are ignored.

### Use emphasis keywords that actually work

`IMPORTANT`, `YOU MUST`, `CRITICAL`, and `NEVER` do influence Claude's behavior in CLAUDE.md -- but only when used sparingly. Overuse dilutes their effect.

## 11.8 Verification Loops

### Give Claude a way to check its work

Boris from the Anthropic team recommends visual verification tools (Puppeteer/Playwright MCP) as the **second-highest-leverage practice** after feedback loops. It provides a 2-3x reliability improvement.

### Self-review pattern

Ask Claude to review its own output before you do:

```
Review the code you just wrote. Look for logic errors, missing edge
cases, performance issues, and security vulnerabilities.
```

### Automated quality gates

Complement Claude with deterministic tools:
- **husky + lint-staged** for pre-commit formatting
- **Hooks** for auto-formatting and security scanning on every edit
- Linting on commit/write provides stronger compliance than CLAUDE.md instructions alone

## 11.9 CLIs Over MCP

A recurring theme from senior developers: convert stateless services to simple CLI wrappers rather than MCP servers.

```bash
# Instead of an MCP server for Jira:
jira-cli list --project MYPROJ --status "In Progress"

# Instead of an MCP server for AWS:
aws s3 ls s3://my-bucket/
```

CLIs are more intuitive for Claude. Write guidance into CLI `--help` text so Claude discovers usage naturally.

Use MCP for: authentication-heavy services, real-time data, tools needing complex state.

Audit unused MCPs: use `MCP-tidy` to check what you actually use. Unused servers still consume context with tool descriptions.

## 11.10 Hidden & Under-Utilized Features

### `/teleport` — Cross-device sessions

Transfer your session to another device:

```
/teleport
```

Generates a link that opens the same session on another device (phone, tablet, another computer).

### `/loop` — Recurring tasks

```
/loop 5m Check if CI has passed and notify me
```

### `/schedule` — Remote scheduled agents

```
/schedule "Run tests and report" --cron "0 9 * * 1-5"
```

Runs on Anthropic's infrastructure, not your machine.

### `/branch` — Quick branch management

Work on branches without leaving Claude Code:

```
/branch feature/new-api
```

### `/btw` — Asynchronous side-notes

Tell Claude something for later without interrupting current work:

```
/btw When you write tests, use Vitest not Jest
```

### `/batch` — Process multiple files

```
/batch "Add JSDoc comments" src/**/*.ts
```

### `/voice` — Voice input

Dictate instead of typing:

```
/voice
```

### `--agent` flag — Headless mode

Run Claude as a fully autonomous agent:

```bash
claude --agent "Implement the feature described in specs/new-feature.md"
```

### Ultrareview — Cloud-powered review

```
/ultrareview
```

Launches a multi-agent cloud review of the current branch. Multiple specialized agents review in parallel.

### Ultraplan — AI-powered planning

```
/ultraplan
```

Multi-agent planning for complex features.

## 11.11 Working with Large Codebases

### Use CLAUDE.md hierarchy

```
monorepo/
├── CLAUDE.md              ← Shared conventions
├── apps/
│   ├── web/CLAUDE.md      ← Web-specific
│   └── api/CLAUDE.md      ← API-specific
└── packages/
    └── shared/CLAUDE.md   ← Shared lib specifics
```

### Use Explore agents for discovery

```
Use an Explore agent to find all files that handle payment processing
```

### Be specific about scope

```
# BAD
Refactor the authentication system

# GOOD
Refactor src/auth/token-validator.ts to extract the JWT verification
into a separate function. Don't change any other files.
```

### Use `--add-dir` for cross-repo context

```bash
claude --add-dir ../shared-types
```

## 11.12 Session Strategy

### Short sessions for focused work

Each session should have a single goal:

```
Session 1: Implement the data model
Session 2: Add API endpoints
Session 3: Build the frontend components
Session 4: Write tests
```

### Long sessions for exploratory work

When exploring or learning a codebase, a longer session is fine. Use `/compact` regularly.

### `--continue` for follow-ups

```bash
# Morning: implement the feature
claude
# ... work ...
# Escape to exit

# Afternoon: fix issues found in review
claude --continue
```

### `--resume` for specific sessions

```bash
claude --resume
# Shows a list of recent sessions to choose from
```

## 11.13 Common Anti-Patterns

| Anti-pattern | Why it fails | Better approach |
|-------------|-------------|-----------------|
| "Read the entire codebase" | Fills context, slow | "Find files related to X" |
| Reusing sessions for unrelated tasks | Context degradation | One task per session |
| Not providing file paths | Claude guesses wrong | Give explicit paths |
| Asking Claude to "be careful" | No effect on behavior | Set specific constraints |
| Over-approving without reading | Dangerous changes slip through | Read the diff |
| Using Claude for simple grep | Slow, wastes tokens | Use `grep` directly |
| Long initial prompts | Most gets compressed | Front-load the critical info |
| Editing files manually mid-session | Busts prompt cache | Let Claude do all edits |
| Continuing after a wrong direction | Wrong output pollutes context | `/rewind` or `/clear` immediately |
| Over-stuffing CLAUDE.md (>200 rules) | Compliance drops significantly | Keep under 60 lines, be selective |
| Trusting AI-generated tests blindly | Misses edge cases, weak assertions | Treat as scaffolding, review carefully |
| Wrestling with corrections vs restarting | Diminishing returns | The slot machine pattern: accept or restart |

## 11.14 Language & Task Suitability

### Where Claude Excels

- **Python, JavaScript/TypeScript, Go** -- abundant training data
- Boilerplate/scaffolding and CRUD apps
- Refactoring to modern patterns
- SQL generation and web app prototyping
- Targeted refactoring of existing code

### Where to Exercise Caution

- **Rust** -- complex ownership semantics
- **Elasticsearch queries** -- subtle DSL issues
- Race conditions and concurrency bugs
- Complex multi-file changes across very large codebases
- Greenfield exploration without clear specs

## 11.15 Cost Optimization

### Typical costs (community data)

- **$0.50-0.75** per focused task
- **A few dollars** for prototype exploration
- Developer time typically exceeds API costs substantially

### Default to Sonnet for 80% of tasks

Opus costs ~5x more per token. Only switch to Opus for deep analysis or complex refactoring. Tactical model switching reduces costs **60-80%**.

| Task | Model | Why |
|------|-------|-----|
| Complex architecture | Opus | Best reasoning, better first-attempt adherence |
| Standard coding | Sonnet | Good quality, lower cost |
| Simple questions | Haiku | Fast and cheap |

### Use `--print` for one-shots

Non-interactive mode is cheaper -- no session management overhead.

### Compact aggressively

Less context = fewer input tokens per turn = lower cost. But steer compaction:

```
/compact Focus on the database migration and current errors
```

### Use subagents for research

Subagent context is discarded after returning results. The main session only pays for the summary.

### Monitor costs actively

Use `/cost` to track spending within a session. Set budget limits. Developers who actively monitor report **40-70% cost reductions** from the combination of model switching, clearing, and specific prompting.

### Scope tool permissions for subagents

| Agent Type | Tools to Grant |
|------------|---------------|
| Read-only (reviewers, auditors) | Read, Grep, Glob |
| Research agents | + WebFetch, WebSearch |
| Code writers | Read, Write, Edit, Bash, Glob, Grep |

This is both a security and a cost optimization practice.

---

## Exercise 11.1: Optimize Your Setup

1. Create a global `~/.claude/CLAUDE.md` -- keep it under 60 lines
2. Set up your most-used permissions in `~/.claude/settings.json`
3. Add a PostToolUse hook to auto-format files Claude writes
4. Try `--bare` for a quick question

## Exercise 11.2: The Four-Question Prompt

Take a real task and write it using the four-question framework:
1. What specifically needs to change?
2. Where in the codebase?
3. Why?
4. How should Claude verify?

Compare the result quality against a vague version of the same prompt.

## Exercise 11.3: The Slot Machine Pattern

1. Commit your current state
2. Give Claude a medium-complexity task
3. Let it run autonomously for 10-15 minutes
4. Evaluate: accept or `git reset` and try with a different prompt
5. Compare this against your usual back-and-forth approach

## Exercise 11.4: Git Worktree Parallelism

1. Create two worktrees for independent tasks:
   ```bash
   git worktree add ../task-a -b feature/task-a
   git worktree add ../task-b -b feature/task-b
   ```
2. Run Claude in each with clear, focused plans
3. Review each branch's output independently
4. Merge the best results

## Exercise 11.5: Try Hidden Features

1. Try `/teleport` to transfer a session
2. Try `/loop` to monitor something
3. Try `/voice` for voice input
4. Try different thinking keywords: "think", "think hard", "ultrathink"

---

## Further Reading

- [Community Tips & Insights](../resources/community-tips.md) -- deep dives on every topic in this module
- [Plugins, Tools & Ecosystem Directory](../resources/plugins-and-tools.md) -- curated directory of 800+ plugins and tools
- [Configuration Best Practices](../resources/config-best-practices.md) -- production-grade config guide

---

## Key Takeaways

1. **Give Claude a feedback loop** -- the single highest-leverage practice
2. **One task per session** -- the #1 productivity tip
3. **Use the four-question prompt framework** -- what, where, why, how to verify
4. **Don't bust the cache** -- avoid manual edits mid-session
5. **The slot machine pattern** -- accept or restart, don't wrestle
6. **Git worktrees** -- the power user unlock for parallelism
7. **Keep CLAUDE.md under 60 lines** -- specific rules beat verbose docs
8. **Match thinking budget to task** -- ultrathink for complex, default for routine
9. **Match model to task** -- Opus for complexity, Sonnet for 80% of work
10. **Read the diff before approving** -- trust but verify
