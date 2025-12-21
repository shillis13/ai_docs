# Desktop Claude ⇄ CLI Control & Monitoring Architecture

**Version:** 1.0 (Draft)  
**Created:** 2025-11-10  
**Purpose:** Define bidirectional control, monitoring, and coordination between Desktop Claude and CLI instances

---

## Overview

This architecture enables Desktop Claude to **monitor** CLI instance health and activity, **control** CLI execution and priorities, and **receive** real-time notifications and results. CLI instances can similarly **report** status, **request** intervention, and **coordinate** with Desktop Claude.

### Key Design Principles

1. **Multi-channel communication** - Use appropriate layer (Layer 1/2/3) for message urgency
2. **Graceful degradation** - If real-time fails, fall back to async
3. **Observable state** - All CLI activity visible to Desktop Claude
4. **Non-blocking** - Desktop Claude doesn't wait for CLI unless explicitly requested
5. **Autonomous CLIs** - Can work independently when Desktop not available

---

## Architecture Layers

### Layer 1: Real-Time (Synchronous)
- **Instant messaging** - Back-and-forth conversations
- **iTerm2 API** - Direct prompt injection
- **Latency:** <1 second

### Layer 2: Polling (Near Real-Time)
- **Notifications** - Status updates, alerts
- **Direct input injection** - Prompt queue monitoring
- **Latency:** 1-60 seconds

### Layer 3: Async (Filesystem-Based)
- **Task coordination** - Work assignment and results
- **Note passing** - Complex communication
- **Latency:** Minutes to hours (human-mediated)

---

## Desktop Claude → CLI Control Mechanisms

### 1. Task Assignment (Layer 3)

**Purpose:** Delegate work to CLI instances with structured specifications

**Mechanism:**
```
Desktop writes task
    ↓
~/.claude/coordination/to_execute/req_XXXX_taskname.md
    ↓
CLI discovers task (user approval: "check inbox")
    ↓
CLI claims task (moves to in_progress/)
    ↓
CLI executes and writes response
    ↓
CLI completes (moves to completed/)
    ↓
Desktop reads response (user: "check responses")
```

**Control capabilities:**
- Task priority (LOW, NORMAL, HIGH, URGENT)
- Task type (Standard, Research, Installation, Testing)
- Expected deliverables
- Success criteria
- Timeout parameters

**Example task file:**
```markdown
# Request #1015: Analyze System Logs

**Type:** Research  
**Priority:** HIGH  
**Posted:** 2025-11-10 08:00:00

## Task Description
Search system logs for errors in last 24 hours

## Task Commands
```bash
grep -i error /var/log/system.log | tail -n 100
```

## Expected Response
Summary of errors found with counts and severity
```

### 2. Direct Prompt Injection (Layer 2)

**Purpose:** Immediate commands that appear in CLI's active terminal

**Mechanism:**
```
Desktop writes prompt
    ↓
claude_cli/prompting/incoming/check_tasks.prompt.txt
    ↓
Daemon detects file (1-2 sec polling)
    ↓
iTerm2 API injects into CLI session
    ↓
Prompt appears in CLI terminal
    ↓
CLI processes immediately
```

**Control capabilities:**
- Emergency interrupts
- Quick status checks
- Priority escalation
- REPL commands
- Targeted to specific CLI (by PID)

**Example prompts:**
```bash
# General prompt (any CLI)
echo "Check coordination inbox now" > \
  claude_cli/prompting/incoming/check_inbox.prompt.txt

# Targeted prompt (specific CLI)
echo "Cancel current operation and report status" > \
  claude_cli/prompting/incoming/cli_12345_cancel.prompt.txt
```

### 3. Notifications (Layer 2)

**Purpose:** Alert CLI about important events or changes

**Mechanism:**
```
Desktop writes notification
    ↓
claude_cli/notifications/pending/priority_change.txt
    ↓
CLI polls periodically (or daemon pushes)
    ↓
CLI reads and processes
    ↓
Moves to processed/
```

**Control capabilities:**
- Priority changes to existing tasks
- New high-priority work available
- System maintenance warnings
- Coordination protocol updates

### 4. Instant Messaging (Layer 1)

**Purpose:** Real-time discussion requiring back-and-forth

**Mechanism:**
```
Desktop starts conversation
    ↓
orchestrator/instant_messaging/active/conv_001/thread.jsonl
    ↓
CLI monitors conversation
    ↓
CLI adds responses to thread
    ↓
Desktop continues discussion
```

