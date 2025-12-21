# AI Ecosystem Architecture Overview (Condensed)

**Source:** `../architecture_overview.md` | **Version:** 2.0.0

## Five-Layer Architecture

| Layer | Purpose | Key Components |
|-------|---------|----------------|
| 1. Foundation | OS infrastructure | filesystem, processes, IPC |
| 2. Communication | AI-to-AI messaging | sync (AppleScript), polling (files), async (notifications) |
| 3. Coordination | Task orchestration | roles, lifecycle, instant messaging |
| 4. Memory | Persistent knowledge | ai_memories/ (exported → preprocessed → converted → histories) |
| 5. Presentation | User interfaces | Desktop app, CLI, Browser |

## AI Instances

| Instance | Type | Coordination Role | Capabilities |
|----------|------|-------------------|--------------|
| Claude Desktop | desktop_app | Delegator/Orchestrator | filesystem, bash, AppleScript, MCP |
| Claude CLI | terminal | Worker (→Orchestrator) | full terminal, iTerm2 API, Puppeteer |
| Codex CLI | terminal | Worker | full terminal |

## Communication Queues

```
claude-desktop → ~/Documents/AI/ai_root/ai_comms/claude/prompting/incoming/
claude-cli     → ~/.claude/coordination/tasks/to_execute/
chatgpt        → ~/Documents/AI/ai_root/ai_comms/chatgpt/inbox/
```

## Task Lifecycle

```
staged/ → to_execute/ → in_progress/ → completed/ | error/ | cancelled/
```

**Claim prefix:** `claimed_{YYYYMMDD_HHMMSS}_{PID}_`

## Core Scripts (`~/Documents/AI/ai_root/ai_general/scripts/`)

| Script | Purpose | Usage |
|--------|---------|-------|
| `ai_isBusy.sh` | Check readiness | `ai_isBusy.sh {desktop\|cli\|webui} {id}` → 0=idle, 1=busy |
| `send_prompt.sh` | Queue prompts | `send_prompt.sh {target} "{message}"` |
| `send_notification.sh` | Async alerts | `send_notification.sh {target} {id} "{msg}" [--priority] [--sound]` |
| `monitor.sh` | Read AI output | `monitor.sh {type} {id} [--pattern REGEX]` |

## Scheduling System

- **Schedule dir:** `~/.claude/coordination/scheduling/`
- **Daemon:** `scheduled_prompts_daemon.sh start` (60s scan interval)
- **Types:** `+30m` (relative), `@hourly`/`@daily` (periodic)

## CLI Session Tracking

- **ENV:** `ITERM_SESSION_ID`, `CLI_INSTANCE_ID` ($$)
- **Liveness:** `~/.claude/cli_instances/.cli_instance_{PID}`
- **Cleanup trap:** `trap "rm -f ~/.claude/cli_instances/.cli_instance_$$" EXIT`

## Design Principles

1. **File-based coordination** - debuggable (ls, cat), survives crashes, atomic mv
2. **Flat-plus-internal layout** - entry points at scripts/ root, channels in _internal/
3. **Separate notifications from prompts** - different semantics and guarantees
4. **Daemon over cron** - supports relative times, dynamic schedules
