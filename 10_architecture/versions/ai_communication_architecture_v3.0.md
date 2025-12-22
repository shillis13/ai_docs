# Architecture: AI Communication And Monitoring System

## 1. Overview

### Purpose and Goals
- Enable Desktop Claude, CLI agents, and automation daemons to communicate across synchronous, near-real-time, and asynchronous channels without human babysitting.
- Provide deterministic monitoring so any AI can prove liveness, accept interrupts, and document task history via the coordination system v4.0.
- Deliver a blueprint that balances reliability (watchdogs, heartbeats) with operator agility (phone-call style overrides, polling-based nudges, async delegation).

### High-Level System Components
- **Interface layer:** Desktop Claude, Claude CLI instances, ChatGPT Desktop/Web sessions reached via Puppeteer relay.
- **Transport layer:** Layer 1 (synchronous hooks), Layer 2 (file-based polling loops), Layer 3 (coordination system v4.0).
- **Automation layer:** Monitoring daemon worker, watchdog cron supervisor, adaptive heartbeat controller, AppleScript prompt injector, Puppeteer chat orchestrator.
- **Data layer:** Duration/state/log/prompt files under `~/.claude/monitoring/` plus coordination trees in `~/Documents/AI/ai_comms/`.
- **Observability layer:** Heartbeat metrics, state dashboards, log tailers, notification scripts.

### Key Design Principles
1. **Layered urgency:** Always choose the lightest layer that meets latency/interaction requirements.
2. **File-first coordination:** Every signal is persisted (duration files, prompt queues, coordination folders) so workers can recover from crashes.
3. **Two-process resilience:** Long-running workers never run alone; watchdogs restart them and record failure context.
4. **Tool neutrality:** AppleScript for Desktop apps, Puppeteer for browsers, Bash/Python glue—pick per surface and provide fallbacks.
5. **Deterministic control:** Claude’s control interface must be auditable (behavior matrices, decision trees) and adjustable at runtime.

ASCII high-level map:

```
          ┌──────────────────────────┐
          │   Desktop Claude (UI)    │
          │  + AppleScript injector  │
          └────────────┬─────────────┘
                       │ Layer 1 (real-time hooks)
┌──────────────────────┴──────────────────────┐
│      Three-Layer Communication Stack        │
│ ┌──────────────┐ ┌──────────────┐ ┌────────┐│
│ │Layer 1       │ │Layer 2       │ │Layer 3 ││
│ │Sync Hooks    │ │Polling Loops │ │Async FS││
│ └──────────────┘ └──────────────┘ └────────┘│
└────────────┬──────────────┬────────────┬────┘
             │              │            │
      Monitoring Daemon  Watchdog Cron  Coordination v4.0
             │              │            │
        Adaptive Heartbeats & File Stores
```

## 2. Three-Layer Communication Architecture

### 2.1 Layer 1 – Synchronous “Phone Call” Channel
**Purpose:** Override anything else when a human/AI needs immediate, conversational control (<1s latency).

**Components**
- **iTerm Python API watcher:** Streams CLI output as it is printed.
- **Puppeteer chat orchestrator:** Relays ChatGPT ⇄ Claude messages with sub-second turnaround.
- **AppleScript prompt injection:** Types directly into Claude Desktop / ChatGPT Desktop.

ASCII data flow:

```
Desktop Claude ──AppleScript──▶ Active Chat Window
        ▲                         │
        │                         ▼
 iTerm Python API ◀──tmux pane── CLI Session
        │
        └─HTTP/WebSocket──▶ Puppeteer Browser Relay
```

**Sample iTerm watcher** (`~/bin/monitor/iterm_relay.py`):

```python
#!/usr/bin/env python3
import asyncio, iterm2, pathlib, subprocess, textwrap

STREAM_FILE = pathlib.Path.home() / ".claude/monitoring/cli_pipe"

async def stream_cli(connection):
    app = await iterm2.async_get_app(connection)
    session = app.current_terminal_window.current_tab.current_session
    last = 0
    while True:
        text = await session.async_get_screen_contents()
        payload = "\n".join(text.splitlines()[last:])
        if payload:
            STREAM_FILE.write_text(payload)
            subprocess.run(
                ["osascript", "~/bin/scripts/notify_desktop_claude.scpt", payload[:500]],
                check=False,
            )
            last = len(text.splitlines())
        await asyncio.sleep(0.25)

iterm2.run_until_complete(stream_cli)
```