**Control capabilities:**
- Interactive problem solving
- Design discussions
- Clarifying requirements
- Collaborative debugging

---

## CLI → Desktop Claude Reporting Mechanisms

### 1. Task Responses (Layer 3)

**Purpose:** Deliver completed work results

**Mechanism:**
```
CLI completes task
    ↓
Writes response.md in completed/req_XXXX/
    ↓
Desktop checks responses directory
    ↓
Reads and processes results
```

**Reporting capabilities:**
- Execution status (COMPLETED, FAILED, PARTIAL)
- Command outputs
- Discovered issues
- Follow-up recommendations
- Artifacts created

**Example response:**
```markdown
# Response: Request #1015

**Status:** COMPLETED  
**Completed:** 2025-11-10 08:15:00  
**CLI Instance:** 12345

## Execution Summary
Found 47 errors in system logs from last 24 hours

## Results
- 23 network timeout errors (non-critical)
- 15 disk I/O warnings (monitor)
- 9 authentication failures (investigate)

## Follow-up Needed
Authentication failures may indicate security issue
```

### 2. Notifications to Desktop (Layer 2)

**Purpose:** Alert Desktop Claude about important events

**Mechanism:**
```
CLI detects event
    ↓
Writes notification to claude/notifications/pending/
    ↓
Pulse daemon aggregates notifications
    ↓
Desktop receives prompt injection
    ↓
Desktop reviews notification
```

**Reporting capabilities:**
- Task completion alerts
- Error conditions
- Resource warnings
- Request for intervention
- Discovery of important information

**Example notifications:**
```bash
# Task completion
echo "req_1015 completed - 9 authentication failures found (INVESTIGATE)" > \
  ~/Documents/AI/ai_root/ai_comms/claude/notifications/pending/cli_12345_task_1015_complete.txt

# Error condition
echo "CLI_12345: Disk space critically low (<5GB) - pausing operations" > \
  ~/Documents/AI/ai_root/ai_comms/claude/notifications/pending/cli_12345_disk_warning.txt

# Request intervention
echo "Task req_1016 requires user decision - cannot proceed" > \
  ~/Documents/AI/ai_root/ai_comms/claude/notifications/pending/cli_12345_intervention_needed.txt
```

### 3. Direct Prompt to Desktop (Layer 2)

**Purpose:** Immediate alert that appears in Desktop Claude's chat

**Mechanism:**
```
CLI needs urgent attention
    ↓
Writes prompt to claude/prompting/incoming/
    ↓
Cron job (1 min polling) or daemon detects
    ↓
AppleScript injects into Claude Desktop
    ↓
Desktop Claude sees prompt in active chat
```

**Reporting capabilities:**
- Emergency alerts
- Critical failures
- User intervention required
- Time-sensitive discoveries

**Example:**
```bash
# Emergency alert
echo "CRITICAL: All CLI tasks failing - database unreachable" > \
  ~/Documents/AI/ai_root/ai_comms/claude/prompting/incoming/emergency_database_down.prompt.txt
```

### 4. Instant Messaging (Layer 1)

**Purpose:** Request real-time discussion with Desktop

**Mechanism:**
```
CLI starts conversation
    ↓
Writes to orchestrator/instant_messaging/active/
    ↓
Desktop monitors for new conversations
    ↓
Desktop responds in thread
```

**Reporting capabilities:**
- Ask clarifying questions
- Discuss unexpected findings
- Collaborative problem solving
- Architecture decisions

---

## Monitoring Components

### 1. CLI Registry

**Purpose:** Track all active CLI instances and their status

**Location:** `claude_cli/cli_registry.json`

**Schema:**
```json
{
  "cli_instances": [
    {
      "pid": 12345,
      "session_id": "w0t0p0:1A2B3C4D",
      "tab_name": "CLI Worker 1",
      "started": "2025-11-10T08:00:00Z",
      "last_seen": "2025-11-10T08:25:00Z",
      "status": "active",
      "current_task": "req_1015",
      "task_started": "2025-11-10T08:10:00Z",
      "capabilities": ["python", "node", "docker"],
      "load": "0.45"
    }
  ],
  "updated": "2025-11-10T08:25:00Z"
}
```

**Desktop Claude can query:**
- How many CLIs are active?
- Which CLI is handling which task?
- When did CLI last check in?
- Is CLI idle or busy?
- What are CLI's capabilities?

**Update mechanisms:**
- CLI heartbeat (every 60 seconds)
- Task claim/complete events
- Registry manager daemon
- Health check script

