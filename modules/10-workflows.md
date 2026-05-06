# Module 10: Real-World Workflows & Patterns

**Duration**: 60 minutes | **Level**: Advanced

## TL;DR

This module covers production-tested workflow patterns for common engineering tasks: feature development, bug fixing, code review, refactoring, and CI/CD integration. These patterns combine commands, skills, agents, and hooks into complete workflows.

---

## 10.1 The "Spec Kit" Workflow

Write a specification before coding. This is the most impactful workflow for non-trivial features.

### The process

1. **Write the spec** in a markdown file
2. **Have Claude review** the spec for gaps
3. **Implement from the spec** — Claude follows it as the source of truth
4. **Validate against the spec** — check that implementation matches

### Setup

Create `.claude/commands/spec.md`:

```markdown
---
name: spec
description: Create or implement from a feature specification
arguments:
  - name: action
    description: create, review, implement, or validate
    required: true
  - name: feature
    description: Feature name
    required: true
---

# Spec: {{action}} {{feature}}

Based on the action:

**create**: Write a specification in `specs/{{feature}}.md` with:
- Problem statement
- Proposed solution
- API/interface design
- Edge cases
- Success criteria
- Out of scope

**review**: Read `specs/{{feature}}.md` and identify gaps, ambiguities,
and missing edge cases.

**implement**: Read `specs/{{feature}}.md` and implement it step by step.
Check off each requirement as you complete it.

**validate**: Compare the implementation against `specs/{{feature}}.md`.
Report any deviations or missing requirements.
```

### Usage

```
/spec create user-auth
# ... review and refine the spec ...
/spec implement user-auth
/spec validate user-auth
```

## 10.2 The "Get It Done" Workflow

For straightforward tasks where you want Claude to work autonomously with minimal back-and-forth.

### Setup

```bash
claude --permission-mode auto
```

Combined with a focused CLAUDE.md:

```markdown
## Working Mode
- When given a task, implement it fully without asking clarifying questions
- Run tests after every change
- Fix any test failures before reporting done
- Commit with conventional commit messages
```

### Usage

```
Add pagination to the /api/posts endpoint. Support cursor-based pagination
with a default page size of 20. Update the frontend to use infinite scroll.
Run the test suite when done.
```

Claude will implement, test, fix, and report.

## 10.3 Bug Fix Workflow

A systematic approach to debugging:

### Step 1: Reproduce

```
The login form shows "Invalid credentials" even with correct password.
Steps to reproduce:
1. Go to /login
2. Enter valid email and password
3. Click "Sign In"
4. Error message appears

Expected: Redirect to dashboard
Actual: "Invalid credentials" error
```

### Step 2: Let Claude investigate

Claude will:
1. Read the auth handler code
2. Check password hashing logic
3. Trace the request flow
4. Find the root cause

### Step 3: Fix and verify

```
Fix the bug and add a test that covers this case.
```

### Pro tip: Use subagents for investigation

```
Use an Explore agent to find all files related to authentication
and password verification. Then fix the bug.
```

The Explore agent searches without consuming main context.

## 10.4 Code Review Workflow

### Built-in review

```
/review
```

Reviews uncommitted changes for bugs, style, and improvements.

### PR review

```
/pr-review 123
```

Reviews a GitHub pull request.

### Custom review with specialized agents

Create a review command that spawns parallel agents:

```markdown
---
name: deep-review
description: Deep code review with multiple specialized agents
arguments:
  - name: target
    description: File or directory to review
    required: true
---

# Deep Review: {{target}}

Spawn these agents in parallel:

1. **Security Agent**: Check for OWASP Top 10 vulnerabilities
2. **Performance Agent**: Check for N+1 queries, unnecessary renders, large imports
3. **Quality Agent**: Check for code smells, naming, complexity

Combine all findings into a single prioritized report.
```

## 10.5 Refactoring Workflow

### Safe refactoring pattern

1. **Understand first**: Ask Claude to explain the current code
2. **Test first**: Ensure existing tests pass
3. **Refactor in small steps**: Each step should pass tests
4. **Run tests after each step**: Catch regressions immediately

