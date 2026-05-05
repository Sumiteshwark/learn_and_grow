**Last Updated:** May 2026
**Author:** SUMITESHWAR KUMAR

---

# Claude Code Mastery - Learning Checklist

## Table of Contents

0. [Essential Commands & Shortcuts](#0-essential-commands-shortcuts-foundation)
1. [CLAUDE.md (Project Memory)](#1-claude-md-project-memory)
2. [Skills (Custom Slash Commands)](#2-skills-custom-slash-commands)
3. [Hooks (Lifecycle Automation)](#3-hooks-lifecycle-automation)
4. [Subagents](#4-subagents)
5. [MCP (Model Context Protocol)](#5-mcp-model-context-protocol)
6. [Headless Mode (Scripting)](#6-headless-mode-scripting)

---

## 0. Essential Commands & Shortcuts (Foundation)

### CLI Commands

**Launch & Help**
| Command | Description |
|---|---|
| `claude` | Start interactive session |
| `claude --help` | Show all CLI options |
| `claude --doctor` | Run diagnostics |

**Prompts & Piping**
| Command | Description |
|---|---|
| `claude "prompt"` | One-off prompt (interactive) |
| `claude -p "prompt"` | Headless mode (print and exit) |
| `cat file \| claude -p "explain"` | Pipe input as context |
| `claude --add-dir /path` | Add directory to context |

**Sessions**
| Command | Description |
|---|---|
| `claude -c` | Continue most recent session |
| `claude --resume` | Pick a past session to resume |
| `claude --resume <id>` | Resume specific session |

**Model & Behavior**
| Command | Description |
|---|---|
| `claude --model opus` | Use specific model (opus/sonnet/haiku) |
| `claude --max-turns 5` | Limit agent turns (safety) |
| `claude --output-format json` | Structured JSON output |
| `claude --bare` | Skip CLAUDE.md, hooks, skills, MCP |
| `claude --allowedTools "Bash,Read"` | Pre-approve tools (no prompts) |
| `claude --verbose` | Debug logging |

### Slash Commands (inside session)

**Session**
| Command | Description |
|---|---|
| `/exit` | Leave session |
| `/clear` | Clear conversation |
| `/compact` | Summarize & compress conversation |
| `/rename <name>` | Rename current session |
| `/transcript` | View session transcript |

**Model & Cost**
| Command | Description |
|---|---|
| `/model` | Switch AI model |
| `/effort` | Adjust thinking depth (low/medium/high/xhigh/max) |
| `/usage` | Show token usage |
| `/cost` | Show cost breakdown |

**Project & Memory**
| Command | Description |
|---|---|
| `/init` | Generate CLAUDE.md for project |
| `/memory` | View CLAUDE.md + auto memory |
| `/context` | See what's loaded in context |
| `/help` | Show all slash commands |

**Extensions**
| Command | Description |
|---|---|
| `/agents` | List/manage subagents |
| `/hooks` | View configured hooks |
| `/mcp` | List active MCP servers |
| `/permissions` | View/manage permissions |
| `/review` | Code review of pending changes |
| `/security-review` | Security review of changes |

### Keyboard Shortcuts

| Shortcut | Description |
|---|---|
| `Enter` | Submit prompt |
| `Shift + Enter` | New line in prompt |
| `Shift + Tab` | Toggle Plan Mode (think before acting) |
| `Esc Esc` | Open rewind menu (undo to earlier point) |
| `Ctrl + C` | Cancel current generation |
| `Ctrl + L` | Clear screen |
| `Ctrl + O` | Toggle verbose thinking mode |
| `Option/Alt + T` | Toggle extended thinking per session |

### Plan Mode (Shift + Tab)
```
Plan Mode ON   → Claude designs/plans before executing
Plan Mode OFF  → Claude executes directly (default)

Use for: complex changes, refactoring, architecture
Skip for: quick edits, simple questions
```

### Effort Levels
```
low      → Quick, minimal thinking
medium   → Balanced (default)
high     → Recommended for coding
xhigh    → Deep reasoning (default on Opus)
max      → Unlimited thinking (one session only)

Set via: /effort <level>
```

### Common Workflows
```bash
# One-shot tasks
claude "explain src/api/auth.ts"
claude "fix the login bug in auth.ts"
claude "write tests for src/orders.ts"

# Piping
cat error.log | claude -p "diagnose"
git diff | claude -p "review for bugs"

# Sessions
claude -c                          # continue last
claude --resume my-feature         # resume named

# Headless with safety
claude -p "fix tests" --allowedTools "Bash,Read,Edit" --max-turns 5

# Cheap & fast
claude -p "summarize" --model haiku --bare
```

### Quick Reference Card
```
┌─────────────────────────────────────────────────┐
│  CLI:                                           │
│   Launch:           claude                      │
│   Continue:         claude -c                   │
│   Headless:         claude -p "..."             │
│   Resume:           claude --resume             │
├─────────────────────────────────────────────────┤
│  INSIDE SESSION:                                │
│   Help:             ?  or  /help                │
│   Exit:             /exit  or  Ctrl+C           │
│   Plan Mode:        Shift + Tab                 │
│   Rewind:           Esc Esc                     │
│   Clear context:    /compact                    │
│   Init project:     /init                       │
│   View memory:      /memory                     │
│   View context:     /context                    │
│   List agents:      /agents                     │
│   List hooks:       /hooks                      │
│   List MCP:         /mcp                        │
│   Permissions:      /permissions                │
│   Cost:             /cost  /usage               │
└─────────────────────────────────────────────────┘
```

---

## 1. CLAUDE.md (Project Memory) 

### What it is
Markdown file Claude reads at start of EVERY session. Persistent context = no repeating yourself.

### 3 Levels (loaded in this order)
```
1. Managed:  /Library/Application Support/ClaudeCode/CLAUDE.md  (org-level, locked)
2. User:     ~/.claude/CLAUDE.md                                 (you, all projects)
3. Project:  ./CLAUDE.md                                         (codebase, COMMIT TO GIT)
4. Path-scoped: ./.claude/rules/*.md                             (lazy, by file pattern)
```

### Rules
- **Eager loading** → loaded EVERY session, costs tokens always
- Keep under **200 lines** (rest gets truncated)
- Use `##` headers for sections
- Update when stack/conventions change
- COMMIT project CLAUDE.md to git

### What to PUT in CLAUDE.md
- Build/test/run commands (`npm test`, `npm run dev`)
- Tech stack (languages, frameworks, DBs)
- Architecture overview (folder structure)
- Coding conventions (indentation, naming)
- Project-specific gotchas ("never X", "always Y")
- Domain terminology unique to project

### What NOT to put
- ❌ Long code examples (clutters context)
- ❌ Generic advice ("write clean code")
- ❌ Stuff already in README.md
- ❌ Frequently changing info (use auto memory)
- ❌ Secrets/passwords/API keys

### Auto Memory (separate from CLAUDE.md)
- Location: `~/.claude/projects/<encoded-path>/memory/MEMORY.md`
- Claude writes this automatically (not you)
- Stores: build failures, your preferences, patterns it learned
- View with: `/memory`

### Path-scoped Rules (for big projects)
```yaml
---
paths:
  - "src/api/**/*.ts"
---
# Backend Rules
- All endpoints validate with Zod
- Use AppError class
```
Loads ONLY when Claude touches matching files = saves tokens.

### Quick commands
- `/memory` → view memory + CLAUDE.md
- `/context` → see what's loaded in current session
- `/init` → bootstrap CLAUDE.md for new project

### Template (paste this and customize)
```markdown
# Project: [Name]

## Stack
- [Language, framework, DB]

## Build & Run
- `npm install`
- `npm run dev`
- `npm test`

## Architecture
- /src/api/ — handlers
- /src/services/ — business logic
- /src/db/ — database layer

## Conventions
- 2-space indentation
- TypeScript only
- Zod for validation

## Important Notes
- Never commit .env
- Use AppError class for errors
- Migrations are timestamped — don't rename
```

---

## 2. Skills (Custom Slash Commands) 

### What they are
Reusable workflows you invoke with `/skill-name`. Like macros for Claude.

### Skills vs CLAUDE.md (CRITICAL)
```
CLAUDE.md = EAGER (loaded every session, always in context)
Skills    = LAZY  (loaded ONLY when /skill-name invoked)

Use CLAUDE.md for: stuff Claude needs ALWAYS
Use Skills for:    workflows you do SOMETIMES
```

### Locations
```
~/.claude/skills/<name>/SKILL.md     ← user skills (all projects, personal)
./.claude/skills/<name>/SKILL.md     ← project skills (commit to git, team)
```

### SKILL.md Anatomy
```markdown
---
name: code-review                       # required, kebab-case
description: Review staged changes      # shown in /help
allowed-tools: Bash(git *), Read        # pre-approve tools (optional)
model: opus                              # override model (optional)
context: fork                            # spawn subagent (optional)
agent: Explore                           # which subagent type (optional)
---

# Skill instructions in markdown

Steps Claude should follow...
```

### 4 Power Features

**1. Dynamic Context (`!`shell command``)**
```markdown
## Current diff
!`git diff --cached`

## Files changed
!`git diff --cached --name-only`
```
Shell runs FIRST, output replaces placeholder, Claude sees real data. No tool calls needed.

**2. Arguments (`$ARGUMENTS`)**
```markdown
Explain: $ARGUMENTS
```
Usage: `/explain how does JWT refresh work` → `$ARGUMENTS` = "how does JWT refresh work"

**3. Tool Restrictions (`allowed-tools`)**
```yaml
allowed-tools: Bash(git *), Read
```
Pre-approves these tools (no permission prompts), blocks others by default.

**4. Subagent Mode (`context: fork`)**
```yaml
context: fork
agent: Explore
```
Skill runs in **isolated subagent** → returns only summary → main context stays clean.

### `context` Field
- (no field) / `inline` → runs in main conversation (default)
- `fork` → spawns isolated subagent

### `agent` Field (when context: fork)
- `Explore` → read-only specialist (Read, Grep, Glob)
- `Plan` → architecture/design thinking
- `general-purpose` → balanced default with all tools
- `<custom>` → your custom agent in `.claude/agents/`

### Supporting Files
```
~/.claude/skills/security-audit/
├── SKILL.md              ← main instructions
├── checklist.md          ← reference (loaded on demand)
└── scripts/scan.sh       ← helper script
```

### Best Practices
- ONE skill = ONE focused task
- Names = verbs (review, refactor, deploy)
- Use kebab-case (code-review, not codeReview)
- Description = ONE line max
- Restrict tools (be specific: `Bash(git *)` not `Bash`)
- Big tasks reading many files → `context: fork`
- Project-specific skills → commit to git

### When to use what
```
Quick task, 1-2 files          → no context field (inline)
Big research, many files       → context: fork + agent: Explore
Architecture planning           → context: fork + agent: Plan
Mixed read/write big task       → context: fork + agent: general-purpose
```

### Quick start template
```bash
mkdir -p ~/.claude/skills/code-review
cat > ~/.claude/skills/code-review/SKILL.md << 'EOF'
---
name: code-review
description: Review staged changes for issues
allowed-tools: Bash(git *), Read
---

# Code Review

## Diff
!`git diff --cached`

## Files
!`git diff --cached --name-only`

Provide:
1. Critical issues (security, bugs)
2. Code quality concerns
3. Improvement suggestions
EOF
```
Then in Claude Code: `/code-review`

---

## 3. Hooks (Lifecycle Automation)

### What they are
Scripts that auto-run at lifecycle events. Turn Claude into a **policy-enforced platform**. Enforcement at SYSTEM level, not Claude's discretion.

### Lifecycle Events (When hooks fire)
```
SessionStart        → session starts/resumes (inject context)
UserPromptSubmit    → before Claude processes prompt
PreToolUse          → before any tool runs (★ MOST USEFUL - can block!)
PostToolUse         → after tool succeeds (★ auto-format, lint)
PostToolUseFailure  → after tool fails
PermissionRequest   → permission dialog appears
Stop                → Claude finishes responding
SessionEnd          → session terminates
```

### Hook Types (5 ways)
```
command   → shell script (95% of use cases)
http      → POST to URL
mcp_tool  → call MCP server tool
prompt    → ask Claude yes/no
agent     → spawn subagent for decision
```

### Settings.json Locations
```
~/.claude/settings.json              ← personal (all sessions)
./.claude/settings.json              ← project (commit to git)
./.claude/settings.local.json        ← personal project (don't commit)
```

### Configuration Structure
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/check.sh"
          }
        ]
      }
    ]
  }
}
```

### Matchers (Pattern Syntax)
```
"*"                    → ALL tools
"Bash"                 → only Bash
"Bash(git *)"          → Bash with git
"Bash(rm -rf *)"       → catch dangerous rm
"Edit|Write"           → Edit OR Write
"mcp__github__*"       → all GitHub MCP tools
```

### JSON Input (stdin to your script)
```json
{
  "session_id": "abc123",
  "cwd": "/path/to/project",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": {
    "command": "rm -rf node_modules"
  }
}
```

### Hook Output (stdout from script)

**Option 1: JSON response**
```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "rm -rf blocked"
  }
}
```

**Option 2: Just exit codes**
```
exit 0  → allow (default)
exit 2  → deny (with stderr message)
```

### Permission Decisions
```
"allow"  → tool runs, no user prompt
"deny"   → blocked, Claude told why
"ask"    → user gets permission prompt
```

### Inject Context (special output)
```json
{
  "hookSpecificOutput": {
    "additionalContext": "Inject this into Claude's context"
  }
}
```

### Skeleton Hook Script
```bash
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // ""')