### 2. Task Status Dashboard

**Purpose:** Quick overview of all task states

**Implementation:** Script that Desktop Claude can run

**Location:** `~/bin/ai/cli_dashboard.sh`

**Output:**
```
CLI COORDINATION DASHBOARD
Generated: 2025-11-10 08:30:00

ACTIVE CLI INSTANCES: 3
├─ CLI 12345 (Worker 1) - BUSY - Task: req_1015 (5m ago)
├─ CLI 12346 (Worker 2) - IDLE - (15m idle)
└─ CLI 12347 (Worker 3) - BUSY - Task: req_1017 (2m ago)

TASK QUEUE STATUS:
├─ To Execute: 2 tasks
│  ├─ req_1018 (HIGH) - System analysis
│  └─ req_1019 (NORMAL) - Log rotation
├─ In Progress: 2 tasks
│  ├─ req_1015 (HIGH) - CLI 12345 (5m)
│  └─ req_1017 (NORMAL) - CLI 12347 (2m)
├─ Completed (last hour): 4 tasks
└─ Errors: 0

NOTIFICATIONS:
├─ Pending: 3
│  ├─ cli_12345_task_1015_complete.txt
│  ├─ cli_12346_idle_available.txt
│  └─ system_health_check.txt
└─ Processed (today): 12

HEALTH STATUS: ✅ ALL SYSTEMS NOMINAL
```

### 3. Log Monitoring

**Purpose:** Track detailed CLI activities and debugging

**Locations:**
- `claude_cli/logs/cli_12345_session.log` - Per-CLI session logs
- `claude_cli/logs/coordination_events.log` - System-wide events
- `/tmp/cli-prompt-daemon.log` - Prompt injection daemon
- `/tmp/pulse-daemon.log` - Notification aggregation

**Desktop Claude can:**
- Tail logs in real-time
- Search for errors
- Track task execution details
- Debug coordination issues

**Example log query:**
```bash
# Recent errors
grep ERROR claude_cli/logs/*.log | tail -n 20

# Specific CLI activity
tail -f claude_cli/logs/cli_12345_session.log

# Task timeline
grep "req_1015" claude_cli/logs/coordination_events.log
```

### 4. Notification Queue

**Purpose:** See all pending alerts

**Location:** `claude/notifications/pending/`

**Desktop Claude workflow:**
```bash
# Check pending notifications
ls -lht ~/Documents/AI/ai_root/ai_comms/claude/notifications/pending/

# Read specific notification
cat claude/notifications/pending/cli_12345_task_1015_complete.txt

# Process notification (acknowledge)
mv claude/notifications/pending/cli_12345_task_1015_complete.txt \
   claude/notifications/processed/
```

---

## Control Operations

### 1. Assign New Task

**Desktop Claude:**
```bash
# Create task specification
cat > ~/.claude/coordination/to_execute/req_1020_system_report.md << 'EOF'
# Request #1020: System Report
**Type:** Standard
**Priority:** HIGH
[Task details...]
EOF

# Notify CLI via prompt injection
echo "New HIGH priority task req_1020 available" > \
  claude_cli/prompting/incoming/new_task_alert.prompt.txt
```

**CLI receives:**
1. Prompt appears in terminal
2. CLI runs: "check inbox"
3. Discovers req_1020
4. User approves
5. CLI claims and executes

### 2. Check CLI Status

**Desktop Claude:**
```bash
# View registry
cat claude_cli/cli_registry.json

# Check what's in progress
ls -lh ~/.claude/coordination/in_progress/

# Run dashboard
~/bin/ai/cli_dashboard.sh
```

### 3. Cancel Task

**Desktop Claude:**
```bash
# Cancel specific task
mv ~/.claude/coordination/in_progress/req_1015_analysis/ \
   ~/.claude/coordination/cancelled/

# Notify CLI immediately
echo "Task req_1015 cancelled - stop work immediately" > \
  claude_cli/prompting/incoming/cli_12345_cancel_1015.prompt.txt
```

### 4. Priority Override

**Desktop Claude:**
```bash
# Urgent notification
echo "URGENT: Elevate req_1018 to HIGH priority and execute immediately" > \
  claude_cli/notifications/pending/priority_override_1018.txt

# Follow with prompt for faster delivery
echo "Check notifications - priority change for req_1018" > \
  claude_cli/prompting/incoming/priority_alert.prompt.txt
```

### 5. Request Status Update

