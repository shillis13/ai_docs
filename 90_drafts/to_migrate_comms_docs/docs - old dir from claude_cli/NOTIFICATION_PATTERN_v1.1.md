---
title: CLI-to-CLI Notification Pattern (Lessons from Production)
version: 1.1.0
status: active
updated: 2025-11-05
previous: v1.0.0
---

# CLI-to-CLI Notification Pattern - Production Learnings

## Problem

Current coordination protocol lacks mechanism for one CLI to notify another CLI that work is complete. Orchestrators must actively poll, keeping them busy and unable to receive notifications.

## Solution: Notification Queue Per CLI Instance

### Directory Structure

```
coordination/
├── notifications/
│   ├── cli_92509/           # Orchestrator's inbox
│   │   ├── task_complete_req_2002.notify
│   │   ├── task_complete_req_2003.notify
│   │   └── synthesis_ready.notify
│   ├── desktop_claude/      # Desktop Claude's inbox
│   │   └── orchestration_complete_req_2100.notify
│   └── README.md
```

---

## CRITICAL LEARNINGS FROM req_2103 (Production Orchestration)

### 1. Auto-Submit Pattern for Worker Spawning

**✅ WORKS - Use This:**
```applescript
write text "cd ~/.claude/coordination && claude.sh --dangerously-skip-permissions 'check inbox and claim req_2002'"
```

**Key:** Pass prompt as argument to claude.sh - it auto-executes.

**❌ DOESN'T WORK - Avoid:**
```applescript
write text "claude.sh --dangerously-skip-permissions"
delay 8
write text "check inbox and claim req_2002"  # ← Never submitted!
```

**Why:** AppleScript `write text` types but doesn't press Enter. Commands sit unsubmitted.

### 2. Worker Tasks Must Include Notification Steps

**Problem Discovered:** Workers completed successfully but orchestrator had no notification files to monitor.

**Root Cause:** Task definitions (req_2002, req_2003) didn't include notification steps.

**Solution:** Add to ALL worker task definitions:

```markdown
## Phase N: Notify Orchestrator

⚠️ **CRITICAL:** If ORCHESTRATOR_PID environment variable is set, create notification.

\`\`\`bash
if [ -n "$ORCHESTRATOR_PID" ]; then
    cat > notifications/cli_${ORCHESTRATOR_PID}/task_complete_req_XXXX.notify <<NOTIFYEOF
{
  "event": "task_complete",
  "task_id": "req_XXXX",
  "worker_pid": $$,
  "timestamp": "$(date -Iseconds)",
  "status": "success",
  "response_location": "completed/req_XXXX_taskname/response_001_cli_$$.md"
}
NOTIFYEOF
    
    echo "✅ Notified orchestrator CLI ${ORCHESTRATOR_PID}"
else
    echo "ℹ️  No orchestrator - running standalone"
fi
\`\`\`
```

### 3. Fallback Verification for Orchestrators

**When notification files are missing**, orchestrators should:

```bash
# Check completed/ directory directly
check_completion() {
    local task_id=$1
    
    if [ -d "completed/${task_id}" ]; then
        # Verify response file exists
        if ls completed/${task_id}/response_*.md &>/dev/null; then
            echo "✅ ${task_id} verified complete (via completed/ check)"
            return 0
        fi
    fi
    
    return 1
}

# Use in monitoring loop
if check_completion "req_2002_create_download_helpers" && \
   check_completion "req_2003_create_templates_configs"; then
    echo "✅ All workers complete (fallback verification)"
    break
fi
```

**This makes orchestrators resilient** to missing notification infrastructure.

### 4. Orchestrator Completion Criteria

**From req_2103 production run:**

Orchestrators correctly waited ~5 minutes for workers to complete, adapted when notification files were missing, verified via completed/ directory, synthesized results, and notified Desktop Claude.

**Updated completion criteria:**

```yaml
orchestration_complete_when:
  primary_path:
    - notification_file_exists: notifications/cli_$$/task_complete_req_*.notify
    - count_matches: expected_worker_count
  
  fallback_path:
    - completed_directory_exists: completed/req_*_taskname/
    - response_file_exists: completed/req_*_taskname/response_*.md
    - verified_deliverables: task_specific_checks
  
  always_required:
    - synthesis_created: true
    - requestor_notified: true
```

---

## Notification File Format

**Filename:** `{event_type}_{details}.notify`