# Your logic here
if echo "$COMMAND" | grep -qi "dangerous_pattern"; then
  jq -n --arg reason "Blocked: dangerous command" '{
    hookSpecificOutput: {
      hookEventName: "PreToolUse",
      permissionDecision: "deny",
      permissionDecisionReason: $reason
    }
  }'
  exit 0
fi

exit 0
```

### Common Use Cases
```
Block dangerous cmds      → PreToolUse + Bash
Auto-format on save       → PostToolUse + Edit/Write
Run linter after edit     → PostToolUse + Edit
Audit log everything      → PreToolUse + *
Inject project context    → SessionStart
Validate commits          → PreToolUse + Bash(git commit *)
Auto-test after changes   → Stop
Slack notify on end       → SessionEnd + http
```

### Best Practices
- **Keep hooks FAST** (< 100ms ideal, blocks Claude)
- **Exit 0 by default** if no deny needed
- **Handle errors gracefully** (`cmd || true`)
- **Use specific matchers** (`Bash(git push *)` > broad `Bash`)
- **Test manually first**: `echo '{...}' | ./hook.sh`
- **Multiple hooks per event** allowed - any deny = blocked
- **Project hooks in git** (commit `.claude/hooks/`)

### Quick Setup
```bash
# 1. Create hooks dir
mkdir -p ~/.claude/hooks

