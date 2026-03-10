---
title: Hook & UI Event Architecture
version: draft_0.1
created: 2026-03-08
status: DRAFT
origin: Claude Code CLI session - feature-dev + task-coordination integration discussion
---

# Problem Statement

The AI ecosystem has multiple event sources (Claude Code hooks, tmux session state changes, MCP server events, UI interactions) but no unified architecture for:
1. Detecting when CLI agents reach decision gates and need orchestrator input
2. Routing events from hooks to the appropriate handler (orchestrator, notification system, logging)
3. Bridging interactive workflows (like feature-dev) with autonomous task coordination
4. Providing a consistent event model across different CLI platforms (Claude, Codex, Gemini, Cline)

# Scope

This document covers:
- Claude Code hook events as primary event source
- Event detection, routing, and handling patterns
- Integration with task coordination protocol
- Integration with interactive skill workflows (feature-dev, brainstorming, etc.)

Out of scope:
- Redesigning the task coordination protocol itself
- Modifying Claude Code's hook system (we work with what exists)

# Event Sources

## Claude Code Hooks (Native)

Available hook events and their orchestration relevance:

| Event | Fires When | Orchestration Use |
|-------|-----------|-------------------|
| `SessionStart` | Session begins | Register worker, announce availability |
| `Stop` | Claude finishes a turn | **Primary checkpoint detection** - parse last message for gates |
| `SubagentStop` | Sub-agent completes | Track feature-dev phase progress (explorer/architect/reviewer done) |
| `UserPromptSubmit` | User/orchestrator sends input | Log orchestrator decisions, audit trail |
| `PreToolUse` | Before tool execution | Guard rails, resource limits |
| `PostToolUse` | After tool execution | Progress tracking, side-effect logging |
| `Notification` | Claude sends notification | Forward to orchestrator |
| `TeammateIdle` | Teammate agent idle | Coordination signal |
| `TaskCompleted` | Background task done | Dependency resolution |
| `PreCompact` | Before context compaction | Save state before context loss |
| `InstructionsLoaded` | CLAUDE.md loaded | Verify bootstrap completed |
| `ConfigChange` | Config files change | Hot-reload coordination settings |
| `WorktreeCreate` | Worktree created | Track isolated workspaces |
| `WorktreeRemove` | Worktree removed | Cleanup tracking |

## Stop Hook Payload (Detail)

The Stop hook receives JSON via stdin:

```json
{
  "hook_event_name": "Stop",
  "session_id": "abc-123",
  "last_assistant_message": "...full text of Claude's final response...",
  "transcript_path": "/path/to/transcript.jsonl",
  "working_dir": "/path/to/project",
  "agent_id": "optional-agent-id",
  "agent_type": "optional-agent-type",
  "worktree": {
    "name": "feature-branch",
    "path": "/path/to/worktree",
    "branch": "feature-branch",
    "original_repo_directory": "/path/to/original"
  }
}
```

Environment variables: `CLAUDE_PROJECT_DIR`, `CLAUDE_SESSION_ID`, `CLAUDE_PLUGIN_ROOT`

## External State Sources (Polling-Based)

| Source | MCP Server | What It Provides |
|--------|-----------|-----------------|
| `is_busy()` | prompting | Whether target is currently processing |
| `observe_session()` | prompting | Tmux pane content snapshot |
| `wait_response()` | prompting | Block until pattern appears in output |
| `get_status()` | cli-agent | Session info with last output |

# Architecture

## Event Flow

```
┌─────────────────────────────────────────────────────────┐
│ CLI Agent (tmux session)                                │
│                                                         │
│  Claude Code ──→ Hook fires (Stop, SubagentStop, etc.)  │
│                      │                                  │
└──────────────────────┼──────────────────────────────────┘
                       │ JSON via stdin
                       ▼
              ┌─────────────────┐
              │  Hook Handler   │
              │  (shell script) │
              └────────┬────────┘
                       │
              ┌────────┼────────────────┐
              ▼        ▼                ▼
         ┌────────┐ ┌──────────┐ ┌───────────┐
         │Classify│ │  Log     │ │  Notify   │
         │ Event  │ │  Event   │ │  Always   │
         └───┬────┘ └──────────┘ └───────────┘
             │
     ┌───────┼──────────┐
     ▼       ▼          ▼
┌─────────┐┌────────┐┌──────────┐
│Checkpoint││Progress││Completion│
│ Detected ││ Update ││  Signal  │
└────┬─────┘└───┬────┘└────┬─────┘
     │          │          │
     ▼          ▼          ▼
┌──────────────────────────────────┐
│     Orchestrator / Notification  │
│     System / Task Coordination   │
└──────────────────────────────────┘
```

## Event Classification

The hook handler's primary job is classifying what kind of turn just completed. Classification is based on pattern matching `last_assistant_message`:

### Checkpoint Detection (Stop Hook)

