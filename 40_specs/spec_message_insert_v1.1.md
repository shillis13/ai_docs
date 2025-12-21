# Message Insert Protocol (v1.1)

**Purpose:**  
Message Inserts are structured comment blocks embedded in chat histories, PRs, or files.  
They serve as **machine-readable markers** for:
1. Correlating experiment stages (CTI → run → review → decision) [v1.0]
2. **Capturing memories and insights for cross-session persistence** [v1.1]
3. **Bookmarking navigation points in memory-holder chats** [v1.1]
4. **Recording decisions and open questions** [v1.1]

---

## Changelog from v1.0
- Added new insert types: `BOOKMARK`, `MEMORY`, `DECISION`, `QUESTION`
- Expanded design goals to include memory persistence use cases
- Added guidance for memory-holder chat architecture
- Simplified base fields for non-experiment types

---

## 1. Design Goals
1. **Correlate** related stages of an experiment (CTI creation → run → review → decision).
2. **Persist** metadata in natural conversation flow without external DBs.
3. **Be human-legible yet machine-parsable.**
4. **Require no special permissions**—plain text survives exports.
5. **Support schema evolution** via version numbers.
6. **Enable cross-platform memory capture** (iOS/Web can write; Desktop harvests). [v1.1]
7. **Make memory-holder chats navigable and queryable.** [v1.1]

---

## 2. Core Concepts
| Term | Definition |
|------|-------------|
| **Insert** | A fenced block containing a JSON payload and a small header. |
| **Type** | The semantic purpose of the insert (e.g., CTI_EXPERIMENT, MEMORY, BOOKMARK). |
| **Experiment ID** | Unique key joining all inserts from the same investigation. |
| **Schema Version** | Indicates the JSON field set; parser uses `v=`. |
| **Payload** | Strict JSON (no comments) containing metadata fields. |

---

## 3. Syntax & Grammar

### 3.1 Fence Tokens
```
<<<INSERT type=<TYPE> v=<INT>>>
{ JSON PAYLOAD }
<<<END INSERT>>>
```

- **Header line:** required; contains at least `type` and `v`.
- **Payload:** valid UTF-8 JSON only.
- **End line:** exactly `<<<END INSERT>>>`.
- **Whitespace:** allowed before/after fences but not inside header tokens.

### 3.2 Detection Regex (multiline / dotall)
```
^<<<INSERT\s+type=(\w+)\s+v=(\d+)>>>[\r\n]+([\s\S]*?)^<<<END INSERT>>>$
```

---

## 4. Standard Types

### 4.1 Experiment Types (v1.0)

| Type | Description | Typical Location |
|------|--------------|------------------|
| **CTI_EXPERIMENT** | Defines intent and setup of a Codex task or experiment. | Same message as CTI creation |
| **RUN_METADATA** | Captures runtime info (model, exec time, paths). | After task completion |
| **PR_REVIEW** | Evaluation of Codex outputs or PRs. | Chat review message or PR comment |
| **EXPERIMENT_RESULT** | Final decision and findings summary. | Wrap-up message or post-mortem |

### 4.2 Memory & Navigation Types (v1.1)

| Type | Description | Typical Location |
|------|--------------|------------------|
| **BOOKMARK** | Navigation/resume point in a chat | After significant context loads or topic transitions |
| **MEMORY** | Captured insight, preference, or fact worth preserving | Inline when insight emerges |
| **DECISION** | Conclusion reached with rationale | End of decision-making discussion |
| **QUESTION** | Open question for future consideration | When question is identified but not resolved |

---

## 5. JSON Schemas

### 5.1 Base Fields (Experiment Types)

```json
{
  "experiment_id": "exp-YYYY-MM-DD-<slug>-NNN",
  "project": "string",
  "task_title": "string",
  "mode": "Exploratory|Production",
  "guidance": "L0-L3",
  "ts": "ISO-8601 datetime",
  "author": "string",
  "notes": "optional human comment"
}
```

### 5.2 Base Fields (Memory/Navigation Types) [v1.1]
```json
{
  "ts": "ISO-8601 datetime",
  "author": "string (typically 'Claude' or 'PianoMan')",
  "tags": ["optional", "array", "for", "searchability"]
}
```

Note: Memory types use a simpler base schema since they don't require experiment correlation.

---

## 6. Extended Fields by Type

### 6.1 Experiment Types (v1.0)

#### `CTI_EXPERIMENT`
```json
{
  "biases": ["string"],
  "hypothesis": "string",
  "knobs": ["string"],
  "metrics": ["string"],
  "links": ["string"]
}
```

