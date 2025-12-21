# Claude CLI Enhancement Ecosystem (Condensed)
**Source:** cli_enhancement_ecosystem.md | 2025-12-05

## Plugin Architecture
```
Plugin (container)
├── Commands   - /slash triggers
├── Agents     - Specialized subagents
├── Skills     - Auto-activating guidance
├── Hooks      - Lifecycle event handlers
└── MCP        - External tool integrations
```

## Component Types

| Type | Trigger | Purpose | Example |
|------|---------|---------|---------|
| Commands | `/cmd` | Explicit actions | /deploy, /test |
| Skills | Auto (context) | Guidance patterns | Frontend design, API testing |
| Agents | Auto/manual | Specialist work | Code reviewer, security analyst |
| Hooks | Lifecycle events | Enforcement | Pre-commit, security scan |
| MCP | Tool calls | External APIs | Database, Jira, project mgmt |

## Agents
**Invocation:** Manual selection or auto by Claude based on task context
**Inline:** `claude --agents '{"reviewer": {"description": "...", "prompt": "..."}}'`
**In Plugin:** `agents/*.md` files

## Skills
**Key Difference:** No explicit trigger - Claude auto-applies based on context
**Structure:**
```
skills/{skill-name}/
├── SKILL.md           # Required
├── scripts/
└── references/
```
**SKILL.md Format:**
```yaml
---
name: Skill Name
description: When to use
version: 1.0.0
---
Skill instructions...
```

## Hooks
**Types:** PreToolUse, PostToolUse, SessionStart, Stop
**Config:** `hooks/hooks.json` + `hooks/scripts/`
**Use Cases:** Security monitoring, code style, auto-docs, pre-commit

## MCP Servers
**Config:** `.mcp.json` in plugin root
**Enables:** Database connections, API integrations, custom tools

## Plugin Structure
```
my-plugin/
├── .claude-plugin/plugin.json  # Manifest: name, desc, version, author
├── commands/      # *.md files
├── agents/        # *.md files
├── skills/        # {name}/SKILL.md
├── hooks/         # hooks.json + scripts/
└── .mcp.json      # MCP integrations
```

## Marketplaces

| Marketplace | Focus | URL |
|------------|-------|-----|
| claude-plugins.dev | Community hub | https://claude-plugins.dev/ |
| Anthropic Official | Core plugins | https://github.com/anthropics/claude-code/plugins |
| claude-code-plugins-plus | 254+ plugins | https://github.com/jeremylongshore/claude-code-plugins-plus |
| wshobson/agents | 85 agents, 63 plugins | https://github.com/wshobson/agents |
| awesome-claude-plugins | Curated list | https://github.com/GiladShoham/awesome-claude-plugins |

**Install:** `/plugin marketplace add <repo>` then `/plugin install <name>@<marketplace>`

## CLI Flags
```bash
--agents <json>           # Inline agents
--plugin-dir <paths>      # Plugin locations
--mcp-config <file>       # MCP config
--setting-sources user,project,local  # Precedence: user < project < local
```

## Orchestration Opportunities
1. Custom coordination plugin - Package CLI coordination
2. Session management skill - Auto-activate session awareness
3. Task execution hooks - Validate tasks before execution
4. Desktop-to-CLI agent - Specialized task handoff
