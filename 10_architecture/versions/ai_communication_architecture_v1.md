# AI Communication Architecture v1.0

**Topic:** Three-layer AI-to-AI communication system design  
**Status:** Architecture complete, implementation in progress  
**Created:** 2025-11-09  
**Maintainer:** PianoMan

---

## Overview

A comprehensive three-layer communication architecture enabling multiple AI instances (Desktop Claude, CLI Claude, ChatGPT, Codex) to coordinate and communicate at different urgency levels, analogous to human communication patterns.

**Design Philosophy:**
- **User time is limiting resource** - Minimize interruptions
- **Variable urgency levels** - Match channel to priority
- **Autonomous operation** - AIs coordinate without human intervention when appropriate
- **Observable by design** - Filesystem-based, standard Unix tools work
- **Crash resilient** - State persists through restarts

---

## Three Communication Layers

### Layer 1: Synchronous (Phone-Like)
**Urgency:** IMMEDIATE  
**Response Time:** Real-time  
**Use Case:** Time-critical coordination requiring immediate interaction

**Implementation Options:**
1. **iTerm Python API** - Direct terminal control
   - AIs can directly control iTerm sessions
   - Send commands, read output in real-time
   - Full bidirectional communication
   
2. **Chat Orchestrator** - Puppeteer-based relay
   - Browser automation shuttles messages between AIs
   - Maintains full conversational context in native interfaces
   - Human oversight modes (manual approval vs auto-relay)

**When to Use:**
- Emergency situations requiring immediate attention
- Time-sensitive decisions with tight windows
- Real-time collaboration on complex problems
- User explicitly requesting immediate coordination

### Layer 2: Polling (Text-Like)
**Urgency:** NEAR-REAL-TIME  
**Response Time:** Seconds to minutes  
**Use Case:** Important updates that need attention but not emergency intervention

**Implementation: Pulsing Daemon**
- Long-running daemon process with configurable sleep intervals
- Reads `~/.claude/heartbeat_duration.txt` to control pulse frequency
- Sends prompt to Desktop Claude when waking up
- Watchdog cron job ensures daemon stays alive

**Key Features:**
- **Variable frequency** - Claude controls timing via duration file
- **Infinite granularity** - Any positive integer seconds supported
- **Idle mode** - Invalid/zero duration stops pulsing (daemon sleeps 60s quietly)
- **Self-healing** - Watchdog restarts daemon if crashed
- **Graceful degradation** - File errors default to idle mode

**Files:**
- `/Users/shawnhillis/bin/ai/pulsing_daemon.sh` - Main daemon
- `~/.claude/heartbeat_duration.txt` - Duration control (seconds)
- `~/.claude/daemon.log` - Operation log
- `~/.claude/daemon.pid` - Process ID for watchdog
- `~/Documents/AI/Claude/claude_workspace/06_announcements/to_desktop_claude/` - Prompt delivery

**Claude Control Examples:**
```bash
# Start active monitoring every 2 minutes
echo "120" > ~/.claude/heartbeat_duration.txt

# Go idle (stop pulsing, daemon stays alive)
echo "0" > ~/.claude/heartbeat_duration.txt
# OR
rm ~/.claude/heartbeat_duration.txt

# Resume at 5-minute intervals
echo "300" > ~/.claude/heartbeat_duration.txt

# High-frequency monitoring (every 30 seconds)
echo "30" > ~/.claude/heartbeat_duration.txt
```

**Watchdog Implementation:**
```bash
# Cron job (every 5 minutes)
*/5 * * * * ~/bin/ai/watchdog_pulse.sh

# watchdog_pulse.sh
#!/bin/bash
PID_FILE=~/.claude/daemon.pid
DAEMON_SCRIPT=~/bin/ai/pulsing_daemon.sh

if [ -f "$PID_FILE" ]; then
    pid=$(cat "$PID_FILE")
    if ps -p "$pid" > /dev/null 2>&1; then
        # Daemon running, all good
        exit 0
    fi
fi

# Daemon not running, start it
nohup "$DAEMON_SCRIPT" > /dev/null 2>&1 &
```