```bash
# Patterns indicating a decision gate
CHECKPOINT_PATTERNS=(
  "Which approach"
  "before proceeding"
  "your preference"
  "want to fix"
  "Should I"
  "Would you like"
  "want me to"
  "Please answer"
  "waiting for your"
  "approve"
)

# Patterns indicating phase completion (feature-dev specific)
PHASE_PATTERNS=(
  "Phase 1.*complete"
  "Phase 2.*complete"
  "Clarifying Questions"      # entering Phase 3
  "Architecture Design"       # entering Phase 4
  "Quality Review"            # entering Phase 6
)
```

### Event Types

| Type | Meaning | Action |
|------|---------|--------|
| `checkpoint_reached` | Agent is waiting for a decision | Notify orchestrator, include questions |
| `phase_progress` | Agent completed a workflow phase | Update task progress, log |
| `subagent_completed` | Explorer/architect/reviewer finished | Track parallel work |
| `turn_complete` | Normal turn, no gate | Log only |
| `error_reported` | Agent reported an error | Escalate to orchestrator |
| `task_finished` | Agent indicates work is done | Move task to completed, notify |

## Integration with Task Coordination

### Mapping Feature-Dev Phases to Task Progress

When a task specifies `methodology: feature-dev`, the hook handler maps phases to task coordination signals:

| Feature-Dev Phase | Task Signal | IM Message |
|-------------------|-------------|------------|
| Phase 1: Discovery | `w2o_` progress update | "Discovery complete, requirements understood" |
| Phase 2: Exploration | `w2o_` progress update | "Codebase explored, {N} key files identified" |
| Phase 3: Questions | `w2o_` **checkpoint** | Full question list for orchestrator |
| Phase 4: Architecture | `w2o_` **checkpoint** | Architecture options for orchestrator |
| Phase 5: Implementation | `w2o_` progress updates | Incremental progress |
| Phase 6: Review | `w2o_` **checkpoint** | Review findings for orchestrator |
| Phase 7: Summary | flag file: `*_completed` | Final summary |

### Task Spec Extension

Tasks that use interactive methodologies can include orchestration hints:

```yaml
title: Add full-text search
methodology: feature-dev
orchestration:
  autonomy: checkpointed     # full | checkpointed | interactive
  architecture: pragmatic     # minimal | clean | pragmatic | ask
  quality_gate: fix-critical  # fix-all | fix-critical | skip | ask
  pre_answers:                # optional pre-supplied answers to common questions
    - pattern: "database.*preference"
      answer: "Use SQLite for now, PostgreSQL later"
    - pattern: "backward.*compat"
      answer: "No backward compatibility needed, greenfield"
```

When `autonomy: checkpointed`, the hook handler notifies the orchestrator at gates.
When `autonomy: full` with pre-answers, the hook handler can auto-respond by matching question patterns.

## Notification Routing

### Where Events Go

```
Hook fires
  │
  ├─→ Log file (always): ~/.claude/logs/hook_events.log
  │
  ├─→ Task IM folder (if task context):
  │     ai_comms/claude_cli/tasks/in_progress/{task}/w2o_{ts}.md
  │
  ├─→ Orchestrator notification (if checkpoint):
  │     notify_desktop_task_checkpoint.sh or equivalent
  │
  └─→ Notification pipeline (if configured):
        ~/bin/ai/notifications/
```

# Implementation Plan

## Phase 1: Stop Hook Handler Script

Create a reusable hook handler that:
- Reads JSON payload from stdin
- Classifies the event type
- Logs to hook event log
- Notifies orchestrator for checkpoints

## Phase 2: Task Coordination Integration

- Extend task spec schema with `orchestration:` block
- Wire hook handler to write IM messages for active tasks
- Map phase detection to progress signals

## Phase 3: Auto-Response (Optional)

- Pattern-match checkpoint questions against pre-answers in task spec
- Auto-inject responses via `send_keys()` for fully autonomous execution
- Requires high confidence in pattern matching to avoid wrong answers

## Phase 4: Multi-Platform Extension

- Adapt patterns for Codex CLI, Gemini CLI, Cline CLI
- Each platform needs its own event detection (hooks may differ)
- Normalize to common event model

# Open Questions

1. **Hook registration**: Should the Stop hook be installed globally (`~/.claude/settings.json`) or per-project? Global makes sense for orchestration.
2. **Race conditions**: If the orchestrator responds before the hook handler finishes classifying, could input arrive at the wrong time?
3. **Transcript parsing**: Should the handler also read the transcript file for richer context, or is `last_assistant_message` sufficient?
4. **SubagentStop granularity**: Can we distinguish which sub-agent completed (explorer vs architect vs reviewer) from the payload?
5. **HTTP hooks**: Would an HTTP hook posting to a local orchestration server be cleaner than shell scripts + file-based IM?

# Related Documents

- REF:ai_general/docs/30_protocols/protocol_taskCoordination.latest.yml
- REF:ai_general/docs/40_specs/spec_response_footer.latest.condensed.yml
- Feature-dev plugin: ~/.claude/plugins/cache/claude-code-plugins/feature-dev/1.0.0/
- CLI-Agent MCP: ai_general/apps/mcps/cli-agent/
- Prompting MCP: ai_general/apps/mcps/prompting/