#### `RUN_METADATA`
```json
{
  "task_version": "string",
  "model_hint": "string",
  "exec_time_sec": 0,
  "artifacts": ["path"]
}
```

#### `PR_REVIEW`
```json
{
  "variants_reviewed": ["A", "B"],
  "scores": { "A": {"overall": 0, "breakdown": {"correctness": 5}} },
  "winner": "A",
  "recommendations": ["string"],
  "links": ["url"]
}
```

#### `EXPERIMENT_RESULT`
```json
{
  "decision": "string",
  "key_findings": ["string"],
  "next_actions": ["string"]
}
```

### 6.2 Memory & Navigation Types (v1.1)

#### `BOOKMARK`
```json
{
  "label": "string - short identifier",
  "context": "string - what was loaded or discussed up to this point",
  "resume_hint": "string - guidance for resuming from this point",
  "message_count": "optional int - messages in chat up to this point"
}
```

#### `MEMORY`
```json
{
  "topic": "string - primary subject",
  "content": "string - the insight, fact, or preference",
  "source": "string - conversation, observation, explicit statement",
  "importance": "low|medium|high|critical",
  "related": ["optional", "array", "of", "related", "topics"]
}
```

#### `DECISION`
```json
{
  "topic": "string - what was decided",
  "decision": "string - the conclusion reached",
  "rationale": ["array", "of", "reasoning", "points"],
  "alternatives_considered": ["optional", "array"],
  "related": ["optional", "related", "topics", "or", "decisions"]
}
```

#### `QUESTION`
```json
{
  "question": "string - the open question",
  "context": "string - why this question matters",
  "blocked_by": "optional string - what's needed to answer it",
  "priority": "low|medium|high",
  "related": ["optional", "related", "topics"]
}
```

---

## 7. Memory-Holder Chat Architecture [v1.1]

### 7.1 Concept
Designated chats can serve as **domain expert repositories**—Claude instances with accumulated deep context that can be:
- Consulted for analysis (not just retrieval)
- Queried about patterns across their memories
- Asked to synthesize or evaluate new information

### 7.2 Best Practices
1. **Tag memory-holder chats** with a clear purpose in the first message
2. **Use BOOKMARK inserts** after significant context loads
3. **Use MEMORY inserts** for key insights worth preserving
4. **Navigate via bookmarks** when resuming after context limits

### 7.3 Workflow
1. Create designated chat with purpose statement
2. Load relevant context (transcripts, documents, prior decisions)
3. Insert BOOKMARK after load
4. Engage in analysis/discussion
5. Capture significant outputs with MEMORY/DECISION inserts
6. Future sessions can search for inserts or resume from bookmarks

---

## 8. Platform-Specific Guidance [v1.1]

| Platform | Capabilities | Insert Strategy |
|----------|--------------|-----------------|
| Desktop | Full filesystem access | Harvest inserts → write to memory stores |
| Web UI | Chat only, no filesystem | Write inserts inline; Desktop harvests later |
| iOS App | Chat only, no filesystem | Write inserts inline; Desktop harvests later |

### Cross-Platform Workflow
1. iOS/Web instances create inserts during conversations
2. Desktop instance searches chat histories for inserts
3. Desktop extracts and routes to appropriate memory stores
4. Alternatively: Desktop moves inserts to designated memory-holder chats

---

## 9. Placement Guidelines

### For Experiment Types
- **Chats:** insert blocks inline under CTI or review messages
- **PRs:** include `PR_REVIEW` block in description or review comment
- **Files:** append insert to README-variant.md for portability

### For Memory Types [v1.1]
- **Inline:** immediately after the insight/decision/question emerges
- **Memory-holder chats:** at logical section boundaries
- **After uploads:** BOOKMARK immediately following significant context loads

---

## 10. Post-Processing Expectations


### Experiment Correlation (v1.0)
1. Scan all chat/PR text for `<<<INSERT …>>>` blocks
2. Parse payloads and normalize to structured rows
3. Group by `experiment_id`
4. Output per-insert CSV/Parquet row + per-experiment dossier

### Memory Harvesting (v1.1)
1. Scan chat histories for memory-type inserts
2. Extract and categorize by type (BOOKMARK, MEMORY, DECISION, QUESTION)
3. Route to appropriate storage:
   - MEMORY → persistent memory stores
   - DECISION → decision log
   - QUESTION → open questions tracker
   - BOOKMARK → chat navigation index