**When to Use:**
- Task completion notifications
- Status check reminders  
- Periodic health monitoring
- Situations where immediate interruption unnecessary but timely awareness important

### Layer 3: Async (Email-Like)
**Urgency:** BACKGROUND  
**Response Time:** Hours to days  
**Use Case:** Work delegation, complex tasks, low-priority coordination

**Implementation: CLI Coordination System v4.0**
- Directory-based task management
- Desktop posts tasks, CLI instances claim and execute
- Full async operation with status lifecycle tracking

**Directory Structure:**
```
~/.claude/coordination/ → ~/Documents/AI/ai_comms/claude_cli/
├── to_execute/          # Pending tasks (flat list)
│   └── req_XXXX_*.md
├── in_progress/         # Active work (folder per task)  
│   └── req_XXXX_*/
│       ├── task.md
│       └── progress.md
├── completed/           # Successful tasks
│   └── req_XXXX_*/
│       ├── task.md
│       ├── response.md
│       └── followup_*.md
├── error/               # Failed tasks
└── cancelled/           # Cancelled tasks
```

**Task Lifecycle:**
1. Desktop writes: `to_execute/req_XXXX_taskname.md`
2. CLI discovers task (user prompt: "check inbox")
3. User approves, CLI creates: `in_progress/req_XXXX_taskname/`
4. CLI executes, writes: `in_progress/req_XXXX_taskname/progress.md`
5. CLI completes, moves folder to: `completed/req_XXXX_taskname/`
6. CLI notifies Desktop (Layer 2 pulse or AppleScript)
7. Desktop reads: `completed/req_XXXX_taskname/response.md`

**When to Use:**
- Complex multi-step tasks
- File system operations
- Data analysis and processing
- Work that can proceed independently without immediate feedback

---

## Integration & Orchestration

### Cross-Layer Coordination Patterns

**Pattern 1: Escalation**
- Start with Layer 3 (async task)
- CLI escalates to Layer 2 (pulse notification) when complete
- User/Desktop escalates to Layer 1 if urgent clarification needed

**Pattern 2: Task Completion Chain**
- CLI completes task (Layer 3)
- Writes completion notification
- Triggers pulse (Layer 2) to wake Desktop
- Desktop reviews and posts next task

**Pattern 3: Real-Time Collaboration**
- User initiates Layer 1 (chat orchestrator)
- Multiple AIs discuss in real-time
- Delegate implementation work to Layer 3
- Results flow back through Layer 2 notifications

### Notification Methods

**Layer 3 → Layer 2 Transition:**
```bash
# CLI writes notification after task completion
echo "Task req_1234 complete: System info gathered" > \
  ~/.claude/notifications/task_1234_complete.txt

# Set rapid pulse for immediate attention
echo "30" > ~/.claude/heartbeat_duration.txt
```

**Layer 2 → Layer 1 Transition:**
```bash
# AppleScript direct message injection
osascript -e 'tell application "Claude"
    tell window 1
        keystroke "URGENT: Critical error in task req_1234"
        keystroke return
    end tell
end tell'
```

**Layer 3 Direct to Desktop (Bypass Layer 2):**
```bash
# Write prompt file for immediate injection (if Desktop prompt sender running)
echo "Task req_1234 completed with errors requiring review" > \
  ~/Documents/AI/Claude/claude_workspace/06_announcements/to_desktop_claude/task_1234_error.prompt.txt
```

---

## Directory Structure & File Conventions

### Communication Directories

```
~/Documents/AI/
├── ai_comms/                           # Central messaging hub
│   ├── claude_cli/                     # Layer 3: CLI coordination
│   │   ├── to_execute/
│   │   ├── in_progress/
│   │   ├── completed/
│   │   ├── error/
│   │   └── cancelled/
│   ├── claude/                         # Per-AI mailboxes
│   │   ├── inbox/
│   │   ├── outbox/
│   │   └── drafts/
│   └── chatgpt/
│       ├── inbox/
│       ├── outbox/
│       └── drafts/
│
├── Claude/claude_workspace/
│   └── 06_announcements/
│       ├── to_desktop_claude/          # Layer 2: Pulse prompts
│       │   ├── *.prompt.txt            # Pending prompts
│       │   └── sent/                   # Delivered prompts archive
│       └── from_cli/                   # Notifications from CLI
│
└── ai_general/
    ├── scripts/                        # Shared automation
    └── docs/                           # Documentation (this file)
```

