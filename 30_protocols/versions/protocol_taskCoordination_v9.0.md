# Task Coordination Protocol v9.0

**Version:** 9.0  
**Date:** 2026-01-25  
**Status:** active  
**Supersedes:** v8.0 (2025-12-14)

## Summary of Changes from v8.0

### 1. Simplified Claiming (BREAKING CHANGE)
Replaces filename renaming with flag files:

| Old (v8.0) | New (v9.0) |
|------------|------------|
| Rename task file to `{task}.$$.md` | Task file unchanged |
| Rename directory to `claimed_{ts}_{pid}_{slug}/` | Directory name unchanged |
| PID encoded in filename | Flag file tracks claim |

**New Pattern:**
- Move directory from `to_execute/` to `in_progress/`
- Add zero-sized flag file: `{reqId}_{dateTime}_started`

**Rationale:** Simpler parsing, cleaner directories, no rename race conditions.

### 2. Completion Flag (BREAKING CHANGE)
Replaces `.completion.md` with zero-sized flag:

| Old (v8.0) | New (v9.0) |
|------------|------------|
| `{task}.$$.completion.md` with YAML content | `{reqId}_{dateTime}_completed` (zero-sized) |
| Contains timestamp, status, summary | Completion details in execution log or response file |

### 3. Directory Naming Convention
Standardized to `{reqId}_{slug}/` pattern:

```
req_1234_yaml_migration/
├── req_1234_yaml_migration.md     # Task definition (unchanged through lifecycle)
├── req_1234_20260125T143022_started   # Zero-sized flag (added at claim)
├── execution_log.md               # Worker's execution notes
├── artifacts/                     # Output files
└── req_1234_20260125T155500_completed  # Zero-sized flag (added at completion)
```

---

## Complete Directory Structure

```yaml
coordination_root:
  staged/:
    purpose: "Pre-pipeline holding - not yet released"
    contents: "Task directories"
    
  to_execute/:
    purpose: "Ready for workers to claim"
    contents: "Task directories (claimable)"
    
  in_progress/:
    purpose: "Active work"
    contents: "Task directories with _started flag"
    
  completed/:
    purpose: "Finished work"
    contents: "Task directories with _completed flag"
    
  error/:
    purpose: "Failed tasks"
    contents: "Task directories with error report"
```

---

## Task Directory Structure

### Naming Pattern

```
{reqId}_{slug}/
```

| Component | Description | Example |
|-----------|-------------|---------|
| `reqId` | Task identifier (e.g., `req_1234`) | `req_1234` |
| `slug` | Kebab-case description | `yaml_migration` |

### Required Contents

```
{reqId}_{slug}/
├── {reqId}_{slug}.md              # Task definition file
└── (created during lifecycle)
    ├── {reqId}_{dateTime}_started     # Zero-sized claim flag
    ├── execution_log.md               # Worker notes (optional)
    ├── artifacts/                     # Output directory (optional)
    └── {reqId}_{dateTime}_completed   # Zero-sized completion flag
```

---

## Task Lifecycle

### State Transitions

```
staged/          # Pre-release holding
    ↓ (orchestrator releases)
to_execute/      # Available for claiming
    ↓ (worker claims: mv dir + add _started flag)
in_progress/     # Active work
    ↓ (worker completes: add _completed flag + mv dir)
completed/       # Done
    
    ↓ (on failure)
error/           # Failed (with error report)
```


### Claiming a Task

```bash
# Worker claims task from to_execute/
cd coordination_root
TASK_DIR="req_1234_yaml_migration"
DATETIME=$(date +%Y%m%dT%H%M%S)

# Move directory to in_progress (atomic on same filesystem)
mv "to_execute/${TASK_DIR}" "in_progress/${TASK_DIR}"

# Create zero-sized flag file to mark claim
touch "in_progress/${TASK_DIR}/${TASK_DIR%%_*}_${DATETIME}_started"
```

**Key Points:**
- Directory name unchanged during claim
- Task file name unchanged
- Flag file format: `{reqId}_{YYYYMMDDTHHMMSS}_started`
- Flag is zero-sized (existence is the signal)

### Completing a Task

```bash
# Worker completes task
TASK_DIR="req_1234_yaml_migration"
DATETIME=$(date +%Y%m%dT%H%M%S)

# Create zero-sized completion flag
touch "in_progress/${TASK_DIR}/${TASK_DIR%%_*}_${DATETIME}_completed"

# Move to completed
mv "in_progress/${TASK_DIR}" "completed/${TASK_DIR}"
```

**Key Points:**
- Add completion flag before moving
- Directory name unchanged
- Completion details go in execution_log.md or response file

### Reporting Milestones

For milestone/checkpoint reports, create response files (not completion):

```bash
# Worker reports milestone (stays in in_progress)
cat > "in_progress/${TASK_DIR}/checkpoint_001.md" << EOF
# Checkpoint 001
**Timestamp:** $(date -Iseconds)
**Status:** awaiting_input

## Summary
What was accomplished...

## Needs
What is needed to continue...
EOF
```

### Error Handling

```bash
# Worker encounters fatal error
TASK_DIR="req_1234_yaml_migration"

# Create error report
cat > "in_progress/${TASK_DIR}/error_report.md" << EOF
# Error Report
**Timestamp:** $(date -Iseconds)
**Error Type:** [type]

## What Happened
Description of failure...

## What Was Tried
Recovery attempts...
EOF

# Move to error/
mv "in_progress/${TASK_DIR}" "error/${TASK_DIR}"
```

---

## Flag File Reference

### Started Flag
- **Pattern:** `{reqId}_{YYYYMMDDTHHMMSS}_started`
- **Example:** `req_1234_20260125T143022_started`
- **Size:** 0 bytes (zero-sized)
- **Created:** When worker claims task
- **Location:** Inside task directory in `in_progress/`

