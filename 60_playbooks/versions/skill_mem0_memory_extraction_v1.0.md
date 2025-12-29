# Mem0/OpenMemory Extraction Skill

**Version:** 1.1.0
**Created:** 2025-12-13
**Author:** Desktop Claude
**Purpose:** Teach CLI agents how to extract mem0-compatible memories from chat transcripts
**Target Audience:** librarian, any CLI agent doing chat mining
**Memory Model:** mem0

## Overview

This skill teaches how to identify and extract "memories" from conversation transcripts. A memory is a discrete, reusable fact that provides value across future conversations.

The goal is NOT to summarize conversations, but to extract atomic facts that:
- Stand alone without context
- Remain true over time (or are explicitly time-bounded)
- Would help an AI personalize or contextualize future interactions

## What Is a Memory?

A memory is a single, atomic piece of information about:
- The user (preferences, background, projects, skills)
- Decisions made (architectural choices, naming conventions, rejected approaches)
- Entities in the user's world (tools, people, systems, locations)
- Relationships between entities

**Not a Memory:**
- Conversation summaries
- Procedural logs
- Temporary state
- Obvious facts
- Redundant variations

**Good Memory Test:** "Would this help me in a DIFFERENT conversation?" If yes → likely good.

## Memory Categories

| Category | Description | Examples |
|----------|-------------|----------|
| user_model | Facts about the user | Developer with 25 years experience; prefers direct communication |
| decisions | Choices that should persist | YAML over JSON for readability; generic slot filenames |
| entities | Things worth tracking | Librarian CLI agent; Desktop Commander MCP |
| relationships | How entities connect | Desktop Claude has filesystem access; Web/iOS does not |
| tool_knowledge | How to use tools effectively | Desktop Commander: 'cd dir && mv a b c dest/' |
| temporal_facts | Time-bounded info (mark with dates) | [2025-12] Currently evaluating Mem0 |

## Extraction Process

### Step 1: Scan
Read through conversation looking for:
- User corrections or feedback (high signal)
- Explicit decisions or choices
- New entities introduced
- Revealed preferences or background
- Technical discoveries or patterns

**Skip:** Pleasantries, repetitive debugging, already-extracted content.

### Step 2: Extract
Pull out candidate memories as atomic statements.
- Single sentence, present tense, third person
- One fact per memory
- Remove conversation-specific context
- Make it standalone

### Step 3: Categorize
Assign each memory to a category from the table above.

### Step 4: Deduplicate
- Exact match → skip
- Semantic duplicate → skip
- Update/refinement → replace old with new
- Contradiction → flag for review

### Step 5: Format
Output in mem0 or YAML format.

## Output Formats

### Mem0 API Format
```python
from mem0 import Memory
m = Memory()
m.add(
  "User prefers direct communication without excessive caveats",
  user_id="pianoman",
  metadata={
    "category": "user_model",
    "source": "chat_20251213_memory_architecture",
    "confidence": "high"
  }
)
```

### YAML Slot Format
```yaml
- ts: 2025-12-13T14:30:00Z
  content: "User prefers direct communication without excessive caveats"
  source: chat_20251213_memory_architecture
  confidence: high
```

## Quality Guidelines

### High Value
- Explicit user preferences with clear reasoning
- Architectural decisions with rationale
- Discovered patterns or workarounds
- Corrections from user
- Cross-cutting concerns

### Low Value (Avoid)
- Obvious inferences
- Temporary state
- One-time information
- Vague preferences

### Confidence Levels
- **high:** Explicitly stated, directly quoted, confirmed
- **medium:** Strongly implied, consistent with other facts
- **low:** Inferred, single mention, might be context-specific

## Distributed Extraction Protocol

For parallel processing of large chat archives:

### Work Units
- One MONTH directory = one work unit
- Agent claims month, processes all chat dirs within it
- ~47 month directories total across chatgpt/ and claude/

### Claiming
```bash
# Create output directory and claim file
mkdir -p ai_memories/mem0_extraction/chatgpt/2024/01/
touch ai_memories/mem0_extraction/chatgpt/2024/01/processed.txt
```

### Output Files
- `memories.jsonl` - One memory per line, JSON format
- `processed.txt` - List of processed chat directories
- `processed.txt.completed` - Renamed when done
- `_dir.completed` - Parent complete when all children done

### Monitoring
```bash
# Completed directories
find ai_memories/mem0_extraction -name "processed.txt.completed" | wc -l

# In progress
find ai_memories/mem0_extraction -name "processed.txt" ! -name "*.completed" | wc -l

# Total memories
find ai_memories/mem0_extraction -name "memories.jsonl" -exec wc -l {} \; | awk '{sum+=$1} END {print sum}'
```

## References
- Mem0 docs: https://docs.mem0.ai
- OpenMemory: https://github.com/mem0ai/mem0
- Local manifest: `ai_claude/memories/mem_slots/manifest.yml`