**Sample Puppeteer relay snippet** (`send-message.js`):

```javascript
import puppeteer from "puppeteer";

export async function relay(chatUrl, message) {
  const browser = await puppeteer.launch({headless: false, args:["--remote-debugging-port=9222"]});
  const page = await browser.newPage();
  await page.goto(chatUrl);
  await page.waitForSelector("textarea");
  await page.type("textarea", message);
  await page.keyboard.press("Enter");
  // Pipe transcript back to CLI
  const reply = await page.waitForSelector("[data-message-complete]");
  console.log(await reply.evaluate(el => el.textContent));
}
```

**AppleScript prompt injection** (`claude_desktop_prompt_sender.scpt`):

```applescript
on run argv
    set payload to item 1 of argv
    tell application "Claude" to activate
    delay 0.3
    tell application "System Events"
        keystroke payload
        keystroke return
    end tell
end run
```

**Use Cases**
- Interrupt runaway CLI before it writes bad data.
- Mediate real-time ChatGPT ↔ Claude debates.
- Deliver emergency instructions (“Stop task 310 immediately; disk full.”).
- Stream live progress to operators listening in.

**Example Scenario**
1. iTerm watcher notices `ERROR` in CLI log and posts snippet.
2. Desktop Claude injects `Ctrl+C` via AppleScript plus instructions.
3. Puppeteer orchestrator updates chat logs so both AIs see the interrupt transcript.

### 2.2 Layer 2 – Polling Loop “Text/IM” Channel
**Purpose:** Near real-time (1–5s) updates while tolerating brief disconnects.

**Mechanics**
- File-based conversation threads under `~/.claude/polling/threads/<id>/`.
- Each thread contains `incoming/`, `outgoing/`, `state.json`.
- Polling agents (Python/Bash) read/write files and sleep between passes.
- Supports interrupts by dropping `interrupt.flag` into thread root.

ASCII topology:

```
Desktop Claude
   │ writes summaries
   ▼
thread_X/outgoing/msg_*.md
   ▲
   │ poll loop reads every 2 s
CLI Agent / Background Worker
```

**Reference poller** (`poll_threads.sh`):

```bash
#!/usr/bin/env bash
set -euo pipefail

THREAD_ROOT="$HOME/.claude/polling/threads"
SLEEP_MIN=1
SLEEP_MAX=5

while true; do
  for thread in "$THREAD_ROOT"/*; do
    [[ -d "$thread/incoming" ]] || continue
    if [[ -f "$thread/interrupt.flag" ]]; then
      echo "$(date): interrupt detected in $(basename "$thread")"
      bash "$thread/hooks/on_interrupt.sh"
      rm "$thread/interrupt.flag"
    fi
    for msg in "$thread/incoming"/*.md; do
      [[ -f "$msg" ]] || continue
      cat "$msg" >> "$thread/log.md"
      mv "$msg" "$thread/archive/"
    done
  done
  sleep $(python3 - <<'PY'
import random; print(round(random.uniform(1,5),2))
PY
)
done
```

**Use Cases**
- Desktop posts clarifying questions; CLI responds within seconds.
- Background model monitors `git status` and pings operator when diff changes.
- Automatic “typing indicators” by writing heartbeat entries to `state.json`.

### 2.3 Layer 3 – Async File-Based “Email” Channel
**Purpose:** Durable coordination when no response time guarantee is required.

**Foundation:** Coordination system v4.0 (`~/Documents/AI/ai_comms/claude_cli/`, symlinked as `~/.claude/coordination`).

ASCII workflow:

```
Desktop Claude → to_execute/req_XXXX_*.md
CLI Worker     → moves to in_progress/req_XXXX/
CLI Worker     → writes response.md
CLI Worker     → moves folder to completed/
Desktop Claude → reads completed/req_XXXX_/response.md
```

**Sample task file**

