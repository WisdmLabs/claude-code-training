# Module 7: Hooks — Event-Driven Automation

**Duration**: 45 minutes | **Level**: Advanced

## TL;DR

Hooks are shell commands that run automatically in response to Claude Code events — before/after tool calls, on session start, on prompt submit. They enable validation, logging, notifications, and guardrails without manual intervention.

---

## 7.1 What Are Hooks?

Hooks are event-driven shell commands configured in `settings.json`. They fire at specific moments in the Claude Code lifecycle:

```
User submits prompt → [PreToolUse hook] → Claude uses tool → [PostToolUse hook] → ...
```

Hooks are **deterministic** — unlike prompts, they execute exactly the same way every time. This makes them ideal for:

- Validation gates (block unsafe operations)
- Logging and auditing
- Notifications (sound, desktop alert)
- Auto-formatting code after edits
- Pre/post-processing

## 7.2 Hook Events

| Event | When it fires | Use cases |
|-------|---------------|-----------|
| `SessionStart` | Claude Code starts | Load context, check prerequisites |
| `UserPromptSubmit` | User submits a prompt | Validate input, inject context |
| `PreToolUse` | Before a tool runs | Block dangerous commands, validate paths |
| `PostToolUse` | After a tool completes | Format code, log actions, notify |
| `Stop` | Session ends | Cleanup, save state |

## 7.3 Configuration

Hooks live in `settings.json` (project or global):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "command": "prettier --write $CLAUDE_FILE_PATH",
        "description": "Auto-format files after edit"
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "command": "echo $CLAUDE_TOOL_INPUT | grep -q 'rm -rf' && exit 1 || exit 0",
        "description": "Block rm -rf commands"
      }
    ],
    "SessionStart": [
      {
        "command": "echo 'Session started at $(date)'",
        "description": "Log session start"
      }
    ]
  }
}
```

### Hook configuration fields

| Field | Purpose |
|-------|---------|
| `matcher` | Regex pattern to match tool names (e.g., `Edit\|Write`) |
| `command` | Shell command to execute |
| `description` | Human-readable description |
| `timeout` | Max execution time in ms |

## 7.4 Environment Variables in Hooks

Claude Code sets these variables before running hooks:

| Variable | Available in | Value |
|----------|-------------|-------|
| `$CLAUDE_FILE_PATH` | PostToolUse (Edit/Write) | Path of the edited file |
| `$CLAUDE_TOOL_NAME` | PreToolUse, PostToolUse | Name of the tool |
| `$CLAUDE_TOOL_INPUT` | PreToolUse | Tool input as JSON |
| `$CLAUDE_SESSION_ID` | All hooks | Current session ID |

## 7.5 Hook Patterns

### Auto-format after edits

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "command": "prettier --write \"$CLAUDE_FILE_PATH\" 2>/dev/null || true",
        "description": "Format files after editing"
      }
    ]
  }
}
```

### Sound notification on completion

```json
{
  "hooks": {
    "Stop": [
      {
        "command": "afplay /System/Library/Sounds/Glass.aiff",
        "description": "Play sound when Claude finishes"
      }
    ]
  }
}
```

### Log all tool usage

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "command": "echo \"$(date): $CLAUDE_TOOL_NAME\" >> ~/.claude/tool-log.txt",
        "description": "Log tool usage"
      }
    ]
  }
}
```

### Block writes to protected files

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "command": "echo $CLAUDE_TOOL_INPUT | grep -q '\"file_path\".*\\.env' && echo 'BLOCKED: Cannot edit .env files' && exit 1 || exit 0",
        "description": "Prevent editing .env files"
      }
    ]
  }
}
```

### Inject context on prompt submit

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "command": "echo 'Current branch: '$(git branch --show-current)'; Last commit: '$(git log -1 --oneline)",
        "description": "Add git context to every prompt"
      }
    ]
  }
}
```

## 7.6 Hook Exit Codes

| Exit code | Effect |
|-----------|--------|
| `0` | Success — continue normally |
| `1` | Failure — **block the action** (for PreToolUse) |
| Other | Error — logged but doesn't block |

For `PreToolUse` hooks, exit code 1 prevents the tool from executing. This is how you create guardrails.

## 7.7 Hooks vs Skills vs CLAUDE.md

| Mechanism | Deterministic | Runs when | Best for |
|-----------|--------------|-----------|----------|
| **Hooks** | Yes | Event-triggered | Automation, validation, guardrails |
| **Skills** | No (AI-driven) | Intent-matched | Reusable procedures |
| **CLAUDE.md** | No (advisory) | Always in context | Behavioral guidelines |

Key insight: If you need something to happen **every single time** without exception, use a hook. If you need Claude to follow guidelines **most of the time**, use CLAUDE.md. If you need a reusable procedure, use a skill.

---

## Exercise 7.1: Auto-Format Hook

1. Add a PostToolUse hook that runs Prettier on edited files
2. Edit a file with messy formatting
3. Verify it's automatically formatted after Claude's edit

## Exercise 7.2: Safety Guardrail

1. Add a PreToolUse hook that blocks editing of `package-lock.json`
2. Ask Claude to modify `package-lock.json`
3. Verify the hook blocks the edit

## Exercise 7.3: Notification Hook

1. Add a Stop hook that plays a sound or shows a desktop notification
2. Ask Claude to do a task
3. Verify the notification fires when Claude finishes

## Exercise 7.4: Logging Hook

1. Add a PostToolUse hook that logs all tool usage to a file
2. Have a conversation with several tool calls
3. Review the log file to see what Claude did

---

## Common Pitfalls

- **Slow hooks**: Hooks run synchronously. A slow hook blocks Claude Code. Keep them fast (<1 second).
- **Hooks that break**: A failing hook can block all tool use. Always test hooks in isolation first.
- **Over-engineering hooks**: Start with 1-2 hooks. Don't build a full pipeline on day one.
- **Not handling errors**: Use `|| true` or `2>/dev/null` for non-critical hooks.
- **Forgetting the matcher**: Without a `matcher`, the hook runs for ALL tools of that event type.

## Key Takeaways

1. Hooks are deterministic event-driven automation — they always run, unlike prompts
2. Five event types: SessionStart, UserPromptSubmit, PreToolUse, PostToolUse, Stop
3. PreToolUse hooks with exit code 1 can block dangerous operations
4. Use hooks for things that must happen every time — formatting, logging, guardrails
5. Keep hooks fast — they block Claude Code while running
6. Configure in `settings.json` with `matcher`, `command`, and `description`
