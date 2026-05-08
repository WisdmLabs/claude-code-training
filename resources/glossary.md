# Claude Code Glossary

## Core Concepts

**Agentic Loop**: The think → tool → observe → think cycle Claude uses to complete tasks. Each iteration may involve multiple tool calls.

**CLAUDE.md**: Markdown files with persistent instructions loaded into every session. Forms a hierarchy from global to project-specific.

**Command**: A user-initiated slash-command (`/name`) that serves as an entry point for workflows. Runs in the main session context. Never auto-invoked.

**Compact**: The `/compact` command compresses conversation history to free context space. Essential for long sessions.

**Context Window**: The finite working memory of a session. Includes system prompt, CLAUDE.md, conversation history, and tool results. Degrades as it fills.

**Context Degradation**: Loss of recall and quality as the context window fills with conversation history and tool outputs. Mitigate with fresh sessions and `/compact`.

**Memory System**: Persistent, file-based storage in `~/.claude/projects/<path>/memory/` for facts and learnings that survive across sessions. Types: user, feedback, project, reference.

**Prompt Caching**: Automatic reuse of processed context (system prompt, CLAUDE.md, early conversation) across turns. Cache TTL is ~5 minutes. Reduces cost and latency.

## Extensions

**Agent (Subagent)**: An isolated Claude instance spawned within a session. Has its own context window. Used for research, review, and parallel work.

**Agent Teams**: Multiple independent Claude Code sessions coordinating via shared files. Experimental feature for large, multi-component work.

**Hook**: A shell command that fires on a Claude Code event (session start, tool use, etc.). Deterministic — always runs the same way. Used for validation, logging, and automation.

**MCP (Model Context Protocol)**: A standard protocol for connecting Claude to external tools and data sources. MCP servers expose tools and resources.

**MCP Server**: A process that provides tools to Claude via the MCP protocol. Examples: Playwright (browser), Context7 (docs), PostgreSQL (database).

**Plugin**: An installable package that bundles agents, skills, hooks, and MCP servers from a marketplace. Installed with `/install`.

**Marketplace**: A repository of plugins. Official marketplace is `claude-plugins-official`; community marketplaces are third-party.

**Skill**: A reusable procedure defined in markdown. Can be auto-invoked based on user intent or called explicitly. Runs inline in the current context.

## Architecture

**Command → Agent → Skill**: The canonical orchestration pattern. Commands are entry points, agents are autonomous workers, skills are reusable procedures.

**Data Contract**: An agreed-upon data format between components (command, agent, skill). Ensures consistent, reliable data flow.

**Orchestration**: Coordinating multiple agents and skills to complete a complex task. Typically managed by a command.

**PROACTIVE**: A keyword in agent descriptions that triggers automatic spawning when the user's intent matches.

**Worktree Isolation**: Running an agent in a git worktree — an isolated copy of the repo. Changes don't affect the main working directory.

## Tools

**Agent Tool**: Spawns a subagent. The core mechanism for delegation and parallel work.

**Bash Tool**: Executes shell commands. Used for running tests, building, git operations, etc.

**Edit Tool**: Makes targeted string replacements in files. Preferred over Write for modifications.

**Read Tool**: Reads files from disk. The primary way Claude understands code.

**WebFetch Tool**: Fetches content from URLs. Used by MCP servers and for documentation lookup.

**Write Tool**: Creates new files or fully rewrites existing ones. Use Edit for partial modifications.

## Modes & Flags

**Auto Mode**: Permission mode that allows reads, writes, and safe commands without prompting.

**Bare Mode** (`--bare`): Minimal startup — skips MCP servers and extensions for faster launch.

**Fast Mode** (`/fast`): Faster output with the same model. Toggle during a session.

**Plan Mode**: Permission mode that requires approval of a plan before implementation begins.

**Print Mode** (`--print`): Non-interactive, one-shot execution. Outputs result and exits. Used for scripting and CI/CD.

## Features

**Teleport** (`/teleport`): Transfer a session to another device via a shareable link.

**Loop** (`/loop`): Schedule a task to run at recurring intervals within a session.

**Schedule** (`/schedule`): Create a scheduled remote agent that runs on a cron schedule.

**Ultrareview** (`/ultrareview`): Multi-agent cloud-powered code review of the current branch.

**Batch** (`/batch`): Process multiple files with the same prompt.

## Files & Configuration

**`.mcp.json`**: Project-level MCP server configuration. Committed to git.

**`~/.claude.json`**: User-level MCP server configuration. Not committed.

**`settings.json`**: Configuration for permissions, hooks, and environment variables. Exists at global (`~/.claude/`) and project (`.claude/`) levels.

**`settings.local.json`**: Local settings overrides. Gitignored. Highest precedence.

**`.claudeignore`**: File listing patterns to exclude from Claude's context (like `.gitignore`). Prevents reading build artifacts, binaries, etc.

**`MEMORY.md`**: Index file in the memory directory. Loaded every session. Maps to individual memory files.

**`keybindings.json`**: Custom keyboard shortcuts in `~/.claude/keybindings.json`.

## Thinking Keywords

**think**: Triggers extended thinking with a standard token budget.

**think hard**: Triggers extended thinking with a larger token budget for complex problems.

**ultrathink**: Triggers maximum thinking budget. Use for architecture decisions and gnarly debugging.