```markdown
# Request #2203: Refresh Notification Daemon

**Type:** Maintenance  
**Priority:** HIGH  
**Posted:** 2025-11-08T08:15:00-08:00

## Task Description
Restart monitoring daemon after macOS security update. Follow watchdog guidelines.

## Task Commands
\`\`\`bash
launchctl unload ~/Library/LaunchAgents/ai.daemon.monitor.plist
launchctl load ~/Library/LaunchAgents/ai.daemon.monitor.plist
tail -n 50 ~/.claude/logs/monitor.log
\`\`\`

## Expected Response
- Confirmation of restart
- Last 50 log lines
- Next heartbeat ETA
```

**Use Cases**
- Large install/upgrade tasks.
- Delegated research (multi-hour).
- File conversions / data processing.
- Multi-step workflows tracked over days.

### 2.4 Communication Layer Comparison Matrix

| Dimension                  | Layer 1 – Synchronous | Layer 2 – Polling Loop | Layer 3 – Async File-Based |
|---------------------------|-----------------------|------------------------|----------------------------|
| Latency target            | < 1s                  | 1–5s                   | Minutes to days            |
| Transport                 | iTerm API, Puppeteer, AppleScript | File threads + polling daemons | Coordination v4.0 directories |
| Typical payload size      | Small (<2 KB, live text) | Moderate (status markdown, clips) | Large (full task folders) |
| Interrupt support         | Immediate (Ctrl+C, AppleScript) | Flag + handler | Post follow-up request |
| Reliability               | Depends on live sessions | High (files persisted) | Very high (fully durable) |
| Best for                  | Emergencies, pairing, teaching moments | Monitoring loops, quick clarifications | Delegated tasks, audit logs |
| Drawbacks                 | Requires active UI focus; complex | Slight lag, requires pollers | Slow, no inline back-and-forth |

### 2.5 Layer Selection Decision Tree

```
                     Start
                       │
           ┌───────────┴───────────┐
           │ Need response < 2 s?  │
           └───────┬───────────────┘
                   │Yes
                   ▼
        Use Layer 1 synchronous hooks
                   │
                   └──▶ Requires Desktop focus? Ensure AppleScript permission.
                   │
                   ▼No
           ┌───────┴─────────────┐
           │ Need answer < 2 min?│
           └───────┬─────────────┘
                   │Yes
                   ▼
          Use Layer 2 polling threads
                   │
                   └──▶ Enable interrupts if human may stop it.
                   │
                   ▼No
        Use Layer 3 coordination tasks
                   │
                   └──▶ Bundle logs + artifacts for audit.
```

## 3. Daemon/Watchdog Architecture

### 3.1 Two-Process Design Pattern
- **Worker daemon** (long-running Bash/Python) handles adaptive sleeping, heartbeats, Claude pulses.
- **Watchdog supervisor** (cron-invoked every minute) verifies worker health, detects hangs, and restarts safely.

ASCII depiction:

```
┌─────────────┐      heartbeat      ┌──────────────┐
│ Worker      │ ──────────────────▶ │ Duration File│
│ (daemon.sh) │ ◀──── control ───── │ control.json │
└──────┬──────┘                     └─────┬────────┘
       │  logs                               │
       ▼                                     ▼
 ~/.claude/logs/monitor.log         Watchdog cron (1 min)
                                           │
                                           ▼
                                   restart / alert scripts
```

### 3.2 Monitoring Daemon (Worker)
Responsibilities:
1. Read duration file to determine next wake interval.
2. Sleep dynamically (idle vs active vs emergency).
3. Wake, gather telemetry (CLI heartbeats, poll thread states).
4. Pulse Claude via AppleScript + notifications.
5. Emit structured logs for watchdog consumption.

**Full Bash reference implementation** (`~/.claude/monitoring/daemon_worker.sh`):