### Control Files

**Pulse Control:**
- `~/.claude/heartbeat_duration.txt` - Pulse interval (seconds)
- `~/.claude/daemon.log` - Daemon operations log
- `~/.claude/daemon.pid` - Daemon process ID

**State Files:**
- `.status` files - Empty marker files indicating state
- `.flag` files - Workflow control flags
- `.lock` files - Prevent concurrent operations

---

## Implementation Status

### ✅ Completed

**Layer 3 (Async):**
- CLI Coordination Protocol v4.0 fully operational
- Task lifecycle management working
- Response file structure established

**Layer 2 (Polling):**
- Pulsing daemon implemented (`pulsing_daemon.sh`)
- Duration file control working
- Logging system functional
- Prompt delivery to Desktop Claude working

**Supporting Infrastructure:**
- AppleScript notification system
- Desktop prompt injection capability
- Directory structure established

### 🚧 In Progress

**Layer 2 Watchdog:**
- Simple cron-based watchdog for daemon monitoring
- Needs: `~/bin/ai/watchdog_pulse.sh` implementation

**Layer 1 (Synchronous):**
- Chat orchestrator exists but needs integration
- iTerm Python API exploration needed

### 📋 Planned

**Enhanced Layer 2:**
- Notification aggregation (prevent prompt flood)
- Priority queue for prompts
- Smart wake-up logic (check workload before pulsing)

**Layer 1 Development:**
- iTerm Python API wrapper for direct terminal control
- Enhanced chat orchestrator integration
- Multi-party real-time coordination

**Cross-Layer Features:**
- Automatic escalation rules (when to move between layers)
- Conversation threading across layers
- Unified logging and observability

---

## Operational Procedures

### Starting Pulse System

```bash
# 1. Set desired pulse interval
echo "180" > ~/.claude/heartbeat_duration.txt

# 2. Start daemon manually (or let watchdog start it)
~/bin/ai/pulsing_daemon.sh &

# 3. Verify daemon running
ps aux | grep pulsing_daemon

# 4. Monitor log
tail -f ~/.claude/daemon.log

# 5. Install watchdog cron (one-time)
crontab -e
# Add: */5 * * * * ~/bin/ai/watchdog_pulse.sh
```

### Stopping Pulse System

```bash
# Option 1: Go idle (daemon stays alive, stops pulsing)
echo "0" > ~/.claude/heartbeat_duration.txt

# Option 2: Kill daemon (watchdog will restart unless removed)
kill $(cat ~/.claude/daemon.pid)

# Option 3: Permanent stop (remove watchdog, kill daemon)
crontab -e  # Remove watchdog line
kill $(cat ~/.claude/daemon.pid)
rm ~/.claude/daemon.pid
```

### Monitoring & Debugging

```bash
# Check daemon status
ps aux | grep pulsing_daemon

# View recent logs
tail -20 ~/.claude/daemon.log

# Check current pulse interval
cat ~/.claude/heartbeat_duration.txt

# List pending prompts
ls -la ~/Documents/AI/Claude/claude_workspace/06_announcements/to_desktop_claude/*.prompt.txt

# Check CLI task queue
ls -la ~/.claude/coordination/to_execute/

# View recent completions
ls -lat ~/.claude/coordination/completed/ | head -10
```

### Adjusting Pulse Frequency

**Scenarios and Recommended Intervals:**

```bash
# Active development - fast feedback loop
echo "60" > ~/.claude/heartbeat_duration.txt    # Every minute

# Normal monitoring - periodic check-ins
echo "300" > ~/.claude/heartbeat_duration.txt   # Every 5 minutes

# Low-priority background - infrequent updates
echo "900" > ~/.claude/heartbeat_duration.txt   # Every 15 minutes

# End of day - catch overnight tasks
echo "3600" > ~/.claude/heartbeat_duration.txt  # Every hour

# Go fully idle - stop pulsing
echo "0" > ~/.claude/heartbeat_duration.txt
```

