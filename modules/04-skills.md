# Module 4: Skills — Reusable Procedures

**Duration**: 45 minutes | **Level**: Intermediate

## TL;DR

Skills are reusable, composable procedures that Claude can auto-invoke based on user intent or call explicitly via `/skill-name`. They execute inline in the main context and are the lightest-weight extension mechanism.

---

## 4.1 What Are Skills?

Skills are markdown files that define repeatable procedures. They differ from commands in two critical ways:

1. **Auto-invocation** — Claude can trigger skills automatically when it detects matching intent
2. **Inline execution** — Skills run in the current context, not in a separate process

Think of skills as "recipes" that Claude follows when a situation matches.

## 4.2 Skill File Structure

Skills live in `.claude/skills/` and use YAML frontmatter:

```
.claude/
└── skills/
    └── my-skill.md
```

### Frontmatter fields

```yaml
---
name: code-review-checklist
description: >
  Run a standardized code review checklist on changed files.
  Checks for security, performance, accessibility, and style issues.
arguments:
  - name: scope
    description: Files or directories to review
    required: false
    default: "git diff --name-only"
context:
  - path: .eslintrc.js          # Files to auto-load as context
    required: false
  - path: tsconfig.json
    required: false
allowed-tools:
  - Read
  - Bash
  - Edit
model: sonnet                   # Model to use
user-invocable: true            # Can be called with /skill-name
---
```

### Key frontmatter fields

| Field | Purpose |
|-------|---------|
| `name` | Identifier used for invocation |
| `description` | How Claude decides whether to auto-invoke (critical!) |
| `arguments` | Named parameters with defaults |
| `context` | Files to preload into context |
| `paths` | Directories where this skill is relevant |
| `allowed-tools` | Restrict which tools the skill can use |
| `model` | Override the model for this skill |
| `user-invocable` | Whether users can call it with `/name` |

## 4.3 Built-in Skills

Claude Code ships with several official skills:

| Skill | Purpose |
|-------|---------|
| `simplify` | Review changed code for reuse, quality, and efficiency |
| `loop` | Run a prompt on a recurring interval |
| `claude-api` | Build and debug Claude API applications |
| `fewer-permission-prompts` | Scan transcripts and add permission allowlists |
| `init` | Initialize CLAUDE.md for a project |
| `review` | Code review of current changes |
| `security-review` | Security-focused code review |

## 4.4 Writing Effective Skills

### Example: API Endpoint Skill

File: `.claude/skills/new-api-endpoint.md`

```markdown
---
name: new-api-endpoint
description: >
  Create a new REST API endpoint with validation, error handling,
  and tests. Use when the user asks to add an API route, endpoint,
  or handler.
arguments:
  - name: method
    description: HTTP method (GET, POST, PUT, DELETE)
    required: true
  - name: path
    description: URL path (e.g., /api/users)
    required: true
  - name: description
    description: What the endpoint does
    required: true
context:
  - path: src/lib/db/schema.ts
    required: false
  - path: src/app/api
    required: false
user-invocable: true
---

# Create API Endpoint: {{method}} {{path}}

## Steps

1. **Check existing patterns**: Read 2-3 existing API routes in `src/app/api/`
   to match the project's conventions

2. **Create the route handler** at the appropriate path under `src/app/api/`:
   - Input validation using zod schemas
   - Proper error responses with status codes
   - TypeScript types for request/response

3. **Add database queries** if needed:
   - Query functions in `src/lib/db/queries/`
   - Use existing Drizzle ORM patterns

4. **Create tests**:
   - Unit test for the handler logic
   - Integration test with test database

5. **Update API documentation** if an API doc file exists

## Conventions
- Return consistent JSON: `{ data: T }` for success, `{ error: string }` for errors
- Use HTTP status codes correctly (201 for creation, 204 for deletion)
- Log errors but don't expose internals to clients
```

### Example: Preloaded Agent Skill

Skills can be preloaded into agents (covered more in Module 5):

File: `.claude/skills/data-fetcher.md`

```markdown
---
name: data-fetcher
description: Fetch and parse data from external APIs
user-invocable: false
allowed-tools:
  - Bash
  - WebFetch
---

# Data Fetcher

When asked to fetch data from an external API:

1. Use WebFetch to call the API
2. Parse the response as JSON
3. Extract the relevant fields
4. Return structured data in this format:

{
  "source": "<api-url>",
  "timestamp": "<iso-datetime>",
  "data": <extracted-data>
}

Handle errors gracefully — return null for failed fetches with an error message.
```

