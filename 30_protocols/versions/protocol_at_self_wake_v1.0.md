# AT Self-Wake Pattern for Autonomous Monitoring

**Version:** 1.0.0
**Created:** 2025-12-10
**Status:** active
**Category:** pattern

## Summary

Desktop Claude sets an AT job to self-prompt after delegating long-running work, enabling periodic monitoring without human intervention.

## Problem

Desktop Claude has no inherent agency between responses. After delegating work to CLI, Claude said "I'll check in after a few minutes" - but with what mechanism? Claude only wakes when prompted. CLI can't reliably prompt Desktop due to conversation locks.

## Solution

AT job as self-wake alarm. Desktop Claude schedules a future prompt to itself, enabling autonomous monitoring loops.

## Key Insight

> "AT is my wake-up alarm, not CLI's doorbell."

CLI completion isn't important enough to force-interrupt. Desktop wakes on schedule, checks progress, and either:
- Sets another AT if work continues
- Wraps up if complete
- Intervenes if stuck

## Implementation

### Set Wake Alarm
```bash
cat << 'EOF' | at now + 3 minutes
/path/to/send_prompt.sh claude-desktop "AT wake-up: Check status" --force --submit
EOF
```

### On Wake Check
- Is tmux session still alive?
- Has progress been made (new/updated files)?
- Is worker stuck waiting for something?

### Decision Tree
| Condition | Action |
|-----------|--------|
| Worker still running + making progress | Set another AT |
| Worker complete | Clean up, report results |
| Worker stuck | Intervene or escalate |

## Prerequisites

- `send_prompt.sh` with `--force` flag (bypasses busy/lock checks)
- AT daemon running (launchd on macOS)
- Desktop Claude app accessible via AppleScript

## Pattern Flow

1. Desktop Claude creates task specification
2. Desktop launches CLI worker in tmux (`-t` flag)
3. Desktop sets AT job for N minutes in future
4. Desktop goes to sleep (stops responding)
5. AT fires → `send_prompt.sh --force` injects wake message
6. Desktop wakes, checks worker status
7. Repeat 3-6 or wrap up

## Example Session

### Delegation
```bash
# Launch CLI worker
claude_cli.sh -t -a -s my_worker -A librarian "Execute task req_XXXX"
```

### Set Alarm
```bash
cat << 'EOF' | at now + 5 minutes
send_prompt.sh claude-desktop "AT wake-up: Check my_worker status" --force --submit
EOF
```

### Wake Check
```bash
# Check tmux session
tmux list-sessions | grep my_worker

# Check output files for progress
ls -la /path/to/output/

# Capture recent activity
tmux capture-pane -t my_worker -p -S -50
```

## Advantages

- No conversation lock bypass needed from worker side
- Predictable wake intervals
- Desktop maintains oversight without blocking
- Worker runs fully autonomous
- Human not required in loop

## Limitations

- AT granularity is 1 minute minimum
- Requires AT daemon running
- `send_prompt.sh --force` must work reliably
- No interrupt for urgent issues (only scheduled wake)

## Related

- `send_prompt.sh` (--force flag addition)
- CLI coordination protocol
- Desktop Claude context preservation rules

## Future Enhancements

- Priority channel for urgent worker alerts
- Adaptive AT intervals based on task complexity
- Multiple concurrent worker monitoring
- Escalation to user if worker fails repeatedly
