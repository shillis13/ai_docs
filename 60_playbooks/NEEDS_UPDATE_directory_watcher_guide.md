# Directory Watcher Implementation Guide

## Current Implementation: Cron-Based Polling ✅

**Strategy:** Keep it simple until proven inadequate

### Active Watchers (macOS)

```bash
# Cron jobs running every minute:
* * * * * ~/bin/ai/send_scheduled_prompt.sh           # Prompt delivery
* * * * * ~/bin/ai/process_notifications_cron.sh      # Notification processing
*/5 * * * * ~/bin/ai/watchdog_pulse.sh                # Daemon health check
```

**What they watch:**
- `~/Documents/AI/Claude/claude_workspace/06_announcements/to_desktop_claude/*.prompt.txt`
- `~/.claude/notifications/pending/*.txt`
- `~/.claude/daemon.pid`

**Performance:**
- 60-second max latency (good enough for task completions)
- Minimal overhead (only runs when needed)
- Simple to debug (just check logs)

---

## Installation

### macOS (Current Platform)

```bash
# One-time setup
~/bin/ai/install_cron_jobs.sh

# Verify
crontab -l | grep "AI Coordination"

# Monitor
tail -f ~/.claude/cron.log
```

---

## Alternative Approaches (Future Consideration)

### Option A: fswatch (Cross-Platform, Still Bash)

**When to use:** Need <10s latency, want to stay in bash

**Installation:**
```bash
brew install fswatch  # macOS
# apt-get install fswatch  # Linux
# Windows: Use WSL or Python approach
```

**Implementation:**
```bash
#!/bin/bash
# ~/bin/ai/watch_prompts_fswatch.sh

PROMPT_DIR="$HOME/Documents/AI/Claude/claude_workspace/06_announcements/to_desktop_claude"

echo "Watching $PROMPT_DIR for new prompts..."

fswatch -0 "$PROMPT_DIR" | while read -d "" event; do
    # Only process .prompt.txt files
    if [[ "$event" == *.prompt.txt ]]; then
        echo "[$(date)] New prompt detected: $event"
        ~/bin/ai/send_scheduled_prompt.sh
    fi
done
```

**Pros:** Real-time, still bash, works on macOS/Linux  
**Cons:** Needs daemon management, Windows requires WSL

---

### Option B: Python watchdog (True Cross-Platform)

**When to use:** Friend needs Windows support, ready to convert to Python

**Installation:**
```bash
pip install watchdog
```

**Implementation:**
```python
#!/usr/bin/env python3
# ~/bin/ai/watch_prompts.py

import time
from pathlib import Path
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler
import subprocess

class PromptHandler(FileSystemEventHandler):
    def on_created(self, event):
        if event.src_path.endswith('.prompt.txt'):
            print(f"[{time.strftime('%Y-%m-%d %H:%M:%S')}] New prompt: {event.src_path}")
            # Call existing bash script or Python equivalent
            subprocess.run(['/Users/shawnhillis/bin/ai/send_scheduled_prompt.sh'])

class NotificationHandler(FileSystemEventHandler):
    def on_created(self, event):
        if event.src_path.endswith('.txt'):
            print(f"[{time.strftime('%Y-%m-%d %H:%M:%S')}] New notification: {event.src_path}")
            subprocess.run(['/Users/shawnhillis/bin/ai/process_notifications_cron.sh'])

if __name__ == "__main__":
    prompt_dir = Path.home() / "Documents/AI/Claude/claude_workspace/06_announcements/to_desktop_claude"
    notif_dir = Path.home() / ".claude/notifications/pending"
    
    observer = Observer()
    observer.schedule(PromptHandler(), str(prompt_dir), recursive=False)
    observer.schedule(NotificationHandler(), str(notif_dir), recursive=False)
    
    observer.start()
    print(f"Watching directories...")
    print(f"  - {prompt_dir}")
    print(f"  - {notif_dir}")
    
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        observer.stop()
    observer.join()
```

**Pros:** Works everywhere, Python is portable  
**Cons:** More code, need daemon management

**Daemon management (macOS launchd):**
```xml
<!-- ~/Library/LaunchAgents/com.pianoman.ai-watcher.plist -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.pianoman.ai-watcher</string>
    <key>ProgramArguments</key>
    <array>
        <string>/Users/shawnhillis/bin/ai/watch_prompts.py</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/Users/shawnhillis/.claude/watcher.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/shawnhillis/.claude/watcher.error.log</string>
</dict>
</plist>
```

