# Module 5: Subagents — Parallel Workers

**Duration**: 60 minutes | **Level**: Intermediate

## TL;DR

Subagents are isolated Claude instances spawned within a session. They have their own context window, can run in parallel, and are perfect for research, review, and tasks that would pollute the main context.

---

## 5.1 What Are Subagents?

A subagent is a separate Claude instance that:

- Has its **own context window** (isolated from the main session)
- Can run **in the background** while you continue working
- Returns a **result summary** to the main session
- Can be **specialized** for specific tasks

The main session spawns subagents via the `Agent` tool. Think of it as delegating work to a colleague.

## 5.2 Why Use Subagents?

### Context isolation
Reading a 2000-line file in the main session costs context. A subagent reads it in its own context and returns a summary.

### Parallel execution
Need to review 5 files? Spawn 5 subagents simultaneously instead of reviewing sequentially.

### Specialization
Different agent types have different tool access and behaviors.

### Protected main context
Research tasks produce lots of output. Subagents absorb that, returning only the conclusions.

## 5.3 Official Agent Types

| Type | Purpose | Tools |
|------|---------|-------|
| **general-purpose** | Default. Research, code, multi-step tasks | All tools |
| **Explore** | Fast read-only search. Finding files, symbols, definitions | Read, Bash, WebFetch, WebSearch (no Edit/Write) |
| **Plan** | Architecture and planning. Returns strategies | Read, Bash, WebFetch (no Edit/Write) |
| **statusline-setup** | Configure status line | Read, Edit |
| **claude-code-guide** | Answer questions about Claude Code | Read, Bash, WebFetch, WebSearch |

## 5.4 Custom Subagent Definitions

Create custom agents in `.claude/agents/`:

```
.claude/
└── agents/
    └── my-agent.md
```

### Frontmatter fields

```yaml
---
name: security-reviewer
description: >
  Review code for security vulnerabilities. Use when
  the user asks for a security audit or review.
tools:
  - Read
  - Bash
  - WebSearch
model: opus
permissionMode: default
maxTurns: 10
memory: |
  You are a security expert. Focus on OWASP Top 10 vulnerabilities.
  Check for: SQL injection, XSS, CSRF, auth bypass, secrets exposure.
isolation: worktree        # Run in git worktree (isolated copy)
color: red                 # Status line color
---
```

### Key fields

| Field | Purpose |
|-------|---------|
| `name` | Agent identifier |
| `description` | When to auto-spawn this agent |
| `tools` | Allowlisted tools |
| `model` | Model override |
| `maxTurns` | Max agentic loop iterations |
| `memory` | Instructions preloaded into the agent's context |
| `isolation` | `worktree` creates an isolated git copy |
| `color` | Visual indicator in the UI |

## 5.5 Spawning Agents in Practice

### From the main session

Claude spawns agents automatically when appropriate, but you can guide it:

```
You: Research how authentication is implemented in this project.
     Use a subagent so it doesn't clutter our context.
```

### From custom commands/skills

Your commands can instruct Claude to spawn agents:

```markdown
# In a command file
Spawn an Explore agent to find all files related to authentication.
Then spawn a general-purpose agent to analyze the auth flow.
Report findings to the user.
```

### Parallel agents

When tasks are independent, agents run simultaneously:

```
You: Review these three modules in parallel:
     1. src/auth/ for security issues
     2. src/api/ for performance issues  
     3. src/db/ for query optimization
```

Claude spawns 3 agents at once, each with its own context.

## 5.6 Agent with Preloaded Skills

Agents can have skills preloaded into their context. This is the Command → Agent → Skill architecture:

### Example: Weather Agent

File: `.claude/agents/weather-agent.md`

```yaml
---
name: weather-agent
description: >
  Fetch weather data and create visualizations.
  PROACTIVE: Use when the user mentions weather, temperature, or forecast.
tools:
  - Bash
  - WebFetch
  - Read
  - Write
model: sonnet
maxTurns: 5
skills:
  - weather-fetcher     # Preloaded skill for data retrieval
---
```

The agent automatically has the `weather-fetcher` skill available without the user needing to invoke it.

### Direct vs Preloaded Skills

| | Direct Skill | Preloaded Agent Skill |
|---|---|---|
| Invocation | `/skill-name` or auto | Agent uses it automatically |
| `user-invocable` | `true` | `false` |
| Context | Main session | Agent's isolated context |
| Use case | User-facing procedures | Agent's internal tools |

## 5.7 Agent Communication Patterns

### One-shot delegation

```
Main session → Agent (does work) → Returns result → Main session continues
```

### Sequential chain

```
Main session → Agent A (research) → Result A → Agent B (implement using A) → Result B
```

### Parallel fan-out

```
Main session → Agent A (task 1) ─┐
             → Agent B (task 2) ─┤→ All results → Main session
             → Agent C (task 3) ─┘
```

### Hierarchical

```
Main session → Coordinator Agent → Worker Agent 1
                                 → Worker Agent 2
                                 → Worker Agent 3
```

## 5.8 Worktree Isolation

When `isolation: worktree` is set:

1. Git creates a temporary worktree (isolated copy of the repo)
2. The agent works on that copy
3. Changes don't affect the main working directory
4. If the agent makes changes, the worktree path and branch are returned
5. If no changes, the worktree is cleaned up automatically

Use this for:
- Experimental changes that might break things
- Parallel work on different approaches
- Safe exploration without affecting your work

## 5.9 Background Agents

Agents can run in the background while you continue working:

```
You: Research the best pagination library for our stack. Run in background.
```

Claude spawns the agent in the background and notifies you when it's done. You can continue working in the main session.

---

## Exercise 5.1: Your First Subagent

1. In Claude Code, ask: `Use a subagent to find all files in this project that import from a database module`
2. Notice the agent spawns, searches, and returns results
3. Observe that your main context stays clean

## Exercise 5.2: Parallel Review

1. Ask Claude to review 3 different directories in parallel using subagents
2. Compare the time with doing it sequentially
3. Notice how each agent returns independent findings

## Exercise 5.3: Custom Agent

1. Create `.claude/agents/test-writer.md`:
```yaml
---
name: test-writer
description: Write tests for a given file
tools:
  - Read
  - Write
  - Bash
model: sonnet
maxTurns: 8
---
```

Add instructions in the body for writing tests following your project's patterns.

2. Restart Claude Code
3. Ask Claude to write tests for a file — it should use your custom agent

## Exercise 5.4: Explore Agent

1. Ask: `Use an Explore agent to find where the user authentication middleware is defined`
2. Notice the Explore agent is read-only — it can search but can't modify files
3. Compare with a general-purpose agent doing the same search

---

## Common Pitfalls

- **Over-delegating**: Simple tasks don't need subagents. Reading one file is faster inline.
- **Not providing enough context in the prompt**: Agents start cold — they don't see your conversation. Include relevant details.
- **Expecting agents to coordinate automatically**: Agents are isolated. They don't see each other's work unless you pass results between them.
- **Using agents for quick lookups**: Use `grep` or `find` via Bash for simple searches.

## Key Takeaways

1. Subagents have isolated context — they protect the main session from clutter
2. Five official types: general-purpose, Explore, Plan, statusline-setup, claude-code-guide
3. Custom agents live in `.claude/agents/` with YAML frontmatter
4. Agents can run in parallel for independent tasks
5. Skills can be preloaded into agents for automatic use
6. Worktree isolation lets agents experiment safely
7. Background agents let you continue working while research happens
