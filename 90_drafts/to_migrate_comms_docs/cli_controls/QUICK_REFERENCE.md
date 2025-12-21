# CLI Control Quick Reference

## Basic Operations

### Launch CLI_B
```bash
session=$(./launch_cli_b.sh)
# Or with custom name:
session=$(./launch_cli_b.sh "my_task_worker")
```

### Send Command
```bash
./send_to_cli_b.sh "$session" "What is 2+2?"
```

### Monitor Screen
```bash
# Last 50 lines (default)
./monitor_cli_b.sh "$session"

# Specific number of lines
./monitor_cli_b.sh "$session" 100

# All lines
./monitor_cli_b.sh "$session" all
```

### Kill Session
```bash
./kill_cli_b.sh "$session"
```

---

## Using Utilities

### Source utilities first
```bash
source ./utils.sh
```

### Wait for response pattern
```bash
wait_for_response "$session" "pattern" 30  # 30s timeout
```

### Send and wait
```bash
send_and_wait "$session" "What is 2+2?" "(4|four)" 20
```

### Check if session exists
```bash
if session_exists "$session"; then
    echo "Session is alive"
fi
```

### List all CLI_B sessions
```bash
list_sessions
```

### Get last response
```bash
response=$(get_last_response "$session" 30)
echo "$response"
```

---

## Common Patterns

### Sequential Commands
```bash
session=$(./launch_cli_b.sh)
./send_to_cli_b.sh "$session" "First question"
sleep 10
./send_to_cli_b.sh "$session" "Second question"
```

### Wait and Parse
```bash
./send_to_cli_b.sh "$session" "Analyze logs"
wait_for_response "$session" "Analysis complete" 60
results=$(./monitor_cli_b.sh "$session" 50 | grep -A 20 "Results:")
```

### Error Handling
```bash
if ! ./send_to_cli_b.sh "$session" "command"; then
    echo "Failed to send command"
    ./kill_cli_b.sh "$session"
    exit 1
fi
```

### Parallel Work
```bash
# Launch CLI_B (non-blocking)
session=$(./launch_cli_b.sh)
./send_to_cli_b.sh "$session" "Long task..."

# Do other work immediately
process_other_tasks()

# Check back later
if wait_for_response "$session" "done" 300; then
    results=$(get_last_response "$session")
fi
```

---

## Demos

### Run full demonstration
```bash
./demo_cli_control.sh
```

### Run coordination integration example
```bash
./coordination_integration.sh
```

---

## Troubleshooting

### Session not found
- Check session name is correct
- Use `list_sessions` to see active sessions
- Session may have been closed

### Command not submitting
- System Events sends to frontmost window
- Make sure iTerm2 is active
- CLI_B session should be visible (can be in background tab)

### Response not appearing
- Wait longer (Claude can take 10-20s)
- Check with `./monitor_cli_b.sh "$session" all`
- Look for "-- INSERT --" indicator

### Multiple enters needed
- First enter may add newline (multi-line prompt mode)
- Second enter submits
- This is Claude Code's normal behavior

---

## Integration with Coordination v4.0

```bash
# CLI_A launches worker
session=$(./launch_cli_b.sh "req_1234_worker")

# Send coordination task
task=$(cat ~/.claude/coordination/to_execute/req_1234_task.md)
./send_to_cli_b.sh "$session" "$task"

# Monitor progress
wait_for_response "$session" "completed\|done" 300

# Capture results
./monitor_cli_b.sh "$session" 100 > ~/.claude/coordination/responses/req_1234_response.md

# Cleanup
./kill_cli_b.sh "$session"
```

---

## Key Points

1. **Non-blocking** - CLI_A continues immediately after sending commands
2. **Persistent** - CLI_B maintains context across multiple commands
3. **Monitorable** - Full screen contents readable anytime
4. **Reliable** - System Events keyboard submission works with TUI
5. **Clean** - Simple bash scripts, no complex dependencies

---

**Location:** `/tmp/cli_control/`
**Main Scripts:** launch_cli_b.sh, send_to_cli_b.sh, monitor_cli_b.sh, kill_cli_b.sh
**Utilities:** utils.sh (source it for helper functions)
**Demos:** demo_cli_control.sh, coordination_integration.sh
