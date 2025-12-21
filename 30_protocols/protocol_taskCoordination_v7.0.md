# Task Coordination Protocol v7.0

**Version:** 7.0  
**Date:** 2025-12-13  
**Status:** active  
**Supersedes:** v6.0 (2025-12-11)

## Summary of Changes from v6.0

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
orch_o1_task/
├── orch_o1_task.$$.md           # Parent task (claimed)
├── orch_o1_task_t1.md           # Subtask 1 definition
├── orch_o1_task_t2.md           # Subtask 2 definition
└── artifacts/                    # Shared artifacts
```

**Subtask Naming:** `{parent}_t{N}.md` where N is sequence number.
**Nested Subtasks:** `{parent}_t{N}.{M}.md` for sub-subtasks.

### 3. Symlinks to Completed Children (NEW)
Orchestrators create symlinks to track completed child task dirs:

```
orch_o1_task/                           
├── orch_o1_task.$$.md
├── orch_o1_task_t1@ ──────────► ../../../claude_cli/Tasks/completed/orch_o1_task_t1/
└── orch_o1_task_t2@ ──────────► ../../../codex_cli/Tasks/completed/orch_o1_task_t2/
```

**Benefits:**
- Parent can inspect child results without knowing worker locations
- Audit trail of delegation preserved
- Single directory contains full task tree (via links)

### 4. Claim Format (UNCHANGED from v6)
Directory prefix: `claimed_{timestamp}_{pid}_`

```
in_progress/
└── claimed_20251213T143022_12345_orch_o1_task/
    └── orch_o1_task.$$.md
```

**Note:** The `$$` in filename is a secondary marker; the directory prefix is authoritative.

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
    contents: "Task directories with claimed_ prefix"
    claim_mechanism: "claimed_{timestamp}_{pid}_ prefix on directory name"
    
  completed/:
    purpose: "Finished work"
    contents: "Task directories (prefix removed)"
    
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
    ↓ (worker claims: mv + add prefix)
in_progress/     # Active work
    ↓ (worker completes: mv + remove prefix + add .completion.md)
completed/       # Done
    
    ↓ (on failure)
error/           # Failed (with error report)
```

### Claiming a Task

```bash
# Worker claims task
cd to_execute/
TASK="orch_o1_task"
TIMESTAMP=$(date +%Y%m%dT%H%M%S)
PID=$$
mv "$TASK" "../in_progress/claimed_${TIMESTAMP}_${PID}_${TASK}"

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
├── Task: orch_o1_task
│   ├── SubTask: orch_o1_task_t1 → claude_cli [clA]
│   │   ├── SubTask: orch_o1_task_t1.1 → claude_cli [clB]
│   │   └── SubTask: orch_o1_task_t1.2 → claude_cli [clC]
│   └── SubTask: orch_o1_task_t2 → codex_cli [coA]
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
    │                                     │                              │
    ├─── create task dir ────────────────►│ to_execute/task/             │
    │                                     │                              │
    │                                     │◄──── claim (mv + prefix) ────┤
    │                                     │ in_progress/claimed_*_task/  │
    │                                     │                              │
    │◄──────────── .response.md ──────────│◄───── milestone report ──────┤
    │                                     │                              │
    │─────── prompt to continue ─────────►│                              │
    │                                     │                              │
    │◄──────────── .completion.md ────────│◄───── task complete ─────────┤
    │                                     │ completed/task/              │
    │                                     │                              │
    ├─── create symlink ──────────────────┼──────────────────────────────┤
    │    task/child_task@ ───────────────►│ completed/child_task/        │
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
| 7.0 | 2025-12-13 | Response/completion distinction, subtask nesting, symlinks |
| 6.0 | 2025-12-11 | Directory-always model, bundle support |
| 5.0 | 2025-11-23 | Directory-based with flat-to-folder transition |
| 4.0 | 2025-11-02 | Original directory-based coordination |
