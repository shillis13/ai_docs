# Protocol: Condensed File Resolution

**Version:** 1.0
**Purpose:** Efficient resolution of REF:/see: pointers using condensed versions when available

## Resolution Algorithm

**Trigger:** Any REF: or see: pointer to .yml or .md file

### Steps

1. **Parse Target:** Extract target path from pointer
2. **Check Condensed:** Check if `{basename}.condensed.yml` exists in same directory
   - Check locations:
     - `{directory}/{basename}.condensed.yml`
     - `{directory}/condensed/{basename}.condensed.yml`
3. **Load:**
   - Condensed exists → Load condensed version
   - Condensed missing → Load original file

## Rationale

### Why Condensed First
- Reduces token consumption by 60-80%
- Faster context loading
- Preserves essential semantics

### When Original Needed
- Deep implementation details required
- Condensed version missing or outdated
- Explicit request: `REF:full:` or `see:full:`

## Examples

### Standard Resolution
| Element | Value |
|---------|-------|
| Pointer | `REF: protocol_taskCoordination_latest.yml` |
| Check | `protocol_taskCoordination_latest.condensed.yml` exists? |
| If Yes | Load condensed version |
| If No | Load original |

### Explicit Full
| Element | Value |
|---------|-------|
| Pointer | `REF:full: protocol_taskCoordination_latest.yml` |
| Action | Always load original, skip condensed check |

### Subdirectory Condensed
| Element | Value |
|---------|-------|
| Pointer | `see: 30_protocols/protocol_ai_chat_orchestration_latest.yml` |
| Check Paths | `30_protocols/protocol_ai_chat_orchestration_latest.condensed.yml` |
| | `30_protocols/condensed/protocol_ai_chat_orchestration_latest.condensed.yml` |

## Edge Cases

| Case | Handling |
|------|----------|
| Symlinks | Resolve symlink first, then check for condensed of target |
| Circular refs | Track visited files, skip already-loaded |
| Version mismatch | Condensed `metadata.source_version` must match original |
| Missing condensed | Silent fallback to original (not an error) |

## Validation

### Condensed Must Have
- `metadata.source_file`: original filename
- `metadata.condensed_date`: ISO timestamp

### Staleness Check
Compare `condensed_date` to original file mtime