```bash
#!/usr/bin/env bash
set -euo pipefail

ROOT="$HOME/.claude/monitoring"
DURATION_FILE="$ROOT/duration.json"
STATE_FILE="$ROOT/state.json"
LOG_FILE="$ROOT/logs/daemon.log"
CLAUDE_NOTIFY="$HOME/bin/scripts/notify_desktop_claude.scpt"

jq_or_default() {
  jq -r "$1 // \"$2\"" "$DURATION_FILE"
}

pulse_claude() {
  local msg="$1"
  if ! osascript "$CLAUDE_NOTIFY" "$msg" >/dev/null 2>&1; then
    echo "$(date -Is) notify_fail $msg" >> "$LOG_FILE"
  fi
}

while true; do
  NOW=$(date +%s)
  MODE=$(jq_or_default '.mode' 'idle')
  INTERVAL=$(jq_or_default '.interval_seconds' '60')
  ACTIVE_UNTIL=$(jq_or_default '.active_until_epoch' '0')

  if (( NOW < ACTIVE_UNTIL )); then
    MODE="active"
    INTERVAL=$(jq_or_default '.active_interval_seconds' '10')
  fi

  NEXT=$(( NOW + INTERVAL ))
  echo "{\"last_pulse\":$NOW,\"next_pulse\":$NEXT,\"mode\":\"$MODE\"}" > "$STATE_FILE"

  # Collect status snapshot
  python3 "$ROOT/bin/collect_status.py" >> "$LOG_FILE" 2>&1 || true
  pulse_claude "🫀 Heartbeat ($MODE) next in ${INTERVAL}s"

  sleep "$INTERVAL"
done
```

### 3.3 Watchdog Cron (Supervisor)
Responsibilities:
- Runs every minute via `launchd` or `cron`.
- Reads `state.json` timestamp; if stale or corrupt, restarts worker.
- Detects “stuck” worker (no log change for N minutes).
- Ensures only one worker runs (`daemon.pid` lock).

**Cron entry** (`crontab -e`):

```
* * * * * /Users/shawnhillis/.claude/monitoring/watchdog.sh >> /Users/shawnhillis/.claude/logs/watchdog.log 2>&1
```

**Watchdog script**:

```bash
#!/usr/bin/env bash
set -euo pipefail

ROOT="$HOME/.claude/monitoring"
PID_FILE="$ROOT/daemon.pid"
STATE_FILE="$ROOT/state.json"
WORKER="$ROOT/daemon_worker.sh"
MAX_STALE=90   # seconds
MAX_RUNTIME=43200 # 12h safety restart

start_worker() {
  nohup "$WORKER" >/dev/null 2>&1 &
  echo $! > "$PID_FILE"
  echo "$(date -Is) restarted worker" >> "$ROOT/logs/watchdog.log"
}

if [[ -f "$PID_FILE" ]]; then
  PID=$(cat "$PID_FILE")
  if ! kill -0 "$PID" 2>/dev/null; then
    start_worker
    exit 0
  fi
else
  start_worker
  exit 0
fi

LAST=$(jq -r '.last_pulse // 0' "$STATE_FILE" 2>/dev/null || echo 0)
NOW=$(date +%s)

if (( NOW - LAST > MAX_STALE )); then
  kill "$PID" 2>/dev/null || true
  start_worker
  exit 0
fi

START=$(stat -f %B "$PID_FILE" 2>/dev/null || echo 0)
if (( NOW - START > MAX_RUNTIME )); then
  kill "$PID" 2>/dev/null || true
  start_worker
fi
```

### 3.4 Monitoring Signals
- **Duration file**: Worker obeys Claude’s requested sleep/active intervals.
- **State file**: Worker writes `last_pulse`, `next_pulse`, `mode`, `notes`.
- **Log file**: Worker + watchers append JSON lines for debugging.
- **Claude pulse**: Worker notifies Desktop via AppleScript when waking.

## 4. Adaptive Heartbeat System

### 4.1 Duration File Mechanism
`~/.claude/monitoring/duration.json` example:

```json
{
  "mode": "idle",
  "interval_seconds": 60,
  "active_interval_seconds": 10,
  "active_until_epoch": 1730995200,
  "sustain_minutes": 30,
  "owner": "Claude Desktop Layer 2"
}
```

- Worker reads every cycle—no restart needed to change cadence.
- Claude Desktop writes updates through control interface.

### 4.2 Claude’s Control Interface
- Desktop issues updates via `~/bin/scripts/update_duration.py --mode active --minutes 15`.
- Script validates JSON schema, bumps `active_until_epoch`, logs to `control.log`.
- UI macros (Layer 1) can trigger “panic mode” by writing `{"mode":"emergency","interval_seconds":3}`.

### 4.3 Dynamic Interval Adjustment Logic
1. Worker wakes, parses duration file.
2. If `active_until_epoch > now`, use `active_interval_seconds`.
3. Else if `mode == "idle"`, default to 60s.
4. If `mode == "sleep"`, extend to 300s (night mode).
5. Emergency override (Layer 1) sets interval to 3s until cleared.

