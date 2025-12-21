# Task: Notification Fallback Design

**Priority:** HIGH  
**Estimated Effort:** 2-3 hours  
**Created:** 2025-11-08  
**Status:** Ready for execution

---

## Goal

Design fallback notification system for when messages can't be injected into Claude Desktop (because Claude is busy or user is typing).

---

## Problem

When CLI or automation needs to notify PianoMan but can't inject into Claude:
- Claude is responding (detected by Task 2.1)
- User is actively typing
- Claude Desktop is not active window
- Need urgent attention but can't interrupt

Current workaround: Write files to coordination directories and hope user checks.

Better solution: System notifications with action buttons.

---

## Requirements

### Notification Backends (Priority Order)

1. **terminal-notifier** (Primary)
   - Best macOS integration
   - Supports action buttons
   - Persistent until dismissed
   - Can open URLs/apps on click
   - **Status:** Needs installation (see req_2111)

2. **osascript display notification** (Fallback #1)
   - Built-in, no installation
   - Basic notifications
   - Limited actions
   - Disappears quickly

3. **osascript display dialog** (Fallback #2)
   - Built-in, no installation
   - Blocks until dismissed
   - Can have buttons
   - Intrusive (use for URGENT only)

---

## Notification Priorities

### URGENT (Immediate attention)
- Use: osascript display dialog (blocking)
- Sound: System alert sound
- Example: "CLI process failed, requires intervention"

### HIGH (Soon, but not blocking)
- Use: terminal-notifier with action buttons
- Sound: Notification sound
- Persist: Until dismissed
- Example: "Task completed, results ready for review"

### NORMAL (FYI, check when convenient)
- Use: terminal-notifier, no sound
- Persist: 10 seconds then auto-dismiss
- Example: "Background task started"

### LOW (Optional, informational)
- Use: osascript display notification
- Auto-dismiss: 5 seconds
- Example: "Periodic status update"

---

## Message Format

### Template Structure
```
TITLE: {source} - {priority}
BODY: {message}
ACTIONS: [Open Chat] [View Details] [Dismiss]
```

### Example Messages

**High Priority:**
```
Title: CLI_12345 - Task Complete
Body: Chat chunker testing finished. 5 issues found.
Actions: [Open Results] [Open Chat] [Dismiss]
```

**Urgent:**
```
Title: CLI_12345 - ERROR
Body: Installation failed. User intervention required.
Actions: [View Logs] [Open Terminal]
```

**Normal:**
```
Title: Background Task
Body: History conversion processing: 45/100 files
Actions: [View Progress] [Dismiss]
```

---

## Backend Selection Logic

```python
def notify(message, priority, source, actions=None):
    """
    Choose notification backend based on:
    - Priority level
    - terminal-notifier availability
    - User preferences
    """
    
    if priority == "URGENT":
        # Always use blocking dialog
        return osascript_display_dialog(message, actions)
    
    if terminal_notifier_available():
        if priority == "HIGH":
            return terminal_notifier(message, actions, sound=True, persist=True)
        elif priority == "NORMAL":
            return terminal_notifier(message, actions, sound=False, timeout=10)
        else:  # LOW
            return terminal_notifier(message, sound=False, timeout=5)
    else:
        # Fallback to osascript
        if priority in ["HIGH", "NORMAL"]:
            return osascript_display_notification(message)
        else:
            # Skip LOW priority if terminal-notifier unavailable
            pass
```

---

## Rate Limiting

**Problem:** Too many notifications = noise = ignored

**Solution:** Rate limiting by source and priority

```
Rules:
- URGENT: No limit (always deliver)
- HIGH: Max 1 per 5 minutes per source
- NORMAL: Max 1 per 15 minutes per source  
- LOW: Max 1 per hour per source

If rate exceeded:
- Queue the notification
- Batch multiple messages
- Deliver summary notification
```

---

## Action Button Design

### Standard Actions

**[Open Chat]**
- Opens Claude Desktop to specific chat
- URL: `claude://chat/{chat_id}` or fallback to AppleScript

**[View Details]**
- Opens file in default editor
- Path: Coordination directory with results

**[View Logs]**
- Opens log file or terminal
- Useful for debugging failures

**[Dismiss]**
- Standard close action

---

## Integration Points

### AppleScript Message Sender
- Before sending message, check Claude state
- If busy → Send notification instead
- Include action: "Open Chat" to return to conversation

### CLI Coordination
- CLI completion → Notification with results link
- CLI error → Urgent notification with logs link
- CLI progress → Normal notification (rate limited)

### Cron Pulsing System
- Check if Claude listening
- If not → Notification instead of injection
- Retry logic: notification → wait → check state → inject

---

## Configuration

### User Preferences File
`~/.claude/notification_config.yml`

```yaml
notification:
  enabled: true
  backend: terminal-notifier  # or osascript
  priorities:
    urgent: true
    high: true
    normal: true
    low: false
  rate_limits:
    high: 300      # seconds
    normal: 900    # seconds
    low: 3600      # seconds
  sounds:
    urgent: true
    high: true
    normal: false
    low: false
```

---

## Success Criteria

- [ ] Design document complete
- [ ] Backend selection logic defined
- [ ] Rate limiting strategy defined
- [ ] Message format templates created
- [ ] Integration points identified
- [ ] Configuration schema defined

---

## Deliverables

1. **This design document** (complete)
2. **Message format examples**
3. **Backend selection flowchart**
4. **Configuration schema**

---

## Implementation Tasks (Follow-up)

After design approval:
1. Install terminal-notifier (req_2111)
2. Implement notification wrapper script
3. Integrate with AppleScript sender
4. Integrate with CLI coordination
5. Test with various scenarios

---

## Notes

- This unblocks pulsing system even before state detection is perfect
- Provides graceful degradation path
- Can be implemented independently and incrementally
- Rate limiting CRITICAL to avoid notification fatigue
- Consider analytics: track notification open rates to tune strategy

