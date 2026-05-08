## Project: Example App

### Tech Stack
- Next.js 15 with App Router
- TypeScript strict mode
- Tailwind CSS v4
- Drizzle ORM with PostgreSQL
- Vitest for testing

### Coding Standards
- Use server components by default; add `'use client'` only for interactivity
- API routes in `app/api/` using route handlers
- Database queries in `lib/db/queries/`
- Shared types in `lib/types/`

### Commands
- Dev: `pnpm dev`
- Test: `pnpm test`
- Lint: `pnpm lint`
- Build: `pnpm build`
- DB migrate: `pnpm db:migrate`

### Git
- Conventional commits: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`
- One logical change per commit
- Run `pnpm test` before committing

### Architecture Rules
- No circular dependencies between modules
- Business logic in `lib/`, never in components
- Error boundaries at route segment level