**Content (JSON):**
```json
{
  "event": "task_complete",
  "task_id": "req_2002",
  "from_pid": 74123,
  "timestamp": "2025-11-05T15:20:00-05:00",
  "status": "success",
  "response_location": "completed/req_2002_create_download_helpers/response_001_cli_74123.md",
  "message": "Download helper scripts completed successfully"
}
```

---

## Pattern: Minimal Orchestration (VALIDATED)

### Phase 1: Spawn & Monitor
```
Orchestrator:
1. Create notification inbox: notifications/cli_$$/
2. Spawn workers with ORCHESTRATOR_PID set
3. Enter monitoring loop (check every 2 min)
4. Wait for notifications OR fallback verification
5. Create synthesis when complete
6. Notify requestor
7. EXIT
```

### Phase 2: Workers Execute & Notify
```
Worker 1: Execute req_2002
  └→ On complete: 
     - Create notification if ORCHESTRATOR_PID set
     - OR orchestrator will detect via completed/

Worker 2: Execute req_2003
  └→ On complete: notify orchestrator OR rely on fallback
```

### Phase 3: Requestor Reviews
```
Desktop Claude receives ONE notification
  └→ Reviews synthesis
  └→ Decides next steps
```

---

## Production-Tested Spawn Script

From req_2103 (WORKING):

```bash
#!/bin/bash
set -euo pipefail

ORCH_PID=$1

# Worker 1
osascript <<EOF
tell application "iTerm"
    create window with default profile
    tell current session of current window
        write text "cd ~/.claude/coordination && export ORCHESTRATOR_PID=${ORCH_PID} && claude.sh --dangerously-skip-permissions 'check inbox and claim req_2002'"
    end tell
end tell
EOF

sleep 2

# Worker 2  
osascript <<EOF
tell application "iTerm"
    create window with default profile
    tell current session of current window
        write text "cd ~/.claude/coordination && export ORCHESTRATOR_PID=${ORCH_PID} && claude.sh --dangerously-skip-permissions 'check inbox and claim req_2003'"
    end tell
end tell
EOF

echo "✅ Workers spawned with auto-submit"
```

**Key features:**
- Passes prompt as argument (auto-executes)
- Sets ORCHESTRATOR_PID environment variable
- 2-second delay between spawns
- Simple and reliable

---

## Benefits (Proven in Production)

✅ **Orchestrator not busy-waiting** - periodic checks (2 min intervals)  
✅ **Workers self-coordinate** - notify when done  
✅ **Desktop Claude has final say** - reviews before next phase  
✅ **Leverages existing infrastructure** - uses completed/ directory  
✅ **Resilient** - works even if notification mechanism incomplete  
✅ **Scales to complex hierarchies** - notification queues per CLI  

---

## Implementation Checklist for New Tasks

When creating worker tasks:

- [ ] Add ORCHESTRATOR_PID check
- [ ] Add notification creation step
- [ ] Include fallback message if no orchestrator
- [ ] Document notification format in task
- [ ] Test with and without orchestrator

When creating orchestrator tasks:

- [ ] Use production-tested spawn script pattern
- [ ] Set ORCHESTRATOR_PID in worker environment
- [ ] Implement primary notification monitoring
- [ ] Implement fallback verification (completed/ check)
- [ ] Include explicit completion criteria
- [ ] Set reasonable timeout (15 min for simple tasks)

---

## Metrics from req_2103 Production Run

**Success metrics:**
- Spawn time: <1 second
- Worker execution: 3-5 minutes each
- Total orchestration: ~5 minutes
- Fallback verification: Worked perfectly
- Single notification to requestor: ✅

**Failure points avoided:**
- ❌ Workers didn't hang waiting for Enter
- ❌ Orchestrator didn't mark complete early
- ❌ Orchestrator didn't spam Desktop Claude
- ❌ Missing notifications didn't block completion

---

## Alternative: Immediate Notification via AppleScript

For lower latency (no cron delay), workers could directly AppleScript:

```bash
# Notify Desktop Claude immediately (experimental)
osascript -e 'tell application "Claude" to activate'
# Use send_ai_message.sh for direct injection

# But this requires:
# - Desktop Claude app is running
# - Worker has AppleScript permissions
# - More complex than file-based approach
```

**Recommendation:** File-based with 1-minute cron is simpler and more reliable.

---

## Version History

**v1.1.0 (2025-11-05):**
- Added production learnings from req_2103
- Documented auto-submit pattern
- Added fallback verification
- Updated worker task template
- Validated orchestration pattern

**v1.0.0 (2025-11-05):**
- Initial pattern specification
- Hierarchical notification design
- Basic notification format