**Desktop Claude:**
```bash
# Ask for progress report
echo "Report progress on req_1015 - estimated completion time?" > \
  claude_cli/prompting/incoming/cli_12345_status_req_1015.prompt.txt
```

**CLI responds:**
```bash
# CLI writes progress note
cat > ~/Documents/AI/ai_root/ai_comms/claude/note_passing/incoming/note_cli_12345_progress.md << 'EOF'
From: CLI_12345
Re: req_1015 Progress

Currently 60% complete. Processing 3rd of 5 log files.
Estimated completion: 10 minutes
EOF

# And sends notification
echo "Progress update for req_1015 in note_passing/incoming/" > \
  ~/Documents/AI/ai_root/ai_comms/claude/notifications/pending/cli_12345_progress_update.txt
```

---

## Monitoring Operations

### 1. Check for Completed Work

**Desktop Claude workflow:**
```bash
# List recent completions
ls -lht ~/.claude/coordination/completed/ | head -20

# Read specific response
cat ~/.claude/coordination/completed/req_1015_analysis/response.md

# Check for new response notifications
ls claude/notifications/pending/ | grep complete
```

### 2. Health Check All CLIs

**Desktop Claude:**
```bash
# Run health check script
~/bin/ai/cli_health_check.sh

# Expected output:
# CLI 12345: ✅ Active (last seen 30s ago)
# CLI 12346: ✅ Active (last seen 45s ago)
# CLI 12347: ⚠️  Stale (last seen 5m ago)
# 
# RECOMMENDATION: Check CLI 12347 - may have crashed
```

### 3. Monitor Real-Time Activity

**Desktop Claude:**
```bash
# Watch coordination directory
watch -n 2 'ls -lh ~/.claude/coordination/*/*/task.md 2>/dev/null | tail -20'

# Tail CLI logs
tail -f claude_cli/logs/coordination_events.log

# Monitor prompt queue
watch -n 1 'ls -lh claude_cli/prompting/incoming/'
```

### 4. Review Notifications

**Desktop Claude:**
```bash
# Pending notifications
for f in claude/notifications/pending/*; do
    echo "=== $f ==="
    cat "$f"
    echo ""
done

# Process notifications
for f in claude/notifications/pending/*; do
    cat "$f"
    read -p "Mark as processed? (y/n) " answer
    if [ "$answer" = "y" ]; then
        mv "$f" claude/notifications/processed/
    fi
done
```

---

## Coordination Patterns

### Pattern 1: Desktop Assigns → CLI Executes → Desktop Reviews

```
[Desktop Claude]
1. Create task in to_execute/
2. Optional: Send notification or prompt
3. Wait for completion (async)

[CLI Instance]
4. Discover task (user: "check inbox")
5. Claim task (moves to in_progress/)
6. Execute task
7. Write response
8. Complete task (moves to completed/)
9. Send notification to Desktop

[Desktop Claude]
10. Receive notification via prompt
11. Read response from completed/
12. Review results
13. Optional: Post follow-up task
```

### Pattern 2: CLI Reports Emergency → Desktop Responds

```
[CLI Instance]
1. Detect critical issue
2. Write emergency prompt to claude/prompting/incoming/
3. Also write detailed note to note_passing/incoming/

[Desktop Claude]
2. Receive prompt in active chat (within 60 seconds)
3. Read emergency details from note
4. Assess situation
5. Issue response via prompt to CLI
6. May cancel other tasks or adjust priorities
```

### Pattern 3: Multi-CLI Parallel Execution

```
[Desktop Claude]
1. Break large task into sub-tasks
2. Post req_1020_part1, req_1020_part2, req_1020_part3
3. Send broadcast prompt "3 new tasks available"

[CLI Instances]
4. CLI_A claims part1
5. CLI_B claims part2
6. CLI_C claims part3
7. All execute in parallel

[Desktop Claude]
8. Monitor via registry (3 CLIs busy)
9. Receive completion notifications as each finishes
10. Read all responses
11. Synthesize final result
```

### Pattern 4: Real-Time Collaboration

```
[Desktop Claude]
1. Encounter complex problem
2. Start instant message conversation
3. Write initial problem statement

[CLI Instance]
4. Monitor instant_messaging/active/
5. Read problem
6. Add response with initial thoughts

[Desktop & CLI]
7. Back-and-forth discussion (3-5 exchanges)
8. Reach consensus on approach

[CLI Instance]
9. Execute agreed approach
10. Report results in same thread

[Desktop Claude]
11. Review results
12. Archive conversation
13. Document decision
```