```
I want to refactor the UserService class. Before changing anything:
1. Run the existing tests to confirm they pass
2. Explain the current structure and responsibilities
3. Propose a refactoring plan
4. Wait for my approval before implementing
```

### Use plan mode

```bash
claude --permission-mode plan
```

Claude creates a plan, you approve, then it executes. Perfect for refactoring where you want oversight.

## 10.6 CI/CD Integration with Print Mode

Claude Code can run in CI/CD pipelines using `--print` mode:

### GitHub Actions example

```yaml
name: AI Code Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: Review PR
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          claude --print "Review the changes in this PR. Focus on bugs, 
          security issues, and performance problems. Output as a markdown 
          checklist." > review.md

      - name: Post review comment
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const review = fs.readFileSync('review.md', 'utf8');
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: review
            });
```

### Automated commit message generation

```bash
git diff --staged | claude --print "Write a conventional commit message for these changes"
```

### Pre-commit hook with Claude

```bash
#!/bin/bash
# .git/hooks/pre-commit
claude --print "Review the staged changes for any obvious bugs or security issues. 
Reply with PASS if okay, or FAIL: <reason> if not." | grep -q "^PASS" || exit 1
```

## 10.7 Scheduled Tasks with `/loop`

For recurring tasks:

```
/loop 5m Check if the build succeeded on CI and notify me
```

```
/loop 10m Monitor the error rate in the production logs
```

Tasks auto-expire after 3 days. Cancel with `cron cancel`.

### Scheduled remote agents with `/schedule`

```
/schedule "Run test suite and report failures" --cron "0 8 * * *"
```

This creates a remote agent that runs daily at 8 AM.

## 10.8 The "Superpowers" Pattern

From the community: give Claude explicit permission to be bold.

```
You have superpowers. You can:
- Make architectural decisions
- Refactor aggressively
- Delete dead code
- Rename things for clarity
- Reorganize file structure

I trust your judgment. Go ahead and improve this module.
```

This works because Claude is conservative by default. Explicit permission unlocks more ambitious changes.

## 10.9 Multi-File Feature Development

For features spanning many files:

```
Implement the notification system. Here's what I need:

1. Database: notifications table with user_id, type, message, read_at
2. API: CRUD endpoints at /api/notifications
3. Backend: NotificationService with create, markRead, getUnread
4. Frontend: NotificationBell component with dropdown
5. Real-time: WebSocket push for new notifications

Start with the database layer and work up. Run tests at each layer.
```

### Pro tip: Use plan mode for review points

```bash
claude --permission-mode plan
```

Claude proposes each step, you approve, it implements. This gives you review points between layers.

---

## Exercise 10.1: Spec-Driven Feature

1. Create the spec command from section 10.1
2. Use `/spec create` to write a spec for a small feature
3. Use `/spec review` to find gaps
4. Use `/spec implement` to build it
5. Use `/spec validate` to verify completeness

## Exercise 10.2: CI/CD Integration

1. Create a GitHub Actions workflow that uses Claude Code for PR reviews
2. Set up the `ANTHROPIC_API_KEY` secret
3. Open a PR and see the automated review

## Exercise 10.3: Custom Review Pipeline

1. Build the `deep-review` command from section 10.4
2. Run it on a real directory in your project
3. Compare the output with a manual review

---

## Common Pitfalls

- **Skipping the spec**: For features over 2 hours of work, write a spec. It saves time.
- **Not using plan mode for risky changes**: Refactoring without review points leads to regret.
- **Running Claude in CI without cost limits**: Add token limits to prevent runaway costs.
- **Too many parallel agents**: Spawning 10 agents produces noise. 3-5 focused agents is the sweet spot.

## Key Takeaways

1. Spec-driven development is the highest-impact workflow for non-trivial features
2. Print mode (`--print`) enables CI/CD integration and scripting
3. Plan mode gives review points during complex changes
4. Parallel review agents catch issues across different dimensions
5. `/loop` and `/schedule` enable recurring automated tasks
6. Give explicit permission for bold changes when appropriate
