# Task ID Counter Specification

**Version:** 1.0.0  
**Created:** 2025-12-24  
**Status:** Draft  

## Purpose

Defines the atomic, hierarchical task ID generation system for multi-level orchestration. Ensures:
- No ID collisions across concurrent processes
- Hierarchical IDs that encode orchestration depth
- Stale task detection via PID markers
- Session conflict handling

## Counter Location

```
ai_comms/counters/
  global.NextId           # Top-level orchestrations: 0001, 0002...
  0001.NextId             # Tasks under orchestration 0001: 0001_t1, 0001_t2...
  0001_t1.NextId          # Subtasks under 0001_t1: 0001_t1.1, 0001_t1.2...
  0001_t1.1.NextId        # Sub-subtasks: 0001_t1.1.1, 0001_t1.1.2...
```

Each orchestrator creates its own counter file when spawning sub-tasks.

## ID Format

| Level | Format | Example |
|-------|--------|---------|
| Orchestration | `{seq}` | `0001` |
| Task | `{parent}_t{seq}` | `0001_t1` |
| Subtask | `{parent}.{seq}` | `0001_t1.1` |
| Sub-subtask | `{parent}.{seq}` | `0001_t1.1.1` |

Sequence uses hybrid base36: `0001-9999`, then `A000-Z999`, then `AA00-ZZ99`, etc.

## Atomic Claim Protocol

Only one process can claim an ID. Uses rename-as-lock:

```bash
claim_next_id() {
    local prefix="${1:-global}"
    local counter_dir="$AI_ROOT/ai_comms/counters"
    local counter_file="$counter_dir/${prefix}.NextId"
    local claim_file="${counter_file}.claimed.$$"
    
    # Ensure counter exists
    mkdir -p "$counter_dir"
    [[ ! -f "$counter_file" ]] && echo "0000" > "$counter_file"
    
    # Atomic claim: mv succeeds for exactly one process
    while ! mv "$counter_file" "$claim_file" 2>/dev/null; do
        sleep 0.05  # Brief wait, retry
    done
    
    # We own it - read, increment, release
    local current=$(cat "$claim_file")
    local next=$(increment_hybrid_base36 "$current")
    echo "$next" > "$counter_file"    # Release with new value
    rm -f "$claim_file"               # Cleanup claim marker
    
    # Return full ID with prefix
    if [[ "$prefix" == "global" ]]; then
        echo "$current"
    else
        echo "${prefix}.${current}"
    fi
}
```

## Increment Logic

```bash
increment_hybrid_base36() {
    local current="$1"
    # Tier 0: 0001-9999 (numeric)
    # Tier 1: A000-Z999 (1 letter prefix)
    # Tier 2: AA00-ZZ99 (2 letter prefix)
    # Tier 3: AAA0-ZZZ9 (3 letter prefix)
    # Implementation handles tier transitions
}
```

## Task Execution Priority

**Rule:** Deepest available leaves first, highest sequence within depth.

```
Given pending tasks:
  0001              (depth 0, orchestration)
  0001_t1           (depth 1, has children - blocked)
  0001_t1.1         (depth 2, leaf) ← Execute 2nd
  0001_t1.2         (depth 2, leaf) ← Execute 3rd  
  0001_t2           (depth 1, leaf) ← Execute 1st (no children)
  0002              (depth 0, leaf) ← Execute 4th

Sort key: (has_children ASC, depth DESC, sequence DESC)
```

A task with children cannot execute until all children complete.


## PID Markers for Stale Detection

When a worker claims a task, it adds PID to filename:

```
to_execute/0001_t1/0001_t1.md
    ↓ claim
in_progress/0001_t1/0001_t1.$$.md     # $$ = actual PID
```

**Stale detection on startup:**

```bash
detect_stale_tasks() {
    local in_progress="$AI_ROOT/ai_comms/$AGENT_TYPE/tasks/in_progress"
    
    for task_dir in "$in_progress"/*; do
        [[ -d "$task_dir" ]] || continue
        
        # Find PID marker in filename
        local task_file=$(find "$task_dir" -name "*.[0-9]*.md" | head -1)
        if [[ -n "$task_file" ]]; then
            local pid=$(echo "$task_file" | grep -oE '\.[0-9]+\.' | tr -d '.')
            
            # Check if PID still running
            if ! kill -0 "$pid" 2>/dev/null; then
                echo "STALE: $task_dir (PID $pid dead)"
                # Move to stale/ for inspection
                mv "$task_dir" "$AI_ROOT/ai_comms/$AGENT_TYPE/tasks/stale/"
            fi
        fi
    done
}
```

