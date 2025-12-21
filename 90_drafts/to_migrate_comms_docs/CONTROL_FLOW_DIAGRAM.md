# Desktop Claude ⇄ CLI Control Flow Diagram

## Bidirectional Communication Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           DESKTOP CLAUDE                                     │
│                     (Coordinator & Strategist)                               │
└────────────┬─────────────────────────────────────────────────┬──────────────┘
             │                                                   │
             │ CONTROL (Desktop → CLI)                           │ MONITORING (CLI → Desktop)
             │                                                   │
             ↓                                                   ↑
    ┌────────────────────┐                            ┌────────────────────┐
    │  Task Assignment   │                            │  Task Responses    │
    │    (Layer 3)       │                            │    (Layer 3)       │
    │                    │                            │                    │
    │ to_execute/        │                            │ completed/         │
    │  req_XXXX.md       │                            │  response.md       │
    └────────────────────┘                            └────────────────────┘
             │                                                   ↑
             ↓                                                   │
    ┌────────────────────┐                            ┌────────────────────┐
    │ Prompt Injection   │                            │  Notifications     │
    │    (Layer 2)       │                            │    (Layer 2)       │
    │                    │                            │                    │
    │ prompting/         │                            │ notifications/     │
    │  *.prompt.txt      │                            │  *.txt             │
    └────────────────────┘                            └────────────────────┘
             │                                                   ↑
             ↓                                                   │
    ┌────────────────────┐                            ┌────────────────────┐
    │  Notifications     │                            │ Prompt Injection   │
    │    (Layer 2)       │                            │    (Layer 2)       │
    │                    │                            │                    │
    │ notifications/     │                            │ prompting/         │
    │  pending/          │                            │  incoming/         │
    └────────────────────┘                            └────────────────────┘
             │                                                   ↑
             ↓                                                   │
    ┌────────────────────┐                            ┌────────────────────┐
    │ Instant Messaging  │◄──────────────────────────►│ Instant Messaging  │
    │    (Layer 1)       │    Bidirectional           │    (Layer 1)       │
    │                    │                            │                    │
    │ orchestrator/      │                            │ orchestrator/      │
    │  conv_*/thread     │                            │  conv_*/thread     │
    └────────────────────┘                            └────────────────────┘
             │                                                   ↑
             ↓                                                   │
    ═════════════════════════════════════════════════════════════════════════
             │                                                   │
             ↓                                                   ↑
┌────────────────────────────────────────────────────────────────────────────┐
│                              CLI INSTANCES                                  │
│                        (Execution & Analysis)                               │
│                                                                             │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐            │
│  │ CLI_A    │    │ CLI_B    │    │ CLI_C    │    │ CLI_D    │            │
│  │ PID:123  │    │ PID:456  │    │ PID:789  │    │ PID:012  │            │
│  │ [BUSY]   │    │ [IDLE]   │    │ [BUSY]   │    │ [IDLE]   │            │
│  │ Task:101 │    │ ------   │    │ Task:103 │    │ ------   │            │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘            │
└────────────────────────────────────────────────────────────────────────────┘
             │                                                   ↑
             └───────────────── CLI Registry ─────────────────────┘
                           (Heartbeat updates)
```

## Communication Layer Details

### Layer 3: Async (Filesystem-Based)
**Latency:** Minutes to hours (human-mediated)
**Use:** Complex work delegation, detailed responses

```
Desktop: Write task spec → to_execute/req_XXXX.md
User:    Tell CLI "check inbox"
CLI:     Claims task → in_progress/req_XXXX/
CLI:     Executes commands
CLI:     Writes response → completed/req_XXXX/response.md
User:    Tell Desktop "check responses"
Desktop: Reviews results
```

### Layer 2: Polling (Near Real-Time)
**Latency:** 1-60 seconds
**Use:** Status updates, alerts, quick coordination

```
PROMPTS (Direct Input Injection):
Desktop → File → Daemon (polling) → iTerm2 API → CLI terminal
CLI → File → Daemon (polling) → AppleScript → Desktop chat

NOTIFICATIONS (Status Updates):
CLI → notifications/pending/ → Pulse daemon → Desktop prompt
Desktop → notifications/pending/ → CLI checks periodically
```

### Layer 1: Synchronous (Real-Time)
**Latency:** <1 second
**Use:** Interactive discussions, immediate responses

```
INSTANT MESSAGING:
Desktop: Write to orchestrator/instant_messaging/active/conv_001/
CLI:     Monitor conversation thread
CLI:     Add response to thread.jsonl
Desktop: Read response, continue discussion
Either:  Archive when complete
```

## Control Operations

### 1. Assign Task (Desktop → CLI)
```
Desktop Claude
  ↓
Write task specification
  ↓
~/.claude/coordination/to_execute/req_1020.md
  ↓
Optional: Send prompt for faster notification
  ↓
claude_cli/prompting/incoming/new_task.prompt.txt
  ↓
CLI Instance
  ↓
User: "check inbox"
  ↓
CLI claims task
```

### 2. Report Completion (CLI → Desktop)
```
CLI Instance
  ↓
Completes work
  ↓
Write response to completed/req_1020/response.md
  ↓
Write notification to claude/notifications/pending/
  ↓
Pulse daemon aggregates
  ↓
Desktop receives prompt in active chat
  ↓
Desktop: "check responses"
  ↓
Reviews results
```

### 3. Emergency Alert (Either Direction)
```
Source (CLI or Desktop)
  ↓
Write prompt to target's prompting/incoming/
  ↓
