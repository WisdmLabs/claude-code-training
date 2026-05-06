# Module 8: Orchestration — Command + Agent + Skill

**Duration**: 60 minutes | **Level**: Advanced

## TL;DR

The Command → Agent → Skill architecture separates concerns: commands are entry points, agents are autonomous workers, skills are reusable procedures. This pattern enables complex, multi-step workflows with clean isolation.

---

## 8.1 The Architecture Pattern

```
User invokes /command
    ↓
Command orchestrates the workflow
    ↓
Command spawns Agent(s)
    ↓
Agent uses preloaded Skill(s)
    ↓
Agent returns results
    ↓
Command presents output (may invoke a rendering Skill)
```

Each layer has a distinct role:

| Layer | Role | Analogy |
|-------|------|---------|
| **Command** | Entry point, orchestration | Manager |
| **Agent** | Autonomous execution | Worker |
| **Skill** | Reusable procedure | Tool/recipe |

## 8.2 Why This Matters

Without orchestration:
```
You: Get the weather for Dubai and make a nice visualization
Claude: [reads weather API docs] [writes fetch code] [runs it]
        [gets confused about SVG] [loses context] [produces mediocre result]
```

With orchestration:
```
You: /weather Dubai
Command: Spawns weather-agent with weather-fetcher skill preloaded
Agent: Uses skill to fetch data cleanly
Command: Passes data to weather-svg-creator skill
Result: Clean SVG visualization
```

The benefits:
1. **Separation of concerns** — each component does one thing
2. **Context isolation** — agent's research doesn't pollute main session
3. **Reusability** — skills work across commands and agents
4. **Reliability** — structured workflow beats ad-hoc prompting

## 8.3 Building an Orchestrated Workflow

Let's build a complete example: a code audit workflow.

### Step 1: Create the Skills

**Skill: Security Scanner**
File: `.claude/skills/security-scan.md`

```markdown
---
name: security-scan
description: Scan code for security vulnerabilities
user-invocable: false
allowed-tools:
  - Read
  - Bash
---

# Security Scan

Scan the target files for:
1. SQL injection vulnerabilities
2. XSS vectors
3. Hardcoded secrets or API keys
4. Insecure crypto usage
5. Path traversal risks

For each finding, report:
- File and line number
- Vulnerability type
- Severity (Critical/High/Medium/Low)
- Suggested fix

Use grep and file reading — do not modify any files.
```

**Skill: Performance Analyzer**
File: `.claude/skills/perf-analyze.md`

```markdown
---
name: perf-analyze
description: Analyze code for performance issues
user-invocable: false
allowed-tools:
  - Read
  - Bash
---

# Performance Analysis

Analyze the target files for:
1. N+1 query patterns
2. Missing database indexes (check schema)
3. Unnecessary re-renders (React)
4. Large bundle imports
5. Synchronous operations that should be async

For each finding, report:
- File and line number
- Issue type
- Impact estimate
- Suggested optimization
```

### Step 2: Create the Agent

File: `.claude/agents/code-auditor.md`

```yaml
---
name: code-auditor
description: >
  Comprehensive code auditor that checks security and performance.
  PROACTIVE: Use when the user asks for a code audit, review, or quality check.
tools:
  - Read
  - Bash
model: opus
maxTurns: 15
skills:
  - security-scan
  - perf-analyze
---
```

Body:
```markdown
# Code Auditor Agent

You are a code auditor. When given files or directories to audit:

1. First use the `security-scan` skill on all target files
2. Then use the `perf-analyze` skill on the same files
3. Correlate findings — some security fixes also improve performance
4. Generate a unified report sorted by severity

Output format:
## Audit Report
### Critical Issues (fix immediately)
### High Priority
### Medium Priority
### Low Priority / Suggestions
### Summary Statistics
```

### Step 3: Create the Command

File: `.claude/commands/audit.md`