## Session Conflict Handling

CLI wrappers support `--on-conflict` flag:

| Option | Behavior |
|--------|----------|
| `fork` | Create parallel session with unique ID, run immediately (default) |
| `exit` | Check session lock, fail immediately if busy |
| `queue` | Write task to `to_execute/`, return immediately |

**Session lock file:**

```
~/.claude/locks/{agent_type}_session.lock
Contents: PID, timestamp, tmux_session
```

**Lock check (uses ps, not kill -0 due to macOS permissions):**

```bash
check_session_lock() {
    local session_name="$1"
    local lock_file="$LOCK_DIR/${session_name}.lock"
    
    if [[ -f "$lock_file" ]]; then
        local lock_pid=$(head -1 "$lock_file" 2>/dev/null || echo "")
        # Use ps to check if process exists (kill -0 fails on processes we can't signal)
        if [[ -n "$lock_pid" ]] && ps -p "$lock_pid" >/dev/null 2>&1; then
            return 1  # Session is busy
        fi
        # Stale lock - process dead
        rm -f "$lock_file"
    fi
    return 0  # Session available
}
```

acquire_session_lock() {
    local agent_type="$1"
    local lock_file="$HOME/.claude/locks/${agent_type}.lock"
    mkdir -p "$(dirname "$lock_file")"
    echo -e "$$\n$(date -Iseconds)\n${SESSION_ID:-default}" > "$lock_file"
}

release_session_lock() {
    local agent_type="$1"
    rm -f "$HOME/.claude/locks/${agent_type}.lock"
}
```

## CLI Wrapper Integration

```bash
# In claude_cli.sh - default is fork for parallelization

case "$on_conflict" in
    exit)
        if ! check_session_lock "$lock_name"; then
            echo "ERROR: Session busy" >&2
            exit 1
        fi
        acquire_session_lock "$lock_name"
        trap 'release_session_lock "$lock_name"' EXIT
        ;;
    
    fork)
        if ! check_session_lock "$lock_name"; then
            # Generate unique forked session name
            fork_session="${lock_name}_fork_$$_$(date +%s)"
            tmux_session="$fork_session"
            session_mode="new"
            lock_name="$fork_session"
        fi
        acquire_session_lock "$lock_name"
        trap 'release_session_lock "$lock_name"' EXIT
        ;;
    
    queue)
        if ! check_session_lock "$lock_name"; then
            task_id=$("$HOME/bin/ai/next_id.sh")
            task_dir="$AI_ROOT/ai_comms/$AGENT_TYPE/tasks/to_execute/${task_id}"
            mkdir -p "$task_dir"
            echo "$PROMPT" > "$task_dir/${task_id}.md"
            echo "Queued: $task_id"
            exit 0
        fi
        ;;
esac
```

## Counter Initialization

On first use of any counter:

```bash
init_counter() {
    local prefix="$1"
    local counter_file="$AI_ROOT/ai_comms/counters/${prefix}.NextId"
    
    if [[ ! -f "$counter_file" ]]; then
        mkdir -p "$(dirname "$counter_file")"
        echo "0001" > "$counter_file"
    fi
}
```

## Orchestrator Creating Sub-counters

When an orchestrator spawns sub-tasks:

```bash
create_subtask() {
    local parent_id="$1"
    local task_content="$2"
    
    # Initialize counter for this orchestration level
    init_counter "$parent_id"
    
    # Claim next subtask ID
    local subtask_id=$(claim_next_id "$parent_id")
    
    # Create task
    local task_dir="$AI_ROOT/ai_comms/$TARGET_AGENT/tasks/to_execute/${subtask_id}"
    mkdir -p "$task_dir"
    echo "$task_content" > "$task_dir/${subtask_id}.md"
    
    echo "$subtask_id"
}
```

## File Locations

| File | Purpose |
|------|---------|
| `ai_comms/counters/*.NextId` | Sequence counters per orchestration level |
| `~/.claude/locks/*.lock` | Session locks per agent type |
| `ai_comms/{agent}/tasks/to_execute/` | Pending tasks |
| `ai_comms/{agent}/tasks/in_progress/` | Claimed tasks (with PID marker) |
| `ai_comms/{agent}/tasks/completed/` | Finished tasks |
| `ai_comms/{agent}/tasks/stale/` | Orphaned tasks (dead PID) |

## Related Documents

- `protocol_taskCoordination.md` - Full task lifecycle
- `spec_orchestration_protocol.md` - Hierarchical orchestration
- `cli_session_persistence.md` - Session management
