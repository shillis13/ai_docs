# Task Coordination Protocol v6.0 - Key Changes from v5.0

## Summary of Changes

### 1. Directory-Always Model (NEW)

**v5.0:** Tasks start as flat files, become folders when claimed
**v6.0:** Tasks are ALWAYS directories from creation

```
# v5.0 (old)
to_execute/req_1001_task.md          # Flat file
in_progress/req_1001_task/           # Becomes folder when claimed

# v6.0 (new)  
to_execute/req_1001_task/            # Always a directory
├── req_1001_task.md                 # Task definition inside
└── (artifacts accumulate here)
```

**Rationale:** Whole folder moves between states. Artifacts (responses, data, logs) stay together.

### 2. Bundle Support (NEW)

Bundles group related phases that must be orchestrated together.

**Structure:**
```
req_XXXX_bundle_name/
├── req_XXXX_bundle_name.md          # Bundle overview (status, flow, phases)
├── req_XXXXa_phase_name/
│   ├── req_XXXXa_phase_name.md      # Phase task definition
│   └── response.md                  # Phase deliverable
├── req_XXXXb_phase_name/
│   └── req_XXXXb_phase_name.md
└── req_XXXXc_phase_name/
    └── req_XXXXc_phase_name.md
```

**Bundle overview file must contain:**
- Phase table with status and assignments
- Flow diagram (sequential or parallel)
- Blocking dependencies
- Current state summary

**Phase naming:** `req_XXXXa`, `req_XXXXb`, etc. (letter suffix)

### 3. Bundle Lifecycle

**Location rules:**
- Bundle stays in ONE directory (staged/, to_execute/, in_progress/, completed/)
- Location reflects bundle's aggregate state, not individual phases
- Phases complete internally; bundle moves only when ALL phases done OR orchestrator moves it

**State transitions:**
```
staged/           # Bundle not yet released
  ↓ (orchestrator releases)
to_execute/       # Phases available for claiming
  ↓ (worker claims phase)
in_progress/      # At least one phase active
  ↓ (all phases complete)
completed/        # Bundle done
```

**Parallel phases:** Multiple workers can claim different phases simultaneously within same bundle.

### 4. Updated Directory Structure

```yaml
coordination_root:
  staged/:
    purpose: "Pre-pipeline holding - bundles and tasks not yet released"
    contents: "Task directories, bundle directories"
    
  to_execute/:
    purpose: "Ready for workers to claim"
    contents: "Task directories, bundle directories with claimable phases"
    
  in_progress/:
    purpose: "Active work"
    contents: "Task directories, bundle directories with active phases"
    claim_mechanism: "claimed_{timestamp}_{pid}_" prefix on directory name
    
  completed/:
    purpose: "Finished work"
    contents: "Task directories, bundle directories (all phases done)"
    
  error/:
    purpose: "Failed tasks"
    contents: "Task directories with error reports"
```

### 5. Task File Naming (Unchanged)

Discrete tasks: `req_XXXX_slug/req_XXXX_slug.md`
Bundle phases: `req_XXXXa_slug/req_XXXXa_slug.md`

### 6. Response File Location (Clarified)

**v6.0:** Response files go INSIDE the task/phase directory:
```
req_1001_task/
├── req_1001_task.md
└── response.md              # Or response_NNN_cli_PID.md for multiple
```

NOT in a separate responses/ directory.

## Migration Notes

Existing flat files in to_execute/ should be converted to directories:
```bash
# For each req_XXXX_slug.md in to_execute/
mkdir req_XXXX_slug
mv req_XXXX_slug.md req_XXXX_slug/
```

### 7. Session Persistence (NEW)

**Problem solved:** Agent expertise dies with context. Each fresh instance starts at zero.

**Solution:** Use `--session-id` to maintain persistent agent identities.

**Registry:** `~/.claude/session_registry.yml`

**Core persistent agents:**

| Agent | Session ID | Purpose |
|-------|------------|---------|
| Librarian | `librarian-001` | Condensing, chat processing, memory management |
| Custodian | `custodian-001` | Maintenance, cleanup, organization |
| Reviewer | `reviewer-001` | Code quality, PR reviews |

**Launch pattern:**
```bash
# Persistent agent (continues prior context)
claude --project ~/Documents/AI/ai_root --session-id librarian-001 "process chats"

# Ephemeral task (fresh context)
claude --project ~/Documents/AI/ai_root "one-off task"
```

**When to use persistent sessions:**
- Role-based agents that accumulate expertise
- Long-running projects with context continuity
- Agents that need to remember prior work

**When to use ephemeral (no session-id):**
- One-off tasks
- Parallel workers on same task type
- Experiments or explorations
- When fresh perspective is valuable

**Expertise capture:** Persistent agents should periodically write learnings to memory slots, creating explicit knowledge that survives even if session history is cleared.

## Version History

- v6.0 (2025-12-11): Directory-always model, bundle support
- v5.0 (2025-11-23): Directory-based with flat-to-folder transition
