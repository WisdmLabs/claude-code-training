# Module 11: Tips, Tricks & Optimization

**Duration**: 45 minutes | **Level**: All

## TL;DR

A curated collection of practical tips from the Claude Code community and Anthropic engineers. Organized by topic: prompting, context, performance, debugging, and hidden features.

---

## 11.1 Prompting Tips

### Be specific, not vague

```
# BAD
Fix the bug

# GOOD
The /api/users endpoint returns 500 when the email field is empty.
Fix the validation in src/api/users.ts to return a 400 with a clear error message.
```

### Give Claude the "why"

```
# BAD
Add a cache to the getUser function

# GOOD
The getUser function hits the database on every call, including repeated
calls in the same request. Add an in-memory cache with a 5-minute TTL
to reduce database load.
```

### Use file references

In Claude Code, you can reference files directly:

```
Look at src/auth/middleware.ts — the token validation on line 42 doesn't
handle expired tokens. Fix it.
```

### Steering mid-conversation

If Claude goes in the wrong direction, redirect firmly:

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

### Use `/compact` proactively

Don't wait for degradation. Compact after:
- Reading large files
- Long command outputs
- Completing a subtask before starting the next

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

## 11.6 Hidden & Under-Utilized Features

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

## 11.7 Working with Large Codebases

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

## 11.8 Session Strategy

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

## 11.9 Common Anti-Patterns

| Anti-pattern | Why it fails | Better approach |
|-------------|-------------|-----------------|
| "Read the entire codebase" | Fills context, slow | "Find files related to X" |
| Reusing sessions for unrelated tasks | Context degradation | One task per session |
| Not providing file paths | Claude guesses wrong | Give explicit paths |
| Asking Claude to "be careful" | No effect on behavior | Set specific constraints |
| Over-approving without reading | Dangerous changes slip through | Read the diff |
| Using Claude for simple grep | Slow, wastes tokens | Use `grep` directly |
| Long initial prompts | Most gets compressed | Front-load the critical info |

## 11.10 Cost Optimization

### Use the right model

| Task | Model | Why |
|------|-------|-----|
| Complex architecture | Opus | Best reasoning |
| Standard coding | Sonnet | Good quality, lower cost |
| Simple questions | Haiku | Fast and cheap |

### Use `--print` for one-shots

Non-interactive mode is cheaper — no session management overhead.

### Compact aggressively

Less context = fewer input tokens per turn = lower cost.

### Use subagents for research

Subagent context is discarded after returning results. The main session only pays for the summary.

---

## Exercise 11.1: Optimize Your Setup

1. Create a global `~/.claude/CLAUDE.md` with your preferences
2. Set up your most-used permissions in `~/.claude/settings.json`
3. Configure environment variables for your defaults
4. Try `--bare` for a quick question

## Exercise 11.2: Session Strategy

1. Plan a multi-session approach for a medium feature:
   - Session 1: spec and planning
   - Session 2: implementation
   - Session 3: testing and review
2. Execute it, using `--continue` for follow-ups

## Exercise 11.3: Try Hidden Features

1. Try `/teleport` to transfer a session
2. Try `/loop` to monitor something
3. Try `/voice` for voice input
4. Try `/batch` to process multiple files

---

## Key Takeaways

1. Be specific in prompts — file paths, line numbers, clear constraints
2. One task per session is the #1 productivity tip
3. Use `/compact` proactively, not reactively
4. Subagents protect main context from research clutter
5. Hidden features like `/teleport`, `/loop`, and `/batch` are productivity multipliers
6. Match the model to the task: Opus for complexity, Sonnet for standard work, Haiku for simple queries
7. Read the diff before approving — trust but verify