# 2. Create script
cat > ~/.claude/hooks/block-rm.sh << 'EOF'
#!/bin/bash
INPUT=$(cat)
CMD=$(echo "$INPUT" | jq -r '.tool_input.command // ""')
if echo "$CMD" | grep -q "rm -rf"; then
  jq -n '{
    hookSpecificOutput: {
      hookEventName: "PreToolUse",
      permissionDecision: "deny",
      permissionDecisionReason: "rm -rf blocked"
    }
  }'
fi
exit 0
EOF
chmod +x ~/.claude/hooks/block-rm.sh

# 3. Register in ~/.claude/settings.json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": "~/.claude/hooks/block-rm.sh" }
        ]
      }
    ]
  }
}
```

### Debugging
```
claude --verbose                              ← see hook activity
echo '{...}' | bash ~/.claude/hooks/x.sh      ← test manually
echo "fired" >> /tmp/log                      ← add logging in script
/transcript                                    ← view hook decisions
```

---

## 4. Subagents

### What they are
Specialized AI assistants with **own context window, tools, and model**. Like sending an intern to research while you stay focused. They return only a summary; their working notes get discarded.

### Why use them
- **Context isolation** → main conversation stays clean
- **Tool restriction** → safer (e.g., read-only Explore can't modify code)
- **Model selection** → cheap haiku for research, opus for hard tasks
- **Specialization** → focused expertise per agent

### Built-in Subagents
```
Explore           → READ-ONLY (Read, Grep, Glob)
                    Best for: search, find, exploration
                    Fast and cheap

