# Task Coordination Protocol v8.0

**Version:** 8.0  
**Date:** 2025-12-14  
**Status:** active  
**Supersedes:** v7.0 (2025-12-13)

## Summary of Changes from v7.0

### 1. Response File Types (NEW)
Distinguishes milestone reports from final completion:

| File Pattern | Purpose | When Created |
|--------------|---------|--------------|
| `{task}.$$.response.md` | Milestone/interim report | Worker hits checkpoint, needs input |
| `{task}.$$.completion.md` | Final completion marker | Task fully done |

**Rationale:** Clearer semantics for orchestrators monitoring child tasks.

### 2. Subtask Nesting (NEW)
Tasks can contain subtasks within their directory:

```
task_0000_orch_chat_refactor/
├── task_0000.arch_chat_refactor.$$.md           # Parent task (claimed)
├── task_0000_01.arch_chat_refactor.md           # Subtask 1 definition
├── task_0000_02.arch_chat_refactor.md           # Subtask 2 definition
└── artifacts/                    # Shared artifacts

**Subtask Naming:** `{parentTaskNum}_{nn}.{slug}.md` where nn is sequence number.

### 3. Claim Format 
Task is claimed by the addition of {pid} ($$) to the Task filename
Task must be in a directory with same slug as Task minus extension
Task dir is moved to Task/{state} dir, e.g., Task/in_progress

```
in_progress/
└── Task_0000.arch_chat_refactor
    └── Task_0000.arch_chat_refactor.$$.md
    └── Task_0000_01.arch_chat_refactor@
    └── Task_0000_02.arch_chat_refactor@
```

---

## Complete Directory Structure

```yaml
coordination_root:
  staged/:
    purpose: "Pre-pipeline holding - not yet released"
    contents: "Task directories, bundle directories"
    
  to_execute/:
    purpose: "Ready for workers to claim"
    contents: "Task directories, bundle directories with claimable phases"
    
  in_progress/:
    purpose: "Active work"
    contents: "Task directories"
    claim_mechanism: "{.pid} field added to Task filename: {taskId}.{slug}{.pid}.md"
    
  completed/:
    purpose: "Finished work"
    contents: "Task directories"
    
  error/:
    purpose: "Failed tasks"
    contents: "Task directories with error reports"
```

---

## Task Lifecycle

### State Transitions

```
staged/          # Pre-release holding
    ↓ (orchestrator releases)
to_execute/      # Available for claiming
    ↓ (worker claims Task, ensures Task file is in a dir bundle, mv Task dir to in_progress)
in_progress/     # Active work
    ↓ (worker completes: adds pid..completed.md file, mv Task dir to completed)
completed/       # Done
    
    ↓ (on failure)
error/           # Failed (with error report)
```

### Claiming a Task

```bash
# Worker claims task
cd to_execute/
TASK="task_1234.task_title_slug"
PID=$$
mv "${TASK}.md" "${TASK}.${PID}.md"

# Mark task file as claimed
cd "../in_progress/claimed_${TIMESTAMP}_${PID}_${TASK}"
mv "${TASK}.md" "${TASK}.$$.md"
```

### Completing a Task

```bash
# Worker completes task
CLAIMED_DIR="claimed_20251213T143022_12345_orch_o1_task"
TASK="orch_o1_task"

# Create completion marker
echo "completed: $(date -Iseconds)" > "${TASK}.$$.completion.md"

# Move to completed (strip claim prefix)
mv "$CLAIMED_DIR" "../completed/${TASK}"
```

### Reporting Milestones

```bash
# Worker reports milestone (stays in in_progress)
echo "milestone: checkpoint-1" > "${TASK}.$$.response.md"
echo "status: awaiting_input" >> "${TASK}.$$.response.md"

# Notify parent orchestrator (implementation-specific)
# Options: file watch, notification daemon, direct prompt
```

---

## Multi-Level Orchestration

### Hierarchy Model

```
Claude Desktop (top orchestrator)
├── Task: task_1234.arch_chat_refactor.md
│   ├── SubTask: task_1234_01.arch_chat_refactor.md → claude_cli [clA]
│   │   ├── SubTask: task_1234_01_01.arch_chat_refactor.md → claude_cli [clB]
│   │   └── SubTask: task_1234_01_02.arch_chat_refactor.md → claude_cli [clC]
│   └── SubTask: task_1234_02.arch_chat_refactor.md → codex_cli [coA]
```

### Orchestrator Responsibilities

1. **Decompose** task into subtasks
2. **Distribute** subtasks to worker queues (`to_execute/`)
3. **Monitor** for `.response.md` (milestone) and `.completion.md` (done)
4. **Coordinate** inter-task dependencies
5. **Aggregate** results via symlinks
6. **Complete** own task when all children done

### Worker Responsibilities

1. **Claim** task from `to_execute/`
2. **Execute** work (may further decompose/delegate)
3. **Report** milestones via `.response.md`
4. **Complete** via `.completion.md` + move to `completed/`
5. **Notify** parent (implementation-specific)

### Delegation Flow

```
Orchestrator                          Worker Queue                    Worker
    │                                     │                                 │
    ├─── create task dir ────────────────►│ to_execute/task/                │
    │                                     │                                 │
    │                                     │◄──── claim in_progress/{task}/──┤
    │                                     │         {task}{.PID}.md         │
    │                                     │                                 │
    │◄──────────── .response.md ──────────│◄───── milestone report ──────   ┤
    │                                     │                                 │
    │─────── prompt to continue ─────────►│                                 │
    │                                     │                                 │
    │◄──────────── .completion.md ────────│◄───── task complete ─────────   ┤
    │                                     │ completed/task/                 │
    │                                     │                                 │
    ├─── create symlink ──────────────────┼──────────────────────────────   ┤
    │    task/child_task@ ───────────────►│ completed/child_task/           │
```

---

## File Reference

### Task File: `{task}.md` or `{task}.$$.md`

Required fields (per schema_taskFile):
- title
- type  
- priority
- posted (ISO timestamp)
- description
- expected_response

Optional fields:
- target_worker
- session
- entry_criteria
- exit_criteria
- subtasks (list)

### Response File: `{task}.$$.response.md`

```yaml
milestone: "checkpoint-name"
status: "awaiting_input|blocked|continuing"
summary: "What was accomplished"
needs: "What is needed to continue (if any)"
timestamp: "ISO timestamp"
```

### Completion File: `{task}.$$.completion.md`

```yaml
completed: "ISO timestamp"
status: "success|partial|failed"
summary: "Final outcome"
artifacts:
  - path: "relative/path/to/output"
    description: "What this artifact is"
```

---

## Migration from v6.0

1. **Response files:** Rename `response.md` to `.completion.md` for completed tasks
2. **No structural changes:** Directory model unchanged
3. **Add symlinks:** For existing orchestrated tasks, create symlinks to child dirs

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 8.0 | 2025-12-14 | Response/completion distinction, subtask nesting, symlinks, claim format |
| 7.0 | 2025-12-13 | Response/completion distinction, subtask nesting, symlinks |
| 6.0 | 2025-12-11 | Directory-always model, bundle support |
| 5.0 | 2025-11-23 | Directory-based with flat-to-folder transition |
| 4.0 | 2025-11-02 | Original directory-based coordination |
