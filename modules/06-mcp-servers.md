# Module 6: MCP Servers & Plugins — Extending Capabilities

**Duration**: 60 minutes | **Level**: Intermediate

## TL;DR

MCP servers and plugins extend Claude Code beyond its built-in tools. MCP servers connect Claude to browsers, databases, and APIs. Plugins bundle agents, skills, and tools from community marketplaces. Configure MCP in `.mcp.json`; install plugins with `/install`.

---

## 6.1 What is MCP?

The Model Context Protocol is a standard for connecting AI agents to external tools and data sources. Each MCP server exposes:

- **Tools** — Functions Claude can call (e.g., "take a screenshot", "query database")
- **Resources** — Data Claude can read (e.g., documentation, schemas)

MCP servers run as local processes that Claude Code communicates with via stdio or HTTP.

## 6.2 Recommended MCP Servers

### Daily Use Essentials

| Server | Purpose | Why Use It |
|--------|---------|------------|
| **Context7** | Documentation lookup | Current docs for any library — beats training data |
| **Playwright** | Browser automation | Test UIs, take screenshots, fill forms |
| **DeepWiki** | Knowledge retrieval | Query documentation repositories |

### Specialized Servers

| Server | Purpose |
|--------|---------|
| **Claude in Chrome** | Interact with the user's browser session |
| **Excalidraw** | Create diagrams and visualizations |
| **PostgreSQL** | Query databases directly |
| **Sentry** | Query error tracking |

### Recommended Workflow

```
Research (Context7, DeepWiki)
    ↓
Implement (built-in tools)
    ↓
Debug (Playwright, Chrome)
    ↓
Document (built-in tools)
```

## 6.3 Configuration

### Project-level: `.mcp.json`

Committed to git, shared with the team:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@anthropic-ai/mcp-playwright@latest"],
      "type": "stdio"
    },
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"],
      "type": "stdio"
    },
    "deepwiki": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-deepwiki@latest"],
      "type": "stdio"
    }
  }
}
```

### User-level: `~/.claude.json`

Personal servers, not shared:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://..."
      },
      "type": "stdio"
    }
  }
}
```

### Transport types

| Type | How it works | Use when |
|------|-------------|----------|
| `stdio` | Runs as a subprocess, communicates via stdin/stdout | Default. Most servers. |
| `http` | Connects to a running HTTP server | Remote servers, shared services |

## 6.4 Using MCP Tools

Once configured, MCP tools appear in Claude Code's tool list. Claude uses them automatically when relevant.

### Example: Context7 for Documentation

```
You: How do I use the new Drizzle ORM batch API?
Claude: Let me look up the current Drizzle documentation...
[Uses context7 resolve-library-id to find drizzle-orm]
[Uses context7 query-docs to fetch batch API docs]
Here's how to use Drizzle's batch API...
```

### Example: Playwright for Testing

```
You: Open our dev server and check if the login page works
Claude: I'll navigate to the login page and test it...
[Uses playwright browser_navigate to http://localhost:3000/login]
[Uses playwright browser_snapshot to see the page]
[Uses playwright browser_fill_form to enter credentials]
[Uses playwright browser_click to submit]
[Uses playwright browser_take_screenshot to capture result]
The login page works correctly. Here's what I see...
```

## 6.5 Permission Settings

Control which MCP tools Claude can use:

### In settings.json

```json
{
  "permissions": {
    "allow": [
      "mcp__context7__resolve-library-id",
      "mcp__context7__query-docs",
      "mcp__playwright__browser_navigate",
      "mcp__playwright__browser_snapshot"
    ],
    "deny": [
      "mcp__playwright__browser_run_code_unsafe"
    ]
  }
}
```

### Scope hierarchy

```
Project .claude/settings.json     ← Project-specific permissions
User ~/.claude/settings.json      ← Personal defaults
```

## 6.6 Managing MCP Servers

### Install a server

```
You: /install @anthropic-ai/mcp-playwright
```

Or use the `/mcp` command to manage servers.

### Check server status

```
You: /mcp
Claude: Active MCP servers:
  ✓ context7 — running
  ✓ playwright — running
  ✗ deepwiki — not configured
```

### Diagnose issues

```bash
claude doctor
```

