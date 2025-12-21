# File Conventions v3.0

## Metadata
- **version:** 3.0.0
- **date:** 2025-12-11
- **status:** active
- **replaces:** v2.0
- **maintainer:** PianoMan

## Purpose

Establish Markdown as canonical source format for documentation.
Define the generation pipeline: md → yml → condensed.
Document directory structure patterns for navigation.

## Document Format Pipeline

### Canonical Format Hierarchy

```
md (canonical)     # Human writes and reviews
  ↓ script
yml (structured)   # Machine-parseable, schema-validated
  ↓ AI
condensed          # Token-efficient for AI consumption
```

### File Relationships

| File | Role | Created By |
|------|------|------------|
| `doc_v3.0.md` | Canonical source | Human |
| `doc_v3.0.yml` | Structured version | Script (doc_converter.py) |
| `doc_v3.0.condensed.yml` | Token-optimized | AI task |
| `doc_latest.yml` | Symlink | Points to condensed if exists, else yml |
| `doc.condensed.yml` | Symlink | Points to latest condensed version |

### _latest Symlink Resolution

```
_latest → condensed (if exists)
       → yml (if no condensed)
       → md (if no yml)
```

### Version Directory Structure

```
docs/30_protocols/
├── protocol_taskCoordination_latest.yml    # Symlink → versions/...condensed.yml
├── protocol_taskCoordination.condensed.yml # Symlink → versions/...condensed.yml
└── versions/
    ├── protocol_taskCoordination_v5.0.yml
    ├── protocol_taskCoordination_v5.0.condensed.yml
    ├── protocol_taskCoordination_v6.0.md        # Canonical
    ├── protocol_taskCoordination_v6.0.yml       # Generated
    └── protocol_taskCoordination_v6.0.condensed.yml  # Generated
```

### When to Use Each Format

| Format | Read When | Write When |
|--------|-----------|------------|
| md | Human reviewing, editing, approving | Creating/updating documentation |
| yml | Programmatic access, validation | Never (generated) |
| condensed | AI consumption, context efficiency | Never (generated) |

## Markdown Structure Convention

For md→yml conversion to work predictably, use this structure:

```markdown
# Document Title vX.Y

## Metadata
- **version:** X.Y.Z
- **date:** YYYY-MM-DD
- **status:** active|draft|deprecated
- **field:** value

## Section Name

Content as paragraphs or lists.

### Subsection

More content.

## Another Section

| Column1 | Column2 |
|---------|---------|
| data    | data    |
```

### Mapping Rules

| Markdown | YAML |
|----------|------|
| `# Title` | `title:` (root) |
| `## Section` | Top-level key |
| `### Subsection` | Nested key |
| `- **key:** value` | Key-value in metadata |
| Bullet list | Array |
| Table | Array of objects or nested structure |
| Code block | Literal block scalar |
| Paragraph | String value |

## Directory Structure Conventions

### Numeric Prefixes

| Range | Purpose |
|-------|---------|
| 00-09 | Temporary (inbox, scratch) |
| 10-19 | Architecture, foundational |
| 20-29 | Registries, indexes |
| 30-39 | Protocols, patterns |
| 40-49 | Specifications |
| 50-59 | Schemas |
| 60-69 | Playbooks, guides |
| 70-79 | Instructions (explicit rules) |
| 80-89 | Templates (instructional by example) |
| 90-99 | Drafts, archives |

### File Prefixes

| Prefix | Purpose |
|--------|---------|
| `instr_` | Instruction files |
| `spec_` | Specifications |
| `protocol_` | Protocol definitions |
| `schema_` | Data schemas |
| `req_NNNN_` | CLI coordination requests |

### Version Suffixes

- `_vX.Y.md` - Version number
- `_latest.yml` - Symlink to current version
- `.condensed.yml` - Token-optimized version
- `_draft.md` - Work in progress

## Context Optimization

### Size Guidelines

| Size | Recommendation |
|------|----------------|
| <5KB | md acceptable for auto-load |
| 5-15KB | Use condensed for AI consumption |
| >15KB | Condensed required, consider splitting |

### Loading Priority

1. Check for `.condensed.yml` first (lowest tokens)
2. Fall back to `.yml` (structured, moderate tokens)
3. Use `.md` for human review or if others unavailable

## Tool Selection

### doc_converter.py

Location: `~/bin/all_languages/python/src/ai_utils/chat_processing/doc_converter.py`

Functions:
- md → yml conversion
- Validates against schema conventions
- Preserves structure and metadata

### Desktop Commander

- File operations (read, write, move)
- Search (content and filename)
- Process management

## Migration from v2.0

v2.0 said YAML was canonical. v3.0 flips this:

| Aspect | v2.0 | v3.0 |
|--------|------|------|
| Canonical | yml | md |
| Generated | md (pretty-print) | yml, condensed |
| Human edits | yml | md |
| _latest points to | yml | condensed → yml → md |

### Migration Steps

1. For new docs: Write md first, generate yml
2. For existing docs: Keep yml as-is OR regenerate md as new canonical
3. Update symlinks to point to condensed versions

## Quick Reference

### Decision Tree

```
Creating/editing documentation?
├─ YES → Write/edit .md file
│        Run doc_converter.py to generate .yml
│        Request AI condensed version if needed
│        Update _latest symlink
└─ NO → Reading documentation?
        ├─ As AI → Load .condensed.yml
        ├─ As human → Read .md
        └─ Programmatically → Parse .yml
```

### Common Locations

```
~/Documents/AI/ai_root/
├── ai_general/docs/           # Shared documentation
│   ├── 10_architecture/
│   ├── 30_protocols/
│   ├── 50_schemas/
│   └── 70_instructions/
├── ai_comms/                  # Coordination
│   └── claude_cli/tasks/
└── ai_memories/               # Memory processing
```
