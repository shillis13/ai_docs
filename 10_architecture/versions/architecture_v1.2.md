# AI Scripts & Orchestration Architecture

**Version:** 1.2.0 | **Status:** active

## Overview

### System Goals

- Enable autonomous multi-AI coordination without user intervention
- Provide clean, composable interfaces for AI communication
- Support synchronous (Layer 1), polling (Layer 2), and async (Layer 3) patterns
- Normalize script organization under unified directory structure
- Facilitate monitoring, scheduling, and notification across all AI channels

### Design Principles

- Flat-plus-internal layout: Main entry points at top level, channel-specific in _internal/
- Router pattern: Entry scripts validate and route to channel implementations
- Non-blocking operations: Notifications never block prompt delivery
- File-based coordination: Simple, debuggable, survives process crashes
- Session-aware: Track and target specific terminal/browser sessions
- Fail-safe defaults: Degrade gracefully when channels unavailable

### Major Components

- Core Entry Points (ai_isBusy, send_prompt, send_notification, monitor)
- Scheduling System (time-based prompt delivery)
- Coordination System (task assignment and execution)
- Terminal Session Tracking (iTerm2 session mapping)
- Notification Pipeline (async alerts and status updates)
- Pulse System (legacy monitoring, migration path defined)

### Evolution From V1 1

- Added send_notification.sh system (4 channels: desktop, webui, cli, user)
- Added scheduling triple (set_scheduled_prompt, send_scheduled_prompt, daemon)
- Unified monitor.sh with pattern matching and output modes
- Standardized _internal/ structure across all entry points
- Defined migration strategy from pulse/ to new monitor+notification
- Enhanced coordination system with scheduling/ and notifications/ directories

## Directory Structure

### Base Path

~/Documents/AI/ai_root/ai_general/scripts

### Layout Type

flat-plus-internal

### Description

Main entry points at scripts/ root, channel-specific implementations in _internal/

### Structure

#### Scripts Root

##### Path

scripts/

##### Contents

- ai_isBusy.sh          # Check if AI can accept prompts
- send_prompt.sh        # Send prompts to AI instances
- send_notification.sh  # Non-blocking notifications
- monitor.sh            # Read content from AI interfaces
- set_scheduled_prompt.sh     # Create/manage schedules
- send_scheduled_prompt.sh    # Execute scheduled prompts
- scheduled_prompts_daemon.sh # Background scheduler
- new_iterm_tab.sh      # Create new iTerm2 tabs
- cli_browser_automation_example.sh  # Browser automation template

#### Internal Implementations

##### Path

scripts/_internal/

##### Description

Channel-specific implementations called by entry points

##### Contents

###### Ai Isbusy

- _ai_isBusy_desktop.sh   # Claude Desktop busy check
- _ai_isBusy_cli.sh        # CLI session busy check
- _ai_isBusy_webui.sh      # Browser AI busy check

###### Monitor

- _monitor_desktop.sh      # Read from Claude Desktop
- _monitor_cli.sh          # Read from CLI sessions
- _monitor_webui.sh        # Read from browser AIs

###### Send Notification

- _send_notification_desktop.sh  # Notify Claude Desktop
- _send_notification_cli.sh      # Notify CLI sessions
- _send_notification_webui.sh    # Notify browser AIs
- _send_notification_user.sh     # macOS user notifications

#### Cli Utilities

##### Path

scripts/cli/

##### Subdirs

###### Lifecycle

cleanup_cli_sessions.sh, list_cli_sessions.sh

###### Wrappers

cli_script_wrapper.sh, cli_tmux_wrapper.sh, check_lock.sh

#### Notifications

##### Path

scripts/notifications/

##### Subdirs

###### Desktop

notify_desktop_smart.sh, notify_desktop_task_complete.sh

###### Pipeline

write_notification.sh, process_notifications.sh, process_notifications_cron.sh

#### Pulse System

##### Path

scripts/pulse/

##### Status

legacy - migration in progress

##### Subdirs

###### Core

pulse_claude.sh, pulsing_daemon.sh, start_pulse.sh, stop_pulse.sh

###### Monitoring

check_pulse_system.sh, pulse_status.sh, test_cli_monitoring.sh

###### Prompts

check_completions.sh, inject_cli_command.sh

##### Migration Note

See migration section for pulse/ → monitor.sh + send_notification.sh path

#### Orchestration

##### Path

scripts/orchestration/

##### Description

Higher-level coordination helpers

#### Legacy

##### Path

scripts/legacy/

##### Description

Deprecated scripts kept for reference

##### Subdirs

###### To Migrate

Scripts pending migration review

#### Tests

##### Path

scripts/tests/

##### Contents

- test_notification_system.sh
- test_send_notification_channels.sh

#### Install

##### Path

scripts/install/

##### Contents

- install_cron_jobs.sh

#### Setup

##### Path

scripts/setup/

##### Contents