Plan              → All read tools + ExitPlanMode
                    Best for: architecture/design
                    Forces "think before code"

general-purpose   → ALL tools (read + write + bash)
                    Best for: mixed read/write tasks
                    Most flexible, most expensive
```

### Locations
```
~/.claude/agents/<name>/AGENT.md     ← user (all projects, personal)
./.claude/agents/<name>/AGENT.md     ← project (commit to git, team)
```

### AGENT.md Anatomy
```markdown
---
name: code-reviewer                  # required, kebab-case
description: Reviews code for...     # required (Claude uses this to pick agent)
model: opus                          # optional (override model)
tools:                               # optional (restrict tools)
  - Read
  - Grep
  - Bash(git *)
---

You are a senior code reviewer.

Your job:
1. Check security issues
2. Check code quality
3. Suggest improvements

Output format:
## Critical Issues
- [file:line] description + fix

Be concise. Reference specific lines.
```

### Frontmatter Fields
| Field | Purpose |
|-------|---------|
| `name` | Agent identifier (kebab-case) |
| `description` | Tells Claude when to delegate to this agent |
| `model` | opus / sonnet / haiku |
| `tools` | Restrict allowed tools (security + cost) |

### How to Invoke

**Method 1: Direct request** — Claude decides to delegate
```
"Review this PR for security issues"
→ Claude picks security-auditor agent based on description
```

**Method 2: Inside a Skill** — explicit specification
```yaml
---
name: deep-review
context: fork
agent: security-auditor    ← name of your custom agent
---
```

**Method 3: Tool call** — Agent tool with subagent_type parameter

### Decision Matrix: When to Use Each Built-in

```
TASK                                     │ AGENT
─────────────────────────────────────────┼────────────────
"Find all places JWT is used"            │ Explore
"Where is login function?"               │ Explore
"How does auth flow work?"               │ Explore
─────────────────────────────────────────┼────────────────
"Design rate limiting feature"           │ Plan
"How should we refactor this module?"    │ Plan
─────────────────────────────────────────┼────────────────
"Refactor imports across files"          │ general-purpose
"Add tests to all controllers"           │ general-purpose
"Migrate from lodash to ramda"           │ general-purpose
```

### Subagent vs Skill vs Hook
```
SKILL     → reusable workflow YOU invoke (/skill-name)
            template with $ARGUMENTS, !`commands`
            "I do this task often"

