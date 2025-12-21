# Document Type Taxonomy

**Version:** 1.0.0  
**Status:** CANONICAL  
**Location:** `ai_general/docs/10_architecture/`  
**Created:** 2025-11-29  
**Graduated:** 2025-11-29  

---

## Hierarchy

| Level | Type         | Purpose                                           | Example                          |
|:-----:|:-------------|:--------------------------------------------------|:---------------------------------|
|   1   | Architecture | WHY — design rationale, vision, layer models      | `architectural_layer_model.yml`  |
|   2   | Registry     | WHAT EXISTS — inventories, catalogs, indexes      | `INVENTORY.md`                   |
|   3   | Protocol     | HOW IT WORKS — process flows, lifecycles          | `protocol_taskCoordination_v5.yml` |
|   4   | Spec         | HOW IT WORKS — interface contracts, APIs          | `spec_ai_message_sender_v1.1.yml` |
|   5   | Schema       | HOW IT WORKS — data structures, file formats      | `schema_taskFile_v1.yml`         |
|   6   | Playbook     | WHAT TO DO — platform-agnostic operations         | `maintenance_playbook.md`        |
|   7   | Instruction  | HOW TO DO IT — platform-specific implementation   | `instr_claude_environment.md`    |
|   8   | Quick Ref    | CHEAT SHEET — condensed reference                 | `QUICK_REFERENCE.md`             |

## Non-Hierarchical Types

| Type      | Purpose                                        | Location             |
|:----------|:-----------------------------------------------|:---------------------|
| Data      | Runtime artifacts, working files               | `data/`              |
| Knowledge | Stable reference facts, learnings, context     | `ai_memories/50_digests/knowledge/` |
| Prompts   | Triggers for AI action                         | `prompts/`           |
| Templates | Scaffolding for new artifacts                  | `templates/`         |
| Manifests | Deployment configs per AI                      | `manifests/`         |

## Key Distinctions

### Protocol vs Spec vs Schema

| Type     | Defines                | Example                                              |
|:---------|:-----------------------|:-----------------------------------------------------|
| Protocol | Process flow           | "Tasks move `to_execute/` → `in_progress/` → `completed/`" |
| Spec     | Interface contract     | "Message sender accepts X params, returns Y structure" |
| Schema   | File format            | "Task file has these fields with these types"        |

### Playbook vs Instruction

| Type        | Scope             | Example                                    |
|:------------|:------------------|:-------------------------------------------|
| Playbook    | Platform-agnostic | "Check queues, review stale tasks"         |
| Instruction | Platform-specific | "Claude: use Desktop Commander to `ls`..." |

**Analogy:** Playbook is the interface; Instruction is the implementation per platform.

## Implemented Directory Structure

```
ai_general/
├── docs/                          # Canonical documentation
│   ├── 10_architecture/           # Level 1 - WHY
│   ├── 20_registries/             # Level 2 - WHAT EXISTS
│   ├── 30_protocols/              # Level 3 - HOW IT WORKS (process)
│   ├── 40_specs/                  # Level 4 - HOW IT WORKS (interface)
│   ├── 50_schemas/                # Level 5 - HOW IT WORKS (structure)
│   ├── 60_playbooks/              # Level 6 - WHAT TO DO
│   ├── 70_instructions/           # Level 7 - HOW TO DO IT
│   │   ├── claude/
│   │   └── chatgpt/
│   ├── 80_quickref/               # Level 8 - CHEAT SHEETS
│   └── 90_drafts/                 # Staging area
│
├── manifests/                     # Deployment configs per AI
├── prompts/                       # Triggers for AI action
├── templates/                     # Scaffolding for new artifacts
├── scripts/                       # Automation tools
├── projects/                      # Active development work
├── research_and_reports/
├── logs/
├── data/                          # Runtime data
└── tmp/
```

## Design Decisions

| Decision                              | Rationale                                          |
|:--------------------------------------|:---------------------------------------------------|
| Numbered prefixes (10–90)             | Makes hierarchy explicit in filesystem listings    |
| Quick Refs in separate `80_quickref/` | Standalone access, not buried with parent doc type |
| Prompts & Templates as peers to docs/ | First-class artifacts, not hidden under `data/`    |
| Knowledge in ai_memories              | Consolidates all memory/digest content             |
| `90_drafts/` as staging area          | Artifacts reviewed before permanent filing         |