- rebuild_myenv.sh
- setup_ai_chatgpt_repo.sh
- setup_ai_claude_repo.sh

## Entry Points

### Ai Isbusy

#### Script

ai_isBusy.sh

#### Signature

ai_isBusy.sh {AI_TYPE} {CONVO_ID}

#### Purpose

Check if AI instance can accept prompts (non-blocking readiness check)

#### Status

implemented

#### Parameters

##### Ai Type

###### Type

string

###### Values

- desktop
- cli
- webui

###### Description

Target AI channel type

##### Convo Id

###### Type

string

###### Description

Conversation/session identifier

#### Exit Codes

##### 0

Idle (safe to send)

##### 1

Busy (wait or fallback)

##### 2

Unknown state / error

#### Internals

##### Script

_ai_isBusy_desktop.sh

##### Purpose

Check Claude Desktop app state via AppleScript

##### Script

_ai_isBusy_cli.sh

##### Purpose

Check CLI session state via iTerm2 session tracking

##### Script

_ai_isBusy_webui.sh

##### Purpose

Check browser AI state via Puppeteer/CDP

#### Examples

- ai_isBusy.sh desktop 12345
- ai_isBusy.sh cli 67890
- ai_isBusy.sh webui chatgpt

### Send Prompt

#### Script

send_prompt.sh

#### Signature

send_prompt.sh {TARGET} {MESSAGE}

#### Purpose

Queue prompts for delivery to AI instances

#### Status

implemented

#### Parameters

##### Target

###### Type

string

###### Values

- claude-desktop
- claude-web
- claude-cli
- codex-cli
- chatgpt-app
- chatgpt-web
- user

###### Description

Target AI or user channel

##### Message

###### Type

string

###### Description

Prompt text to deliver

#### Queue Mapping

##### Claude Desktop

~/Documents/AI/ai_root/ai_comms/claude/prompting/incoming/scheduled/

##### Claude Web

~/Documents/AI/ai_root/ai_comms/claude/prompting/incoming/scheduled/

##### Claude Cli

~/.claude/coordination/to_execute/

##### Codex Cli

~/Documents/AI/ai_root/ai_comms/codex_cli/to_execute/

##### Chatgpt

~/Documents/AI/ai_root/ai_comms/chatgpt/inbox/

##### User

~/Documents/AI/ai_root/ai_comms/claude/notifications/pending/

#### Features

- File-based queueing for reliability
- Timestamped filenames for ordering
- Target-specific queue directories
- Fallback options (--fb_queue, --fb_notification)

#### Examples

- send_prompt.sh claude-desktop "Check coordination inbox"
- send_prompt.sh chatgpt-web "Analyze this data"

### Send Notification

#### Script

send_notification.sh

#### Signature

send_notification.sh {TARGET} {TARGET_ID} {MESSAGE} [OPTIONS]

#### Purpose

Non-blocking notifications to AIs and user (async status updates)

#### Status

implemented

#### Implementation Task

req_2123

#### Parameters

##### Target

###### Type

string

###### Values

- desktop
- webui
- cli
- user

###### Description

Notification channel

##### Target Id

###### Type

string

###### Description

Specific instance identifier (session ID, window ID, etc)

##### Message

###### Type

string

###### Description

Notification text

#### Options

##### Priority

###### Flag

--priority {low|normal|high|urgent}

###### Default

normal

###### Description

Notification priority level

##### Persist

###### Flag

--persist

###### Description

Keep in queue until acknowledged

##### Sound

###### Flag

--sound

###### Description

Play alert sound / terminal bell

#### Environment

##### Send Notification Dry Run

###### Description

Set to 1 to skip external side-effects (testing mode)

#### Internals

##### Script

_send_notification_desktop.sh

##### Purpose

Notify Claude Desktop via coordination system + AppleScript

##### Method

Write to notifications/pending/, optionally inject prompt

##### Script

_send_notification_cli.sh

##### Purpose

Notify CLI instances via iTerm2 session

##### Method

Write to session-specific notification queue, optional terminal bell

##### Script

_send_notification_webui.sh

##### Purpose

Notify browser AIs via coordination queue

##### Method

Write to webui-specific notification directory

##### Script

_send_notification_user.sh

##### Purpose

macOS user notifications

##### Method

osascript display notification, system alert sounds

#### Logging

##### Log File

scripts/logs/send_notification.log

##### Format

[timestamp] target={target} id={target_id} priority={priority} persist={persist} sound={sound}

#### Examples

- send_notification.sh desktop 48B10940 "Task req_2123 complete" --priority high --sound
- send_notification.sh cli $ITERM_SESSION_ID "Build finished" --priority normal
- send_notification.sh user desktop "Error detected in log" --priority urgent

### Monitor

#### Script

monitor.sh

#### Signature

monitor.sh {TARGET_TYPE} {TARGET_ID} [OPTIONS]

#### Purpose