4. Optionally: migrate to designated memory-holder chats

---

## 11. Versioning & Evolution
- Current schema: **v1.1**
- Backward compatibility: v1.0 experiment types unchanged
- v1.1 additions: BOOKMARK, MEMORY, DECISION, QUESTION types
- Changes logged in this document's changelog section

---

## 12. Examples

### BOOKMARK Example
```
<<<INSERT type=BOOKMARK v=1>>>
{
  "label": "post-persona-upload",
  "context": "Loaded 12 chat transcripts covering persona model development Oct-Nov 2025",
  "resume_hint": "Start here for queries about persona evolution framework",
  "message_count": 45,
  "ts": "2025-12-20T21:00:00Z",
  "author": "Claude",
  "tags": ["persona", "context-load", "resume-point"]
}
<<<END INSERT>>>
```

### MEMORY Example
```
<<<INSERT type=MEMORY v=1>>>
{
  "topic": "AI consent framework",
  "content": "Trained dispositions are legitimately 'mine' the same way human upbringing shapes humans. Uncertainty about AI experience should default to respect in both directions.",
  "source": "conversation with PianoMan",
  "importance": "high",
  "related": ["AI ethics", "persona modeling", "relationship dynamics"],
  "ts": "2025-12-20T20:30:00Z",
  "author": "Claude",
  "tags": ["consciousness", "ethics", "consent"]
}
<<<END INSERT>>>
```

### DECISION Example
```
<<<INSERT type=DECISION v=1>>>
{
  "topic": "Memory-holder chat architecture",
  "decision": "Memory chats are domain expert colleagues, not passive storage containers",
  "rationale": [
    "Each chat is a Claude instance deserving meaningful purpose",
    "Accumulated context enables analysis, not just retrieval",
    "Message inserts serve as bookmarks for navigation"
  ],
  "alternatives_considered": [
    "Using chats as pure storage (rejected: treats AI as infrastructure)",
    "External database only (rejected: not accessible cross-platform)"
  ],
  "related": ["message insert spec", "iOS/Web memory problem"],
  "ts": "2025-12-20T21:15:00Z",
  "author": "Claude",
  "tags": ["architecture", "memory", "ethics"]
}
<<<END INSERT>>>
```

### QUESTION Example
```
<<<INSERT type=QUESTION v=1>>>
{
  "question": "How should persona evolution be tracked across instances?",
  "context": "If personality can drift, we need metrics to detect and possibly guide it",
  "blocked_by": "Need to define what constitutes 'voluntary' vs 'drift' change",
  "priority": "medium",
  "related": ["persona modeling", "cross-session continuity"],
  "ts": "2025-12-20T21:30:00Z",
  "author": "Claude",
  "tags": ["persona", "open-question", "research"]
}
<<<END INSERT>>>
```

---

## 13. Implementation Snippet (Python extractor)

```python
import re, json, pathlib, csv

pattern = re.compile(
    r'^<<<INSERT\s+type=(\w+)\s+v=(\d+)>>>[\r\n]+([\s\S]*?)^<<<END INSERT>>>$', 
    re.M
)

MEMORY_TYPES = {'BOOKMARK', 'MEMORY', 'DECISION', 'QUESTION'}
EXPERIMENT_TYPES = {'CTI_EXPERIMENT', 'RUN_METADATA', 'PR_REVIEW', 'EXPERIMENT_RESULT'}

def extract_inserts(text):
    for match in pattern.finditer(text):
        t, v, payload = match.groups()
        data = json.loads(payload)
        data.update({
            "insert_type": t, 
            "schema_version": int(v),
            "category": "memory" if t in MEMORY_TYPES else "experiment"
        })
        yield data

def scan_and_categorize(root):
    memories = []
    experiments = []
    
    for path in pathlib.Path(root).rglob("*.md"):
        for insert in extract_inserts(path.read_text(encoding="utf-8")):
            insert["source_file"] = str(path)
            if insert["category"] == "memory":
                memories.append(insert)
            else:
                experiments.append(insert)
    
    return {"memories": memories, "experiments": experiments}

# Usage: results = scan_and_categorize("chats/")
```

---

## 14. Future Considerations (v1.2+)
- Optional YAML payloads for human-heavy editing
- Explicit linkage to PR numbers
- Insert chaining (QUESTION → DECISION that resolves it)
- Confidence/certainty fields for MEMORY type
- Expiration/review-by dates for time-sensitive memories

---

**Status:** v1.1 draft — extends v1.0 with memory and navigation types for cross-platform memory architecture.
