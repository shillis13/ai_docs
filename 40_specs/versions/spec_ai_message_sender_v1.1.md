# AI Message Sender Specification

**Version:** 1.1.0
**Created:** 2025-11-01
**Author:** PianoMan
**Status:** production
**Location:** `/Users/shawnhillis/bin/projects/chat_orchestrators/send_ai_message.sh`

## Summary

Universal mechanism for external processes to send messages to AI desktop applications (Claude, ChatGPT) via AppleScript automation. Enables system notifications, scheduled prompts, CLI completion alerts, and cross-AI communication without requiring human intervention.

## Interface

### Command
```bash
send_ai_message.sh <app> <message>
```

### Parameters

| Position | Name | Type | Required | Description |
|----------|------|------|----------|-------------|
| 1 | app | string | yes | Target AI application: `claude` or `chatgpt` |
| 2 | message | string | yes | Message text to send (can be multi-line) |

### Return Codes
- **0:** Success - message sent
- **1:** Error - invalid parameters or AppleScript failure

### Examples

```bash
# Send to Claude
send_ai_message.sh claude "Task completed successfully"

# Send to ChatGPT
send_ai_message.sh chatgpt "Please analyze this result"

# Multi-line message
send_ai_message.sh claude "Line 1
Line 2
Line 3"

# From variable
result=$(some_command)
send_ai_message.sh claude "Analysis complete: $result"

# Scheduled via cron
0 9 * * * send_ai_message.sh claude "Daily standup: What are priorities?"
```

## Behavior

### Activation
- Activates target app and brings to foreground
- 1 second delay after activation
- May interrupt user's current focus

### Message Input
- Method: Keystroke simulation via AppleScript
- Target: Currently active conversation in app
- 0.5 second delay before Enter key

### Conversation Targeting
Messages are sent to whichever conversation is currently open/active in the target app. This allows targeting specific conversations by opening them first.

### Concurrency
Not thread-safe - simultaneous calls may interfere. Serialize calls with lock file or queue.

## Use Cases

- **System Notifications:** Disk space warnings, backup completion, error conditions
- **CLI Completion:** Notify Desktop AI of task completion
- **Scheduled Prompts:** Cron jobs send periodic reminders or triggers
- **Cross-AI Communication:** One AI sends updates to another
- **Async Collaboration:** Leave messages for AI to process when convenient

## Technical Details

### Implementation
- Language: Bash wrapper around AppleScript
- Dependencies: macOS AppleScript, System Events permissions, target app installed

### Timing
- Activation delay: 1.0s
- Typing delay: 0.5s
- Total minimum: ~1.5s per message

### Limitations
- No response reading
- No conversation URL targeting (uses active conversation)
- No retry logic
- Single-threaded (concurrent calls may conflict)
- Requires app focus (interrupts user)

## Related Systems
- CLI Coordination Protocol v3.1
- Chat Orchestrator (Puppeteer-based)
- Desktop Claude Prompt Sender v2.0 (file-based queue)
