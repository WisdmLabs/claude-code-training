# Module 9: Agent Teams — Multi-Session Coordination

**Duration**: 60 minutes | **Level**: Advanced

## TL;DR

Agent Teams run multiple independent Claude Code sessions that coordinate via shared task lists. Each session has a role, works in isolation, and shares results through a common project structure.

---

## 9.1 What Are Agent Teams?

Agent Teams are multiple Claude Code instances (typically in separate terminal panes) that work together on the same project. Unlike subagents (which run within a single session), Agent Teams are:

- **Independent sessions** — each with full Claude Code capabilities
- **Coordinated via shared state** — task lists, files, and git
- **Parallel by nature** — truly concurrent work on different aspects
- **Experimental** — requires opt-in via environment variable

## 9.2 Setup

### Enable the feature

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

### Terminal setup

Agent Teams work best with a terminal multiplexer:

**Using tmux:**
```bash
# Create a new tmux session with 3 panes
tmux new-session -s team
tmux split-window -h
tmux split-window -v
```

**Using iTerm2:**
- Split panes: Cmd+D (horizontal), Cmd+Shift+D (vertical)

### Start each team member

In each terminal pane, start Claude Code with a role:

```bash
# Pane 1: Architect
claude --system-prompt "You are the Command Architect. You design the command structure and data contracts."

# Pane 2: Backend
claude --system-prompt "You are the Agent Engineer. You implement agent logic and tool integrations."

# Pane 3: Frontend
claude --system-prompt "You are the Skill Designer. You create reusable skills and output formatters."
```

## 9.3 Coordination Mechanisms

### Shared task list

Agent Teams coordinate through a shared task file:

```markdown
# tasks.md (shared file in the project)

## Pending
- [ ] Design data contract for weather API response
- [ ] Implement weather-fetcher agent
- [ ] Create SVG output skill

## In Progress
- [ ] @architect: Defining command structure

## Done
- [x] Project setup complete
```

Each team member reads and updates this file.

### Shared project structure

Team members work in a common project directory:

```
project/
├── .claude/
│   ├── commands/     ← Architect writes here
│   ├── agents/       ← Engineer writes here
│   └── skills/       ← Designer writes here
├── tasks.md          ← Shared coordination
└── contracts.md      ← Shared data contracts
```

### Git-based coordination

Team members can use branches and commits:
- Each member works on their component
- They commit their work
- Other members pull to see changes

## 9.4 Team Roles Pattern

### Role: Command Architect

Responsibilities:
- Define command entry points
- Establish data contracts between components
- Design the user-facing interface
- Write integration tests

### Role: Agent Engineer

Responsibilities:
- Implement agent definitions
- Build tool integrations
- Handle error cases and retries
- Optimize agent performance

### Role: Skill Designer

Responsibilities:
- Create reusable skills
- Build output formatters
- Write validation procedures
- Design templates

## 9.5 Example: Time Display Workflow

Three team members build a time display system:

### Data Contract (agreed upon first)

```json
{
  "time": "14:30:00",
  "timezone": "Asia/Dubai",
  "formatted": "2:30 PM GST",
  "utcOffset": "+04:00"
}
```

### Architect creates: `.claude/commands/show-time.md`

```markdown
---
name: show-time
description: Display current time in a specified timezone
arguments:
  - name: timezone
    description: IANA timezone name
    required: true
---

# Show Time: {{timezone}}

1. Spawn the `time-agent` to fetch current time for {{timezone}}
2. Pass the result to the `time-formatter` skill
3. Display the formatted output
```

### Engineer creates: `.claude/agents/time-agent.md`

```yaml
---
name: time-agent
description: Fetch current time for a timezone
tools:
  - Bash
model: sonnet
maxTurns: 3
skills:
  - time-fetcher
---
```

### Designer creates: `.claude/skills/time-formatter.md`

```markdown
---
name: time-formatter
description: Format time data into a readable display
user-invocable: false
---

Format the time data as:

🕐 Current Time
━━━━━━━━━━━━━━━
Location: {{timezone}}
Time: {{formatted}}
UTC Offset: {{utcOffset}}
━━━━━━━━━━━━━━━
```

## 9.6 Communication Protocol

Since team members can't talk to each other directly, use these patterns:

### 1. File-based messaging

Write status updates to a shared file:

```markdown
# team-log.md

## 2024-01-15 14:30 — Architect
Command structure defined. Data contract in contracts.md.
Agent Engineer: you can start on the time-agent now.

## 2024-01-15 14:35 — Engineer
time-agent implemented. Uses Bash to call `date` command.
Skill Designer: output format is JSON matching the contract.
```

### 2. Task list updates

```markdown
## In Progress
- [x] @architect: Command structure ✓ (see .claude/commands/show-time.md)
- [ ] @engineer: Time agent (blocked on: data contract confirmation)
- [ ] @designer: Formatter skill
```

### 3. Contract files

```markdown
# contracts.md

## Time Data Contract v1.0
Status: APPROVED by all team members

Input: timezone (string, IANA format)
Output:
{
  "time": "HH:MM:SS",
  "timezone": "Area/Location",
  "formatted": "human readable",
  "utcOffset": "+/-HH:MM"
}
```

## 9.7 When to Use Agent Teams vs Subagents

| | Agent Teams | Subagents |
|---|---|---|
| **Sessions** | Multiple independent sessions | Spawned within one session |
| **Context** | Each has full, independent context | Isolated but spawned from main |
| **Coordination** | File-based, manual | Automatic via Agent tool |
| **Parallelism** | Truly concurrent | Concurrent within session limits |
| **Best for** | Large, multi-component work | Research, reviews, focused tasks |
| **Setup** | Multiple terminals, env var | None — built in |

**Use Agent Teams when:**
- Building multiple interconnected components
- Team members need full autonomy
- Work spans hours, not minutes
- Different roles need different system prompts

**Use Subagents when:**
- Tasks can be defined in a single prompt
- Results feed back into the main session
- Work takes minutes, not hours
- No coordination protocol needed

---

## Exercise 9.1: Two-Member Team

1. Open two terminal panes
2. Set `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`
3. In pane 1: Start Claude as "Backend Engineer"
4. In pane 2: Start Claude as "Frontend Engineer"
5. Create a shared `tasks.md` file
6. Have each member complete their tasks and update the file

## Exercise 9.2: Data Contract First

1. Before any coding, have team members agree on a data contract
2. Write it to `contracts.md`
3. Have each member implement their component against the contract
4. Verify the components integrate correctly

## Exercise 9.3: Full Three-Member Team

1. Set up three panes: Architect, Engineer, Designer
2. Build a complete Command → Agent → Skill workflow
3. Use `team-log.md` for communication
4. Iterate until the workflow works end-to-end

---

## Common Pitfalls

- **No data contract upfront**: Team members build incompatible interfaces. Define contracts first.
- **Not checking shared state**: Members work in isolation and miss updates. Check task list regularly.
- **Race conditions on files**: Two members editing the same file. Assign clear ownership.
- **Over-communicating**: The task file becomes a chat log. Keep entries concise and actionable.

## Key Takeaways

1. Agent Teams are independent Claude Code sessions coordinating via shared files
2. Requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` environment variable
3. Terminal multiplexers (tmux, iTerm2) make multi-pane setup easy
4. Coordination via shared task lists, data contracts, and team logs
5. Define data contracts BEFORE team members start implementing
6. Use Agent Teams for large, multi-component work; subagents for focused tasks