### 4.4 Idle vs Active Modes
| Mode        | Interval | Trigger                                   | Effect                                      |
|-------------|----------|-------------------------------------------|---------------------------------------------|
| `active`    | 5–15 s   | Running critical task / user watching     | Frequent pulses, immediate interrupt honors |
| `idle`      | 60 s     | Default background monitoring             | Regular heartbeats, low CPU                 |
| `sleep`     | 300 s    | After-hours, no pending tasks             | Watchdog still alive, but minimal noise     |
| `emergency` | 3 s      | Panic button, Layer 1 override            | Rapid pulses + forced desktop notifications |
| `paused`    | ∞        | Maintenance window, watchdog holds lock   | Worker stops but watchdog records rationale |

### 4.5 Behavior Matrix (All States)

| State        | Worker Behavior                          | Watchdog Response                       | Claude UI Feedback                    |
|--------------|-------------------------------------------|-----------------------------------------|---------------------------------------|
| Active       | Sleep short, poll threads + CLI logs      | Expect state <30s old                    | Banner “Monitoring: ACTIVE (5s)”      |
| Idle         | Sleep 60s, only high-level status         | Accept state <90s                       | Status chip “Idle heartbeat”          |
| Sleep        | Sleep 300s, skip pulses                   | Accept state <360s                      | grey badge “Night mode”               |
| Emergency    | Sleep 3s, include CLI tail + alerts       | Expect state <10s, auto-escalate if >15 | Red toast + AppleScript prompt        |
| Paused       | Worker stopped; writes reason             | Watchdog suppresses restart if `pause.flag` | Desktop tooltip “Paused for maintenance” |

## 5. File Structures and Formats

### 5.1 Directory Layout

```
~/.claude/monitoring/
├── daemon_worker.sh
├── watchdog.sh
├── duration.json
├── state.json
├── logs/
│   ├── daemon.log
│   └── watchdog.log
├── pulses/
│   └── 2025-11-08T08-00-00.json
├── prompts/
│   └── layer1_interrupt_*.md
└── threads/ (Layer 2)
    └── thread_<uuid>/
        ├── incoming/
        ├── outgoing/
        ├── log.md
        └── interrupt.flag
```

### 5.2 Duration File Format
- JSON, UTF-8, single object.
- Required keys: `mode`, `interval_seconds`.
- Optional: `active_interval_seconds`, `active_until_epoch`, `owner`, `notes`.

### 5.3 State File Format (Optional)
Stored at `state.json`:

```json
{
  "last_pulse": 1730992000,
  "next_pulse": 1730992060,
  "mode": "idle",
  "layer": "monitoring-daemon",
  "notes": "watching 2 CLI sessions",
  "recent_findings": [
    "cli_2441 heartbeat 4s ago",
    "thread_ab12 idle 90s"
  ]
}
```

### 5.4 Log File Format
- Append-only JSON Lines for easy parsing.
- Example:

```json
{"ts":"2025-11-08T08:00:01-08:00","component":"daemon","event":"pulse","mode":"active","interval":5}
{"ts":"2025-11-08T08:00:05-08:00","component":"daemon","event":"claude_notify","result":"ok"}
{"ts":"2025-11-08T08:00:20-08:00","component":"layer2","thread":"thread_ab12","status":"interrupt_handled"}
```

### 5.5 Prompt File Naming and Location
- Stored under `~/.claude/monitoring/prompts/`.
- Format: `layer<layer>_<context>_<timestamp>.md` e.g. `layer1_interrupt_cli2441_20251108T0815.md`.
- Each file contains:
  - `# Prompt`
  - `## Target`
  - `## Message`
  - `## Expected Action`

### 5.6 Directory Structures
- **Layer 1 assets:** `~/bin/scripts/notify_desktop_claude.scpt`, `~/bin/projects/chat_orchestrators/`.
- **Layer 2 threads:** `~/.claude/polling/threads/<uuid>/`.
- **Layer 3 coordination:** `~/Documents/AI/ai_comms/claude_cli/` (symlinked).

## 6. Process Interactions (Sequence Diagrams)

### 6.1 Normal Operation Cycle

