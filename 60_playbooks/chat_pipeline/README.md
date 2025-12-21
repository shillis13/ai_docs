# Chat Pipeline Documentation

Documentation for the AI chat history processing pipeline.

## Contents

| Document | Purpose |
|----------|---------|
| `PIPELINE.md` | Full pipeline documentation - stages, directories, naming conventions |
| `SCRIPTS.md` | Script reference - usage, options, examples |
| `LIBRARIAN_TASK.md` | Maintenance tasks for keeping the pipeline healthy |
| `BULK_IMPORT_202512.md` | Log of December 2025 bulk import (1,424 conversations) |

## Quick Start

```bash
# Process new ChatGPT bulk export
cd ~/Documents/AI/ai_root/ai_memories
python scripts/process_stage4.py --platform chatgpt

# Process new Claude exports  
python scripts/process_stage4.py --platform claude
```

## Related Locations

| Resource | Path |
|----------|------|
| Scripts | `ai_general/scripts/chat_pipeline/` |
| Data | `ai_memories/40_histories/` |
| Schema | `ai_general/docs/50_schemas/chat_history_v2.0.yml` |
| Converter library | `bin/python/src/ai_utils/chat_processing/` |
