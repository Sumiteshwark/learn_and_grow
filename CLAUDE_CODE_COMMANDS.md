**Last Updated:** November 2025  
**Author:** SUMITESHWAR KUMAR

---

# Claude Code CLI — Complete Cheatsheet

> A quick-reference guide for all important Claude Code commands and shortcuts.

---

## 🚀 Getting Started

| Command | Description |
|---|---|
| `claude` | Launch Claude Code (interactive REPL) |
| `claude --version` | Check installed version |
| `claude --help` | Show all available CLI options |
| `claude --doctor` | Run diagnostics to check setup & health |

---

## 💬 Running Prompts

| Command | Description |
|---|---|
| `claude` | Start an interactive chat session |
| `claude "your prompt"` | Send a one-off prompt |
| `claude -p "prompt"` | Print mode — outputs response and exits (good for piping) |
| `cat file.ts \| claude "explain this"` | Pipe file content as context |
| `claude --add-dir /path` | Add a directory to the conversation context |

---

## 🔄 Session Management

| Command | Description |
|---|---|
| `claude --continue` | Continue the most recent conversation |
| `claude -c` | Shorthand for `--continue` |
| `claude --resume` | Open and resume a past session (shows session picker) |
| `claude --resume "session-name"` | Resume a specific named session |
| `claude --resume <session-id>` | Resume a session by its ID |

---

## ⚙️ Configuration & Model

| Command | Description |
|---|---|
| `claude config` | View/edit configuration |
| `claude config set model <model>` | Set the default model |
| `--model <model>` | Use a specific model for this session only |
| `--max-turns N` | Limit agentic turns in non-interactive mode |
| `--output-format json` | Get structured JSON output |
| `--verbose` | Enable verbose/debug logging |

---

## 🎮 Inside the REPL — Slash Commands

### Navigation & Session

| Command | Description |
|---|---|
| `/exit` | Leave Claude Code |
| `/quit` | Same as `/exit` |
| `Ctrl + C` | Cancel current action or exit |
| `/clear` | Clear conversation history |
| `/rename <name>` | Rename the current session |
| `/compact` | Summarize & compact the conversation (saves context) |

### Model & Usage

| Command | Description |
|---|---|
| `/model` | Change the AI model |
| `/usage` | Show token usage & cost for the session |
| `/cost` | Show cost breakdown |
| `/effort` | Adjust effort level (low/medium/high) |

### Tools & Configuration

| Command | Description |
|---|---|
| `/init` | Create a `CLAUDE.md` project file with instructions |
| `/theme` | Change terminal color theme |
| `/doctor` | Run diagnostics inside the REPL |
| `/voice` | Enable voice mode |
| `?` | Show all keyboard shortcuts & help |

---

## ⌨️ Keyboard Shortcuts

| Shortcut | Description |
|---|---|
| `Shift + Tab` | Toggle **Plan Mode** (think before acting) |
| `Ctrl + C` | Cancel current generation / exit |
| `Enter` | Submit prompt |
| `Shift + Enter` | New line in prompt |

---

## 📋 Plan Mode (Shift + Tab)

Plan mode tells Claude to **think and plan** before making changes. It cycles through:

- **Plan mode ON** — Claude will outline its plan before executing
- **Plan mode OFF** — Claude executes directly

> [!TIP]
> Use Plan Mode for complex tasks where you want to review the approach before Claude makes changes.

---

## 🗂️ Project Setup

| Command | Description |
|---|---|
| `/init` | Create a `CLAUDE.md` file in your project root |

The `CLAUDE.md` file lets you define:
- Project-specific instructions
- Coding conventions & style guidelines
- File structure context
- Custom rules Claude should follow

---

## 📊 Effort Levels

Accessible via `/effort` inside the REPL:

| Level | Behavior |
|---|---|
| **Low** | Quick, concise responses |
| **Medium** | Balanced (default) |
| **High** | Thorough, detailed responses |

---

## 🔌 MCP Servers (Skills)

Claude Code can be extended with external skills (like fetching live documentation) via the Model Context Protocol (MCP).

| Scope | Command Example | Where it saves |
|---|---|---|
| **Global (All Projects)** | `claude mcp add --scope user fetch npx -y @modelcontextprotocol/server-fetch` | `~/.claude.json` |
| **Local (Current Repo Only)** | `claude mcp add fetch npx -y @modelcontextprotocol/server-fetch` | `[Project Root]/.claude.json` |
| **List Installed Servers** | `claude mcp list` | - |
| **Remove Local Server** | `claude mcp remove <server_name>` | `[Project Root]/.claude.json` |
| **Remove Global Server** | `claude mcp remove --scope user <server_name>` | `~/.claude.json` |

*(Note: To install for the current repository only, omit the `--scope user` flag).*

---

## 💡 Common Workflows

### Explain a file
```bash
claude "explain src/api/auth.ts"
```

### Fix a bug
```bash
claude "fix the login validation bug in auth.ts"
```

### Generate tests
```bash
claude "write unit tests for src/api/orders.ts"
```

### Refactor code
```bash
claude "refactor the ProductCard component to use hooks"
```

### Continue where you left off
```bash
claude --continue
```

### Resume a named session
```bash
claude --resume "my-feature-work"
```

### Pipe and process
```bash
cat src/api/auth.ts | claude -p "find security issues"
```

---

## 🔑 Quick Reference Card

```
┌─────────────────────────────────────────────────┐
│           CLAUDE CODE QUICK REFERENCE           │
├─────────────────────────────────────────────────┤
│  Launch:          claude                        │
│  Diagnostics:     claude --doctor               │
│  Continue:        claude --continue  or  -c     │
│  Resume Session:  claude --resume               │
│  One-off prompt:  claude "prompt"               │
│  Pipe mode:       claude -p "prompt"            │
├─────────────────────────────────────────────────┤
│  INSIDE REPL:                                   │
│  Exit:            /exit  or  Ctrl+C             │
│  Change Model:    /model                        │
│  Rename Session:  /rename <name>                │
│  Show Usage:      /usage                        │
│  Plan Mode:       Shift + Tab                   │
│  Help:            ?                             │
│  Init Project:    /init                         │
│  Compact Chat:    /compact                      │
│  Voice Mode:      /voice                        │
└─────────────────────────────────────────────────┘
```

---