```markdown
---
name: audit
description: Run a comprehensive code audit
arguments:
  - name: target
    description: File or directory to audit
    required: true
  - name: focus
    description: Focus area (security, performance, all)
    required: false
    default: "all"
---

# Code Audit: {{target}}

Run a comprehensive code audit on {{target}} with focus on {{focus}}.

1. Spawn a `code-auditor` agent to analyze the target
2. The agent will use its preloaded security-scan and perf-analyze skills
3. Present the agent's report to the user
4. Ask if they want to auto-fix any of the findings
```

### Usage

```
You: /audit src/api security
```

This triggers: Command → Agent (with preloaded skills) → structured report.

## 8.4 Presentation Delegation

Commands often delegate output formatting to a separate skill:

```markdown
# In the command file
After receiving the audit results, use the `report-formatter` skill
to create a clean markdown report with:
- Executive summary
- Findings table
- Severity distribution chart (text-based)
- Recommended action items
```

This keeps audit logic separate from presentation logic.

## 8.5 Data Flow Between Layers

```
Command defines the task
    ↓ passes arguments
Agent executes autonomously
    ↓ uses skills for specific procedures
Skills return structured data
    ↓ agent aggregates
Agent returns findings
    ↓ command may pass to rendering skill
Final output presented to user
```

### Data contracts

Define clear interfaces between layers:

```markdown
# In the agent definition
Return data in this format:
{
  "findings": [
    {
      "file": "path/to/file",
      "line": 42,
      "type": "security|performance",
      "severity": "critical|high|medium|low",
      "title": "Short description",
      "details": "Full explanation",
      "fix": "Suggested remediation"
    }
  ],
  "summary": {
    "total": 12,
    "critical": 1,
    "high": 3,
    "medium": 5,
    "low": 3
  }
}
```

## 8.6 Design Guidelines

### When to use each layer

| If you need... | Use a... |
|----------------|----------|
| A user-facing entry point | Command |
| Autonomous multi-step work | Agent |
| A reusable procedure | Skill |
| Output formatting | Skill (with `user-invocable: false`) |
| Guardrails and validation | Hook |

### Preference hierarchy

Start with the lightest mechanism:

```
Skill (lightest) → Agent → Command (heaviest)
```

If a skill can do it, don't create an agent. If an agent can do it, don't create a command. Commands are for multi-step orchestration that requires user interaction or coordination.

### PROACTIVE agents

Adding `PROACTIVE` to an agent's description tells Claude to spawn it automatically:

```yaml
description: >
  PROACTIVE: Use when the user mentions deployment, deploy, or ship.
```

---

## Exercise 8.1: Build an Orchestrated Workflow

Build a "new feature" workflow with:
1. A command (`/new-feature`) that takes a feature name
2. An agent that researches existing patterns in the codebase
3. A skill that scaffolds the feature files

## Exercise 8.2: Data Contract

1. Define a clear data contract between your agent and command
2. Have the agent return structured JSON
3. Have the command format it for the user

## Exercise 8.3: Multi-Agent Orchestration

Build a command that spawns two agents in parallel:
1. One for frontend analysis
2. One for backend analysis
3. The command combines their results

---

## Common Pitfalls

- **Putting everything in the command**: Commands should orchestrate, not execute. Delegate to agents and skills.
- **Not defining data contracts**: Without clear interfaces, components return inconsistent formats.
- **Over-engineering simple tasks**: Not everything needs Command → Agent → Skill. Use the simplest mechanism.
- **Forgetting PROACTIVE**: If an agent should auto-trigger, include PROACTIVE in the description.

## Key Takeaways

1. Command → Agent → Skill separates entry point, execution, and procedures
2. Commands orchestrate, agents execute autonomously, skills provide reusable procedures
3. Data contracts between layers ensure consistent, reliable output
4. Presentation can be delegated to a separate skill
5. Start with skills (lightest), escalate to agents, then commands
6. PROACTIVE in descriptions enables auto-triggering
