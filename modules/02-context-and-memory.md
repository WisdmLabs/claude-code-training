# Module 2: Context & Memory System

**Duration**: 45 minutes | **Level**: Beginner

## TL;DR

CLAUDE.md files are persistent instructions that load into every session. They form a hierarchy from global to project-specific. Use them for coding standards, project context, and behavioral rules.

---

## 2.1 What is CLAUDE.md?

CLAUDE.md is a markdown file that Claude Code reads at session start. It acts as persistent memory — instructions that survive across sessions.

Think of it as a `.editorconfig` for AI: it tells Claude how to behave in this project.

## 2.2 The Configuration Hierarchy

CLAUDE.md files load from five levels, in order of precedence:

```
1. ~/.claude/CLAUDE.md              ← Global (all projects)
2. <project-root>/CLAUDE.md         ← Project-wide
3. <project-root>/.claude/CLAUDE.md ← Project-wide (alternative location)
4. <subdirectory>/CLAUDE.md         ← Component-specific (lazy-loaded)
5. Managed policies                 ← Enterprise/org-level (highest precedence)
```

### Loading rules

- **Ancestor files** (levels 1-3): Always loaded at session start
- **Descendant files** (level 4): Lazy-loaded when Claude accesses that directory
- **Sibling files**: Never loaded (a `frontend/CLAUDE.md` won't load when working in `backend/`)

## 2.3 What to Put in CLAUDE.md

### Global `~/.claude/CLAUDE.md`

Personal preferences that apply everywhere:

```markdown
## My Preferences
- Use TypeScript strict mode
- Prefer functional components in React
- Always use pnpm, not npm
- Write tests with Vitest, not Jest
```

### Project-root `CLAUDE.md`

Project-specific standards:

```markdown
## Project: My SaaS App

### Tech Stack
- Next.js 15 with App Router
- TypeScript strict
- Tailwind CSS v4
- Drizzle ORM with PostgreSQL

### Coding Standards
- Use server components by default
- Client components only when interactivity is needed
- API routes in app/api/ using route handlers
- Database queries in lib/db/queries/

### Commands
- Dev server: `pnpm dev`
- Tests: `pnpm test`
- Lint: `pnpm lint`
- Build: `pnpm build`

### Git
- Conventional commits: feat:, fix:, refactor:, docs:, test:
- One logical change per commit
- Always run tests before committing
```

### Subdirectory `frontend/CLAUDE.md`

Component-specific instructions that only load when Claude works in that directory:

```markdown
## Frontend Conventions
- Components in PascalCase directories
- Each component has: index.tsx, styles.module.css, types.ts
- Use React Hook Form for all forms
- Use Tanstack Query for data fetching
```

## 2.4 Monorepo Structure Example

```
my-monorepo/
├── CLAUDE.md                 ← Shared conventions (always loaded)
├── packages/
│   ├── web/
│   │   └── CLAUDE.md         ← Web app specifics (lazy)
│   ├── api/
│   │   └── CLAUDE.md         ← API specifics (lazy)
│   └── shared/
│       └── CLAUDE.md         ← Shared lib specifics (lazy)
```

The root CLAUDE.md might contain:
```markdown
## Monorepo Structure
- packages/web — Next.js frontend
- packages/api — Express backend
- packages/shared — Shared types and utilities

## Shared Conventions
- All packages use TypeScript strict
- Shared types go in packages/shared
- No circular dependencies between packages
```

## 2.5 Settings Files

Beyond CLAUDE.md, Claude Code uses JSON settings files:

### Global settings: `~/.claude/settings.json`

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Edit",
      "Write",
      "Bash(git *)",
      "Bash(pnpm *)"
    ]
  },
  "env": {
    "CLAUDE_MODEL": "opus"
  }
}
```

### Project settings: `.claude/settings.json`

```json
{
  "permissions": {
    "allow": [
      "Bash(pnpm dev)",
      "Bash(pnpm test)",
      "Bash(pnpm lint)"
    ]
  }
}
```

### Settings precedence

```
local (.claude/settings.local.json)  ← Highest
project (.claude/settings.json)
global (~/.claude/settings.json)       ← Lowest
```

## 2.6 Persistent Memory System

Beyond CLAUDE.md (which is instruction-based), Claude Code has a **memory system** that stores facts and learnings across sessions.

### How It Works

Memory files live in `~/.claude/projects/<project-path>/memory/`:

```
~/.claude/projects/<project-path>/memory/
├── MEMORY.md                  ← Index file (always loaded)
├── user_role.md               ← Who you are
├── feedback_testing.md        ← Behavioral corrections
├── project_auth_rewrite.md    ← Project decisions
└── reference_linear.md        ← External system pointers
```

### Memory Types

| Type | What to Store | Example |
|------|---------------|---------|
| **user** | Role, preferences, expertise | "Senior backend engineer, new to React" |
| **feedback** | Corrections to Claude's approach | "Don't mock the database in integration tests" |
| **project** | Decisions, goals, deadlines | "Auth rewrite driven by compliance, not tech debt" |
| **reference** | Pointers to external systems | "Bugs tracked in Linear project INGEST" |

### Using Memory

- **Ask Claude to remember**: "Remember that we use Vitest, not Jest"
- **Check memory**: `/memory` shows current project memories
- **Claude saves automatically** when it learns relevant context from your corrections

### What NOT to Store in Memory

- Code patterns or conventions → put in CLAUDE.md
- Git history or recent changes → use `git log`
- Debugging solutions → the fix is in the code
- Temporary task details → use the conversation

### MEMORY.md

The `MEMORY.md` index file is loaded into every session (like CLAUDE.md). Keep it under 200 lines -- each entry should be one line:

```markdown
- [User role](user_role.md) — Senior backend engineer, Go expert, React beginner
- [Testing feedback](feedback_testing.md) — Always use real DB in integration tests
- [Auth rewrite](project_auth_rewrite.md) — Compliance-driven, deadline March 15
```

## 2.7 Excluding Files from Context

### `.claudeignore`

Create a `.claudeignore` file (like `.gitignore`) to exclude files from Claude's context:

```
# Build artifacts
dist/
build/
.next/