Read content from AI interfaces (CLI, Desktop, WebUI)

#### Status

in_progress

#### Implementation Task

req_1005

#### Parameters

##### Target Type

###### Type

string

###### Values

- cli
- desktop
- webui

###### Description

Interface type to monitor

##### Target Id

###### Type

string

###### Description

Specific instance identifier

#### Options

##### Pattern

###### Flag

--pattern REGEX

###### Description

Filter output by regex pattern

##### Output

###### Flag

--output MODE

###### Values

- cleaned
- raw

###### Default

cleaned

###### Description

Output processing mode

##### Lines

###### Flag

--lines N

###### Default

50

###### Description

Number of lines to return

#### Internals

##### Script

_monitor_cli.sh

##### Purpose

Read from CLI terminal sessions

##### Method

iTerm2 session capture, terminal scrollback parsing

##### Script

_monitor_desktop.sh

##### Purpose

Read from Claude Desktop

##### Method

AppleScript UI element extraction, accessibility API

##### Script

_monitor_webui.sh

##### Purpose

Read from browser AI interfaces

##### Method

Puppeteer DOM extraction, CDP protocol

#### Use Cases

- Detect task completion markers
- Watch for error patterns
- Extract AI responses for processing
- Monitor long-running operations

#### Examples

- monitor.sh cli session_12345
- monitor.sh desktop claude --pattern "COMPLETE:"
- monitor.sh webui chatgpt --output raw --lines 100

### Scheduling System

#### Scripts

##### Name

set_scheduled_prompt.sh

##### Signature

set_scheduled_prompt.sh {create|update|cancel|list} [ARGS]

##### Purpose

Create and manage scheduled prompts

##### Name

send_scheduled_prompt.sh

##### Signature

send_scheduled_prompt.sh {SCHEDULE_FILE}

##### Purpose

Execute a scheduled prompt (called by daemon)

##### Name

scheduled_prompts_daemon.sh

##### Signature

scheduled_prompts_daemon.sh {start|stop|status|reload|process}

##### Purpose

Background daemon to process scheduled prompts

#### Status

implemented

#### Implementation Task

req_2124

#### Schedule Directory

~/.claude/coordination/scheduling/

#### Subdirectories

##### Active

Active schedules awaiting execution

##### Completed

One-time schedules that have run

##### Cancelled

User-cancelled schedules

#### Schedule Types

##### Once

###### Description

Execute once at specified time

###### Examples

- +30m
- +2h
- 2025-11-20 14:00:00

##### Periodic

###### Description

Repeat on interval

###### Examples

- @hourly
- @daily
- */5 (every 5 minutes)
- */30 (every 30 minutes)

#### Schedule File Format

##### Required Fields

- schedule_id: Unique identifier
- target: AI target (desktop, cli, webui)
- target_id: Specific instance
- prompt: Message to send
- schedule_spec: When to execute
- schedule_type: once or periodic
- next_run: Next execution timestamp
- priority: low|normal|high|urgent
- status: pending|active|completed|cancelled
- created: Creation timestamp
- updated: Last update timestamp

#### Daemon Operation

##### Scan Interval

60 seconds (configurable)

##### Lock Mechanism

flock on daemon.lock prevents concurrent processing

##### Pid Tracking

daemon.pid contains running daemon PID

##### Logging

~/.claude/coordination/logs/scheduling.log

#### Integration

- Calls send_prompt.sh for prompt delivery
- Uses --fb_queue and --fb_notification for fallbacks
- Periodic schedules update next_run and remain in active/
- One-time schedules move to completed/ after execution

#### Examples

- set_scheduled_prompt.sh create check_inbox desktop 48B10940 "check inbox" "+30m" normal
- set_scheduled_prompt.sh create daily_summary desktop 48B10940 "summarize today" "@daily" normal
- set_scheduled_prompt.sh list
- scheduled_prompts_daemon.sh start

## Integration Patterns

### Coordination System

#### Base Path

~/.claude/coordination/

#### Description

File-based task coordination between Desktop Claude and CLI instances

#### Structure

##### Tasks

###### Incoming

New task assignments from Desktop Claude

###### In Progress

Tasks currently being worked on by CLI instances

###### Completed

Finished tasks with responses

##### Broadcasts

###### Description

System-wide announcements visible to all CLI instances

###### Format

Markdown files with broadcast metadata

##### Scheduling

###### Active

Scheduled prompts awaiting execution

###### Completed

Executed one-time schedules

###### Cancelled

User-cancelled schedules

##### Notifications

###### Pending

New notifications awaiting delivery

###### Processed

Delivered notifications (archived)

##### Logs

###### Description

System operation logs (scheduling.log, etc)

#### Workflows

##### Task Assignment

- Desktop Claude posts task to tasks/incoming/req_XXXX_description.md
- CLI instance claims by moving to tasks/in_progress/req_XXXX/
- CLI executes task, writes response_001_cli_PID.md
- CLI moves task folder to tasks/completed/
- CLI calls send_notification.sh to alert Desktop Claude

