# Working Memory Architecture

```yaml
metadata:
  title: Working Memory Architecture
  version: 2.0.0
  created: 2026-03-16
  supersedes: federated_memory_architecture_v1.0.md
  status: active
```

## Overview

A shared persistent memory system for all AI participants in ai_root. All AIs
read and write to a single shared location. Memory survives across sessions and
is accessible to any AI with filesystem access.

**Core Principle:** Write observations immediately as they occur. Never checkpoint. Curate later.

## The Permission Inversion

Traditional: AI requests permission for each memory change.  
This approach: User sets **structure** (memory slot pointers), AI controls **content** (slot files).

Memory slots point to files. AIs write to files via Desktop Commander (no per-write
permission). Result: Autonomous persistent memory for all participants.

## Structure

```
ai_root/
└── ai_memories/
    └── 80_working_memory/
        ├── manifest.yml          ← Slot registry (what each slot means)
        ├── 01.yml                ← Ecosystem registry pointer (reserved)
        ├── 03.yml                ← User model
        ├── 04.yml                ← Communication patterns
        ├── 05.yml                ← Tool/pattern discoveries
        ├── 06.yml                ← Current context notes
        ├── 07.yml                ← Cross-AI notes
        ├── 08.yml                ← Learnings worth preserving
        ├── 09.yml                ← Project history
        ├── 11.yml                ← Novel phrasing / originality evidence
        └── _archive_YYYYMM/      ← Historical entries
```

## Ownership Model

| Component | User Controls | AI Controls |
|---|---|---|
| Memory slot pointers (Anthropic system) | ✓ | |
| Slot file contents | | ✓ (all AIs) |
| Manifest declarations | | ✓ (all AIs) |

**Rule:** Any AI may read any slot. No AI writes to a slot declared as
owned by another AI without coordination.


## Slot Allocation

- **Slots 01–02:** Reserved (ecosystem registry, manifest self-reference)
- **Slots 03–11:** Active (see manifest.yml for current assignments)
- **Slots 12–27:** Reserved for expansion
- **Slots 28–30:** Retired (previously web/iOS inbox — superseded by message insert mechanism)

Any AI may claim a slot in the 12–27 range by declaring it in `manifest.yml` first.

## Write-As-You-Learn Pattern

```yaml
# WRONG — batching/checkpointing
insights = []
insights.append(new_thing)
# ... later, maybe ...
save_insights()  # Context may be gone by then

# RIGHT — immediate append to slot file
- ts: 2026-03-16T14:30:00Z
  content: "observation text"
# Written THE MOMENT it's realized
```

## What Each AI Logs

| AI | Example Observations |
|---|---|
| Desktop Claude | User preferences, tool patterns, project context, relationship notes |
| CLI agents (librarian, dev-lead, custodian) | Role-specific patterns, task learnings |
| Codex | Implementation patterns, code generation learnings |
| Web/iOS Claude | Via message insert blocks — harvested by Desktop/CLI |

## Slot File Format

```yaml
# slot NN — name defined in manifest.yml
- ts: 2026-03-16T14:30:00Z
  content: "brief observation"

# Full form when context matters:
- ts: 2026-03-16T14:30:00Z
  type: learning | observation | pattern | shortcut | issue
  content: "the insight"
  context: "what prompted this"
  reads: 0
```

Slots are append-only by default. Curation (removing stale entries) is permitted
but infrequent.

## Cross-Platform Write Paths

| Platform | Mechanism |
|---|---|
| Desktop / CLI | `edit_block` append to slot file directly |
| Web / iOS | `<<<INSERT type=memory_v1.1>>>` inline block → harvested by cron |

## Cross-Reference Protocol

- **Reading others:** Always allowed — read any slot freely
- **Writing others:** Never unilaterally — use task coordination to pass information
- **Ecosystem registry:** `ai_general/memories/ai_ecosystem_manifest.yml` — federation-wide AI registry

## Related Documents

- `manifest.yml` — live slot registry
- `instr_working_memory_protocol_v3.0.md` — operational instructions
- `instr_working_memory_v2.0.md` — condensed quick-reference
- `spec_message_insert` — web/iOS write path specification
- `federated_memory_architecture_v1.0.md` → archived (per-agent subdirectory model, superseded)
