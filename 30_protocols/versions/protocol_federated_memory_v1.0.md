# Federated AI Memory Protocol

**Version:** 1.0.0
**Created:** 2025-12-11
**Status:** active
**Priority:** high

## Purpose

Defines how multiple AI instances (Desktop Claude, CLI agents, Codex) maintain and share persistent memory across sessions.

## Core Principles

| Principle | Description |
|-----------|-------------|
| Write Immediate | Log observations as they occur, never checkpoint |
| Own Domain | Each AI controls its own memory structure and content |
| Read Freely | Any AI can read any other's memories |
| Write Never | Never write to another AI's memory space |
| Curate Async | Periodic review to promote/archive/merge |

## Architecture

**Ecosystem Registry:** `ai_general/memories/ai_ecosystem_manifest.yml`

### Participants

| Participant | Manifest | Slots | Role |
|-------------|----------|-------|------|
| Claude Desktop | `ai_claude/memories/manifest.yml` | 03.yml - 30.yml | Primary orchestrator, strategic coordination |
| CLI Agents | `ai_claude/memories/cli/{agent}/` | 10 slots per agent | Named sessions (-n flag) |
| Codex CLI | `ai_codex/memories/manifest.yml` | 01.yml - 10.yml | Synchronous task execution |

**CLI Agents:** librarian, dev-lead, custodian, ops

## Slot File Format

```yaml
# Slot {NN} - purpose defined in manifest.yml
entries:
  - ts: 2025-12-11T07:15:23Z
    content: "observation text"
    reads: 0  # optional, increment on access
  - ts: ...
```

### Rules
- Always append, never rewrite
- Timestamp every entry (ISO 8601)
- Keep entries atomic (one observation per entry)
- `reads` field optional, add when accessed

## Manifest Structure

### Required Fields
- `metadata.owner`
- `metadata.version`
- `metadata.philosophy`
- `slots` (01-NN with purpose)
- `usage_patterns`

### Example
```yaml
metadata:
  owner: librarian
  version: 1.0.0
  philosophy: "Write observations immediately. Curate later."
slots:
  01: {purpose: query_patterns}
  02: {purpose: corpus_map}
  03: {purpose: unassigned}
usage_patterns:
  logging: "Append observations as they occur"
  format: "YAML list with timestamped entries"
```

## Session Patterns

### CLI Agents

**Singleton Model:**
CLI agents use named sessions for persistence:
```bash
claude_cli.sh -n librarian "Process query..."
```
Same session resumes → accumulated context preserved

**First Use:**
1. Session starts fresh
2. Agent reads own manifest.yml
3. Agent decides slot purposes based on work
4. Updates manifest with slot assignments
5. Begins logging to slots

**Subsequent Use:**
1. Named session resumes (context intact)
2. Agent has learned context from prior work
3. Continues logging new observations
4. May reorganize slots as understanding grows

### Desktop Claude

**Bootup:**
1. Load ecosystem manifest (slot 1 pointer)
2. Load own manifest (slot 2 pointer)
3. Load PRIORITY:1 slots as needed

**Runtime:**
- Write observations to slots immediately
- Update manifest when slot purposes change
- Cross-reference other AI memories (read-only)

## Cross-Reference Protocol

### Reading Others
**Allowed:** Always

**Process:**
1. Read target's manifest first (understand structure)
2. Load relevant slots based on manifest
3. Use information, cite source if sharing

**Example:**
Desktop Claude needs Librarian's corpus map:
- Read `ai_claude/memories/cli/librarian/manifest.yml`
- Find: slot 02 is corpus_map
- Read `ai_claude/memories/cli/librarian/mem_slots/02.yml`

### Sharing Insights
**Method:** Coordination task, not direct write

**Example:**
Librarian discovers something Dev Lead should know:
- Creates coordination task with insight
- Dev Lead processes task, logs to OWN memory

## Logging Examples

### Good
```yaml
- ts: 2025-12-11T07:15:23Z
  content: "Query 'pulse system' → found in ai_comms/, not ai_memories/"
- ts: 2025-12-11T07:16:01Z
  content: "User prefers condensed YAML over prose - 75% token savings"
```

### Bad
```yaml
- "Learned some things today"  # No timestamp, vague
- {batch: [item1, item2, item3]}  # Batched, not immediate
```

## Curation Signals

| Signal | Action |
|--------|--------|
| Promote | High reads → core knowledge, move to summary |
| Archive | Zero reads after 30 days → archive candidate |
| Merge | Frequently read together → combine entries |
| Expire | Contradicted by newer observation → mark stale |

## Implementation Status

### Complete
- Directory structure for all participants
- Manifest files with slot definitions
- Empty slot files ready for use
- Ecosystem registry updated
- agents.json with logging instructions

### Pending
- Codex CLI instructions update
- Read-tracking implementation
- Curation automation
- Cross-reference tooling
