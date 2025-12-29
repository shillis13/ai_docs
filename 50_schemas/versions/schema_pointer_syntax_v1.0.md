# Reference Pointer Syntax Schema

**Version:** 1.0.0
**Created:** 2025-12-05
**Maintainer:** PianoMan
**Status:** active

## Related Documents
- Protocol: `30_protocols/protocol_reference_pointers_v1.0.yml`
- Instructions: `70_instructions/claude/instr_pointer_loading_v1.0.yml`

## Grammar

```
<TYPE>:<TARGET> [| <KEY>:<VALUE>]*
```

**Max Length:** 200 characters (must fit in native memory slot)

## Pointer Types

| Type | Description | Target Format | Example |
|------|-------------|---------------|---------|
| STARTER | Core identity and preferences - load first | relative/path/to/file.yml | `STARTER:ai_memories/60_knowledge/about_user/identity.yml` |
| PROJECT | Current work context | relative/path/to/file.yml | `PROJECT:ai_claude/work/current_focus.yml` |
| PREFS | Communication and behavior preferences | relative/path/to/file.yml | `PREFS:ai_claude/instructions/comm_style.yml` |
| CONTEXT | Situational context (time-sensitive) | relative/path/to/file.yml | `CONTEXT:ai_comms/claude_cli/status_summary.yml` |
| TOOL | Tool configurations and patterns | relative/path/to/file.yml | `TOOL:ai_general/docs/40_specs/cli_coordination.yml` |
| REF | Reference material - on-demand only | relative/path/to/file.yml | `REF:ai_general/docs/40_specs/spec_full_detail.yml` |
| QUERY | Dynamic search against long-term memory | search terms (free text) | `QUERY:CLI timeout solutions \| LIMIT:5 \| SINCE:30d` |
| RECENT | Retrieve N most recent items | category name | `RECENT:session_summaries \| COUNT:3` |

## Modifiers

### PRIORITY
| Value | Meaning |
|-------|---------|
| 1 | Load at conversation start |
| 2 | Load when topic area arises (default) |
| 3 | Load only when explicitly referenced |

### LOAD
| Value | Meaning |
|-------|---------|
| AUTO | Load without AI decision |
| TOPIC | Load when related topic detected (default) |
| DEMAND | Load only when explicitly requested |

### SCOPE
| Value | Meaning |
|-------|---------|
| GLOBAL | Applies across all contexts (default) |
| PROJECT | Applies to specific project |
| SESSION | Temporary, current session only |

### TTL
- Format: `YYYY-MM-DD` or `Nd` (N days)
- Meaning: Review/refresh after this date
- Example: `TTL:7d`

## Query Modifiers

| Modifier | Type | Default | Meaning |
|----------|------|---------|---------|
| LIMIT | integer | 5 | Max results to return |
| SINCE | Nd/Nw/Nm | - | Only results updated within window |
| SOURCE | enum | all | Which memory tier: histories, summaries, knowledge, all |
| SORT | enum | relevance | Sort order: relevance, recent, oldest |

## Validation Rules

### Pointer String
- Must be <= 200 characters
- TYPE must be recognized value
- TARGET must not be empty
- Modifiers must use valid keys

### Path Targets
- Must be relative to ai_root/
- Must end in .yml or .yaml
- Should resolve to existing file (warning if not)

### Query Targets
- Should contain substantive keywords
- Avoid generic terms like "the", "and"

## Examples

**Minimal:**
```
STARTER:ai_memories/60_knowledge/about_user/identity.yml
```
(56 characters)

**With Modifiers:**
```
PROJECT:ai_claude/work/current_focus.yml | PRIORITY:2 | LOAD:TOPIC | SCOPE:PROJECT
```
(88 characters)

**Query Pointer:**
```
QUERY:chat orchestrator design decisions | LIMIT:3 | SINCE:30d
```
(66 characters)

## Regex Patterns

**Full Pointer:**
```regex
^(?P<type>[A-Z]+):(?P<target>[^\|]+?)(?:\s*\|\s*(?P<mods>.+))?$
```

**Modifier Pair:**
```regex
(?P<key>[A-Z]+):(?P<value>[^\|]+)
```
