# Claude Code Installed Plugins

Last updated: 2026-03-25

---

## 1. WisdmLabs Engineering (`wisdmlabs-engineering`)

- **Version:** 5.36.0
- **Source:** `aruneshwisdm/wisdmlabs-engineering-plugin` (GitHub)
- **Installed:** 2026-01-20 | **Last Updated:** 2026-03-23

### What it does

A comprehensive engineering harness with 150+ specialized agents covering the full software development lifecycle. Includes agents for:

- **Code review** — WordPress, Moodle, Shopify, and WooCommerce standards reviewers
- **Architecture** — API design review, dependency analysis, scalability assessment
- **Testing** — E2E test generation (Playwright), unit tests (PHPUnit), TDD loops, visual regression
- **Security** — Vulnerability scanning, penetration test planning, privacy/GDPR compliance
- **Estimation** — Project estimation, task complexity analysis, risk assessment
- **UI/UX** — Figma-to-WordPress conversion, design token extraction, accessibility audits
- **Git workflow** — PR review automation, merge conflict resolution, incident management
- **Documentation** — API docs generation, stakeholder reporting
- **Orchestration** — Multi-agent coordination, CI/CD setup, scaffold generation
- **Business analysis** — Requirements elicitation, BRD generation, stakeholder mapping

Also provides MCP tools for team learnings, estimation tracking, and Playwright browser automation. Includes 60+ slash-command skills for BA workflows, UI/UX design, testing, and orchestration.

### Installation

```bash
# Add the marketplace
claude plugins add-marketplace https://github.com/aruneshwisdm/wisdmlabs-engineering-plugin.git

# Install the plugin
claude plugins install wisdmlabs-engineering

# Set up MCP dependencies (run after install or update)
claude /wisdmlabs-engineering:setup-mcp
```

---

## 2. Context7 (`context7`)

- **Source:** `anthropics/claude-plugins-official` (GitHub)
- **Installed:** 2026-01-29

### What it does

Provides up-to-date documentation and code examples for any library via MCP tools. When you need to look up current API docs, usage patterns, or code samples for a third-party library, Context7 fetches them in real-time rather than relying on training data.

**MCP tools:** `resolve-library-id`, `query-docs`

### Installation

```bash
# The official marketplace is added by default
# If not present:
claude plugins add-marketplace anthropics/claude-plugins-official

# Install
claude plugins install context7
```

---

## 3. Claude Mem (`claude-mem`)

- **Version:** 10.2.3
- **Source:** `thedotmack/claude-mem` (GitHub)
- **Installed:** 2026-02-17

### What it does

Persistent cross-session memory system via MCP. Stores observations and lets you search them in future conversations. Useful for remembering decisions, preferences, and context across sessions.

**MCP tools:** `save_memory`, `search`, `get_observations`, `timeline`
**Skills:** `/claude-mem:do`, `/claude-mem:make-plan`, `/claude-mem:mem-search`

### Installation

```bash
claude plugins add-marketplace thedotmack/claude-mem
claude plugins install claude-mem
```

---

## 4. Frontend Design (`frontend-design`)

- **Source:** `anthropics/claude-plugins-official` (GitHub)
- **Installed:** 2026-01-29

### What it does

Creates distinctive, production-grade frontend interfaces with high design quality. Provides the `/frontend-design:frontend-design` skill for building polished UIs from specifications or descriptions.

### Installation

```bash
claude plugins install frontend-design
# (from the official marketplace)
```

---

## 5. Figma (`figma`)

- **Version:** 1.1.0
- **Source:** `anthropics/claude-plugins-official` (GitHub)
- **Installed:** 2026-02-26

### What it does

Connects Figma designs to code. Provides skills for:

- **Code Connect** — Links Figma components to code components (`/figma:code-connect-components`)
- **Design system rules** — Generates custom design system rules from your codebase (`/figma:create-design-system-rules`)
- **Implement design** — Translates Figma designs into production-ready code with 1:1 visual fidelity (`/figma:implement-design`)

### Installation

```bash
claude plugins install figma
```

---

## 6. Vercel (`vercel`)

- **Version:** 1.0.0
- **Source:** `anthropics/claude-plugins-official` (GitHub)
- **Installed:** 2026-02-26

### What it does

Deploy and manage Vercel projects directly from Claude Code. Provides skills for:

- `/vercel:setup` — Configure Vercel CLI and project
- `/vercel:deploy` — Deploy the current project to Vercel
- `/vercel:logs` — View deployment logs

### Installation

```bash
claude plugins install vercel
```

---

## 7. GitHub (`github`)

