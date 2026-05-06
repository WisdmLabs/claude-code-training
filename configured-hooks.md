# Claude Code Hooks Configuration

Last updated: 2026-03-25

**Note:** There are no user-defined hooks in `~/.claude/settings.json` or any project-level settings. All hooks below are provided by installed plugins.

---

## How Hooks Work

Claude Code hooks are shell commands that execute automatically in response to lifecycle events. They are configured in `settings.json` under the `hooks` key, or provided by plugins via `hooks/hooks.json`.

### Hook Events

| Event | When it fires |
|---|---|
| `Setup` | When the plugin is first set up |
| `SessionStart` | When a new session begins (startup, clear, compact) |
| `UserPromptSubmit` | When the user sends a message |
| `PreToolUse` | Before a tool is executed (can block with exit code 2) |
| `PostToolUse` | After a tool completes successfully |
| `PostToolUseFailure` | After a tool execution fails |
| `PreCompact` | Before conversation context is compacted |
| `Stop` | When Claude finishes its turn |
| `TaskCompleted` | When a task is marked complete |
| `SubagentStop` | When a subagent finishes |
| `SessionEnd` | When the session ends |

### Matcher Syntax

Hooks can use a `matcher` field to filter which tools or events trigger them:
- `"*"` — matches everything
- `"Edit|Write"` — matches specific tools (pipe-separated)
- `"Bash"` — matches a single tool
- `"startup|clear|compact"` — matches specific session start types

---

## Plugin: Security Guidance

**Source:** `anthropics/claude-plugins-official`

### Hook: PreToolUse — Security Reminder

**Triggers on:** `Edit`, `Write`, `MultiEdit`

**What it does:** Scans file edits for dangerous security patterns before they are written. On first detection per file+rule per session, it **blocks the tool call** (exit code 2) and displays a warning. Subsequent edits to the same file for the same rule are allowed through.

**Patterns detected:** 9 rules covering GitHub Actions workflow injection, shell execution, dynamic code construction/evaluation, React XSS patterns, DOM injection, HTML assignment, Python deserialization, and Python shell execution. See the hook script at `plugins/cache/claude-plugins-official/security-guidance/*/hooks/security_reminder_hook.py` for the full list.

**Can be disabled:** Set environment variable `ENABLE_SECURITY_REMINDER=0`.

### Configuration (if adding manually)

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "python3 /path/to/security_reminder_hook.py"
          }
        ]
      }
    ]
  }
}
```

---

## Plugin: Claude Mem

**Source:** `thedotmack/claude-mem`

### Hook 1: Setup — Install Dependencies

**Triggers on:** Plugin setup (`*`)

**What it does:** Runs `scripts/setup.sh` to install required dependencies for the memory system. Timeout: 120s.

### Hook 2: SessionStart — Smart Install + Worker Start + Context Load

**Triggers on:** `startup`, `clear`, `compact`

**What it does (3 sequential steps):**
1. **Smart install** — Checks and installs any missing dependencies (timeout: 300s)
2. **Start worker service** — Launches the background memory worker process (timeout: 60s)
3. **Load context** — Fetches relevant memory context for the current session (timeout: 60s)

### Hook 3: UserPromptSubmit — Session Init

**Triggers on:** Every user message

**What it does (2 sequential steps):**
1. **Ensure worker is running** — Starts the worker service if not already running
2. **Session init** — Initializes/refreshes the session in the memory system

### Hook 4: PostToolUse — Observation Capture

**Triggers on:** Every tool use (`*`)

**What it does (2 sequential steps):**
1. **Ensure worker is running** — Starts the worker service if not already running
2. **Capture observation** — Records observations about tool usage for cross-session memory (timeout: 120s)

### Hook 5: Stop — Summarize + Session Complete

**Triggers on:** Every Claude turn completion

**What it does (2 sequential steps):**
1. **Summarize** — Creates a summary of the conversation turn for memory (timeout: 120s)
2. **Session complete** — Marks the session as complete and persists data (timeout: 30s)

### Configuration (if adding manually)

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "node /path/to/scripts/bun-runner.js /path/to/worker-service.cjs hook claude-code context",
            "timeout": 60
          }
        ]
      }
    ]
  }
}
```

---

## Plugin: WisdmLabs Engineering

**Source:** `aruneshwisdm/wisdmlabs-engineering-plugin` (v5.36.0)

### Hook 1: SessionStart — Capture Session ID

**Triggers on:** Every session start

**What it does:** Captures and stores the current session ID for use by other hooks and scripts. Enables session-scoped transcript logging and context tracking.

### Hook 2: UserPromptSubmit — Inject Project Context

**Triggers on:** Every user message | **Timeout:** 5s

**What it does:** Injects recent project context (session history, recent activity) into the conversation. This is what provides the `# [project] recent context` block seen at the top of each session.