```
Desktop Claude | Daemon Worker | Watchdog | CLI Agent
───────────────┼───────────────┼──────────┼──────────
set duration   |               |          |
──────────────▶|               |          |
               | read duration |          |
               | pulse Claude  |          |
               |───────notify────────────▶|
               | sleep interval|          |
               | write state   |          |
               |──────────────▶|          |
               |               | verify   |
               |               |────────▶ |
               |               | OK       | continues work
```

### 6.2 Daemon Crash and Recovery

```
Daemon Worker unexpectedly exits
       │
Watchdog (t+60s) reads stale state (>90s) 
       │
Watchdog kills stale PID (if any) and relaunches worker
       │
New worker writes state, pulses Claude: "Recovered after crash"
```

ASCII sequence:

```
Desktop | Worker | Watchdog
────────┼────────┼─────────
        | crash! |
        |   X    |
        |        | detect stale
        |        |────────▶
        |        | restart worker
        |◀───────| notify recovery
```

### 6.3 Daemon Stuck Detection

```
Worker alive but not writing log (e.g., stuck in AppleScript call)
Watchdog compares log mtime & state timestamp
If unchanged > MAX_STALE:
 - Send Layer 2 message: "Worker stuck, forcing restart"
 - Kill worker, start new instance
 - Append incident in watchdog.log
```

### 6.4 Layer 1 Interrupt During Layer 2 Monitoring

```
Desktop Claude (Layer1) ──inject interrupt.cr ─▶ CLI
Layer2 poller detects `interrupt.flag` soon after
Poller writes summary to thread log, clears flag
Daemon sees both events, writes joint report, resets mode to active
```

Diagram:

```
Desktop | Layer1 Hook | Layer2 Poller | CLI
────────┼─────────────┼──────────────┼────
Need stop
────────▶ AppleScript sends Ctrl+C
                     │──────────────▶ CLI halts
                     │ set interrupt.flag
                     ▼
             poll loop wakes, handles flag
             log → daemon pulse
```

### 6.5 Transition from Active to Idle

```
Desktop clears active window via control interface
Duration file mode → idle
Worker reads new mode on next wake, lengthens interval
Watchdog observes slower cadence but state still fresh (<90s)
Claude UI shows "Returned to idle heartbeat"
```

## 7. Failure Modes and Recovery

| Failure             | Detection                              | Recovery Steps                                                                                  | Troubleshooting Commands                              |
|---------------------|----------------------------------------|--------------------------------------------------------------------------------------------------|-------------------------------------------------------|
| Daemon crashes      | Watchdog sees missing PID/log          | `watchdog.sh` restarts worker, posts Layer 3 note. Inspect `logs/daemon.log` tail.               | `tail -n 100 ~/.claude/logs/daemon.log`               |
| Daemon hangs        | State timestamp stale but PID alive    | Watchdog kills PID, restarts, raises Layer 2 interrupt for operators.                           | `ps -p $(cat daemon.pid) -o etime,cmd`                |
| File corruption     | `jq` errors parsing duration/state     | Watchdog copies bad file to `logs/corrupt_*`, restores last-known-good from `.bak`, notifies.    | `jq . ~/.claude/monitoring/duration.json`             |
| Directory permissions | Worker cannot write logs/prompts     | Watchdog detects `touch` failure, runs `repair_permissions.sh`, escalates if repeating.         | `ls -ld ~/.claude/monitoring ~/.claude/logs`          |
| Race conditions (Layer2) | Duplicate message processing     | Threads use `mv` to archive processed files; watchers check `.lock`.                             | `ls thread_x/archive`                                 |

**Troubleshooting guide quick actions**
1. **Verify heartbeat:** `jq '.last_pulse' ~/.claude/monitoring/state.json`.
2. **Check watchdog decisions:** `tail -n 50 ~/.claude/logs/watchdog.log`.
3. **Force restart:** `~/.claude/monitoring/watchdog.sh --force`.
4. **Simulate failure:** `pkill -f daemon_worker.sh` (watch auto recovery).
5. **Validate AppleScript path:** `osascript ~/bin/scripts/notify_desktop_claude.scpt "ping"`.

## 8. Implementation Details

