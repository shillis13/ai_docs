# Message Insert Protocol (Condensed v1.1)
**Source:** spec_message_insert_v1.1.md

## Purpose
Machine-readable markers in chats/PRs/files for:
- Experiment lifecycle correlation (v1.0)
- Memory capture & cross-session persistence (v1.1)
- Navigation bookmarks in memory-holder chats (v1.1)

## Syntax
```
<<<INSERT type=<TYPE> v=<INT>>>
{ JSON PAYLOAD }
<<<END INSERT>>>
```
Regex: `^<<<INSERT\s+type=(\w+)\s+v=(\d+)>>>[\r\n]+([\s\S]*?)^<<<END INSERT>>>$`

## Types

### Experiment Types (v1.0)
| Type | Purpose |
|------|---------|
| CTI_EXPERIMENT | Define hypothesis, knobs, metrics |
| RUN_METADATA | Model, exec time, artifacts |
| PR_REVIEW | Variants reviewed, scores, winner |
| EXPERIMENT_RESULT | Decision, findings, next actions |

### Memory Types (v1.1)
| Type | Purpose |
|------|---------|
| BOOKMARK | Navigation/resume point after context loads |
| MEMORY | Captured insight, preference, or fact |
| DECISION | Conclusion reached with rationale |
| QUESTION | Open question for future consideration |

## Base Fields

### Experiment Types
```json
{
  "experiment_id": "exp-YYYY-MM-DD-<slug>-NNN",
  "project": "string", "task_title": "string",
  "mode": "Exploratory|Production", "guidance": "L0-L3",
  "ts": "ISO-8601", "author": "string", "notes": "optional"
}
```

### Memory Types (simpler)
```json
{
  "ts": "ISO-8601",
  "author": "string",
  "tags": ["optional", "searchable", "array"]
}
```

## Extended Fields (v1.1 Memory Types)

**BOOKMARK:** `label`, `context`, `resume_hint`, `message_count`

**MEMORY:** `topic`, `content`, `source`, `importance` (low/medium/high/critical), `related[]`

**DECISION:** `topic`, `decision`, `rationale[]`, `alternatives_considered[]`, `related[]`

**QUESTION:** `question`, `context`, `blocked_by`, `priority` (low/medium/high), `related[]`

## Memory-Holder Chat Architecture
1. Designated chats = domain expert repositories (not passive storage)
2. BOOKMARK after significant context loads
3. MEMORY for key insights
4. Instances can be queried for analysis, not just retrieval

## Cross-Platform Workflow
- iOS/Web: Write inserts inline (can't access filesystem)
- Desktop: Harvest inserts → route to memory stores or memory-holder chats
- Searchable via insert type and tags

## Quick Examples

**BOOKMARK:**
```
<<<INSERT type=BOOKMARK v=1>>>
{"label": "post-upload", "context": "Loaded persona transcripts", "resume_hint": "Query persona evolution here", "ts": "2025-12-20T21:00:00Z", "author": "Claude"}
<<<END INSERT>>>
```

**MEMORY:**
```
<<<INSERT type=MEMORY v=1>>>
{"topic": "AI consent", "content": "Uncertainty defaults to respect", "importance": "high", "ts": "2025-12-20T20:30:00Z", "author": "Claude", "tags": ["ethics"]}
<<<END INSERT>>>
```

**Status:** v1.1 draft — extends v1.0 with memory/navigation types