# Large generated files
*.min.js
*.bundle.js

# Binary files
*.png
*.jpg
*.woff2

# Dependencies
node_modules/
vendor/
```

This prevents Claude from wasting context on files it shouldn't read.

## 2.8 Design Principles

1. **Ancestors always load** — Root-level CLAUDE.md is always in context
2. **Descendants load on demand** — Subdirectory files only load when Claude visits them
3. **Siblings never load** — Working in `frontend/` won't load `backend/CLAUDE.md`
4. **Keep it concise** — Every line consumes context. Trim ruthlessly.
5. **Instructions, not documentation** — Tell Claude what to DO, not how things work

### Anti-patterns

```markdown
# BAD: Too verbose, documenting instead of instructing
## Architecture
Our application uses a microservices architecture where each service
communicates via REST APIs. The frontend is built with React and uses
a component-based architecture pattern inspired by atomic design...

# GOOD: Concise instructions
## Architecture
- Microservices communicate via REST
- Frontend: React with atomic design (atoms → molecules → organisms)
- New services go in services/ with Dockerfile and docker-compose entry
```

---

## Exercise 2.1: Create Your First CLAUDE.md

1. Navigate to a project
2. Create `CLAUDE.md` at the project root with:
   - Project tech stack
   - Build/test/lint commands
   - 3-5 coding conventions
3. Start Claude Code and ask it to add a feature — observe if it follows your conventions

## Exercise 2.2: Hierarchy in Action

1. Create a subdirectory CLAUDE.md (e.g., `src/components/CLAUDE.md`)
2. Put component-specific rules in it
3. Ask Claude to work in that directory — verify it picks up both root and subdirectory rules
4. Ask Claude to work in a different directory — verify the subdirectory rules don't apply

## Exercise 2.3: Settings Configuration

1. Create `.claude/settings.json` in your project
2. Add permission rules for your common commands
3. Restart Claude Code and notice fewer permission prompts for allowed commands

---

## Common Pitfalls

- **Writing too much in CLAUDE.md**: Every word costs context. Keep it under 100 lines.
- **Duplicating info between CLAUDE.md files**: Root file handles shared rules. Subdirectory files only add specifics.
- **Putting secrets in CLAUDE.md**: This file is checked into git. Never put API keys or passwords here.
- **Using CLAUDE.md as documentation**: It's instructions for Claude, not docs for humans.

## Key Takeaways

1. CLAUDE.md is persistent memory — instructions that survive across sessions
2. Five-level hierarchy: global → project → subdirectory → managed
3. Ancestors always load, descendants lazy-load, siblings never load
4. Keep CLAUDE.md concise and instructional, not documentary
5. Settings files control permissions and environment variables
