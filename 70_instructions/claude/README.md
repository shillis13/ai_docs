# Claude Documentation & Knowledge Manifest

## Purpose

This directory contains Claude-specific instructions and the **knowledge manifest** - a comprehensive index enabling efficient documentation loading.

## Key Files

| File | Purpose |
|------|---------|
| `_knowledge_manifest.yml` | Master index of ALL docs with load tiers |
| `_claude_instructions_manifest.yml` | Instructions-only subset |
| `_registry_instructions_latest.yml` | Legacy detailed instruction index |

## How _knowledge_manifest.yml Was Created

### Generator Script

Location: `ai_general/scripts/generate_knowledge_manifest.py`

```bash
cd ~/Documents/AI/ai_root
python ai_general/scripts/generate_knowledge_manifest.py
```

### What It Does

1. **Scans** all numbered directories (10-70) in `ai_general/docs/`
2. **Extracts metadata** from each `.yml`, `.md`, `.yaml` file
3. **Estimates tokens** (~4 chars/token)
4. **Assigns load tiers** based on:
   - Directory number (lower = more foundational)
   - File metadata `load_priority` if present
   - Default: `topic` tier
5. **Detects dependencies** via keyword matching in content
6. **Outputs** structured YAML manifest

### Load Tier Assignment

```
auto   → Core operational docs (operating principles, context rules)
topic  → Architecture, active instructions, protocols
demand → Specs, schemas, playbooks (reference material)
```

### Dependency Detection

The generator scans file content for references to other files:
- `see:` pointers
- `REF:` pointers  
- `depends_on:` metadata
- Filename mentions

Dependencies are listed so loaders know what else to pull.

## Condensation Process (Dec 2025)

### Method

1. **Parallel CLI workers** - 7 Claude CLI instances, one per directory
2. **Iterative validation** - Each worker called Codex to verify no operational info lost
3. **AT self-wake monitoring** - Desktop Claude checked progress every 5 minutes

### Results

- 54 files condensed
- 72% line reduction (12,773 → 3,658 lines)
- ~71,000 tokens saved
- All validated for operational fidelity

### Playbook

See: `60_playbooks/playbook_parallel_orchestration.yml`
Case study: `60_playbooks/parallel_orchestration_case_study.yml`

## Updating the Manifest

After adding/modifying documentation:

```bash
cd ~/Documents/AI/ai_root
python ai_general/scripts/generate_knowledge_manifest.py
```

The generator will:
- Find new files
- Update token estimates
- Recalculate dependencies
- Preserve manual tier overrides (if in file metadata)

## Project File Usage

Load `_knowledge_manifest.yml` as a Claude Project file to give Claude:
- Awareness of all available documentation
- Proper load tier guidance
- Dependency chains for complete context
- Token budget visibility

## Related

- `../README.md` - Documentation system overview
- `30_protocols/protocol_condensed_resolution_latest.yml` - How pointers resolve
- `ai_general/scripts/generate_knowledge_manifest.py` - Generator source