- **Source:** `anthropics/claude-plugins-official` (GitHub)
- **Installed:** 2026-02-26

### What it does

Enhanced GitHub integration beyond the built-in `gh` CLI. Provides additional tools and workflows for working with GitHub issues, PRs, and repositories.

### Installation

```bash
claude plugins install github
```

---

## 8. Security Guidance (`security-guidance`)

- **Source:** `anthropics/claude-plugins-official` (GitHub)
- **Installed:** 2026-02-26

### What it does

Provides security best practices and guidance rules. Helps ensure code follows security standards and avoids common vulnerabilities (OWASP top 10, injection, XSS, etc.).

### Installation

```bash
claude plugins install security-guidance
```

---

## 9. PHP LSP (`php-lsp`)

- **Version:** 1.0.0
- **Source:** `anthropics/claude-plugins-official` (GitHub)
- **Installed:** 2026-02-26

### What it does

PHP Language Server Protocol integration. Provides IDE-like PHP intelligence — go-to-definition, find references, autocompletion, and diagnostics for PHP codebases.

### Installation

```bash
claude plugins install php-lsp
```

---

## 10. TypeScript LSP (`typescript-lsp`)

- **Version:** 1.0.0
- **Source:** `anthropics/claude-plugins-official` (GitHub)
- **Installed:** 2026-02-26

### What it does

TypeScript/JavaScript Language Server Protocol integration. Provides type checking, go-to-definition, find references, and diagnostics for TS/JS codebases.

### Installation

```bash
claude plugins install typescript-lsp
```

---

## 11. PHPActor (`phpactor`)

- **Version:** 0.1.0
- **Source:** `Piebald-AI/claude-code-lsps` (GitHub)
- **Installed:** 2026-01-28

### What it does

Alternative PHP language server providing refactoring, code generation, and navigation capabilities. Complements the official PHP LSP with additional refactoring tools.

### Installation

```bash
claude plugins add-marketplace Piebald-AI/claude-code-lsps
claude plugins install phpactor
```

---

## 12. Pyright (`pyright`)

- **Version:** 0.1.0
- **Source:** `Piebald-AI/claude-code-lsps` (GitHub)
- **Installed:** 2026-01-28

### What it does

Python language server based on Microsoft's Pyright. Provides static type checking, type inference, go-to-definition, and diagnostics for Python codebases.

### Installation

```bash
claude plugins add-marketplace Piebald-AI/claude-code-lsps
claude plugins install pyright
```

---

## 13. VS Code Language Servers (`vscode-langservers`)

- **Version:** 0.1.0
- **Source:** `Piebald-AI/claude-code-lsps` (GitHub)
- **Installed:** 2026-01-28

### What it does

Bundles VS Code's built-in language servers for HTML, CSS, and JSON. Provides validation, autocompletion, and diagnostics for web markup and config files.

### Installation

```bash
claude plugins add-marketplace Piebald-AI/claude-code-lsps
claude plugins install vscode-langservers
```

---

## 14. VTSLS (`vtsls`)

- **Version:** 0.1.0
- **Source:** `Piebald-AI/claude-code-lsps` (GitHub)
- **Installed:** 2026-01-28

### What it does

Alternative TypeScript language server (VS Code TypeScript Language Service wrapper). Provides TypeScript/JavaScript intelligence as a standalone LSP server.

### Installation

```bash
claude plugins add-marketplace Piebald-AI/claude-code-lsps
claude plugins install vtsls
```

---

## 15. AIBotKit Engineering (`aibotkit-engineering`) [Local]

- **Version:** 1.5.0
- **Source:** Local plugin at `/home/arunesh/projects/aibotkit-claude-plugin`
- **Installed:** 2026-01-17

### What it does

Custom local plugin for AIBotKit-specific engineering workflows. Installed from a local path rather than a marketplace.

### Installation

```bash
claude plugins install --local /home/arunesh/projects/aibotkit-claude-plugin
```

---

## Marketplace Summary

| Marketplace | Source | Plugins Installed |
|---|---|---|
| `claude-plugins-official` | `anthropics/claude-plugins-official` | context7, frontend-design, figma, vercel, github, security-guidance, php-lsp, typescript-lsp |
| `wisdmlabs-engineering-marketplace` | `aruneshwisdm/wisdmlabs-engineering-plugin` | wisdmlabs-engineering |
| `claude-code-lsps` | `Piebald-AI/claude-code-lsps` | phpactor, pyright, vscode-langservers, vtsls |
| `thedotmack` | `thedotmack/claude-mem` | claude-mem |
| *(local)* | — | aibotkit-engineering |
