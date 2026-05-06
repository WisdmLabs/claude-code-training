# Module 0: Setup & First Run

**Duration**: 30 minutes | **Level**: Beginner

## TL;DR

Install Claude Code, authenticate, run your first prompt, and understand the basic interaction model.

---

## 0.1 Installation

### Install via npm

```bash
npm install -g @anthropic-ai/claude-code
```

### Verify installation

```bash
claude --version
```

### Authenticate

```bash
claude auth login
```

You can authenticate with:
- **Anthropic API key** — direct API access
- **Claude Max/Pro subscription** — consumer plan usage
- **OAuth** — enterprise SSO

### First launch

```bash
cd your-project
claude
```

Claude Code launches in your terminal with an interactive prompt. It sees the working directory and can read/write files, run commands, and interact with git.

## 0.2 Where Claude Code Runs

Claude Code is available in multiple environments:

| Environment | How to access |
|-------------|---------------|
| **Terminal CLI** | `claude` command in any terminal |
| **Desktop App** | Mac and Windows native app |
| **Web App** | claude.ai/code |
| **VS Code** | Extension from marketplace |
| **JetBrains** | Plugin from marketplace |

All environments share the same core engine. The CLI is the most full-featured.

## 0.3 The Interaction Model

Claude Code is **not** a chatbot. It is an **agent** that:

1. Reads your codebase
2. Plans changes
3. Edits files directly
4. Runs commands (with your permission)
5. Iterates until the task is done

You give it tasks, not questions. Instead of "How do I add a login page?", say "Add a login page with email/password using NextAuth."

## 0.4 Permission Modes

Claude Code asks for permission before taking actions. You control how strict this is:

| Mode | Behavior |
|------|----------|
| **default** | Asks for most file writes and all shell commands |
| **auto** | Allows reads, writes, and safe commands automatically |
| **bypassPermissions** | No prompts (use with caution) |
| **plan** | Requires plan approval before implementation |

Set at launch:

```bash
claude --permission-mode plan
```

## 0.5 Essential First Commands

Once inside Claude Code, try these:

| Command | What it does |
|---------|-------------|
| `/help` | Show available commands |
| `/status` | Show current session info |
| `/model` | Switch the model |
| `/clear` | Clear conversation context |
| `/compact` | Compress conversation to save context |
| `Ctrl+C` | Cancel current operation |
| `Escape` | Exit Claude Code |

---

## Exercise 0.1: Hello Claude Code

1. Open a terminal and navigate to any project directory
2. Run `claude`
3. Type: `What files are in this project? Give me a brief summary.`
4. Observe how Claude reads the directory and responds

## Exercise 0.2: Your First Edit

1. In Claude Code, type: `Create a file called hello.txt with a greeting`
2. Approve the file write when prompted
3. Check that the file was created: `cat hello.txt`
4. Delete the file: `rm hello.txt`

## Exercise 0.3: Permission Modes

1. Exit Claude Code (Escape)
2. Restart with: `claude --permission-mode plan`
3. Ask Claude to make a change — notice it creates a plan first
4. Approve or reject the plan

---

## Common Pitfalls

- **Running Claude outside a project directory**: Claude works best when launched from a project root with meaningful code. Running it from `~` gives it no context.
- **Not reading the permission prompts**: Always read what Claude wants to do before approving, especially shell commands.
- **Treating it like ChatGPT**: Give tasks and instructions, not open-ended questions.

## Key Takeaways

1. Claude Code is an agent that operates on your codebase, not a chatbot
2. Always launch from your project directory
3. Permission modes let you control how much autonomy Claude has
4. The CLI is the most full-featured environment
