# Chat History Processing Pipeline

## Overview

This pipeline converts raw chat exports from various AI platforms (ChatGPT, Claude, etc.) into a normalized, chunked format suitable for search, analysis, and context loading.

## Directory Structure

```
ai_memories/
├── 10_exported/              # Stage 0: Raw exports by platform
│   ├── chatgpt/
│   │   └── archive/bulk_export_YYYYMMDD/
│   └── claude/
├── 20_preprocessed/          # (optional) Multi-chat splits
│   ├── pending/
│   └── converted/
├── 30_converted/             # Stage 2 output: v2.0 schema JSON
│   ├── chatgpt/
│   └── claude/
└── 40_histories/             # Stage 4 output: Final chunked histories
    ├── chatgpt/
    │   └── chatgpt_YYYYMMDD_uuid_title/
    │       ├── chatgpt_YYYYMMDD_uuid_title.001.yml
    │       ├── _intermediate/   # Converted YAML (provenance)
    │       └── _source/         # Symlink to original in 10_exported
    └── claude/
        └── claude_YYYYMMDD_uuid_title/
            └── ...
```

## Pipeline Stages

### Stage 0: Export Collection
**Manual step** - Export chat history from AI platform and place in `10_exported/{platform}/`

| Platform | Export Method | Format |
|----------|---------------|--------|
| ChatGPT | Settings → Data controls → Export | Single JSON with all chats |
| Claude | Use claude-exporter tool or manual | Individual JSON per chat |

### Stage 1: Split Bulk Exports
**Script:** `chat_export_splitter.py`  
**Location:** `bin/python/src/ai_utils/chat_processing/`

Splits a single bulk export file (containing all chats) into individual conversation files.

```bash
python -m chat_processing.chat_export_splitter \
  /path/to/chatgpt_export.json \
  -o ~/Documents/AI/ai_root/ai_memories/10_exported/chatgpt/archive/bulk_export_YYYYMMDD/
```

**Input:** Single JSON with all conversations  
**Output:** Individual JSON files per conversation

### Stage 2: Convert to v2.0 Schema
**Script:** `chat_converter.py`  
**Location:** `bin/python/src/ai_utils/chat_processing/`

Converts platform-specific formats to the unified chat_history_v2.0 schema.

```bash
python -m chat_processing.chat_converter \
  input.json \
  -o output_v2.json \
  -f json
```

**Input:** Platform-native JSON/Markdown  
**Output:** v2.0 schema JSON or YAML

**Supported input formats:**
- ChatGPT JSON (bulk export tree structure)
- Claude JSON (conversation export)
- Markdown (various formats)
- HTML (ChatGPT share pages)

### Stage 3: Chunk Large Conversations
**Script:** `chat_chunker.py`  
**Location:** `bin/python/src/ai_utils/chat_processing/`

Splits large conversations into ~4000 token chunks while preserving Q&A coherence.

```bash
python -m chat_processing.chat_chunker \
  input_v2.yaml \
  -o chunks/ \
  --target-size 4000
```

**Input:** v2.0 schema YAML  
**Output:** Multiple chunk files (`.001.yml`, `.002.yml`, etc.)

### Stage 4: Organize into Histories
**Script:** `process_stage4.py`  
**Location:** `ai_general/scripts/chat_pipeline/` (symlinked from `ai_memories/scripts/`)

Orchestrates stages 2+3 and organizes output into the final directory structure with provenance tracking.

```bash
cd ~/Documents/AI/ai_root/ai_memories
python scripts/process_stage4.py --platform chatgpt
python scripts/process_stage4.py --platform claude
```

**Input:** Files in `30_converted/{platform}/`  
**Output:** Organized directories in `40_histories/{platform}/`

## Naming Conventions

### Directory Names
```
{platform}_{YYYYMMDD}_{uuid8}_{title_slug}/
```
- `platform`: chatgpt, claude, gemini, etc.
- `YYYYMMDD`: Conversation creation date
- `uuid8`: First 8 characters of conversation UUID
- `title_slug`: Sanitized title (max 50 chars, lowercase, underscores)

Example: `chatgpt_20221221_e39c2f79_quantum_computing_overview/`

### Chunk File Names
```
{dirname}.{NNN}.{ext}
```
- `dirname`: Matches parent directory name exactly
- `NNN`: Zero-padded sequence number (001, 002, etc.)
- `ext`: `yml` for ChatGPT, `yaml` for Claude

Example: `chatgpt_20221221_e39c2f79_quantum_computing_overview.001.yml`

## Supporting Scripts

| Script | Purpose |
|--------|---------|
| `chatgpt_tree_converter.py` | Convert ChatGPT tree exports to v2 YAML. Uses LiteralDumper for proper newlines. |
| `process_to_histories.py` | Chunk conversations and organize into year/month structure in 40_histories/ |
| `rename_histories.py` | Bulk rename dirs/chunks to new naming convention |
| `validate_v2_json.py` | Validate v2.0 schema compliance |
| `count_chunks.py` | Count chunks across directories |
| `archive_chatgpt_bulk.sh` | Archive bulk export to dated directory |
| `reset_pipeline_test.sh` | Reset pipeline for testing |

## Quick Reference

### Full Pipeline Run (ChatGPT Bulk Export)
```bash
# 1. Split bulk export
cd ~/bin/python/src/ai_utils
python -m chat_processing.chat_export_splitter \
  ~/Downloads/chatgpt_export.json \
  -o ~/Documents/AI/ai_root/ai_memories/10_exported/chatgpt/archive/bulk_export_$(date +%Y%m%d)/

# 2. Convert all to v2.0
for f in ~/Documents/AI/ai_root/ai_memories/10_exported/chatgpt/archive/bulk_export_*/*.json; do
  python -m chat_processing.chat_converter "$f" -o "${f%.json}_v2.json" -f json
done

# 3. Move converted to 30_converted
mv ~/Documents/AI/ai_root/ai_memories/10_exported/chatgpt/archive/bulk_export_*/*_v2.json \
   ~/Documents/AI/ai_root/ai_memories/30_converted/chatgpt/

# 4. Process through stage 4
cd ~/Documents/AI/ai_root/ai_memories
python scripts/process_stage4.py --platform chatgpt
```

### Single Conversation
```bash
cd ~/bin/python/src/ai_utils

# Convert
python -m chat_processing.chat_converter input.json -o output_v2.yaml -f yaml

# Chunk
python -m chat_processing.chat_chunker output_v2.yaml -o chunks/
```

## Schema Reference

See `ai_general/docs/50_schemas/chat_history_v2.0.yml` for the full v2.0 schema specification.

Key fields:
- `schema_version`: "2.0" or "2.1" (with thinking field)
- `chat_id`: Unique conversation identifier
- `platform`: Source platform
- `title`: Conversation title
- `created_at`, `updated_at`: ISO timestamps
- `messages[]`: Array of message objects
  - `role`: user, assistant, system
  - `content`: Message text
  - `thinking`: (v2.1 only) Claude's extended thinking
  - `timestamp`: Message timestamp

## Related Documentation

- Schema: `ai_general/docs/50_schemas/chat_history_v2.0.yml`
- Architecture: `ai_general/docs/10_architecture/`
- Scripts: `ai_general/scripts/chat_pipeline/`