Daemon detects (1-2 seconds)
  ↓
iTerm2 API or AppleScript injection
  ↓
Appears immediately in target's interface
  ↓
Target responds urgently
```

### 4. Interactive Discussion
```
Desktop or CLI
  ↓
Start conversation: orchestrator/instant_messaging/active/conv_X/
  ↓
Write message to thread.jsonl
  ↓
Other party monitors thread
  ↓
Adds response
  ↓
Back-and-forth exchange (3-5 messages)
  ↓
Decision reached
  ↓
Archive conversation
```

## Monitoring Flows

### Registry Updates (Continuous)
```
CLI Instance
  ↓
Every 60 seconds: Update heartbeat
  ↓
claude_cli/cli_registry.json
  ↓
Update: last_seen, current_task, status
  ↓
Desktop Claude
  ↓
Query registry anytime
  ↓
View all active CLIs and their states
```

### Task Status (On-Demand)
```
Desktop Claude
  ↓
Run: cli_dashboard.sh
  ↓
Scans:
  - to_execute/ (pending)
  - in_progress/ (active)
  - completed/ (done)
  - error/ (failed)
  ↓
Displays: Current state of all tasks
```

### Notification Queue (Event-Driven)
```
CLI Event
  ↓
Write: claude/notifications/pending/event.txt
  ↓
Pulse daemon (every 60s or on trigger)
  ↓
Aggregate notifications
  ↓
Create prompt: claude/prompting/incoming/notify.prompt.txt
  ↓
Cron job (every 60s)
  ↓
AppleScript → Desktop Claude app
  ↓
Desktop sees notification in chat
```

## CLI-to-CLI Coordination

```
CLI_A (Phase 1 complete)
  ↓
Look up CLI_B in registry
  ↓
Write prompt: claude_cli/prompting/incoming/cli_456_phase2.prompt.txt
  ↓
Daemon detects file
  ↓
Read filename: Target = CLI_B (PID 456)
  ↓
iTerm2 API: Inject into CLI_B's session
  ↓
CLI_B receives prompt
  ↓
CLI_B starts Phase 2 immediately
  ↓
No Desktop mediation needed
```

## Error Recovery

### CLI Becomes Unresponsive
```
Desktop Claude
  ↓
Check registry: last_seen > 5 minutes
  ↓
Attempt prompt injection: "Report status"
  ↓
Wait 60 seconds
  ↓
Still no response?
  ↓
Mark CLI as "stale"
  ↓
Move CLI's in_progress task back to to_execute
  ↓
Notify user: "CLI_A not responding - task returned to queue"
  ↓
User investigates CLI terminal
```

### Task Execution Failure
```
CLI Instance
  ↓
Task fails (error, exception, etc.)
  ↓
Write error details: error/req_1020/response.md
  ↓
Send notification: claude/notifications/pending/req_1020_error.txt
  ↓
Desktop receives alert
  ↓
Reviews error details
  ↓
Decides: Retry? Modify? Abandon?
```

## Communication Channel Selection

```
┌─────────────────────────────────────────────────────────────┐
│                    DECISION TREE                             │
└─────────────────────────────────────────────────────────────┘

Is it URGENT/EMERGENCY?
│
├─ YES → Layer 2: Prompt Injection
│         (Appears in seconds)
│
└─ NO → Is it a STATUS UPDATE?
        │
        ├─ YES → Layer 2: Notifications
        │         (Processed in batches)
        │
        └─ NO → Does it need BACK-AND-FORTH?
                │
                ├─ YES → Layer 1: Instant Messaging
                │         (Real-time conversation)
                │
                └─ NO → Layer 3: Task Assignment
                        (Structured delegation)
```

## System Components

```
┌──────────────────────────────────────────────────────────────┐
│                     SYSTEM DAEMONS                            │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────────────┐      ┌──────────────────────┐      │
│  │ CLI Prompt Daemon   │      │  Pulse Daemon        │      │
│  │ (iTerm2 API)        │      │  (Notification       │      │
│  │                     │      │   Aggregation)       │      │
│  │ - Monitors prompts/ │      │                      │      │
│  │ - Injects into CLI  │      │ - Monitors notif/    │      │
│  │ - Routes by PID     │      │ - Creates prompts    │      │
│  └─────────────────────┘      └──────────────────────┘      │
│           │                            │                     │
│           └────────────────┬───────────┘                     │
│                            ↓                                 │
│               ┌────────────────────────┐                     │
│               │  CLI Registry Manager  │                     │
│               │  (Heartbeat Monitor)   │                     │
│               │                        │                     │
│               │ - Tracks active CLIs   │                     │
│               │ - Health checks        │                     │
│               │ - Stale detection      │                     │
│               └────────────────────────┘                     │
└──────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│                    DATA STORES                                │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐    │
│  │ CLI Registry │   │ Task Queue   │   │ Logs         │    │
│  │              │   │              │   │              │    │
│  │ JSON file    │   │ Directories  │   │ Text files   │    │
│  │ - PIDs       │   │ - to_execute │   │ - Per-CLI    │    │
│  │ - Sessions   │   │ - progress   │   │ - System     │    │
│  │ - Heartbeat  │   │ - completed  │   │ - Daemons    │    │
│  └──────────────┘   └──────────────┘   └──────────────┘    │
└──────────────────────────────────────────────────────────────┘
```

---

**Version:** 1.0  
**Created:** 2025-11-10  
**Purpose:** Visual reference for Desktop ⇄ CLI communication flows
