# CLI Agent Operations Guide

**Version:** 1.0.0
**Created:** 2025-12-08
**Maintainer:** PianoMan
**Status:** active

## Overview

Quick reference for running CLI agents. Desktop Claude coordinates strategy; CLI agents execute specialized work autonomously.

**Wrapper Script:** `~/bin/ai/cli/claude_cli.sh`
**Agent Definitions:** `~/.claude/agents.json`

## Architecture

```
Desktop Claude (Architect/Systems)
       │
       ├── Strategic planning, research, design
       ├── Cross-cutting decisions
       ├── Documentation standards
       └── Delegates execution to CLI agents
                │
    ┌───────────┼───────────┬───────────┐
    │           │           │           │
dev-lead   librarian   custodian     ops
```

## Agent Roles

| Agent | Purpose | Responsibilities |
|-------|---------|------------------|
| dev-lead | Development coordination | Task curation, code review, quality enforcement |
| librarian | Knowledge pipeline | Chat exports, summaries, cross-chat threads |
| custodian | Filesystem hygiene | Symlinks, archives, naming conventions |
| ops | Task execution | Monitor coordination dirs, execute tasks |

## Usage

### Launch Agent

**Interactive:**
```bash
claude_cli.sh -A <agent> -a
```

**With Prompt:**
```bash
claude_cli.sh -A <agent> "<prompt>"
```

**Background (tmux):**
```bash
claude_cli.sh -A <agent> -t "<prompt>"
```

### Session Management

| Action | Command |
|--------|---------|
| Continue last | `claude_cli.sh -c "<prompt>"` |
| Resume by ID | `claude_cli.sh -r <session_id> "<prompt>"` |
| Named session | `claude_cli.sh -n <name> "<prompt>"` |
| List sessions | `claude_cli.sh list <project_dir> [limit]` |

### Common Workflows

**Librarian Export Processing:**
1. Export chat to `ai_memories/10_exported/`
2. `claude_cli.sh -A librarian -a`
3. Librarian processes through pipeline

**Dev-Lead Task Curation:**
1. Add rough todo to `ai_general/todos/`
2. `claude_cli.sh -A dev-lead 'Review and refine todos'`
3. Dev-lead creates Task files in `staged/`

**Ops Task Execution:**
1. Task placed in `to_execute/`
2. `claude_cli.sh -A ops -t 'Check inbox and execute'`
3. Monitor: `tail -f <output_log>`

## Flags Reference

### Execution Flags
| Flag | Description |
|------|-------------|
| `-t, --tmux` | Run in background tmux session |
| `-s, --session` | Custom tmux session name |
| `-a, --auto-approve` | Skip permission prompts |
| `-A, --agent` | Load agent persona |
| `-w, --workdir` | Override working directory |
| `--no-mcp` | Run without MCP servers |

### Session Flags
| Flag | Description |
|------|-------------|
| `-c, --continue` | Continue most recent session |
| `-r, --resume` | Resume by session ID |
| `-n, --named` | Resume by registered name |

## Codex CLI

Separate tool for code-focused autonomous work.

**Wrapper:** `~/bin/ai/cli/codex_cli.sh`

**When to Use:**
- Code debugging and fixing
- Autonomous multi-step coding tasks
- Tasks requiring sandbox isolation

**Usage:**
```bash
codex_cli.sh -i                    # Interactive
codex_cli.sh "<prompt>"            # With prompt
codex_cli.sh -t -s <name> -a "<prompt>"  # Background
```

## Related Documents
- `ai_general/docs/40_specs/cli_specialized_agent_roles_v2.md`
- `ai_general/docs/40_specs/cli_coordination_protocol_v4.md`