---

## Dashboard Implementation

### CLI Control Dashboard (for Desktop Claude)

**Location:** `~/bin/ai/claude_cli_dashboard.sh`

**Features:**
- Live CLI registry status
- Task queue overview
- Recent completions
- Pending notifications
- Health indicators
- Quick actions menu

**Usage:**
```bash
# Run dashboard
~/bin/ai/claude_cli_dashboard.sh

# Auto-refresh mode
~/bin/ai/claude_cli_dashboard.sh --watch

# Generate report file
~/bin/ai/claude_cli_dashboard.sh --output dashboard.txt
```

### CLI Web Dashboard (Optional Future)

**Technology:** Simple HTML + JavaScript
**Updates:** Via file watching or 5-second polling
**Features:**
- Visual task pipeline
- CLI status indicators
- Real-time log streaming
- Notification inbox
- Action buttons (cancel, prioritize, etc.)

---

## Error Handling & Recovery

### CLI Becomes Unresponsive

**Detection:**
- Registry last_seen > 5 minutes old
- Task in_progress for >expected duration
- No heartbeat updates

**Recovery:**
```bash
# Desktop Claude workflow:
1. Check CLI health: cli_health_check.sh
2. Attempt prompt injection: "Report status immediately"
3. Wait 60 seconds
4. If still unresponsive:
   - Mark CLI as "stale" in registry
   - Move in_progress task back to to_execute
   - Notify user
   - User manually checks CLI terminal
```

### Task Execution Failure

**Detection:**
- CLI writes to error/ directory
- Notification with ERROR prefix
- Registry shows task_status: "failed"

**Recovery:**
```bash
# CLI writes error details
mv in_progress/req_1015_* error/
echo "req_1015 FAILED - Database connection timeout" > \
  ~/Documents/AI/ai_root/ai_comms/claude/notifications/pending/cli_12345_req_1015_error.txt

# Desktop Claude reviews
cat error/req_1015_*/response.md
# Decides: retry, modify, or abandon task
```

### Prompt Injection Failure

**Detection:**
- Prompt file remains in incoming/ >5 minutes
- Daemon logs show iTerm2 API errors

**Recovery:**
```bash
# Fallback to notification
mv claude_cli/prompting/incoming/failed_prompt.prompt.txt \
   claude_cli/notifications/pending/manual_check_needed.txt

# Desktop Claude: Switch to task-based coordination
# (More reliable but slower)
```

---

## Future Enhancements

### 1. CLI Capability Negotiation
- CLIs advertise capabilities (python, docker, databases, etc.)
- Desktop assigns tasks based on capability matching
- Dynamic workload balancing

### 2. Task Dependencies
- Task B depends on Task A completion
- Automatic chaining
- Parallel execution where possible

### 3. Resource Management
- CPU/memory limits per CLI
- Throttling based on system load
- Priority queue management

### 4. Audit Trail
- Complete history of all coordination events
- Task assignment → execution → completion timeline
- Decision tracking (why certain approach chosen)

---

## Quick Reference

### Desktop Claude → CLI Commands

```bash
# Assign task
echo "..." > ~/.claude/coordination/to_execute/req_XXXX.md

# Send prompt
echo "..." > claude_cli/prompting/incoming/action.prompt.txt

# Send notification
echo "..." > claude_cli/notifications/pending/event.txt

# Check CLI status
cat claude_cli/cli_registry.json

# View pending work
ls ~/.claude/coordination/to_execute/

# Check completions
ls ~/.claude/coordination/completed/
```

### CLI → Desktop Claude Reporting

```bash
# Complete task
mv in_progress/req_XXXX completed/
echo "..." > completed/req_XXXX/response.md

# Send notification
echo "..." > ~/Documents/AI/ai_root/ai_comms/claude/notifications/pending/report.txt

# Send prompt (urgent)
echo "..." > ~/Documents/AI/ai_root/ai_comms/claude/prompting/incoming/alert.prompt.txt

# Update registry
# (Done automatically by heartbeat)

# Start conversation
echo '{"from":"cli_12345","text":"...","ts":"..."}' >> \
  orchestrator/instant_messaging/active/conv_001/thread.jsonl
```

---

**Version History:**
- **v1.0** (2025-11-10) - Initial architecture design

**Status:** Draft  
**Next:** Validate with actual CLI implementation and testing  
**Related:** ARCHITECTURE_v1.1.md, ITERM2_API_GUIDE.md, coordination_system_v4_digest.md