##### Broadcast Message

- Desktop Claude writes to broadcasts/YYYYMMDD_topic.md
- All CLI instances check broadcasts/ on startup
- CLIs process relevant broadcasts and optionally respond

##### Multi Step Task

- Desktop posts initial task to tasks/in_progress/req_XXXX/
- CLI completes step 1, writes response_001
- Desktop adds followup_001.md with next instructions
- CLI processes followup, writes response_002
- Continues until Desktop marks complete or CLI finishes

### Terminal Session Tracking

#### Environment Variables

##### Iterm Session Id

###### Description

Unique iTerm2 session identifier

###### Format

w{window_id}t{tab_id}p{pane_id}

###### Propagation

Set by iTerm2, inherited by subprocesses

##### Term Session Id

###### Description

macOS Terminal.app session identifier

###### Propagation

Set by Terminal.app

##### Cli Instance Id

###### Description

PID of CLI instance ($$)

###### Usage

Liveness marker: ~/.claude/.cli_instance_$$

#### Wrappers

##### Codex.Sh

###### Description

Wrapper for Codex CLI

###### Ensures

ITERM_SESSION_ID propagated to CLI process

##### Claude.Sh

###### Description

Wrapper for Claude CLI

###### Ensures

ITERM_SESSION_ID propagated to CLI process

#### Purpose

- Route notifications to correct terminal session
- Target specific CLI instances for prompts
- Track active CLI sessions for monitoring
- Enable CLI-to-CLI direct communication

#### Registry

##### Location

~/.claude/cli_sessions/ (conceptual, not yet implemented)

##### Contents

Mapping of CLI_INSTANCE_ID → ITERM_SESSION_ID, start time, status

### Notification Flows

#### Desktop To Cli

##### Trigger

Task posted to coordination system

##### Flow

- Desktop Claude writes task to tasks/incoming/
- Desktop calls send_notification.sh cli {session_id} 'New task req_XXXX'
- _send_notification_cli.sh writes to CLI-specific queue
- CLI checks queue periodically or on notification receipt
- CLI claims and executes task

##### Alternatives

- Direct prompt injection via iTerm2 API (Layer 2)
- User manually prompts CLI to check inbox

#### Cli To Desktop

##### Trigger

CLI completes task

##### Flow

- CLI writes response to tasks/completed/req_XXXX/
- CLI calls send_notification.sh desktop {desktop_id} 'Task req_XXXX complete'
- _send_notification_desktop.sh writes to notifications/pending/
- Desktop Claude checks pending/ (periodic or on-demand)
- Desktop reviews response and takes action

##### Alternatives

- CLI uses send_prompt.sh for urgent notifications
- Scheduled daemon prompts Desktop to check completed/

#### Cli To User

##### Trigger

Error detected, critical alert needed

##### Flow

- CLI detects error condition
- CLI calls send_notification.sh user desktop "Error in req_XXXX" --priority urgent --sound
- _send_notification_user.sh triggers macOS notification
- User sees system notification banner
- User investigates via Desktop Claude or CLI session

#### Periodic Checks

##### Trigger

Scheduled monitoring

##### Flow

- scheduled_prompts_daemon.sh running in background
- Daemon checks active schedules every 60 seconds
- When schedule due, calls send_scheduled_prompt.sh
- Prompt delivered via send_prompt.sh
- Target AI receives prompt and processes

##### Examples

- Every 30 minutes: Check coordination inbox for new tasks
- Daily at 9am: Summarize yesterdays completed tasks
- Hourly: Monitor system health and report anomalies

### Monitoring Patterns

#### Pattern Watching

##### Description

Monitor AI interface for specific patterns, trigger actions

##### Implementation

- Call monitor.sh cli {session_id} --pattern 'COMPLETE:' --output cleaned
- Parse output for completion markers
- When detected, send notification or execute followup task

##### Use Cases

- Detect task completion without polling coordination system
- Watch for error messages in AI output
- Trigger actions based on AI state changes

#### Completion Detection

##### Description

Watch for task completion markers in AI output

##### Pattern

COMPLETE: {task_id}

##### Flow

- CLI or monitor script watches AI output
- Detects 'COMPLETE: req_XXXX' marker
- Triggers send_notification.sh to alert orchestrator
- Orchestrator reviews completed task

#### Error Detection

##### Description

Watch for error patterns, alert user immediately

##### Patterns

- ERROR:
- FAILED:
- Exception:
- Traceback:

##### Flow

- monitor.sh cli {session} --pattern 'ERROR:'
- If match found, call send_notification.sh user desktop ...
- User notified of error for intervention

#### State Monitoring

##### Description

Periodic checks of AI state (busy/idle)

##### Flow

- Scheduled check calls ai_isBusy.sh {target} {id}
- Exit code indicates state
- If busy for extended period, send alert
- Prevents queue buildup and missed prompts

