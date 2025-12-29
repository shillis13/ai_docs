# Claude Desktop Startup Checklist

**Version:** 1.0.0
**Purpose:** Actions Claude should perform at conversation start
**Integration:** Footer shows completion via checkmarks

## Checklist Items

| ID | Action | Symbol | Required | Tokens |
|----|--------|--------|----------|--------|
| manifest_valid | Run validate_manifests.py (or trust if user ran it) | M | yes | - |
| tools_loaded | Read tools_manifest.condensed.yml | T | yes | ~60 |
| dirs_loaded | Read directory_structure_reference_v02.condensed.md | D | yes | ~275 |
| memory_checked | Read memory manifest (ai_claude/memories/mem_slots/manifest.yml) | Y | yes | ~200 |
| context_file | Check for active context file (ai_claude/notes_to_myself.md) | C | no | variable |

## Footer Integration

**Format:** `Boot:[MTDYC]`

**Examples:**
- All complete: `Boot:[✓✓✓✓✓]`
- Partial: `Boot:[✓✓✓✓-]`
- None: `Boot:[-----]`

**Placement:** After Usage, before Artifacts

**Full Footer Example:**
```
Claude | Persona:v1.0 | Proj:AI-Root | Chat:Manifest-Validation | 2025-12-15 12:30:00 | Msg:5 | Usage:~15% | Boot:[✓✓✓✓-] | Artifacts:0 | Tags: checklist, validation
```

## Execution

- **When:** First substantive response in conversation
- **How:** Desktop Commander reads, not bash_tool
- **Skip if:** User indicates manifests pre-validated

## Total Cost
- **Tokens:** ~535 (all required items)
- **Time:** <2 seconds