Note `user-invocable: false` — this skill is designed to be loaded into an agent, not called directly.

## 4.5 The Description Field is Everything

The `description` field determines when Claude auto-invokes the skill. It must:

1. **Clearly state what the skill does**
2. **Include trigger phrases** — "Use when the user asks to..."
3. **Be specific enough** to avoid false triggers
4. **Be broad enough** to catch legitimate use cases

```yaml
# BAD: Too vague, will trigger on everything
description: Help with code

# BAD: Too narrow, will rarely trigger
description: Create a React component named UserProfile with Tailwind CSS

# GOOD: Clear scope with trigger hints
description: >
  Create a new React component with TypeScript types, styles, and tests.
  Use when the user asks to create, add, or scaffold a component.
```

## 4.6 Skill Best Practices (from Anthropic)

These are from Thariq Shihipar's published guidelines:

### 1. Avoid the Obvious
Don't write skills for things Claude already does well. A skill for "write a function" adds no value.

### 2. Build Gotchas Sections
Include known issues, edge cases, and workarounds:

```markdown
## Gotchas
- The auth middleware must be applied BEFORE the rate limiter
- PostgreSQL connection pool maxes at 20 — don't open new connections in loops
- The user.email field can be null for OAuth-only accounts
```

### 3. Leverage the File System
Use `context` to pull in relevant files automatically:

```yaml
context:
  - path: src/lib/auth/config.ts
  - path: .env.example
```

### 4. Avoid Railroading
Give Claude room to adapt. Don't script every line:

```markdown
# BAD: Over-specified
Write exactly this code:
```ts
export function getUser(id: string) {
  return db.users.findUnique({ where: { id } })
}
```

# GOOD: Pattern-guided
Follow the query pattern used in existing functions in `src/lib/db/queries/`.
The function should accept an ID parameter and return the full user object.
```

### 5. Design Setup Patterns
Include initialization steps if the skill needs prerequisites:

```markdown
## Setup (run once)
If the testing library is not installed:
1. Run `pnpm add -D vitest @testing-library/react`
2. Create `vitest.config.ts` following the pattern in the docs
```

### 6. Implement Memory Systems
Skills can reference project memory for consistency:

```markdown
Check CLAUDE.md for project-specific conventions before generating code.
If a `conventions.md` exists in the project root, follow those patterns.
```

## 4.7 Skill Categories

Organize skills by purpose:

| Category | Examples |
|----------|---------|
| **Library & API Reference** | API docs lookup, SDK usage patterns |
| **Code Scaffolding** | Component generators, route creators |
| **Code Quality** | Review checklists, refactoring guides |
| **Data Fetching** | API integration, data parsing |
| **Business Process** | Deployment checks, release workflows |
| **CI/CD** | Pipeline configuration, test runners |
| **Runbooks** | Incident response, debugging procedures |

---

## Exercise 4.1: Create a Review Skill

Create `.claude/skills/pr-checklist.md` that:
- Checks for common issues (console.log, TODO, hardcoded values)
- Verifies test coverage for changed files
- Reviews for security issues
- Outputs a pass/fail checklist

## Exercise 4.2: Auto-Invocation Test

1. Create a skill with a clear description trigger
2. Without calling the skill directly, make a request that matches the description
3. Observe whether Claude auto-invokes the skill

## Exercise 4.3: Skill with Context

1. Create a skill that uses the `context` field to preload project config
2. Verify that the skill has access to those files when invoked
3. Compare with a skill that doesn't preload — notice the extra tool calls

---

## Common Pitfalls

- **Skills that are just prompts**: A skill should encode project-specific knowledge, not generic instructions.
- **Over-specifying steps**: Let Claude adapt. Provide patterns, not scripts.
- **Missing description triggers**: If your skill never auto-invokes, the description is too narrow.
- **Skills that should be agents**: If the task needs isolation or parallel execution, use an agent instead.

## Key Takeaways

1. Skills are auto-invocable, inline procedures — the lightest extension mechanism
2. The `description` field drives auto-invocation — craft it carefully
3. Use `context` to preload relevant files automatically
4. Build skills around project-specific gotchas, not generic coding tasks
5. Skills can be user-invocable (`/skill-name`) or agent-preloaded (`user-invocable: false`)
6. Follow Anthropic's guidance: avoid railroading, include gotchas, leverage the file system
