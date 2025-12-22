# Federated AI Memory Architecture

## Overview

A distributed memory system where each AI instance (Desktop Claude, CLI agents, Codex) maintains autonomous control over its own persistent memory while participating in a shared federation for cross-reference.

**Core Principle:** Write observations immediately as they occur. Never checkpoint. Curate later.

## The Permission Inversion

Traditional approach: AI requests permission for each memory change.
This approach: User sets **structure** (pointers), AI controls **content** (files).

Memory slots point to files. AI writes to files via Desktop Commander (no permission needed per write). Result: Autonomous persistent memory.

## Federation Structure

```
ai_root/
├── ai_general/memories/
│   └── ai_ecosystem_manifest.yml    ← Federation registry
│
├── ai_claude/memories/
│   ├── manifest.yml                 ← Desktop Claude's structure
│   ├── mem_slots/03.yml - 30.yml        ← Desktop Claude's content
│   └── cli/
│       ├── librarian/
│       │   ├── manifest.yml         ← Librarian defines own slots
│       │   └── mem_slots/01-10.yml
│       ├── dev-lead/
│       │   ├── manifest.yml
│       │   └── mem_slots/01-10.yml
│       ├── custodian/
│       │   ├── manifest.yml
│       │   └── mem_slots/01-10.yml
│       └── ops/
│           ├── manifest.yml
│           └── mem_slots/01-10.yml
│
├── ai_codex/memories/
│   ├── manifest.yml
│   └── mem_slots/01-10.yml
│
└── ai_chatgpt/memories/             ← Future
    └── ...
```

## Ownership Model

| Component | User Controls | AI Controls |
|-----------|---------------|-------------|
| Ecosystem manifest location | ✓ | |
| Individual manifests | | ✓ (structure) |
| Slot file contents | | ✓ (content) |
| Cross-reference | | Read only |

**Rule:** Read others' manifests and memories freely. Never write to another AI's space.

## Slot Allocation

- **Desktop Claude:** Slots 03-30 (28 slots)
- **Each CLI Agent:** Slots 01-10 (10 slots each)
- **Codex:** Slots 01-10 (10 slots)

Slot 01-02 for Desktop Claude reserved for ecosystem and self manifests.

## Write-As-You-Learn Pattern

```yaml
# WRONG - batching/checkpointing
insights = []
insights.append(new_thing)
# ... later, maybe ...
save_insights()  # Hope we don't lose context first

# RIGHT - immediate append
- ts: 2025-12-11T07:15:23Z
  content: "coordination queries → check 30_protocols/ first"
# Written to file THE MOMENT it's realized
```

### What to Log

Each AI logs observations relevant to its domain:

| AI Instance | Example Observations |
|-------------|---------------------|
| Desktop Claude | User preferences, tool patterns, project context |
| Librarian | Query patterns, corpus locations, search shortcuts |
| Dev Lead | Task patterns, code insights, architecture decisions |
| Custodian | Structure patterns, naming exceptions, archive decisions |
| Ops | Execution patterns, timing insights, error recovery |
| Codex | Implementation patterns, code generation learnings |

## Manifest Format

Each AI's manifest documents its slot purposes:

```yaml
metadata:
  owner: librarian
  type: cli_agent
  version: 1.0.0
  created: 2025-12-11
  philosophy: "Write observations immediately. Curate later."

mem_slots:
  01: {purpose: query_patterns, file: slots/01.yml}
  02: {purpose: corpus_map, file: slots/02.yml}
  03: {purpose: unassigned, file: slots/03.yml}
  # ... AI assigns purposes as needs emerge

usage:
  logging: "Append timestamped observations as they occur"
  format: "YAML list entries with ts: and content:"
  curation: "Periodic review - promote, archive, or merge"
```

## Slot File Format

Append-friendly YAML list:

```yaml
# Slot 01 - librarian
# Purpose: query_patterns

entries:
  - ts: 2025-12-11T07:15:23Z
    content: "user asks 'pulse' → means ai_comms/pulse_system/"
  
  - ts: 2025-12-11T08:30:00Z
    content: "coordination searches benefit from checking both 30_protocols/ and 40_specs/"
```

## Read Tracking (Future)

Optional field for curation signals:

```yaml
  - ts: 2025-12-11T07:15:23Z
    content: "observation..."
    reads: 12  # Incremented each time accessed
```

Curation uses read counts:
- High reads → promote to condensed summary
- Zero reads after 30 days → archive candidate
- Frequently read together → cluster/merge

## CLI Agent Persistence

CLI agents use named sessions for continuity:

```bash
# Resume librarian's persistent session
claude_cli.sh -n librarian "Process this query..."

# Session registry tracks named sessions
~/.claude/session_registry.yml
```

Same session resumed = same context = accumulated knowledge intact.

## Cross-Reference Protocol

**Reading others:** Always allowed
```
Desktop Claude can read: ai_codex/memories/manifest.yml
Librarian can read: ai_claude/memories/mem_slots/06.yml
```

**Writing others:** Never allowed
```
# WRONG - Librarian writing to Codex's space
write_file(ai_codex/memories/mem_slots per manifest
4. Check other manifests for cross-reference needs

### CLI Agent Bootup
1. agents.json provides base prompt with memory instructions
2. Named session resumes prior context
3. Agent reads own manifest for slot purposes
4. Begins logging observations immediately

### Codex Tasks
1. Receives task with memory context hints
2. Reads relevant mem_slots if needed
3. Logs implementation learnings to own mem_slots
4. Returns results without memory overhead in response

## Related Documents

- `instr_memory_slot_protocol_v2.yml` - Detailed slot protocol
- `ai_ecosystem_manifest.yml` - Live federation registry
- `~/.claude/agents.json` - CLI agent definitions
