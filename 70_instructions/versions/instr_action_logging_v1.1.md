# Instruction: Action Logging Protocol

**Version:** 1.1.0
**Created:** 2026-02-13
**Last Updated:** 2026-03-02
**Status:** active
**Priority:** high
**Applies To:** all AI instances

## Purpose

All AI instances log actions before executing them. This enables:
- Recovery from interruptions (last entry = where you were)
- Debugging failures (see what was attempted)
- Audit trails for review

## Core Principle

Log BEFORE you act. Completion is implicit: if the next entry exists, the previous action completed. No explicit status markers needed — single thread per file.

## Format

```
{ISO_TIMESTAMP} | started | {action} | {optional metadata}
```

### Examples

```
2026-03-02T10:15:00 | started | Desktop Commander:read_file | path=.../notes.md
2026-03-02T10:15:30 | started | codex:codex | analyzing todo_mgr.py structure
2026-03-02T10:16:45 | started | assessing results | todo_mgr uses curses for REPL
2026-03-02T10:17:30 | started | delegating to CLI | req_0045 parallel file analysis
```

## Nesting

Nested actions (sub-steps) can use indentation. Completion is inferred from the next line.

```
2026-03-02T10:15:00 | started | analyzing todo_mgr changes
2026-03-02T10:15:10 | started |   reading todo_0063/notes.md
2026-03-02T10:15:15 | started |   reading todo_0064/notes.md
2026-03-02T10:15:30 | started | drafting approach
```

If "drafting approach" exists, all reads completed.

## Log Locations

| Instance | Path |
|---|---|
| Desktop Claude | `ai_claude/logs/{YYYY-MM-DD}/activity.log` |
| Claude CLI | `ai_claude_cli/logs/{YYYY-MM-DD}/activity.log` |
| Codex | `ai_codex/logs/{YYYY-MM-DD}/activity.log` |
| Gemini CLI | `ai_comms/gemini_cli/logs/{YYYY-MM-DD}/activity.log` |

### In Task Context

When working on a task, ALSO log to: `{task_dir}/action_log.txt`

Task context = presence of `task.yml` or `*.status` file in the current working directory.

## What to Log

### Always

- Every tool call (Desktop Commander, Codex MCP, etc.)
- Delegation to CLI agents
- File reads and writes
- Process/command execution
- Significant investigational steps
- Assessment conclusions that drive decisions

### Skip

- Internal reasoning (just thinking)
- Conversation with user (already in chat)
- Reading content already in context window

## Interruption Detection

Last entry in the log = where the interruption occurred. No forensic guessing needed.

## Time Loop Recovery

After a time loop or context loss:

1. Check today's log for the last entry
2. That action was in progress when interrupted
3. Work context is recoverable from log history

## Maintenance

- **Retention:** Daily directories, low volume, keep indefinitely
- **Cleanup:** Manual archival as needed

## Related

- `spec_response_footer.latest.yml`
- `instr_claude_context_conservations.latest.yml`
