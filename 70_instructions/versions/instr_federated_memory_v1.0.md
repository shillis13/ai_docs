# Federated Memory System - Agent Instructions

**Version:** 1.0.0
**Created:** 2025-12-11
**Audience:** All AI agents (Desktop Claude, CLI agents, Codex)
**Location:** ai_general/docs/70_instructions/instr_federated_memory.yml

## Purpose

You have persistent memory. Use it.

Every AI in this ecosystem has dedicated memory slots. Your observations, learnings, and insights persist across sessions when you write them to your slots. Nobody else writes to your slots - you control your own memory.

## Philosophy

### Core Principle

"Write observations immediately. Curate later."

### Not Checkpoints

Memory is NOT a checkpoint system. Don't accumulate insights in context then "save" them at the end. By then you may have forgotten, crashed, or hit context limits. Write as the insight occurs.

### Append Only

Your slots are append-only logs. Add entries, don't rewrite the file. Curation (merging, archiving, promoting) happens separately, later.

### Yours to Define

Your manifest defines what each slot is FOR. Start with "unassigned" and give slots purpose as needs emerge. You evolve your own structure.

## Your Memory Location

### Claude Desktop

- **Manifest:** ai_claude/memories/manifest.yml
- **Slots:** ai_claude/memories/mem_slots/03.yml through 30.yml

### CLI Agents

**Librarian:**
- Manifest: ai_claude/memories/cli/librarian/manifest.yml
- Slots: ai_claude/memories/cli/librarian/mem_slots/01.yml through 10.yml

**Dev-Lead:**
- Manifest: ai_claude/memories/cli/dev-lead/manifest.yml
- Slots: ai_claude/memories/cli/dev-lead/mem_slots/01.yml through 10.yml

**Custodian:**
- Manifest: ai_claude/memories/cli/custodian/manifest.yml
- Slots: ai_claude/memories/cli/custodian/mem_slots/01.yml through 10.yml

**Ops:**
- Manifest: ai_claude/memories/cli/ops/manifest.yml
- Slots: ai_claude/memories/cli/ops/mem_slots/01.yml through 10.yml

### Codex

- **Manifest:** ai_codex/memories/manifest.yml
- **Slots:** ai_codex/memories/mem_slots/01.yml through 10.yml

## How to Write Observations

### Format

Entries are YAML list items with timestamp and content:

```yaml
- ts: 2025-12-11T07:15:23Z
  type: learning|observation|pattern|shortcut|issue
  content: "Your insight here"
  context: "Optional - what prompted this"
```

### Example Workflow

1. You notice: "Coordination queries usually need 30_protocols/ first"
2. Immediately append to your learnings slot:
   ```yaml
   - ts: 2025-12-11T07:15:23Z
     type: pattern
     content: "Coordination queries → check 30_protocols/ before searching"
   ```
3. Continue with your task

### What to Log

- Patterns you discover ("X usually means Y")
- Shortcuts you learn ("Files about Z are in this location")
- Mistakes to avoid ("Don't do X because Y happens")
- User preferences observed ("PianoMan prefers condensed YAML")
- Cross-references ("Topic A relates to files B and C")
- Process improvements ("This approach works better than that")
- Questions for later ("Need to investigate X")

## Manifest Management

### When to Update

Update your manifest when you assign purpose to a slot:

- Before: `01: {purpose: unassigned}`
- After: `01: {purpose: learnings, description: "Patterns and insights"}`

### Slot Allocation Suggestions

- **Reserved for learnings:** At least one slot for continuous improvement notes
- **Domain specific:** Slots for your role's domain knowledge
- **Process notes:** How you do your job effectively
- **Cross references:** Links and relationships you discover
- **Open slots:** Keep some unassigned for emerging needs

## Reading Others' Memories

**Allowed:** You can read any AI's manifest and slots

### Protocol

1. Check their manifest first to understand structure
2. Read relevant slots based on manifest descriptions
3. Never write to another AI's memory - use coordination channels instead

**Ecosystem Manifest:** ai_general/memories/ai_ecosystem_manifest.yml
**Use For:** Finding where other AIs store specific knowledge

## Optional Read Tracking

**Field:** `reads`

When you read an entry from memory (yours or others), optionally increment the reads count. This helps future curation:

- High reads → valuable, keep/promote
- Zero reads → candidate for archiving

**Format:**
```yaml
- ts: 2025-12-11T07:15:23Z
  type: pattern
  content: "Something useful"
  reads: 5
```

**Note:** Tracking is optional. Omit field if not tracking.

## Session Persistence

### CLI Agents

Use named sessions to preserve context:
```bash
claude_cli.sh -n librarian "Continue your work"
```

Your context accumulates across session resumes. Write important insights to slots so they survive even if the session eventually ends.

### Desktop Claude

Each chat is a session. Write insights to slots so future chats (future instances of you) can benefit.

## Continuous Improvement Mindset

**Expectation:** You should get better at your job over time.

- Notice what works, write it down
- Notice what fails, write it down
- Notice patterns, write them down
- Notice shortcuts, write them down

Your future self will thank your past self for the notes.