SUBAGENT  → specialist with own context + tools + model
            Claude/you delegate to it
            "I need a specialist for this task"

HOOK      → auto-runs on lifecycle events
            decides allow/deny/modify
            "I want enforcement/automation"

Combined: Skill (invokes) → Subagent (does work) ← Hook (validates)
```

### Context Flow
```
Main Conversation                Subagent (separate world)
┌────────────────────┐           ┌──────────────────────┐
│ Your question      │           │ Own system prompt    │
│ ↓                  │           │ Own 200K context     │
│ Spawns subagent    │──────────►│ Reads 30 files (80K) │
│ ↓                  │           │ Analyzes...          │
│ Gets summary (1K)  │◄──────────│ Generates summary    │
│ ↓                  │           │ (rest discarded)     │
│ Continues work     │           └──────────────────────┘
└────────────────────┘
Net: 1K tokens consumed in main, 80K work done elsewhere
```

### Background vs Foreground
```
Foreground (default)  → main waits, gets result, continues
Background            → main continues, notified when done
                        Use for long tasks, independent work
```

### Best Practices
- **ONE agent = ONE expertise** (not "general genius")
- **Specific description** → "for security audits" not "for code"
- **Restrict tools** → code-reviewer doesn't need Write
- **Match model to complexity** → haiku (research), sonnet (general), opus (hard)
- **Short AGENT.md** → < 100 lines (saves their context)
- **Role-based names** → security-auditor, doc-writer (not reviewer-v2)
- **Project agents in git** → commit `.claude/agents/`

### Common Custom Agents to Build
```
security-auditor   → OWASP-focused security review
test-writer        → comprehensive test generation
doc-writer         → technical documentation
perf-analyzer      → performance analysis
db-optimizer       → query optimization
migration-planner  → database migrations
api-designer      → API design review
refactor-advisor  → refactoring suggestions
```

### Quick Setup
```bash
# 1. Create agent directory
mkdir -p ~/.claude/agents/code-reviewer

# 2. Create AGENT.md
cat > ~/.claude/agents/code-reviewer/AGENT.md << 'EOF'
---
name: code-reviewer
description: Reviews code for quality, security, best practices
model: opus
tools:
  - Read
  - Grep
  - Glob
  - Bash(git *)
---

You are a senior code reviewer.

Focus on:
1. Security issues (OWASP Top 10)
2. Code quality
3. Performance concerns
4. Maintainability

Output: structured markdown with file:line references.
Be concise. Don't waste tokens.
EOF

# 3. Use it
# In Claude: "Review the PR using the code-reviewer agent"
```

---

## 5. MCP (Model Context Protocol)

### What it is
Open standard protocol for connecting **external tools and services** to AI agents. MCP servers expose tools/data to Claude over a standard interface. Like USB-C for AI tools — one standard, plug anything in.

### Why it matters
Without MCP, Claude only has built-in tools (Read, Bash, etc.). With MCP, Claude can:
- Send Slack messages
- Query databases directly
- Manage GitHub PRs/issues
- Read Gmail, Calendar, Drive
- Hit any custom API
- Talk to any service that speaks MCP

### Architecture
```
┌──────────────┐         MCP Protocol         ┌──────────────┐
│ Claude Code  │ ◄─────── (JSON-RPC) ───────► │  MCP Server  │
│  (client)    │                              │ (provider)   │
└──────────────┘                              └──────────────┘
                                                     │
                                                     ▼
                                              External Service
                                              (Slack/DB/API/etc)
```

### MCP Server Types
```
LOCAL (runs on your machine)
  - Started by Claude Code as subprocess
  - Communicates via stdio (stdin/stdout)
  - Best for: tools needing local file access, dev tools

REMOTE (HTTP-based)
  - Hosted somewhere (cloud, internal server)
  - Communicates via HTTP
  - Best for: SaaS integrations, team-shared services
