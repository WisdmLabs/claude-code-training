# Module 1: Foundations — How Claude Code Works

**Duration**: 45 minutes | **Level**: Beginner

## TL;DR

Claude Code uses a conversation loop with tools. It reads files, edits code, runs commands, and iterates. Understanding context windows, tool use, and the agentic loop is essential to using it well.

---

## 1.1 The Agentic Loop

Claude Code operates in a loop:

```
User prompt → Claude thinks → Claude uses tools → Claude observes results → Claude thinks again → ... → Response
```

Each "turn" can involve multiple tool calls. Claude might:
1. Read 3 files to understand context
2. Edit 2 files to make changes
3. Run a test command to verify
4. Fix a failing test
5. Report back

This loop continues until Claude believes the task is complete or needs your input.

## 1.2 The Tool System

Claude Code has built-in tools:

| Tool | Purpose |
|------|---------|
| **Read** | Read files from disk |
| **Edit** | Make targeted string replacements in files |
| **Write** | Create new files or fully rewrite existing ones |
| **Bash** | Run shell commands |
| **Agent** | Spawn subagent workers |
| **WebFetch** | Fetch URLs |
| **WebSearch** | Search the web |

These are the primitives that everything else is built on. Commands, skills, and agents all ultimately use these tools.

## 1.3 Context Window

The context window is the "working memory" of a session. It has a finite size and includes:

- System prompt
- CLAUDE.md files
- Your conversation history
- Tool call results (file contents, command output)
- Claude's responses

### Context Degradation

**This is the most important concept in Claude Code.**

As your conversation grows, older messages get compressed or dropped. This means:

- Claude may "forget" things discussed earlier in long sessions
- Large file reads consume significant context
- Long command outputs fill context fast

### Strategies to manage context:

1. **Start fresh sessions for new tasks** — don't reuse long conversations
2. **Use `/compact`** to compress conversation history
3. **Use `/clear`** to reset entirely
4. **Be specific** — "Edit line 42 of auth.ts" uses less context than "Find and fix the auth bug"
5. **Use subagents** for research tasks — they have isolated context

### Prompt Caching

Claude Code automatically uses prompt caching to reduce costs and latency. Cached content (system prompt, CLAUDE.md, early conversation) is reused across turns without re-processing.

Key implications:
- **Cache TTL is ~5 minutes** — if you're idle for more than 5 minutes, the cache expires and the next turn is slower/costlier
- **Manually editing files mid-session busts the cache** — let Claude do all file edits
- **Stable CLAUDE.md content maximizes cache hits** — it's loaded every turn
- **`/compact` invalidates the cache** — use it strategically, not reflexively

### The golden rule:

> One task per session. When context degrades, start a new session.

## 1.4 CLI Startup Flags

Control behavior at launch with flags:

### Session Management

```bash
# Continue the last conversation
claude --continue

# Resume a specific conversation
claude --resume

# Start with no conversation history
claude
```

### Model Selection

```bash
# Use a specific model
claude --model opus

# Aliases: opus, sonnet, haiku
claude --model sonnet
```

### Print Mode (Non-Interactive)

```bash
# One-shot question, no interactive session
claude --print "What does this project do?"

# JSON output for scripting
claude --print --output-format json "List all API endpoints"

# Stream JSON for real-time processing
claude --print --output-format stream-json "Analyze this code"
```

### System Prompt

```bash
# Prepend instructions
claude --system-prompt "You are reviewing code for security issues"

# Append instructions
claude --append-system-prompt "Always explain your reasoning"
```

### Security

```bash
# Restrict tools
claude --allowedTools "Read,Edit"

# Block specific tools
claude --disallowedTools "Bash"
```

## 1.5 Plan Mode

Plan mode is one of the most powerful features for complex tasks. Toggle it with **Shift+Tab** (press twice) or launch with:

```bash
claude --permission-mode plan
```

In plan mode:
1. Claude analyzes the task and proposes a step-by-step plan
2. You review, edit, or approve the plan
3. Only after approval does Claude execute

Boris from the Anthropic team says plan mode **multiplies results 2-3x** for complex tasks. Use it for:
- Refactoring across multiple files
- Implementing features with unclear requirements
- Any task where you want to review the approach before execution

## 1.6 Environment Variables

Configure defaults without flags:

```bash
# Set default model
export CLAUDE_MODEL=opus

# Set default permission mode
export CLAUDE_CODE_PERMISSION_MODE=auto

# Enable experimental features
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

## 1.7 The `/doctor` Command

Diagnose issues:

```bash
claude doctor
```

This checks your installation, authentication, MCP servers, and configuration.

---

## Exercise 1.1: Watch the Loop

1. Start Claude Code in a project
2. Ask: `Find all TODO comments in this project and list them with file locations`
3. Watch the tool calls — notice how Claude reads files, searches, and aggregates

## Exercise 1.2: Context Management

1. Start a session and have a long conversation (10+ exchanges)
2. Ask Claude to recall something from the beginning — notice any degradation
3. Try `/compact` and ask again
4. Start a fresh session with `/clear` — notice the difference

## Exercise 1.3: Print Mode for Scripting

1. Run: `claude --print "List the 5 largest files in this directory by line count"`
2. Run: `claude --print --output-format json "What language is this project written in?"`
3. Pipe it: `claude --print "Generate a .gitignore for a Node.js project" > .gitignore`

---

## Common Pitfalls

- **Overloading a single session**: The #1 mistake. Start new sessions for new tasks.
- **Not using `--continue`**: When you need to resume work, use `claude --continue` instead of re-explaining.
- **Ignoring context cost**: Asking Claude to "read the entire codebase" wastes context. Be targeted.

## Key Takeaways

1. Claude Code is a tool-using agent that loops until tasks are complete
2. Context degradation is real — one task per session, use `/compact`
3. CLI flags and environment variables control behavior
4. Print mode (`--print`) enables scripting and automation
5. The agentic loop (think → tool → observe → think) is the core mechanism
