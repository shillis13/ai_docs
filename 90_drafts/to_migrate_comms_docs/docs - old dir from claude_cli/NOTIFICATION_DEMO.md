# CLI to Desktop Notification Demo

## Complete Workflow Example

### Step 1: Desktop Creates Task
```bash
cat > ~/.claude/coordination/to_execute/req_1012_notification_test.md << 'EOF'
# Request #1012: Notification Test

**Type:** Test  
**Priority:** NORMAL  
**Posted:** 2025-11-01 15:00:00

---

## Task Description
Test the CLI → Desktop notification system.

## Task Commands
```bash
echo "Testing notification system"
date
echo "CLI Instance: $$"
sleep 2
echo "Task complete!"
```

## Expected Response
- Confirmation that commands executed
- Notification sent to Desktop Claude
EOF
```

### Step 2: CLI Executes Task
User in CLI terminal:
```bash
claude
# Reports: "Found 1 task: Request #1012: Notification Test"

# User says: "yes, handle 1012"
```

CLI automatically:
1. Creates `in_progress/req_1012_notification_test/`
2. Moves task file into folder
3. Executes commands
4. Writes `response_001_cli_PID.md`

### Step 3: CLI Marks Complete
```bash
mv ~/.claude/coordination/in_progress/req_1012_notification_test \
   ~/.claude/coordination/completed/
```

### Step 4: CLI Notifies Desktop (NEW!)
```bash
~/bin/scripts/notify_desktop_task_complete.sh req_1012_notification_test
```

### Step 5: Desktop Claude Receives Message
Desktop Claude sees in active chat window:
```
✅ Task req_1012 completed by Claude CLI (PID: 65890)

Results available at:
`~/.claude/coordination/completed/req_1012_notification_test/`

Check the response file for details.
```

Desktop can then immediately:
- Review the response file
- Provide followup if needed
- Mark task as verified
- Archive or reference in other work

---

## Manual Testing

### Test the Notification Script Directly
```bash
# Create a test message
osascript ~/bin/scripts/notify_desktop_claude.scpt "Test notification from CLI"
```

### Test with Task Name
```bash
# Assuming you have a completed task
~/bin/scripts/notify_desktop_task_complete.sh req_1011_protocol_test
```

### Expected Result
- Claude Desktop App activates
- Message appears in current chat
- Message is sent automatically

---

## For Codex CLI

Same workflow, different paths:
```bash
# Codex completes task
mv ~/.codex/coordination/in_progress/req_1013_codex_test \
   ~/.codex/coordination/completed/

# Notify Desktop
~/bin/scripts/notify_desktop_task_complete.sh req_1013_codex_test
```

Message will say "Codex CLI" instead of "Claude CLI"

---

## Error Notification

If task fails:
```bash
# Move to error
mv ~/.claude/coordination/in_progress/req_1014_failed_task \
   ~/.claude/coordination/error/

# Notify with error indicator
osascript ~/bin/scripts/notify_desktop_claude.scpt "❌ Task req_1014 failed in Claude CLI. Check error/$TASK_NAME/"
```

---

## Integration with Existing Workflows

### In CLI Automation
After task completion, CLI can:
1. Move to completed/
2. Call notification script
3. Exit or continue to next task

### In Desktop Monitoring
Desktop can:
1. Receive notification
2. Read response file
3. Post followup if needed
4. Move to final archive

### For Multi-Step Tasks
1. CLI completes step 1, notifies Desktop
2. Desktop reviews, posts followup
3. CLI detects followup on next "check inbox"
4. CLI completes step 2, notifies Desktop again
5. Repeat until Desktop marks done

---

## Files Created

- `/Users/shawnhillis/bin/scripts/notify_desktop_claude.scpt` - AppleScript sender
- `/Users/shawnhillis/bin/scripts/notify_desktop_task_complete.sh` - Bash wrapper
- Both are executable and ready to use

---

**Next:** Try creating req_1012 task and watch the complete workflow!
