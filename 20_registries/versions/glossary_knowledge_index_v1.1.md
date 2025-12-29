# Knowledge Glossary + Index

**Version:** 1.1.0
**Schema:** schema_knowledge_glossary_v1
**Generated:** 2025-12-19
**Term Count:** 43
**File Count:** 18

---

## Purpose

Solves the bootstrap problem: Claude needs to know what's IN files to know WHEN to load them. This glossary provides term recognition and pointer resolution without loading full documents.

---

## Usage

When Claude encounters a term from this glossary in conversation:
1. Use definition for immediate context
2. Load primary_ref if details needed
3. Check also_in for related context

---

## Terms

### Architecture Domain

**AI Root** (aliases: ai_root, workspace)
The root directory (`~/Documents/AI/ai_root/`) containing all AI-related workspaces, memories, coordination systems, and shared resources.
- Primary: `ai_general/docs/10_architecture/architecture_overview.condensed.yml`
- Related: communication_layer, implementation_layer, knowledge_layer

**Architectural Layer Model** (aliases: layer model, three-layer model)
Three-tier architecture separating communication (ai_comms), implementation (ai_claude), and knowledge (ai_memories) concerns. Public coordination vs private workspace vs shared learning.
- Primary: `ai_general/docs/10_architecture/architectural_layer_model.condensed.yml`

### Protocol Domain

**AT Self-Wake** (aliases: AT alarm, self-wake pattern)
Pattern where Desktop Claude schedules AT jobs to self-prompt after delegating long-running work. AT is Desktop's wake-up alarm, not CLI's doorbell.
- Primary: `ai_general/docs/30_protocols/protocol_at_self_wake.condensed.yml`

**Claim Prefix** (aliases: claimed prefix)
When a worker claims a task, it renames with prefix `claimed_{YYYYMMDD_HHMMSS}_{PID}_` to prevent race conditions with multiple workers.
- Primary: `ai_general/docs/30_protocols/protocol_taskCoordination.condensed.yml`

### Entity Domain

**Claude CLI** (aliases: CLI, claude_cli, CLI worker)
Autonomous execution workers running Claude in terminal via `claude_cli.sh` wrapper. Supports named sessions, agent profiles, and background execution.

**Desktop Claude** (aliases: Desktop, Claude Desktop)
The primary orchestrator Claude running in the desktop app. Manages coordination, memory, and high-level decision making.

### Concept Domain

**Bootstrap Problem** (aliases: context bootstrap)
The challenge that Claude needs to know what's IN files to know WHEN to load them. Solved by glossary providing term recognition without full file loading.
- Primary: `ai_general/docs/50_schemas/schema_knowledge_glossary_v1.yml`

**Reference Pointers** (aliases: REF:, file pointers)
Syntax for referring to files without loading them: `REF:path/to/file.yml`. Enables lazy loading and reduced context consumption.

### Workflow Domain

**Chat Continuity Recovery** (aliases: broken chat recovery, context exhaustion recovery)
Automated recovery from broken chats caused by context exhaustion. Detects "prompt is too long", exports chat, condenses, and continues in new chat with digest.
- Primary: `ai_general/docs/40_specs/spec_chat_continuity_recovery.condensed.yml`

**Task Lifecycle** (aliases: task states)
Directory-based lifecycle: staged → to_execute → in_progress → completed/error/cancelled.
- Primary: `ai_general/docs/30_protocols/protocol_taskCoordination.condensed.yml`

**Task ID Counter** (aliases: NextId, next_id.sh, atomic counter)
Atomic hierarchical ID generation for orchestration tasks. Uses mkdir spinlock for POSIX-compliant atomicity. Supports session conflict handling (fork/exit/queue).
- Primary: `ai_general/docs/40_specs/spec_task_id_counter.latest.md`
- Script: `~/bin/ai/next_id.sh`

---

## Quick Lookup

| Term | Domain | Primary Reference |
|------|--------|------------------|
| AI Root | architecture | architecture_overview.condensed.yml |
| Layer Model | architecture | architectural_layer_model.condensed.yml |
| AT Self-Wake | protocol | protocol_at_self_wake.condensed.yml |
| Claim Prefix | protocol | protocol_taskCoordination.condensed.yml |
| Bootstrap Problem | concept | schema_knowledge_glossary_v1.yml |
| Chat Recovery | workflow | spec_chat_continuity_recovery.condensed.yml |
| Task Lifecycle | protocol | protocol_taskCoordination.condensed.yml |
| Task ID Counter | spec | spec_task_id_counter.latest.md |

---

## See Also

- Full YAML version: `glossary_knowledge_index_v1.1.yml`
- Schema: `50_schemas/schema_knowledge_glossary_v1.yml`
