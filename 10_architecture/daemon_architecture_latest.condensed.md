# Daemon Scripts Architecture v1.0

**Topic:** Pulsing daemon, watchdog system, and script organization  
**Status:** ✅ Operational  
**Created:** 2025-11-09  
**Location:** `~/bin/ai/` (reorganized structure)

---

## System Overview

A comprehensive daemon-based notification system enabling Layer 2 (polling) communication in the three-layer AI architecture. The system uses:

- **Pulsing Daemon:** Long-running process that wakes at configurable intervals
- **Watchdog Monitor:** Cron-based health check that restarts daemon if crashed
- **Prompt Delivery:** Cron job that injects prompts into Desktop Claude
- **Notification Pipeline:** Aggregates notifications into prompts

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    LAYER 2: POLLING SYSTEM                   │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────────────┐                                    │
│  │  Pulsing Daemon     │  Reads: ~/.claude/heartbeat_duration.txt
│  │  (Long-running)     │  Wakes: Every N seconds            │
│  │                     │  Writes: Prompts to inject         │
│  └──────┬──────────────┘                                    │
│         │                                                     │
│         ├──> Sends prompt: claude/prompting/incoming/        │
│         │                                                     │
│  ┌──────▼──────────────┐                                    │
│  │  Prompt Delivery    │  Cron: Every minute                │
│  │  (send_scheduled_   │  Reads: prompting/incoming/*.prompt.txt
│  │   prompt.sh)        │  Action: AppleScript → Desktop Claude
│  └─────────────────────┘                                    │
│                                                               │
│  ┌─────────────────────┐                                    │
│  │  Notification       │  Cron: Every minute                │
│  │  Pipeline           │  Reads: notifications/pending/     │
│  │  (process_          │  Aggregates: Multiple → Single prompt
│  │   notifications_    │  Writes: prompting/incoming/       │
│  │   cron.sh)          │                                     │
│  └─────────────────────┘                                    │
│                                                               │
│  ┌─────────────────────┐                                    │
│  │  Watchdog           │  Cron: Every 5 minutes             │
│  │  (watchdog_pulse.sh)│  Checks: Daemon still running?     │
│  │                     │  Action: Restart if crashed        │
│  └─────────────────────┘                                    │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

---

## Component Details

### 1. Pulsing Daemon (`~/bin/ai/pulse/core/pulsing_daemon.sh`)

**Purpose:** Long-running background process that wakes at configurable intervals to send "pulse" prompts to Desktop Claude.

**Key Features:**
- Infinite granularity (any positive integer seconds)
- Idle mode (0 or invalid = daemon sleeps, no pulses)
- Self-logging to `~/.claude/daemon.log`
- PID tracking in `~/.claude/daemon.pid`
- Graceful shutdown on SIGTERM

**Control Mechanism:**
```bash
# Duration file controls pulse frequency
~/.claude/heartbeat_duration.txt

# Contains single integer (seconds between pulses)
# Examples:
#   60   = Pulse every minute
#   300  = Pulse every 5 minutes
#   0    = Idle (daemon alive, no pulses)
#   [missing] = Idle
```

**Operation Logic:**
```bash
while true; do
    # Read duration file
    if [ -f ~/.claude/heartbeat_duration.txt ]; then
        duration=$(cat ~/.claude/heartbeat_duration.txt | tr -d '[:space:]')
        
        # Validate: positive integer?
        if [[ "$duration" =~ ^[0-9]+$ ]] && [ "$duration" -gt 0 ]; then
            # Valid duration - send pulse
            send_pulse_prompt
            sleep "$duration"
        else
            # Invalid duration - go idle (sleep 60s, no pulse)
            sleep 60
        fi
    else
        # No duration file - go idle
        sleep 60
    fi
done
```

**Startup:**
```bash
# Manual start
~/bin/ai/pulse/core/pulsing_daemon.sh &

# Or use helper
~/bin/ai/pulse/core/start_pulse.sh

# Watchdog will auto-start if not running
```

**Shutdown:**
```bash
# Kill daemon
kill $(cat ~/.claude/daemon.pid)

# Or use helper
~/bin/ai/pulse/core/stop_pulse.sh
```

### 2. Watchdog Monitor (`~/bin/ai/pulse/monitoring/watchdog_pulse.sh`)

**Purpose:** Cron job that ensures the pulsing daemon stays alive. Restarts if crashed.

**Cron Schedule:** Every 5 minutes
```bash
*/5 * * * * /Users/shawnhillis/bin/ai/pulse/monitoring/watchdog_pulse.sh
```

**Operation Logic:**
```bash
# Check if daemon running
if [ -f ~/.claude/daemon.pid ]; then
    pid=$(cat ~/.claude/daemon.pid)
    if ps -p "$pid" > /dev/null 2>&1; then
        # Daemon running - all good
        log "Daemon healthy (PID: $pid)"
        exit 0
    fi
fi

# Daemon not running - start it
log "Daemon not running, starting..."
nohup ~/bin/ai/pulse/core/pulsing_daemon.sh > /dev/null 2>&1 &
log "Daemon started (PID: $!)"
```

**Logs:** `~/.claude/watchdog.log`

### 3. Prompt Delivery (`~/bin/ai/pulse/prompts/send_scheduled_prompt.sh`)

**Purpose:** Cron job that reads prompt files and injects them into Desktop Claude using AppleScript.

**Cron Schedule:** Every minute
```bash
* * * * * /Users/shawnhillis/bin/ai/pulse/prompts/send_scheduled_prompt.sh
```

**Operation Logic:**
```bash
PROMPT_DIR=~/Documents/AI/ai_root/ai_comms/claude/prompting/incoming
SENT_DIR=~/Documents/AI/ai_root/ai_comms/claude/prompting/read

# Find pending prompts
for prompt_file in "$PROMPT_DIR"/*.prompt.txt; do
    [ -f "$prompt_file" ] || continue
    
    # Read prompt content
    content=$(cat "$prompt_file")
    
    # Inject into Desktop Claude via AppleScript
    osascript -e "tell application \"Claude\"
        activate
        tell window 1
            keystroke \"$content\"
            keystroke return
        end tell
    end tell"
    
    # Move to sent/
    timestamp=$(date +%Y%m%d_%H%M%S)
    basename=$(basename "$prompt_file")
    mv "$prompt_file" "$SENT_DIR/${timestamp}_${basename}"
done
```

**Features:**
- Waits for Desktop Claude to be idle (checks button count)
- Clears existing input before typing
- Archives sent prompts with timestamp
- Logs all operations

### 4. Notification Pipeline (`~/bin/ai/pulse/prompts/process_notifications_cron.sh`)

**Purpose:** Aggregates multiple notifications into single prompts to avoid flooding Desktop Claude.

**Cron Schedule:** Every minute (runs before prompt delivery)
```bash
* * * * * /Users/shawnhillis/bin/ai/pulse/prompts/process_notifications_cron.sh
```

**Operation Logic:**
```bash
NOTIF_DIR=~/Documents/AI/ai_root/ai_comms/claude/notifications/pending
PROMPT_DIR=~/Documents/AI/ai_root/ai_comms/claude/prompting/incoming
PROCESSED_DIR=~/Documents/AI/ai_root/ai_comms/claude/notifications/processed

# Collect all pending notifications
notifications=()
for notif in "$NOTIF_DIR"/*.txt; do
    [ -f "$notif" ] || continue
    notifications+=("$(cat \"$notif\")")
    mv "$notif" "$PROCESSED_DIR/"
done

# If notifications exist, create aggregated prompt
if [ ${#notifications[@]} -gt 0 ]; then
    timestamp=$(date +%Y%m%d_%H%M%S)
    cat > "$PROMPT_DIR/notifications_${timestamp}.prompt.txt" << EOF
You have ${#notifications[@]} new notification(s):

$(printf '%s\n' "${notifications[@]}")

Please review and take appropriate action.
EOF
fi
```

**Benefits:**
- Prevents notification spam (10 notifications → 1 prompt)
- Batch processing more efficient
- Desktop Claude sees organized summary

---

## File Organization (`~/bin/ai/`)

The script directory was reorganized from a flat 40+ file structure into logical subdirectories:

```
~/bin/ai/
├── docs/                    # Documentation
│   ├── CLI_MONITORING_README.md
│   ├── PULSE_INSTALLATION.md
│   ├── PULSE_QUICK_REFERENCE.md
│   ├── PULSE_SYSTEM_README.md
│   └── pulse_quickstart.txt
│
├── pulse/                   # Pulse system (Layer 2)
│   ├── core/                # Main daemon scripts
│   │   ├── pulsing_daemon.sh      # Main daemon
│   │   ├── pulse_claude.sh        # Manual trigger
│   │   ├── start_pulse.sh         # Start helper
│   │   └── stop_pulse.sh          # Stop helper
│   ├── monitoring/          # Health checks
│   │   ├── watchdog_pulse.sh      # Cron watchdog
│   │   ├── check_pulse_system.sh  # Health check
│   │   ├── check_claude_state.scpt
│   │   └── test_cli_monitoring.sh
│   └── prompts/             # Prompt delivery
│       ├── send_scheduled_prompt.sh
│       ├── inject_cli_command.sh
│       └── check_completions.sh
│
├── notifications/           # Notification system
│   ├── desktop/             # Desktop notifications
│   │   ├── notify_desktop_claude.scpt
│   │   ├── notify_desktop_smart.sh
│   │   └── notify_desktop_task_complete.sh
│   └── pipeline/            # Processing
│       ├── write_notification.sh
│       ├── process_notifications.sh  # Interactive
│       └── process_notifications_cron.sh  # Production
│
├── cli/                     # CLI coordination
│   ├── lifecycle/           # Launch, cleanup
│   │   ├── launch_claude_cli_task.scpt
│   │   ├── launch_codex_cli_task.scpt
│   │   ├── cleanup_cli_sessions.sh
│   │   └── list_cli_sessions.sh
│   └── wrappers/            # Wrappers
│       ├── cli_script_wrapper.sh
│       ├── cli_tmux_wrapper.sh
│       ├── monitor_cli_sessions.py
│       └── check_lock.sh
│
├── install/                 # Installation
│   ├── install_cron_jobs.sh  # Install all 3 cron jobs
│   └── codex_push_commits.script
│
├── setup/                   # One-time setup
│   ├── setup_ai_chatgpt_repo.sh
│   ├── setup_ai_claude_repo.sh
│   └── rebuild_myenv.sh
│
├── tests/                   # Testing
│   └── test_notification_system.sh
│
├── legacy/                  # Deprecated
│   ├── install_cron.sh  # OLD installer
│   └── to_migrate/      # Old scripts
│
└── scripts@ → ~/Documents/AI/ai_root/ai_general/scripts/  # Symlink
```

**Benefits of Reorganization:**
- ✅ Logical grouping by function
- ✅ Easy to find related scripts
- ✅ Clear deprecation (legacy/)
- ✅ Each directory has README
- ✅ Scalable structure

---

## Control Files

### Duration File (`~/.claude/heartbeat_duration.txt`)
```bash
# Single integer: seconds between pulses
# Examples:
echo "60" > ~/.claude/heartbeat_duration.txt    # Every minute
echo "300" > ~/.claude/heartbeat_duration.txt   # Every 5 minutes
echo "0" > ~/.claude/heartbeat_duration.txt     # Idle mode

# To check current setting:
cat ~/.claude/heartbeat_duration.txt
```

### Daemon PID (`~/.claude/daemon.pid`)
```bash
# Contains process ID of running daemon
# Used by watchdog to check if alive
cat ~/.claude/daemon.pid
ps -p $(cat ~/.claude/daemon.pid)  # Check if running
```

### Logs
```bash
# Daemon operations
~/.claude/daemon.log

# Watchdog operations  
~/.claude/watchdog.log

# Prompt delivery
# (Embedded in send_scheduled_prompt.sh output)
```

---

## Integration with Three-Layer Architecture

### Layer 1 (Synchronous)
Not used by daemon system. Layer 1 uses chat orchestrator for real-time AI-to-AI communication.

### Layer 2 (Polling) - **Primary Use**
```
Daemon System Components → Layer 2

1. Notification written: claude/notifications/pending/task_done.txt
2. process_notifications_cron.sh aggregates
3. Creates: claude/prompting/incoming/notifications_*.prompt.txt
4. send_scheduled_prompt.sh injects prompt
5. Desktop Claude receives and responds
```

**Pulse frequency controls Layer 2 responsiveness:**
- 60s = Very responsive, high interrupt frequency
- 300s = Balanced, checks every 5 minutes
- 900s = Low-priority, checks every 15 minutes

### Layer 3 (Async)
CLI coordination uses Layer 3. When CLI completes task:
```
1. CLI writes: claude_cli/completed/req_XXXX/response.md
2. CLI writes: claude/notifications/pending/task_XXXX_done.txt
3. Daemon system (Layer 2) delivers notification
4. Desktop Claude reviews Layer 3 completion
```

**Escalation Path:**
```
Layer 3 (Async) → Layer 2 (Pulse notification) → Layer 1 (If urgent)
```

---

## Common Operations

### Start System
```bash
# 1. Set pulse interval
echo "180" > ~/.claude/heartbeat_duration.txt

# 2. Install cron jobs (one-time)
~/bin/ai/install/install_cron_jobs.sh

# 3. Start daemon (or let watchdog start it)
~/bin/ai/pulse/core/start_pulse.sh

# 4. Verify
~/bin/ai/pulse/monitoring/check_pulse_system.sh
```

### Adjust Pulse Frequency
```bash
# Fast (active development)
echo "60" > ~/.claude/heartbeat_duration.txt

# Normal (regular work)
echo "300" > ~/.claude/heartbeat_duration.txt

# Slow (background monitoring)
echo "900" > ~/.claude/heartbeat_duration.txt

# Idle (stop pulsing, keep daemon alive)
echo "0" > ~/.claude/heartbeat_duration.txt
```

### Manual Pulse Trigger
```bash
# Bypass daemon, send immediate pulse
~/bin/ai/pulse/core/pulse_claude.sh
```

### Check System Health
```bash
# Comprehensive health check
~/bin/ai/pulse/monitoring/check_pulse_system.sh

# Output shows:
# - Daemon status (running/not running)
# - Current pulse interval
# - Recent log activity
# - Watchdog status
# - Recent completions
# - Pending prompts
```

### Monitor Logs
```bash
# Watch daemon log
tail -f ~/.claude/daemon.log

# Watch watchdog log
tail -f ~/.claude/watchdog.log

# Check recent pulse activity
tail -20 ~/.claude/daemon.log | grep "Sending pulse"
```

### Stop System
```bash
# Option 1: Go idle (daemon alive, no pulses)
echo "0" > ~/.claude/heartbeat_duration.txt

# Option 2: Kill daemon (watchdog will restart)
~/bin/ai/pulse/core/stop_pulse.sh

# Option 3: Full stop (remove watchdog, kill daemon)
crontab -e  # Remove watchdog line
~/bin/ai/pulse/core/stop_pulse.sh
```

---

## Design Rationale

### Why Daemon Instead of Cron?

**Cron-only approach:**
```bash
# Would need frequent cron (every minute)
* * * * * check_and_pulse.sh

# Problems:
# - High overhead (process start/stop every minute)
# - Can't maintain state between runs
# - Reading same files repeatedly
# - No sub-minute granularity
```

**Daemon approach:**
```bash
# Long-running process
pulsing_daemon.sh runs continuously

# Benefits:
# - Low overhead (one process, sleeps between pulses)
# - Maintains state (can track last pulse time, aggregate data)
# - Infinite granularity (any positive integer seconds)
# - Can adapt behavior based on history
```

### Why Watchdog Instead of systemd/launchd?

**Watchdog advantages:**
- ✅ Simple: Just a cron job checking PID
- ✅ Portable: Works on any Unix system
- ✅ Observable: Easy to debug with logs
- ✅ No complex plist/unit files
- ✅ User-level (no root/sudo required)

**systemd/launchd complexity:**
- Requires root or complex user-level setup
- Platform-specific configurations
- Harder to debug and modify
- Overkill for single daemon

### Why Duration Control Instead of State Machine?

**Duration-based (chosen):**
```bash
echo "120" > duration.txt  # Exactly 2 minutes
echo "37" > duration.txt   # Exactly 37 seconds
```
- ✅ Infinite granularity
- ✅ Claude controls timing precisely
- ✅ Simple to understand and modify
- ✅ No predefined mappings

**State-based (rejected):**
```bash
echo "active" > state.txt   # What does this mean?
echo "idle" > state.txt     # How often to check?
```
- ❌ Limited to predefined states
- ❌ Requires mapping (active=2min? 5min?)
- ❌ Less expressive
- ❌ Brittle

### Why Notification Aggregation?

**Without aggregation:**
```
10 tasks complete → 10 prompts → 10 Desktop Claude interruptions
```

**With aggregation:**
```
10 tasks complete → 1 aggregated prompt → 1 Desktop Claude interruption
Summary shows all 10 completions clearly
```

**Benefits:**
- Reduces interrupt fatigue
- More efficient processing
- Better overview of status
- Preserves Layer 2 intent (polling, not real-time)

---

## Troubleshooting

### Daemon Won't Start

**Check 1: Script executable?**
```bash
ls -la ~/bin/ai/pulse/core/pulsing_daemon.sh
# Should be: -rwxr-xr-x

# Fix:
chmod +x ~/bin/ai/pulse/core/pulsing_daemon.sh
```

**Check 2: Path errors?**
```bash
# Try manual start, watch for errors
~/bin/ai/pulse/core/pulsing_daemon.sh

# Check logs
cat ~/.claude/daemon.log
```

**Check 3: Permissions?**
```bash
# Can daemon write to prompt directory?
ls -la ~/Documents/AI/ai_root/ai_comms/claude/prompting/incoming/

# Should be writable by user
```

### No Prompts Appearing

**Check 1: Daemon running?**
```bash
ps aux | grep pulsing_daemon | grep -v grep
# Should show process
```

**Check 2: Valid duration?**
```bash
cat ~/.claude/heartbeat_duration.txt
# Should be positive integer
```

**Check 3: Prompt delivery working?**
```bash
# Check for pending prompts
ls ~/Documents/AI/ai_root/ai_comms/claude/prompting/incoming/

# Manually trigger delivery
~/bin/ai/pulse/prompts/send_scheduled_prompt.sh
```

**Check 4: Cron jobs running?**
```bash
crontab -l | grep "send_scheduled_prompt\|watchdog"

# Check cron logs (macOS)
log show --predicate 'process == "cron"' --last 1h | grep pulse
```

### Too Many Pulses

**Solution 1: Slow down**
```bash
echo "600" > ~/.claude/heartbeat_duration.txt  # Every 10 min
```

**Solution 2: Go idle temporarily**
```bash
echo "0" > ~/.claude/heartbeat_duration.txt
# Resume later
echo "300" > ~/.claude/heartbeat_duration.txt
```

**Solution 3: Check for runaway notifications**
```bash
# Are notifications flooding the system?
ls ~/Documents/AI/ai_root/ai_comms/claude/notifications/pending/

# If yes, investigate source
# Fix notification-generating process
```

### Watchdog Not Restarting Daemon

**Check 1: Cron installed?**
```bash
crontab -l | grep watchdog_pulse
# Should show entry

# Reinstall if missing
~/bin/ai/install/install_cron_jobs.sh
```

**Check 2: Watchdog executable?**
```bash
chmod +x ~/bin/ai/pulse/monitoring/watchdog_pulse.sh
```

**Check 3: Watchdog working?**
```bash
# Kill daemon manually
kill $(cat ~/.claude/daemon.pid)

# Wait 5 minutes, check if restarted
ps aux | grep pulsing_daemon

# Check watchdog log
tail ~/.claude/watchdog.log
```

---

## Performance Considerations

### CPU Usage
- **Daemon idle:** Minimal (<0.1% CPU, just sleeping)
- **Daemon active:** Brief spike when sending pulse (~1-2s)
- **Watchdog:** Runs for ~0.5s every 5 minutes
- **Cron jobs:** Each runs for ~1-2s per minute

**Total overhead:** Negligible (<1% average CPU usage)

### Disk I/O
- **Daemon:** Reads duration file once per wake cycle
- **Notifications:** Written as needed by other processes
- **Logs:** Append-only, minimal I/O

**Disk usage:** ~10KB/day logs (auto-rotated if needed)

### Memory
- **Daemon:** ~5MB RSS (resident set size)
- **Cron jobs:** Transient, ~2-3MB each during execution

**Total memory:** <10MB persistent

---

## Future Enhancements

### Smart Pulse Logic
```bash
# Only pulse if work to review
if [ $(ls -1 claude/notifications/pending | wc -l) -gt 0 ]; then
    send_pulse
else
    skip_pulse  # Nothing to report
fi
```

### Priority Queue
```bash
# Multiple urgency levels
claude/prompting/incoming/
├── urgent/    # Processed first
├── normal/    # Standard queue
└── low/       # Batch processed
```

### Historical Analytics
```bash
# Track pulse frequency over time
# Identify patterns (busy hours, idle periods)
# Auto-adjust frequency based on activity
```

### Multi-Agent Pulse
```bash
# Each agent controls its own pulse frequency
# Daemon manages multiple agents
chatgpt/pulse_config.txt
gemini/pulse_config.txt
```

---

## Related Documentation

- **Three-Layer Architecture:** `~/Documents/AI/ai_root/ai_general/docs/ai_communication_architecture_v1.md`
- **AI Comms Structure:** `~/Documents/AI/ai_root/ai_comms/README.md`
- **CLI Coordination:** `coordination_system_v4_digest.md`
- **Script Organization:** `~/bin/ai/docs/README.md`
- **Installation Guide:** `~/bin/ai/docs/PULSE_INSTALLATION.md`

---

## Version History

- **v1.0** (2025-11-09) - Initial daemon architecture documentation
  - Pulsing daemon with configurable frequency
  - Watchdog monitor with auto-restart
  - Notification aggregation pipeline
  - Prompt delivery system
  - Script reorganization into logical directories
  - Integration with three-layer architecture

---

**Document Size:** ~12KB  
**Recommended Location:** `~/Documents/AI/ai_root/ai_general/docs/daemon_architecture_v1.md`  
**Refresh Trigger:** Daemon enhancements, performance tuning, or integration with new systems