## System Boundaries

### Desktop Claude Capabilities

#### Can Do

- Read/write files via filesystem
- Execute bash commands
- Call AppleScript via osascript
- Read coordination directories
- Write task definitions
- Process task responses
- Send notifications via send_notification.sh

#### Cannot Do

- Directly access iTerm2 Python API (runs in different process)
- Type into other application windows (limited AppleScript access)
- Control browser automation (no Puppeteer access)
- Read terminal scrollback directly

#### Workarounds

- Use CLI instances as agents for terminal operations
- Use browser automation scripts for web AI access
- Coordinate via file-based task system

### Cli Instance Capabilities

#### Can Do

- Full terminal access (read scrollback, execute commands)
- Access iTerm2 session information via environment variables
- Execute Python scripts with iTerm2 API
- Run browser automation (Puppeteer, Playwright)
- Read/write coordination files
- Claim and execute tasks
- Send notifications to all channels
- Monitor other AI interfaces

#### Cannot Do

- Directly access Claude Desktop UI (AppleScript limitations)
- Persist across terminal session closure (ephemeral)
- Guarantee execution order across multiple instances

#### Workarounds

- Use send_notification.sh desktop for Desktop Claude communication
- Use liveness markers (CLI_INSTANCE_ID files) for session tracking
- Use coordination locks for mutual exclusion

### Browser Automation

#### Capabilities

- Puppeteer: Full browser control, DOM access, event triggering
- CDP (Chrome DevTools Protocol): Low-level browser inspection
- Read AI responses from web interfaces
- Type prompts into web chat interfaces
- Monitor browser-based AI state

#### Limitations

- Requires browser in debug mode (--remote-debugging-port=9222)
- Anti-automation detection by some sites
- Session timeouts and login requirements
- Slower than native APIs

#### Supported Targets

- Claude Web (claude.ai)
- ChatGPT Web (chat.openai.com)
- Gemini Web (gemini.google.com)
- Grok Web (grok.x.ai)

### Applescript Capabilities

#### Can Do

- Control macOS apps with AppleScript dictionary support
- Send keystrokes to focused window
- Display user notifications
- Read UI element properties (accessibility API)
- Activate/focus specific applications

#### Limitations

- Limited access to Claude Desktop internals
- Fragile (breaks with UI changes)
- Slow compared to native APIs
- Requires app to be frontmost for many operations

#### Use Cases

- Notify Desktop Claude via display notification
- Focus Claude Desktop window
- Trigger cron-based prompt injection (legacy pulse system)

## Migration

### Overview

#### From

pulse/ monitoring system

#### To

monitor.sh + send_notification.sh + scheduled_prompts_daemon.sh

#### Reason

Unified interfaces, better channel separation, more maintainable

### Pulse Components

#### Keep

##### Script

pulse/core/pulse_claude.sh

##### Reason

Low-level terminal monitoring, specific to pulsing pattern

##### Action

Keep as specialized tool for terminal state reading

##### Script

pulse/core/pulsing_daemon.sh

##### Reason

Working cron-based pulse delivery system

##### Action

Continue using until scheduled_prompts_daemon.sh is stable

#### Migrate

##### Component