### Completed Flag  
- **Pattern:** `{reqId}_{YYYYMMDDTHHMMSS}_completed`
- **Example:** `req_1234_20260125T155500_completed`
- **Size:** 0 bytes (zero-sized)
- **Created:** When worker finishes task
- **Location:** Inside task directory (before move to `completed/`)

### Detecting Task State

```bash
# Check if task is claimed
if ls "${TASK_DIR}"/*_started 2>/dev/null | head -1; then
    echo "Task is claimed"
    STARTED_FLAG=$(ls "${TASK_DIR}"/*_started | head -1)
    CLAIM_TIME=$(basename "$STARTED_FLAG" | sed 's/.*_\([0-9T]*\)_started/\1/')
fi

# Check if task is complete
if ls "${TASK_DIR}"/*_completed 2>/dev/null | head -1; then
    echo "Task is completed"
fi
```

---

## Multi-Level Orchestration

### Hierarchy Model

```
Claude Desktop (top orchestrator)
├── Task: req_1234_arch_refactor/
│   ├── SubTask: req_1234_01_analyze/ → claude_cli
│   │   ├── SubTask: req_1234_01_01_review/ → claude_cli
│   │   └── SubTask: req_1234_01_02_test/ → codex_cli
│   └── SubTask: req_1234_02_implement/ → codex_cli
```

### Subtask Naming

```
{parentReqId}_{nn}_{slug}/
```

| Component | Description | Example |
|-----------|-------------|---------|
| `parentReqId` | Parent's reqId | `req_1234` |
| `nn` | 2-digit sequence | `01`, `02` |
| `slug` | Subtask description | `analyze` |

**Examples:**
- `req_1234_01_analyze/`
- `req_1234_01_01_review/` (nested subtask)
- `req_1234_02_implement/`


### Orchestrator Responsibilities

1. **Decompose** task into subtasks (if needed)
2. **Create** subtask directories with task files
3. **Distribute** subtasks to worker queues (`to_execute/`)
4. **Monitor** for `_completed` flags and response files
5. **Coordinate** inter-task dependencies
6. **Aggregate** results via symlinks
7. **Complete** own task when all children done

### Worker Responsibilities

1. **Claim** task: move directory to `in_progress/` + add `_started` flag
2. **Execute** work (may further decompose/delegate)
3. **Report** milestones via checkpoint files
4. **Complete**: add `_completed` flag + move to `completed/`
5. **Notify** parent (implementation-specific)

---

## File Reference

### Task File: `{reqId}_{slug}.md`

Required fields (per schema_taskFile):
- title (with reqId)
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

### Example Task Directory

```
req_1234_yaml_migration/
├── req_1234_yaml_migration.md           # Task definition
├── req_1234_20260125T143022_started     # Claim flag (zero-sized)
├── execution_log.md                     # Worker's notes
├── checkpoint_001.md                    # Milestone report
├── artifacts/
│   ├── converted_files.txt
│   └── validation_report.md
└── req_1234_20260125T155500_completed   # Completion flag (zero-sized)
```

---

## Migration from v8.0

### For Existing Tasks

1. **In-progress tasks:**
   - Strip `claimed_` prefix from directory
   - Create `{reqId}_{claimTime}_started` flag file
   - Rename task file from `{task}.$$.md` to `{task}.md`

2. **Completed tasks:**
   - Strip `claimed_` prefix from directory
   - Rename `.completion.md` file to `_completed` flag (if desired)
   - Keep `.completion.md` content in execution_log.md

### Script Example

```bash
#!/bin/bash
# migrate_v8_to_v9.sh - Migrate task from v8 to v9 format

TASK_DIR="$1"

# Extract components from claimed_TIMESTAMP_PID_slug format
if [[ "$TASK_DIR" =~ ^claimed_([0-9T_]+)_([0-9]+)_(.+)$ ]]; then
    TIMESTAMP="${BASH_REMATCH[1]}"
    PID="${BASH_REMATCH[2]}"
    SLUG="${BASH_REMATCH[3]}"
    
    NEW_DIR="${SLUG}"
    
    # Rename directory
    mv "$TASK_DIR" "$NEW_DIR"
    
    # Create started flag
    touch "${NEW_DIR}/${SLUG%%_*}_${TIMESTAMP}_started"
    
    # Rename task file (remove .$$.md)
    for f in "${NEW_DIR}"/*.$$.md; do
        mv "$f" "${f/.\$\$.md/.md}"
    done
    
    echo "Migrated: $TASK_DIR → $NEW_DIR"
fi
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 9.0 | 2026-01-25 | Flag files for claim/complete, no directory/file renaming |
| 8.0 | 2025-12-14 | Response/completion distinction, subtask nesting, symlinks, claim format |
| 7.0 | 2025-12-13 | Response/completion distinction, subtask nesting, symlinks |
| 6.0 | 2025-12-11 | Directory-always model, bundle support |
| 5.0 | 2025-11-23 | Directory-based with flat-to-folder transition |
| 4.0 | 2025-11-02 | Original directory-based coordination |

---

## Quick Reference

```
CLAIM:
  mv to_execute/{dir} in_progress/{dir}
  touch in_progress/{dir}/{reqId}_{datetime}_started

COMPLETE:
  touch in_progress/{dir}/{reqId}_{datetime}_completed
  mv in_progress/{dir} completed/{dir}

ERROR:
  create error_report.md
  mv in_progress/{dir} error/{dir}

DETECT CLAIM:
  ls {dir}/*_started

DETECT COMPLETE:
  ls {dir}/*_completed
```