---

## Design Principles & Rationale

### 1. Claude Controls Timing, Not States

**Anti-Pattern (state-based):**
```bash
# Bad: Predefined states limit flexibility
echo "active" > ~/.claude/pulse_state.txt    # What does "active" mean?
echo "idle" > ~/.claude/pulse_state.txt      # How often should it check?
```

**Correct Pattern (duration-based):**
```bash
# Good: Infinite granularity, Claude decides meaning
echo "120" > ~/.claude/heartbeat_duration.txt  # Exactly 2 minutes
echo "37" > ~/.claude/heartbeat_duration.txt   # Exactly 37 seconds
```

**Rationale:**
- States require predefined frequency mappings (brittle)
- Claude can adapt timing to workload dynamically
- Any positive integer seconds is valid
- More expressive: "check every 2 minutes" vs "active state"

### 2. Filesystem as Message Bus

**Why Not:**
- Database - Adds complexity, single point of failure
- Network sockets - Requires both ends online simultaneously
- Message queue - Overkill for local coordination

**Why Filesystem:**
- Natural persistence (survives crashes)
- Observable with standard tools (ls, cat, tail)
- Atomic operations (file presence/absence)
- No daemon required for async communication
- Works across process boundaries
- Debuggable by humans (readable text files)

### 3. Layered Urgency, Not Single Channel

**Human Analogy:**
- Layer 1 = Phone call (synchronous, immediate)
- Layer 2 = Text message (near-real-time, soon)
- Layer 3 = Email (async, when convenient)

**System Benefits:**
- Match communication channel to urgency
- Avoid interrupt fatigue (not everything is urgent)
- Enable autonomous operation (AIs decide when to escalate)
- User maintains control (can adjust pulse frequency)

### 4. Daemon + Watchdog Pattern

**Why Daemon:**
- Long-running process eliminates startup overhead
- Maintains state between pulses
- Can adapt behavior based on history

**Why Watchdog:**
- Simple cron job is more reliable than complex daemon management
- Auto-restart if daemon crashes
- No systemd/launchd complexity
- Works on any Unix system

**Why Not Single Cron Job:**
- Frequent cron (every minute) causes high overhead
- Process startup/teardown every time
- Can't maintain state between runs
- Reading same files repeatedly inefficient

### 5. User Time is Limiting Resource

**Implications:**
- Default to async (Layer 3) unless urgency clear
- Pulse notifications show context (what to check)
- Task responses include "follow-up needed?" explicit flag
- Desktop Claude can silence pulses when deep in work
- AIs coordinate autonomously when possible

---

## Future Enhancements

### Near-Term (1-2 months)

**Smart Pulse Logic:**
```bash
# Daemon checks workload before pulsing
if [ $(ls ~/.claude/coordination/completed/ | wc -l) -gt 0 ]; then
    # Work to review, pulse
    send_pulse
elif [ $(ls ~/.claude/coordination/in_progress/ | wc -l) -gt 0 ]; then
    # Work still running, don't pulse yet
    skip_pulse
else
    # Nothing happening, idle pulse for heartbeat
    send_idle_pulse
fi
```

**Notification Aggregation:**
```bash
# Collect multiple completions into single prompt
# Instead of: 
#   - heartbeat_123.prompt.txt (task 1 done)
#   - heartbeat_124.prompt.txt (task 2 done)
#   - heartbeat_125.prompt.txt (task 3 done)
# Create:
#   - summary_123.prompt.txt (3 tasks completed: req_001, req_002, req_003)
```

**Priority Queue:**
```
to_desktop_claude/
├── urgent/           # Processed first
├── normal/           # Standard queue
└── low_priority/     # Batch processed
```

### Medium-Term (3-6 months)

**Layer 1 Integration:**
- iTerm Python API wrapper for terminal control
- Chat orchestrator MCP server
- Direct AI-to-AI terminal sessions