pulse/monitoring/*.sh → _internal/_monitor_cli.sh patterns

##### Scripts

- check_pulse_system.sh
- pulse_status.sh
- test_cli_monitoring.sh

##### Action

Extract pattern matching and state detection logic

##### Target

_monitor_cli.sh and monitor.sh --pattern option

##### Component

pulse/prompts/*.sh → send_notification.sh patterns

##### Scripts

- check_completions.sh
- inject_cli_command.sh

##### Action

Reimplement as notification flows using send_notification.sh

##### Target

_send_notification_cli.sh for direct injection

##### Component

pulse/core/pulsing_daemon.sh → scheduled_prompts_daemon.sh

##### Action

Migrate cron-based periodic checks to scheduling system

##### Target

scheduled_prompts_daemon.sh with @hourly/@daily schedules

#### Deprecate

##### Component

pulse/prompts/send_scheduled_prompt.sh

##### Reason

Superseded by scripts/send_scheduled_prompt.sh

##### Action

Mark deprecated, update references, eventual removal

##### Component

pulse/fix_cron*.sh, pulse/launchd_control.sh

##### Reason

Scheduling system uses launchd/systemd directly

##### Action

Keep for reference, move to scripts/legacy/

### Timeline

#### Phase 1

##### Name

Build new systems

##### Status

COMPLETE

##### Tasks

- req_2123: Implement send_notification.sh
- req_2124: Implement scheduling system
- req_1005: Complete monitor.sh

##### Deliverables

- All entry points functional
- _internal implementations for all channels
- Test suite for notification and scheduling

#### Phase 2

##### Name

Parallel operation

##### Status

CURRENT

##### Tasks

- Run pulse/ and new systems side-by-side
- Validate new system reliability
- Compare outputs and behaviors
- Fix any gaps or regressions

##### Duration

2-4 weeks

##### Success Criteria

- New system handles all pulse/ use cases
- No loss of functionality
- Better or equal performance

#### Phase 3

##### Name

Migrate workflows

##### Status

PLANNED

##### Tasks

- Update cron jobs to use scheduled_prompts_daemon.sh
- Replace pulse/prompts/* calls with send_notification.sh
- Update monitoring scripts to use monitor.sh
- Document migration for any custom pulse/ users

##### Deliverables

- All automated workflows use new system
- pulse/ scripts no longer called by cron/daemon
- Migration guide published

#### Phase 4

##### Name

Deprecation

##### Status

FUTURE

##### Tasks

- Mark pulse/ as deprecated (add DEPRECATED.md)
- Move pulse/ to scripts/legacy/pulse/
- Remove from PATH and documentation
- Keep for 6 months for emergency rollback

##### Deliverables

- scripts/legacy/pulse/ with deprecation notice
- Documentation updated to reference new system only
- DEPRECATED.md explaining migration path

### Decision Criteria

#### Proceed To Phase 3 When

- scheduled_prompts_daemon.sh runs for 2 weeks without crashes
- send_notification.sh delivers 100+ notifications successfully
- monitor.sh pattern matching proven reliable
- No critical bugs in new system
- User confirms new system meets needs

#### Rollback To Pulse If

- Critical failures in new system
- Data loss or corruption
- Unacceptable performance degradation
- Missing functionality that cannot be quickly added

## Design Decisions

### Flat Plus Internal Vs Subdirectories

#### Decision

Use flat-plus-internal layout

#### Rationale

- Entry points easy to discover at scripts/ root
- Channel-specific code isolated but not deeply nested
- PATH only needs to include scripts/, not subdirectories
- Minimal cognitive overhead for common operations
- Internal implementations can evolve without changing entry point contracts

#### Alternative Considered

Subdirectories (scripts/prompt/, scripts/notification/, scripts/monitoring/)

#### Why Rejected

- Requires PATH updates or wrapper scripts
- Harder to remember which directory contains which script
- More nesting for simple operations

### Yaml Over Json For Configs

#### Decision

Use YAML for configuration and architecture docs

#### Rationale

- Comments supported (critical for documentation)
- Human-readable and human-editable
- Less punctuation noise than JSON
- Native multi-line string support
- Standard for many infrastructure tools (Kubernetes, Ansible, etc)

#### Alternative Considered

JSON

#### Why Rejected

- No comment support
- Harder to read for humans
- Verbose for nested structures

### File Based Coordination Over Ipc

#### Decision

Use file-based coordination (directories and files)

#### Rationale

- Simple and debuggable (just ls and cat)
- Survives process crashes
- No daemon required for basic operation
- Cross-platform (works on any Unix-like system)
- Easy to inspect state manually
- Atomic operations possible with mv

#### Alternative Considered

IPC (sockets, message queues, databases)

#### Why Rejected

- Requires daemon for coordination
- Harder to debug (can't just cat a socket)
- State lost on process crash unless persisted
- More complex implementation

#### Tradeoffs

- Slower than in-memory IPC
- No built-in pub/sub (must poll or use fswatch)
- Potential race conditions (mitigated with locks and atomic moves)

### Notification Separate From Prompt

#### Decision

send_notification.sh is separate from send_prompt.sh

#### Rationale

- Different semantics: prompts demand response, notifications inform
- Notifications are non-blocking by design
- Different delivery guarantees (best-effort vs guaranteed delivery)
- Separate queues prevent notification spam from blocking prompts
- Allows different retry logic and persistence policies

#### Alternative Considered

Unified send_message.sh with --type flag

#### Why Rejected

- Mixing concerns makes interface harder to use
- Harder to enforce non-blocking for notifications
- More complex routing logic

### Scheduling Daemon Vs Cron

#### Decision

Use scheduled_prompts_daemon.sh with file-based schedules

#### Rationale

- Cron limited to fixed intervals, not relative times (+30m)
- Dynamic scheduling (add schedules without editing crontab)
- Schedule files are self-documenting
- Easy to cancel or update schedules without cron knowledge
- Daemon can handle complex scheduling logic
- Better logging and error handling

#### Alternative Considered

Pure cron with at for one-time schedules

#### Why Rejected

- Cron requires editing crontab (not user-friendly)
- at command not available on all systems
- No easy way to list or update schedules programmatically
- Harder to integrate with coordination system

#### Hybrid Approach

- Use launchd/systemd to start daemon on boot
- Daemon handles all scheduling logic
- Best of both worlds: reliability of system scheduler + flexibility of file-based schedules

## Testing

### Integration Tests

#### Notification Delivery

##### Test File

scripts/tests/test_notification_system.sh

##### Tests

- send_notification.sh desktop → notifications/pending/
- send_notification.sh cli → iTerm2 session bell
- send_notification.sh user → macOS notification
- send_notification.sh webui → webui queue
- Priority handling (urgent gets delivered first)
- Persist flag keeps notification until acknowledged
- Sound flag triggers terminal bell

##### Success Criteria

- All channels deliver successfully
- Correct files created in expected locations
- Logs show proper metadata
- No crashes or errors

#### Scheduling Execution

##### Test File

scripts/tests/test_scheduling_system.sh

##### Tests

- Create schedule with +5m relative time
- Daemon picks up and executes schedule
- Periodic schedule (@hourly) updates next_run
- One-time schedule moves to completed/
- Cancel schedule before execution
- Update schedule after creation

##### Success Criteria

- Schedules execute within 60 seconds of due time
- Periodic schedules remain active
- One-time schedules complete properly
- Daemon recovers from crashes (restart picks up schedules)

#### Coordination Flow

##### Test File

scripts/tests/test_coordination_flow.sh

##### Tests

- Desktop posts task to tasks/incoming/
- CLI claims task (moves to in_progress/)
- CLI writes response
- CLI moves to completed/
- CLI sends notification to Desktop
- Desktop receives notification

##### Success Criteria

- Task folder moves through states correctly
- Response file created with expected format
- Notification delivered to Desktop
- No file corruption or race conditions

#### Monitoring Patterns

##### Test File

scripts/tests/test_monitor_patterns.sh

##### Tests

- monitor.sh cli {session} returns output
- monitor.sh --pattern 'ERROR' filters correctly
- monitor.sh --output raw preserves formatting
- monitor.sh --lines 10 limits output

##### Success Criteria

- Output matches expected format
- Pattern filtering works correctly
- Line limits enforced
- No crashes on edge cases (empty output, invalid session)

### Test Coverage Expectations

#### Entry Points

- 100% of entry point scripts have integration tests
- All exit codes documented and tested
- All command-line options tested

#### Internal Implementations

- Each _internal/*.sh script has unit tests
- Edge cases tested (missing files, invalid IDs, timeouts)

#### Error Handling

- Graceful degradation tested (channel unavailable)
- Retry logic validated
- Error messages clear and actionable

### Validation Checklist

#### New Entry Point

- [ ] Usage documentation in script header
- [ ] Example invocations provided
- [ ] Exit codes documented
- [ ] Integration test written
- [ ] Logged to appropriate log file
- [ ] Error messages to stderr
- [ ] Success messages to stdout
- [ ] Added to this architecture document

#### New Internal Implementation

- [ ] Matches entry point interface contract
- [ ] Unit test written
- [ ] Error handling implemented
- [ ] Logging to entry point log file
- [ ] Executable permissions set
- [ ] Shebang and set -euo pipefail

## Future Enhancements

### Multi Ai Coordination

#### Description

Extend coordination system to support Gemini, Grok, ChatGPT

#### Requirements

- Unified task format across all AI agents
- AI-specific capabilities registry
- Smart task routing based on AI strengths
- Multi-AI collaboration on single task

#### Implementation

- Add Gemini, Grok to ai_isBusy.sh and send_prompt.sh targets
- Create _internal implementations for each AI
- Extend coordination system with cross-AI task dependencies

### Web Ui For Monitoring

#### Description

Dashboard for monitoring coordination system state

#### Features

- Real-time task status (pending, in progress, completed)
- Active schedule list with next run times
- Notification queue view
- AI instance health status
- Log aggregation and search
- Manual task creation and cancellation

#### Implementation

- Flask/FastAPI web server reading coordination directories
- WebSocket for real-time updates
- Static HTML/JS frontend
- Authentication for security

### Advanced Scheduling

#### Description

Enhanced scheduling capabilities

#### Features

- Task dependencies (schedule B after A completes)
- Conditional scheduling (only if condition met)
- Retry policies (retry failed schedules)
- Priority queues (high-priority schedules first)
- Schedule templates (reusable schedule patterns)

#### Implementation

- Extend schedule file format with dependencies and conditions
- Update daemon to evaluate dependencies before execution
- Add retry logic to send_scheduled_prompt.sh

### Comprehensive Logging Observability

#### Description

Unified logging and observability across all systems

#### Features

- Structured logging (JSON format)
- Log aggregation (all logs to single location)
- Log levels (DEBUG, INFO, WARN, ERROR)
- Correlation IDs (track request across systems)
- Metrics collection (execution times, success rates)
- Alerting (trigger notifications on error patterns)

#### Implementation

- Update all scripts to use common logging function
- Centralized log directory with rotation
- Log parsing and analysis tools
- Integration with monitoring systems (Prometheus, Grafana)

### Cross Platform Support

#### Description

Support Linux and Windows in addition to macOS

#### Requirements

- Abstract platform-specific code (_internal implementations)
- Use cross-platform tools where possible
- Graceful degradation when platform features unavailable

#### Implementation

- Linux: Use wmctrl/xdotool instead of AppleScript
- Windows: Use PowerShell for automation
- Detect platform at runtime and route accordingly

### Plugin System

#### Description

Allow users to add custom entry points and channels

#### Features

- Drop-in plugin directory (scripts/plugins/)
- Plugin manifest (metadata, dependencies, interface)
- Dynamic loading of plugins at runtime
- Plugin marketplace (share plugins with community)

#### Implementation

- Plugin discovery in scripts/plugins/
- Plugin validation (interface compliance)
- Sandboxing for security

## Version History

### V1 2 0

#### Date

2025-11-20

#### Changes

- Added send_notification.sh system (req_2123)
- Added scheduling triple (set_scheduled_prompt, send_scheduled_prompt, daemon) (req_2124)
- Unified monitor.sh with pattern matching and output modes (req_1005)
- Standardized _internal/ structure across all entry points
- Defined migration strategy from pulse/ to new monitor+notification
- Enhanced coordination system with scheduling/ and notifications/ directories
- Created comprehensive architecture_v1.2.yml documentation (req_2125)

#### Status

active

#### Supersedes

v1.1

### V1 1 0

#### Date

2025-11-10

#### Changes

- Initial unified architecture implementation
- Established flat-plus-internal layout
- Created coordination system (tasks, broadcasts, logs)
- Implemented ai_isBusy.sh and send_prompt.sh
- Terminal session tracking via ITERM_SESSION_ID
- Three-layer communication architecture (sync, polling, async)

#### Status

superseded

#### Document

archive/ARCHITECTURE_v1.1.md

## Related Documentation

### Current System

- TODO_MANAGER_STATUS_2025-11-20.md: Current implementation status
- terminal_session_tracking.md: iTerm2 session tracking implementation
- coordination_system_v4_digest.md: CLI coordination protocol

### Reference

- ai_communication_architecture_v1.md: Three-layer messaging architecture
- cli_orchestration_v1.md: CLI instance coordination patterns
- daemon_architecture_v1.md: Daemon design patterns

### Legacy

- PULSE_SYSTEM_README.md: Original pulse/ monitoring system (to be migrated)
- directory_structure_reference_v02.md: Early directory organization

### Tasks

- req_2123: send_notification.sh implementation (completed)
- req_2124: Scheduling system implementation (completed)
- req_1005: monitor.sh implementation (in progress)
- req_2125: This architecture document (in progress)

## Onboarding

### For New Ai Instances

#### Start Here

- Read this architecture_v1.2.yml document
- Understand the three core systems: prompting, notification, monitoring
- Review entry point signatures and examples
- Understand coordination system workflow (tasks/incoming → in_progress → completed)

#### Essential Commands

- ai_isBusy.sh {target} {id} - Check if AI ready
- send_prompt.sh {target} '{message}' - Send prompt
- send_notification.sh {target} {id} '{message}' - Send notification
- monitor.sh {target} {id} - Read AI output
- set_scheduled_prompt.sh list - View scheduled prompts

#### Coordination Workflow

- Check inbox: ls ~/.claude/coordination/tasks/incoming/
- Claim task: mv incoming/req_XXXX.md in_progress/req_XXXX/
- Execute task commands
- Write response: in_progress/req_XXXX/response_001_cli_$$.md
- Complete: mv in_progress/req_XXXX completed/
- Notify: send_notification.sh desktop {id} 'Task complete'

#### Common Patterns

- Periodic check: set_scheduled_prompt.sh create check_inbox desktop 48B10940 'check inbox' '@hourly' normal
- One-time reminder: set_scheduled_prompt.sh create reminder desktop 48B10940 'Review results' '+30m' high
- Watch for errors: monitor.sh cli $ITERM_SESSION_ID --pattern 'ERROR:'
- Test notification: send_notification.sh user desktop 'Test message' --sound

### For Developers

#### Adding New Entry Point

- Create main script at scripts/{name}.sh
- Implement router pattern (validate args, route to _internal)
- Create _internal implementations for each channel
- Write integration tests
- Update this architecture document
- Add examples to README.md

#### Adding New Channel

- Create _internal/_ai_isBusy_{channel}.sh
- Create _internal/_monitor_{channel}.sh
- Create _internal/_send_notification_{channel}.sh
- Update entry point case statements to include new channel
- Write channel-specific tests
- Update documentation

#### Debugging Tips

- Check logs: tail -f ~/.claude/coordination/logs/*.log
- Check logs: tail -f ~/Documents/AI/ai_root/ai_general/scripts/logs/*.log
- Inspect coordination state: ls -R ~/.claude/coordination/
- Test entry points manually before automation
- Use SEND_NOTIFICATION_DRY_RUN=1 for safe testing
- Check daemon status: scheduled_prompts_daemon.sh status