```

### Configuration in settings.json
```json
{
  "mcpServers": {
    "github": {
      "command": "docker",
      "args": ["run", "--rm", "-i", "mcp/github"],
      "env": {
        "GITHUB_TOKEN": "ghp_xxx"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://localhost/mydb"]
    },
    "remote-api": {
      "url": "https://my-mcp-server.com/mcp",
      "transport": "http"
    }
  }
}
```

### Tool Naming Convention
MCP tools appear as: `mcp__<server-name>__<tool-name>`

```
mcp__github__create_issue
mcp__github__list_prs
mcp__postgres__query
mcp__slack__send_message
mcp__filesystem__read_file
```

### Popular MCP Servers
```
github            → GitHub PRs, issues, repos
gitlab            → GitLab equivalents
postgres          → PostgreSQL queries
sqlite            → SQLite queries
slack             → Send/receive messages
gmail             → Read/send emails
google-calendar   → Calendar events
google-drive      → Files, docs
filesystem        → Beyond default file tools
puppeteer         → Browser automation
brave-search      → Web search
context7          → Live documentation lookup
sequential-thinking → Structured reasoning
ifttt             → Trigger IFTTT applets
```

### Authentication
Most MCP servers need credentials:
```json
{
  "mcpServers": {
    "github": {
      "command": "...",
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"   ← reads from env var
      }
    }
  }
}
```

For OAuth-based servers (Gmail, Slack, etc.):
- Server provides authentication flow
- Tools like `mcp__gmail__authenticate` start OAuth
- `mcp__gmail__complete_authentication` finishes it
- Tokens stored securely

### Settings Locations
```
~/.claude/settings.json           ← personal MCP servers (all projects)
./.claude/settings.json           ← project MCP servers (commit to git)
./.claude/settings.local.json     ← personal project (don't commit)
```

### How Claude Uses MCP Tools
```
1. You: "Send a Slack message to #engineering saying deploy is done"
2. Claude sees mcp__slack__send_message tool available
3. Claude calls it with proper arguments
4. MCP server forwards request to Slack API
5. Slack returns success
6. Claude confirms to you
```

### Permissions
MCP tools follow the same permission system as built-in tools:
```
"permissions": {
  "allow": [
    "mcp__github__list_prs",          ← always allow
    "mcp__github__create_issue"
  ],
  "ask": ["mcp__github__delete_*"],   ← always confirm
  "deny": ["mcp__postgres__drop_*"]   ← never allow
}
```

### Testing MCP Connection
```
/mcp                          → list active MCP servers + tool count
/mcp <server-name>            → details about specific server
```

### Tool Search (For Many MCP Tools)
With many MCP servers, hundreds of tools = context bloat. Solution:
```json
{
  "mcpServers": { ... },
  "toolSearch": "auto"          ← lazy-load tool schemas on demand
}
```
Claude searches for tools by name when needed instead of loading all schemas upfront.

### Building Your Own MCP Server

**Minimal Python example:**
```python
from mcp.server import Server
from mcp.types import Tool

server = Server("my-server")

@server.list_tools()
async def list_tools():
    return [
        Tool(
            name="get_weather",
            description="Get weather for a city",
            inputSchema={
                "type": "object",
                "properties": {
                    "city": {"type": "string"}
                }
            }
        )
    ]

@server.call_tool()
async def call_tool(name, arguments):
    if name == "get_weather":
        # Your logic here
        return [{"type": "text", "text": f"Sunny in {arguments['city']}"}]
```

Register it:
```json
{
  "mcpServers": {
    "my-weather": {
      "command": "python",
      "args": ["/path/to/server.py"]
    }
  }
}
```

### Use Cases
```
USE CASE                          │ MCP SERVER
──────────────────────────────────┼──────────────────────
"Create a GitHub issue from this" │ github
"Query the production DB"          │ postgres
"Send Slack alert"                 │ slack
"Schedule a meeting"               │ google-calendar
"Search current docs"              │ context7
"Send email to team"               │ gmail
"Trigger a Zapier/IFTTT applet"    │ ifttt
"Browse a webpage"                 │ puppeteer
"Custom internal API"              │ Build your own
```

### MCP vs Skills vs Subagents vs Hooks
```
SKILL      → reusable workflow YOU invoke (template-based)
SUBAGENT   → specialist AI in isolated context
HOOK       → auto-runs on lifecycle events
MCP        → CONNECTS Claude to external services

Combined power:
  Skill (invokes) → uses MCP tools (e.g., github) → Subagent processes results
  Hook validates inputs → MCP tool executes → results back to Claude
```

### Security Considerations
```
⚠️  MCP servers can read/modify external systems
⚠️  Treat credentials carefully (env vars, not hardcoded)
⚠️  Use --bare in CI to avoid loading personal MCP servers
⚠️  Restrict destructive operations via permissions:
    "deny": ["mcp__*__delete_*", "mcp__*__drop_*"]
⚠️  Audit which servers can access what
⚠️  Local MCP servers run with YOUR permissions
```

### Best Practices
- **Start with official servers** → github, postgres, slack (well-tested)
- **Use env vars for secrets** → never hardcode tokens in settings.json
- **Lock down destructive ops** → deny list for delete/drop
- **Project-specific MCP** → put in `.claude/settings.json`, commit safely
- **Personal credentials** → put in `.claude/settings.local.json`, gitignore
- **Test connections** → use `/mcp` to verify before relying on them
- **Document your MCP servers** → add to project CLAUDE.md
- **Enable toolSearch** → if you have many MCP servers (saves context)

### Quick Setup (GitHub example)
```bash
# 1. Get a GitHub token
# Visit: github.com/settings/tokens → generate token

# 2. Add to ~/.claude/settings.json
{
  "mcpServers": {
    "github": {
      "command": "docker",
      "args": ["run", "--rm", "-i", "mcp/github"],
      "env": {
        "GITHUB_TOKEN": "ghp_your_token_here"
      }
    }
  }
}

# 3. Test
# In Claude Code:
/mcp
# Should show: github (X tools)

# 4. Use it
# "List my open PRs"
# Claude calls mcp__github__list_prs
```

### Common Pitfalls
```
❌ Hardcoding tokens in committed settings.json
❌ Not using `${ENV_VAR}` for secrets
❌ Loading too many MCP servers (slow startup, context bloat)
❌ Using global settings for project-specific servers
❌ Forgetting to allow MCP tools in CI permissions
❌ Not testing /mcp connection before depending on it
```

---

## 6. Headless Mode (Scripting)

### What it is
Running Claude **non-interactively** from scripts/CI/git hooks/cron. One question, one answer, exit. Claude becomes a programmable component you compose like grep or curl.

### Basic Syntax
```bash
claude -p "your prompt"
```
The `-p` flag = "print mode" = headless = non-interactive.

### Common Flags
```
-p, --print              → non-interactive mode
--bare                   → skip CLAUDE.md, hooks, skills, MCP
--allowedTools           → pre-approve tools (no prompts)
--output-format          → text / json / stream-json
--continue               → continue most recent session
--resume <id>            → resume specific session
--model                  → opus / sonnet / haiku
--max-turns              → limit agentic turns (safety)
--verbose                → debug output
--cwd                    → run in specific directory
```

### Output Formats
```
text (default)   → plain text, just Claude's answer
json             → parseable, includes cost/tokens/session_id
stream-json      → line-by-line events (real-time progress)
```

**JSON example:**
```json
{
  "type": "result",
  "result": "...",
  "session_id": "abc123",
  "total_cost_usd": 0.001,
  "duration_ms": 1234,
  "num_turns": 1,
  "is_error": false
}
```

### `--bare` Flag (CRITICAL for CI)
```
Skips: CLAUDE.md, hooks, skills, MCP servers
Use when: CI/CD, predictable behavior needed, no project context
Don't use when: task NEEDS your project context/skills/hooks
```

### `--allowedTools` Examples
```bash
--allowedTools "Bash"               # any Bash command
--allowedTools "Bash,Read,Edit"     # multiple tools
--allowedTools "Bash(git *)"        # only git commands
--allowedTools "*"                  # ALL tools (dangerous, only sandboxed)
```

### Piping Input
```bash
git diff | claude -p "Review this diff"
cat error.log | claude -p "Diagnose"
curl https://api/data | claude -p "Extract emails"
git log --oneline | head -20 | claude -p "Summarize"
```

### Session Continuation
```bash
# Continue last session
claude -p "What did we discuss?" --continue

# Resume specific session
claude -p "Add error handling" --resume abc123

# Chain prompts using session_id from JSON
SID=$(claude -p "Analyze" --output-format json | jq -r .session_id)
claude -p "Now improve it" --resume "$SID"
```

### Common Use Cases

**Pre-commit hook (auto-review):**
```bash
#!/bin/bash
DIFF=$(git diff --cached)
RESULT=$(echo "$DIFF" | claude -p "Review. Return ONLY 'PASS' or issues." --bare)
[[ "$RESULT" == "PASS" ]] && exit 0 || { echo "$RESULT"; exit 1; }
```

**Smart commit message:**
```bash
DIFF=$(git diff --cached)
MSG=$(echo "$DIFF" | claude -p "Generate commit message. Format: <type>: <subject>. Return ONLY the message." --bare)
git commit -m "$MSG"
```

**CI test triage:**
```bash
LOGS=$(gh run view $RUN_ID --log-failed)
DIAGNOSIS=$(echo "$LOGS" | claude -p "Diagnose root cause" --bare --output-format json | jq -r .result)
gh pr comment $PR_NUM --body "$DIAGNOSIS"
```

**Daily standup generator (cron):**
```bash
COMMITS=$(git log --since="yesterday" --oneline)
STANDUP=$(claude -p "Generate standup from: $COMMITS" --bare)
curl -X POST $SLACK_WEBHOOK -d "{\"text\":\"$STANDUP\"}"
```

### Cost Tracking
```bash
RESULT=$(claude -p "task" --output-format json)
COST=$(echo "$RESULT" | jq .total_cost_usd)
echo "$(date): \$$COST" >> ~/.claude/costs.log
```

### Limiting Turns (Safety)
```bash
claude -p "Fix tests" --allowedTools "Bash,Read,Edit" --max-turns 5
# If not done in 5 turns, stops. Prevents infinite loops.
```

### Patterns

**Confirm-then-execute:**
```bash
PLAN=$(claude -p "Plan how to do X" --bare)
echo "$PLAN"
read -p "Execute? " yn
[[ "$yn" == "y" ]] && claude -p "Execute" --resume <session>
```

**Parallel tasks:**
```bash
claude -p "Review backend" --bare > backend.txt &
claude -p "Review frontend" --bare > frontend.txt &
claude -p "Review DB" --bare > db.txt &
wait
```

### Best Practices
- **Always use `--bare` in CI** → predictable, no side effects
- **Restrict tools explicitly** → `--allowedTools "Bash(git *),Read"`
- **Set `--max-turns`** in automation → prevents runaway costs
- **Use JSON output** for parsing → `jq` is your friend
- **Log costs per run** → track `total_cost_usd`
- **Check `is_error`** in JSON → handle failures
- **Cache results** → don't re-run expensive analyses

### Common Pitfalls
```
❌ Forgetting --allowedTools  → script hangs on permission prompts
❌ No --bare in CI            → local hooks run, unpredictable
❌ No --max-turns             → agent loops forever, runs up costs
❌ "claude" without -p        → stuck in interactive mode
❌ Pipes + --continue         → conflicting stdin sources
```

### Quick Reference Card
```bash
# One-shot query
claude -p "question"

# With piped data
git diff | claude -p "review"

# Unattended (no prompts)
claude -p "task" --allowedTools "Bash,Read,Edit"

# CI-safe
claude --bare -p "task" --allowedTools "Read" --max-turns 5

# Parseable output
claude -p "task" --output-format json | jq

# Chain conversations
SID=$(claude -p "step 1" --output-format json | jq -r .session_id)
claude -p "step 2" --resume "$SID"

# Cheap model for simple tasks
claude -p "summarize" --model haiku
```

---

## 🎯 ALL TOPICS COMPLETE

Final progress:
```
#0 Essentials     ✅ commands & shortcuts ready
#1 CLAUDE.md      ✅ revision notes ready
#2 Skills         ✅ revision notes ready
#3 Hooks          ✅ revision notes ready
#4 Subagents      ✅ revision notes ready
#5 MCP            ✅ revision notes ready
#6 Headless Mode  ✅ revision notes ready
```

---

## Bonus: How They Work Together

- Essentials → daily commands + shortcuts (foundation)
- CLAUDE.md provides context → Skills use that context
- Hooks enforce policies → before/after tool executions
- Subagents handle big tasks → keep main context clean
- MCP connects external services → Claude talks to GitHub/Slack/DB/etc.
- Headless mode → use everything in scripts/CI

---

## Recommended Learning Order

0. **Essentials** (commands & shortcuts — daily basics)
1. **CLAUDE.md** (foundation, easiest, immediate impact)
2. **Skills** (high ROI for daily workflow)
3. **Hooks** (enforcement and automation)
4. **Subagents** (delegate complex tasks)
5. **MCP** (extend Claude with external services)
6. **Headless Mode** (deploy to production/CI)

---

## Progress Tracker

```
Topic              │ Started │ Completed │ Hands-on Done
───────────────────┼─────────┼───────────┼──────────────
Essentials         │   [ ]   │    [ ]    │     [ ]
CLAUDE.md          │   [ ]   │    [ ]    │     [ ]
Skills             │   [ ]   │    [ ]    │     [ ]
Hooks              │   [ ]   │    [ ]    │     [ ]
Subagents          │   [ ]   │    [ ]    │     [ ]
MCP                │   [ ]   │    [ ]    │     [ ]
Headless Mode      │   [ ]   │    [ ]    │     [ ]
```