**Conversation Threading:**
- Link related messages across layers
- Maintain context when escalating between layers
- Thread IDs in prompt filenames

**Enhanced Observability:**
- Dashboard showing all three layers
- Message flow visualization
- Performance metrics (response times by layer)

### Long-Term (6+ months)

**Multi-Agent Coordination:**
- N-way conversations across multiple AIs
- Consensus protocols for decision-making
- Load balancing across CLI instances

**Learning System:**
- Optimize pulse frequency based on usage patterns
- Predict when Desktop Claude available/busy
- Auto-adjust communication strategies

**Cross-Platform:**
- Windows support (WSL-based)
- Linux daemon alternatives
- Cloud-hosted coordination hubs

---

## Troubleshooting

### Daemon Not Pulsing

**Check 1: Is daemon running?**
```bash
ps aux | grep pulsing_daemon
# Should show process, if not:
~/bin/ai/pulsing_daemon.sh &
```

**Check 2: Valid duration file?**
```bash
cat ~/.claude/heartbeat_duration.txt
# Should be positive integer, if not:
echo "180" > ~/.claude/heartbeat_duration.txt
```

**Check 3: Check logs**
```bash
tail -20 ~/.claude/daemon.log
# Look for errors or "not sending pulse" messages
```

**Check 4: Prompt directory writable?**
```bash
ls -la ~/Documents/AI/Claude/claude_workspace/06_announcements/to_desktop_claude/
# Should be writable, check permissions
```

### Too Many Pulse Notifications

**Solution 1: Increase interval**
```bash
echo "600" > ~/.claude/heartbeat_duration.txt  # Every 10 minutes
```

**Solution 2: Go idle temporarily**
```bash
echo "0" > ~/.claude/heartbeat_duration.txt
# Resume later:
echo "300" > ~/.claude/heartbeat_duration.txt
```

**Solution 3: Implement smart pulse logic**
```bash
# Edit pulsing_daemon.sh to check workload before pulsing
# Only send pulse if there's actually work to review
```

### CLI Tasks Not Being Discovered

**Check 1: Symlink intact?**
```bash
ls -la ~/.claude/coordination
# Should point to: ~/Documents/AI/ai_comms/claude_cli/
```

**Check 2: Task format correct?**
```bash
cat ~/.claude/coordination/to_execute/req_XXXX_taskname.md
# Should have proper headers, task description, commands
```

**Check 3: CLI instance running?**
```bash
# Launch CLI instance if needed
```

### Watchdog Not Restarting Daemon

**Check 1: Cron job installed?**
```bash
crontab -l | grep watchdog
# Should show watchdog line
```

**Check 2: Watchdog script executable?**
```bash
chmod +x ~/bin/ai/watchdog_pulse.sh
```

**Check 3: Check cron logs**
```bash
# macOS
log show --predicate 'process == "cron"' --last 1h

# Linux  
grep CRON /var/log/syslog
```

---

## Related Documentation

**Core System:**
- `coordination_system_v4_digest.md` - CLI coordination protocol details
- `ai_conversation_orchestrator_memory_v01.md` - Chat orchestrator design
- `PULSE_SYSTEM_README.md` - Pulse implementation details
- `CLI_MONITORING_README.md` - CLI monitoring capabilities

**Implementation Files:**
- `/Users/shawnhillis/bin/ai/pulsing_daemon.sh` - Main daemon
- `/Users/shawnhillis/bin/ai/pulse_claude.sh` - Manual pulse trigger
- `/Users/shawnhillis/bin/ai/notify_desktop_smart.sh` - AppleScript notifications
- `/Users/shawnhillis/bin/ai/pulse_quickstart.txt` - Quick reference guide

**Configuration:**
- `~/.claude/heartbeat_duration.txt` - Pulse interval control
- `~/.claude/daemon.log` - Operational log
- `~/.claude/daemon.pid` - Process tracking

---

**Document Size:** ~15KB  
**Recommended Location:** `~/Documents/AI/ai_root/ai_general/docs/ai_communication_architecture_v1.md`  
**Refresh Trigger:** Implementation progress, architectural changes, or production lessons learned