This checks MCP server connectivity among other things.

## 6.7 Building Custom MCP Servers

If you need Claude to interact with a custom system, you can build your own MCP server. The protocol is straightforward:

1. Accept connections via stdio or HTTP
2. Respond to `tools/list` with available tool definitions
3. Handle `tools/call` with tool execution

Example tools a custom server might expose:
- Query your company's internal API
- Interact with your CI/CD system
- Read from your documentation wiki
- Post to your team's Slack channels

## 6.8 Plugins & Marketplaces

Plugins are a higher-level extension system that bundles agents, skills, hooks, and tools into installable packages.

### What Plugins Provide

| Component | What It Adds |
|-----------|-------------|
| **Agents** | Specialized subagent definitions |
| **Skills** | Reusable procedures |
| **Hooks** | Lifecycle automation |
| **MCP Servers** | External tool connections |
| **Slash Commands** | Custom commands |

### Installing Plugins

```
/install context7
```

Or from a specific marketplace:

```
/install security-review@claude-plugins-official
```

### Official vs Community Marketplaces

| Source | Trust Level | Example |
|--------|-------------|---------|
| **Official** (`claude-plugins-official`) | Curated by Anthropic | context7, playwright, github, security-guidance |
| **Community** | Third-party, review before use | Various specialized plugins |

### Managing Plugins

View installed plugins:
```
/mcp
```

Enable/disable in `settings.json`:
```json
{
  "enabledPlugins": {
    "context7": true,
    "phpactor": false
  }
}
```

### Plugin Best Practices

1. **Start with official plugins** -- they're curated and maintained
2. **One LSP per language** -- don't enable both `phpactor` and `php-lsp`
3. **Disable unused plugins** -- each active plugin consumes resources and context
4. **Audit community sources** -- review what a plugin does before installing
5. **Set `enableAllProjectMcpServers: false`** -- never auto-trust project MCP configs from unknown repos

### The Ecosystem

The Claude Code ecosystem (as of mid-2026) includes:
- **4,200+** community skills
- **770+** MCP servers
- **2,500+** marketplace entries

See [Plugins & Tools Directory](../resources/plugins-and-tools.md) for a curated list.

---

## Exercise 6.1: Set Up Context7

1. Create `.mcp.json` in your project root:
```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"],
      "type": "stdio"
    }
  }
}
```

2. Restart Claude Code
3. Ask: `What's the latest API for [a library you use]?`
4. Watch Claude use Context7 to fetch current documentation

## Exercise 6.2: Set Up Playwright

1. Add Playwright to your `.mcp.json`:
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@anthropic-ai/mcp-playwright@latest"],
      "type": "stdio"
    }
  }
}
```

2. Start your dev server
3. Ask Claude to navigate to your app and test a feature
4. Ask for a screenshot of a specific page

## Exercise 6.3: Permission Management

1. Add MCP tool permissions to `.claude/settings.json`
2. Allow documentation tools but restrict browser automation
3. Test that Claude can look up docs but asks permission for browser actions

---

## Exercise 6.4: Install a Plugin

1. Run `/install context7` to install from the official marketplace
2. Check the plugin loaded: `/mcp`
3. Test it: ask Claude about a library's latest API
4. Disable it in settings to see the difference

---

## Common Pitfalls

- **Not restarting after `.mcp.json` changes**: MCP servers load at session start. Restart Claude Code after editing config.
- **Missing npx/node**: MCP servers typically need Node.js installed globally.
- **Forgetting environment variables**: Database servers need connection strings. Use the `env` field.
- **Too many servers/plugins**: Each consumes resources and context. Only configure what you actually use.
- **Enabling all project MCP servers**: Never set `enableAllProjectMcpServers: true` -- untrusted projects could load malicious servers.

## Key Takeaways

1. MCP servers extend Claude Code with external tools -- browsers, databases, docs, APIs
2. Plugins bundle agents, skills, and tools into installable packages from marketplaces
3. Context7 for docs and Playwright for browser automation are the essential servers
4. Configure MCP in `.mcp.json` (project) or `~/.claude.json` (personal)
5. Install plugins with `/install` from official or community marketplaces
6. Permission settings control which MCP tools Claude can use
7. Disable unused plugins and audit community sources before installing