```bash
# Load daemon
launchctl load ~/Library/LaunchAgents/com.pianoman.ai-watcher.plist

# Unload daemon
launchctl unload ~/Library/LaunchAgents/com.pianoman.ai-watcher.plist
```

---

### Option C: Automator Folder Actions (macOS Only)

**When to use:** Never (not portable, too magical)

**Why not:**
- Friend's Windows needs rule it out
- Harder to debug than scripts
- Fragile with macOS updates
- Hidden configuration

---

## Windows Port Strategy (Future)

### Phase 1: Convert to Python
Replace bash scripts with Python equivalents:

```python
# send_scheduled_prompt.py
import pyautogui  # Cross-platform keyboard automation
import pywinauto  # Windows-specific window control

# process_notifications.py
from pathlib import Path
import shutil

# watchdog integration (already Python)
```

### Phase 2: Task Scheduler
Replace cron with Windows Task Scheduler:

```powershell
# Create scheduled task (PowerShell)
$action = New-ScheduledTaskAction -Execute "python" -Argument "C:\Users\Friend\bin\ai\send_scheduled_prompt.py"
$trigger = New-ScheduledTaskTrigger -Once -At (Get-Date) -RepetitionInterval (New-TimeSpan -Minutes 1)
Register-ScheduledTask -Action $action -Trigger $trigger -TaskName "AI-Prompt-Delivery"
```

### Phase 3: Directory Structure
Map Unix paths to Windows:

```python
# cross_platform_paths.py
import os
from pathlib import Path

if os.name == 'nt':  # Windows
    AI_ROOT = Path.home() / "Documents" / "AI"
    CLAUDE_CONFIG = Path.home() / "AppData" / "Roaming" / "Claude"
else:  # macOS/Linux
    AI_ROOT = Path.home() / "Documents" / "AI"
    CLAUDE_CONFIG = Path.home() / ".claude"
```

---

## Decision Matrix

| Need | Current (Cron) | fswatch | Python | Automator |
|------|---------------|---------|--------|-----------|
| **Works now** | ✅ Yes | Need install | Need code | ✅ Yes |
| **Real-time** | ❌ 60s lag | ✅ <1s | ✅ <1s | ✅ <1s |
| **Portable** | ✅ Concept | ⚠️ WSL only | ✅ True | ❌ macOS only |
| **Simple** | ✅ Very | ⚠️ Medium | ❌ Complex | ✅ GUI |
| **Debuggable** | ✅ Easy | ✅ OK | ⚠️ Harder | ❌ Hard |
| **Friend ready** | ✅ Adapt | ❌ No | ✅ Yes | ❌ No |

---

## Recommendation: Phased Approach

### Phase 1: Now (Get It Working)
```bash
# Install cron jobs (current approach)
~/bin/ai/install_cron_jobs.sh

# Test for a few days
# Is 60s latency actually a problem? Probably not!
```

### Phase 2: If Real-Time Needed
```bash
# Try fswatch first (minimal change)
~/bin/ai/watch_prompts_fswatch.sh &

# If that works well, convert to Python later
```

### Phase 3: Windows Port (When Friend Asks)
```python
# Convert bash → Python (documented above)
# Use Task Scheduler instead of cron
# Test on friend's machine
# Iterate based on his feedback
```

---

## Current Status

✅ **Implemented:** Cron-based polling (60s latency)  
📋 **Ready:** fswatch approach (if needed)  
📋 **Documented:** Python conversion path (for Windows)  
❌ **Not doing:** Automator (poor portability)

**Verdict:** Don't complicate it until you need to. Cron works fine for task notifications!

---

## Testing Current Implementation

```bash
# Install cron jobs
~/bin/ai/install_cron_jobs.sh

# Create test notification
mkdir -p ~/.claude/notifications/pending
cat > ~/.claude/notifications/pending/test_001.txt << 'EOF'
seq: 1
timestamp: $(date)
source: claude_cli
task_id: req_9999
status: completed
location: ~/.claude/coordination/completed/req_9999_test/

message:
Test notification from CLI
System check completed successfully
EOF

# Wait up to 60 seconds, should appear in Desktop Claude as prompt

# Check logs
tail -f ~/.claude/cron.log

# Verify notification processed
ls ~/.claude/notifications/processed/
```

---

**Portability Guide** | Updated: 2025-11-09 | Keep it simple until proven necessary