### Hook 3: PostToolUse — Context Monitor

**Triggers on:** Every tool use

**What it does:** Monitors tool usage to track context window consumption and session activity. Provides awareness of how much context has been used.

### Hook 4: PostToolUse — Post-Edit Lint

**Triggers on:** `Write`, `Edit` | **Timeout:** 15s

**What it does:** Runs lightweight lint checks after file edits to catch issues immediately rather than at commit time. Platform-aware (WordPress/Moodle/Shopify standards).

### Hook 5: Stop — Quality Gate

**Triggers on:** Every Claude turn completion | **Timeout:** 10s

**What it does:** Runs a quality gate check when Claude finishes a turn. Validates that completed work meets minimum quality thresholds before the user sees the final result.

### Hook 6: PreCompact — Save State Before Compaction

**Triggers on:** Before context compaction | **Timeout:** 5s

**What it does:** Saves important conversation state (tasks, progress, decisions) before the context window is compacted, preventing loss of critical information.

### Hook 7: PreToolUse — File Guard

**Triggers on:** `Write`, `Edit` | **Timeout:** 5s

**What it does:** Guards against accidental writes to protected files (e.g., plugin core files, vendor directories, lock files). Blocks the tool call if the target file is in a protected path.

### Hook 8: PostToolUseFailure — Failure Learning Search

**Triggers on:** `Bash` failures | **Timeout:** 10s

**What it does:** When a Bash command fails, automatically searches team learnings for similar errors. Surfaces known solutions so the same problems don't need to be re-investigated.

### Hook 9: TaskCompleted — Task Verification

**Triggers on:** Task completion | **Timeout:** 5s

**What it does:** Verifies that a task marked as complete actually meets its acceptance criteria before finalizing.

### Hook 10: SubagentStop — Output Validation

**Triggers on:** Subagent completion | **Timeout:** 10s

**What it does:** Validates subagent output for completeness and quality. Ensures spawned agents return actionable results rather than empty or malformed responses.

### Hook 11: SessionEnd — Learning Extraction

**Triggers on:** Session end | **Timeout:** 15s

**What it does:** Extracts patterns, decisions, and learnings from the completed session and persists them for future reference via the wisdm-engineering MCP server.

### Configuration (if adding manually)

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "node /path/to/pre-tool-file-guard.mjs",
            "timeout": 5
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "node /path/to/post-edit-lint.mjs",
            "timeout": 15
          }
        ]
      }
    ],
    "PostToolUseFailure": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "node /path/to/failure-learning-search.mjs",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

---

## Summary: All Active Hooks by Event

| Event | Plugin | Hook | Purpose |
|---|---|---|---|
| **Setup** | claude-mem | Install deps | One-time dependency setup |
| **SessionStart** | claude-mem | Smart install + worker + context | Start memory system, load context |
| **SessionStart** | wisdmlabs | Capture session ID | Session tracking |
| **UserPromptSubmit** | claude-mem | Session init | Memory session management |
| **UserPromptSubmit** | wisdmlabs | Inject project context | Recent context injection |
| **PreToolUse** | security-guidance | Security reminder | Block dangerous patterns (Edit/Write) |
| **PreToolUse** | wisdmlabs | File guard | Protect critical files (Edit/Write) |
| **PostToolUse** | claude-mem | Observation capture | Cross-session memory |
| **PostToolUse** | wisdmlabs | Context monitor | Track context usage |
| **PostToolUse** | wisdmlabs | Post-edit lint | Lint after edits (Edit/Write) |
| **PostToolUseFailure** | wisdmlabs | Failure learning search | Auto-search team learnings (Bash) |
| **PreCompact** | wisdmlabs | Save state | Preserve state before compaction |
| **Stop** | claude-mem | Summarize + session complete | Persist memory |
| **Stop** | wisdmlabs | Quality gate | Validate work quality |
| **TaskCompleted** | wisdmlabs | Task verification | Verify task acceptance criteria |
| **SubagentStop** | wisdmlabs | Output validation | Validate subagent results |
| **SessionEnd** | wisdmlabs | Learning extraction | Extract and persist learnings |

---

## How to Add Custom Hooks

To add your own hooks, edit `~/.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "/path/to/your/script.sh",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

### Key rules:
- **Exit code 0** = allow (hook passes)
- **Exit code 2** = block the tool call (PreToolUse only) and show stderr as feedback
- **Any other exit code** = hook error (tool proceeds)
- **stdout** is injected into the conversation as additional context
- **stderr** is shown to the user as feedback
- Hook input is provided via **stdin** as JSON with `session_id`, `tool_name`, `tool_input`
- Use `timeout` to prevent hanging hooks from blocking the session
- Plugin hooks use `${CLAUDE_PLUGIN_ROOT}` to reference their install directory
