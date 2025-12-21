# Task Coordination Documentation Update Plan

**Date:** 2025-12-13  
**Status:** ✅ COMPLETED  
**Trigger:** New orchestration workflow visualization requiring protocol alignment  
**Resolution:** Option C (Hybrid) selected and implemented

## Decision Made

Before cascading updates, need decision on **which direction to align**:

### Option A: Update Protocol to Match Workflow
Adopt PianoMan's conventions as canonical:
- `$$` in filename for PID claim
- `.response.md` / `.completion.md` distinction
- Symlinks to completed child task dirs
- Nested subtask support

### Option B: Update Workflow to Match Protocol v6
Keep existing protocol conventions:
- `claimed_{timestamp}_{pid}_` prefix on dir
- `response.md` / `response_{seq}_{cli}_{pid}.md`
- No symlinks (copy or reference by path)

### Option C: Hybrid
- Keep v6 claim prefix (more metadata)
- Adopt `.response.md` / `.completion.md` distinction (clearer semantics)
- Add symlink guidance (useful for orchestration)
- Document subtask nesting pattern

---

## Cascading Updates (once direction chosen)

### Tier 1: Core Protocol (update first)
| File | Current State | Needed Update |
|------|---------------|---------------|
| `30_protocols/protocol_taskCoordination_latest.yml` | v6.0 | Add subtask/symlink patterns, response type distinction |
| `50_schemas/schema_taskFile_latest.yml` | v2.0 | Add subtask fields, response file schemas |

### Tier 2: Architecture Docs (update second)
| File | Current State | Needed Update |
|------|---------------|---------------|
| `10_architecture/cli_orchestration_latest.md` | v1.0, references v4/v5 | Align with v6+, add orchestration hierarchy |
| `10_architecture/daemon_architecture_latest.md` | Active | Add orchestration patterns if relevant |

### Tier 3: Operational Docs (update third)
| File | Current State | Needed Update |
|------|---------------|---------------|
| `ai_comms/README.md` | v1.0, references v4.0 | Update to v6+, orchestration patterns |
| `ai_comms/QUICK_REFERENCE.md` | Stale | Complete rewrite for current model |
| `~/.claude/coordination/README.md` | References missing doc | Update to current protocol |

### Tier 4: Memory/Digest Files (update last)
| File | Current State | Needed Update |
|------|---------------|---------------|
| `ai_memories/60_knowledge/coordination_system_v4_digest.md` | v4.0 digest | Archive or create v6 digest |

---

## New Documents to Create

1. **spec_task_orchestration_workflow_ascii.md** ✅ Created
   - Visual reference for orchestration flows

2. **protocol_taskCoordination_v7.md** (if adopting workflow conventions)
   - Incorporate orchestration patterns
   - Response type distinction
   - Symlink guidance
   - Subtask nesting

3. **playbook_multi_level_orchestration.md**
   - Step-by-step guide for setting up orchestration
   - Example task hierarchies
   - Notification patterns

---

## Recommended Approach

1. **You decide** Option A/B/C above
2. **I update** protocol first (single source of truth)
3. **Cascade** to other docs via delegation to Codex/CLI
4. **Validate** with a test orchestration run

Ready to proceed when you indicate direction.