### Required Files & Scripts
- `~/.claude/monitoring/daemon_worker.sh` – adaptive heartbeat worker.
- `~/.claude/monitoring/watchdog.sh` – cron supervisor.
- `~/bin/scripts/notify_desktop_claude.scpt` – AppleScript injector.
- `~/.claude/polling/poll_threads.sh` – Layer 2 loop.
- `~/bin/projects/chat_orchestrators/chat_orchestrator_puppeteer/send-message.js` – Layer 1/Chat orchestrator.
- `~/Documents/AI/ai_comms/claude_cli/docs/protocol_v04.0_directory_based.md` – Layer 3 spec reference.

### Cron / launchd Configuration
- Install watchdog cron entry (see §3.3).
- Optional `launchctl` plist for worker ensures startup on login:
  - Label: `ai.daemon.monitor`
  - ProgramArguments: `/Users/shawnhillis/.claude/monitoring/daemon_worker.sh`
  - RunAtLoad: true
  - KeepAlive: false (watchdog handles restarts)

### Permissions & Security
- Scripts should be `chmod 750`, owned by user.
- AppleScript automation requires Accessibility permissions (System Settings → Privacy & Security → Accessibility).
- Duration/state/log directories `chmod 700` to prevent other users from reading instructions.
- Node/Puppeteer runner should not store credentials in plain text; rely on logged-in Chrome profile.

### Logging & Debugging
- Standard logs: `~/.claude/logs/daemon.log`, `watchdog.log`, `layer2.log`.
- Enable verbose mode by `touch ~/.claude/monitoring/debug.flag` (worker emits more data until file removed).
- Use `jq -r '.event' daemon.log | sort | uniq -c` to summarize event frequency.

### Testing Procedures
1. **Unit check** – `bats tests/daemon_worker.bats`.
2. **Heartbeat latency** – `python3 tests/check_latency.py --expect <15`.
3. **AppleScript smoke** – `osascript notify_desktop_claude.scpt "test"`.
4. **Layer 2 integration** – `tests/simulate_thread.py --messages 5`.
5. **Failover drill** – `./watchdog.sh --simulate-crash` ensures restart + notification.
6. **Coordination round-trip** – Post dummy task to `to_execute/`, have CLI process it, ensure state change cascade.

## 9. Use Cases and Examples

- **Active AI-to-AI conversations (Layer 1)**: Puppeteer orchestrator opens Claude Web + ChatGPT Web, relays conversation while iTerm watcher mirrors CLI commentary; AppleScript interrupts if either goes off-script.
- **Background task monitoring (Layer 2)**: Polling loop watches `rsync` job logs, writes status markdown every 3s, daemon summarizes every minute.
- **User interrupts**: Desktop hits “Stop” macro → AppleScript sends Ctrl+C (Layer1) + writes `interrupt.flag` (Layer2) + posts `req_XXXX_followup.md` (Layer3) describing reason.
- **Multi-conversation monitoring**: Threads `thread_a`, `thread_b` correspond to different CLI instances; daemon state lists both, and duration file sets `mode=active` while >0 active threads.
- **Emergency escalations**: Watchdog detects repeated crashes, sets duration `mode=emergency`, pulses Claude every 3s until operator acknowledges.

## 10. Integration Points

- **CLI Coordination System v4.0**: Layer 3 tasks originate/complete here; daemon attaches logs to `completed/req_xxxx/monitoring.md` for traceability.
- **Chat Orchestrator (Puppeteer)**: Layer 1 uses orchestrator to keep ChatGPT and Claude Web in sync; orchestrator results stored in Layer2 threads for searchable context.
- **Desktop Prompt Injection (AppleScript)**: Shared utility for daemon pulses, manual interrupts, and Layer1 phone-call workflows; ensure scripts located under `~/bin/scripts`.
- **Future Extensions**
  1. Replace polling with event-driven watchers (fs events) where macOS permits.
  2. Integrate iTerm2 Python daemon for high-fidelity streaming.
  3. Add metrics exporter (Prometheus/StatsD) reading state/log files.
  4. Extend decision tree with ML-based priority scoring for automatic layer selection.

---

This document provides the actionable blueprint—covering layers, daemon/watchdog pattern, adaptive heartbeat logic, file formats, sequence diagrams, and troubleshooting—needed to implement the full AI communication and monitoring system end-to-end.
