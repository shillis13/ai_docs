# Claude CLI Enhancement Ecosystem

**Created:** 2025-12-05  
**Status:** Discovery document  

---

## Overview

Claude CLI (Claude Code) has a comprehensive extension system with five primary enhancement vectors, all bundleable into **Plugins**:

```
Plugin (container)
├── Commands      - Explicit /slash triggers
├── Agents        - Specialized subagents  
├── Skills        - Auto-activating context-aware guidance
├── Hooks         - Event handlers at lifecycle points
└── MCP Servers   - External tool integrations
```

---

## 1. Agents

**What:** Specialized subagents with focused expertise and custom prompts.

**Invocation:** 
- Manual selection by user
- Automatic selection by Claude based on task context

**Inline Definition (--agents flag):**
```bash
claude --agents '{"reviewer": {"description": "Reviews code", "prompt": "You are a code reviewer"}}'
```

**In Plugin Structure:**
```
agents/
├── code-reviewer.md
├── security-analyst.md
└── test-writer.md
```

**Use Cases:**
- Code review specialists
- Security analyzers
- Documentation generators
- Architecture advisors

---

## 2. Skills

**What:** Auto-activating instruction sets that Claude uses based on task context.

**Key Difference from Commands:** No explicit trigger needed - Claude recognizes when to apply.

**Structure:**
```
skills/
├── api-testing/
│   ├── SKILL.md           # Required: skill instructions
│   ├── scripts/
│   └── references/
└── database-migrations/
    ├── SKILL.md
    └── examples/
```

**SKILL.md Format:**
```markdown
---
name: Skill Name
description: When to use this skill
version: 1.0.0
---

Skill instructions and guidance...
```

**Examples:**
- Frontend design patterns
- API debugging workflows
- Database migration guidance
- Security scanning procedures

---

## 3. Commands

**What:** Explicit slash commands for specific actions.

**Invocation:** `/command-name [args]`

**Structure:**
```
commands/
├── hello.md
├── deploy.md
└── test.md
```

**Use Cases:**
- `/deploy` - Trigger deployment workflow
- `/test` - Run test suite
- `/review` - Start code review
- `/gc` - Smart git commit

---

## 4. Hooks

**What:** Custom behaviors triggered at specific lifecycle points.

**Hook Types:**
- `PreToolUse` - Before tool execution
- `PostToolUse` - After tool execution  
- `SessionStart` - When session begins
- `Stop` - When exit attempted

**Structure:**
```
hooks/
├── hooks.json
└── scripts/
    ├── pre-commit-check.sh
    └── security-scan.py
```

**Use Cases:**
- Security pattern monitoring
- Code style enforcement
- Automatic documentation updates
- Pre-commit validations

---

## 5. MCP Servers (in Plugins)

**What:** External integrations packaged within plugins.

**Configuration:** `.mcp.json` in plugin root

**Enables:**
- Database connections
- API integrations
- Project management tools (Jira, Asana)
- Custom tool servers

---

## Plugin Structure

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json         # Manifest
├── commands/               # Slash commands
├── agents/                 # Specialized subagents
├── skills/                 # Auto-activating skills
├── hooks/                  # Event handlers
│   ├── hooks.json
│   └── scripts/
├── .mcp.json               # MCP integrations
└── scripts/                # Shared utilities
```

**plugin.json:**
```json
{
  "name": "my-plugin",
  "description": "Plugin description",
  "version": "1.0.0",
  "author": { "name": "Your Name" }
}
```

---

## Marketplaces

Plugins distributed via marketplaces - Git repos with catalog files.

**Known Marketplaces:**
| Marketplace | Focus | URL |
|------------|-------|-----|
| claude-plugins.dev | Community hub | https://claude-plugins.dev/ |
| Anthropic Official | Core plugins | https://github.com/anthropics/claude-code/plugins |
| claude-code-plugins-plus | 254+ plugins | https://github.com/jeremylongshore/claude-code-plugins-plus |
| wshobson/agents | 85 agents, 63 plugins | https://github.com/wshobson/agents |
| awesome-claude-plugins | Curated list | https://github.com/GiladShoham/awesome-claude-plugins |

**Installation:**
```bash
# Add marketplace
/plugin marketplace add jeremylongshore/claude-code-plugins-plus

# Install plugin
/plugin install devops-automation-pack@claude-code-plugins-plus
```

---

## Setting Sources

```bash
claude --setting-sources user,project,local
```

**Hierarchy:**
- `user` - User-level settings (~/.claude/)
- `project` - Project-level (.claude/ in project)
- `local` - Local overrides

---

## CLI Flags for Enhancements

```bash
# Custom agents inline
--agents <json>

# Setting source control
--setting-sources <sources>

# Load plugins from directory
--plugin-dir <paths...>

# MCP configuration
--mcp-config <file or string>
```

---

## Component Comparison

| Component | Trigger | Scope | Use Case |
|-----------|---------|-------|----------|
| **Commands** | Explicit `/cmd` | Single action | Deploy, test, review |
| **Skills** | Auto (context) | Guidance | Patterns, best practices |
| **Agents** | Auto/manual | Subagent | Specialized expertise |
| **Hooks** | Lifecycle events | Enforcement | Validation, security |
| **MCP** | Tool calls | Integration | External systems |

---

## For Our Orchestration System

**Opportunities:**
1. **Custom coordination plugin** - Package our CLI coordination as a plugin
2. **Session management skill** - Auto-activate session awareness
3. **Task execution hooks** - Validate tasks before execution
4. **Desktop-to-CLI agent** - Specialized agent for task handoff

**Plugin Concept:**
```
ai-orchestration-plugin/
├── .claude-plugin/plugin.json
├── commands/
│   └── task.md              # /task command for coordination
├── agents/
│   └── task-executor.md     # Specialized task execution agent
├── skills/
│   └── session-awareness/
│       └── SKILL.md         # Auto-activate session handling
└── hooks/
    ├── hooks.json
    └── scripts/
        └── task-validator.sh
```

---

## Next Steps

1. Explore existing plugins for patterns
2. Design orchestration plugin structure
3. Consider marketplace for team distribution
4. Integrate with session persistence wrapper
