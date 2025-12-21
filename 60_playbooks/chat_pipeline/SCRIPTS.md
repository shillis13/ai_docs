# Chat Pipeline Scripts Reference

## Core Pipeline Scripts

### chat_export_splitter.py
**Location:** `bin/python/src/ai_utils/chat_processing/`  
**Stage:** 1 - Split bulk exports

Splits ChatGPT bulk exports (single JSON with all conversations) into individual files.

```bash
python -m chat_processing.chat_export_splitter INPUT [-o OUTPUT_DIR]
```

**Features:**
- Handles various export wrapper formats (conversations, data, payload, export keys)
- Single-line JSON support (ChatGPT's default format)
- Generates filenames from conversation metadata

---

### chat_converter.py
**Location:** `bin/python/src/ai_utils/chat_processing/`  
**Stage:** 2 - Format conversion

Converts platform-specific formats to v2.0 schema.

```bash
python -m chat_processing.chat_converter INPUT -o OUTPUT [-f FORMAT]
```

**Options:**
- `-f json` or `-f yaml` - Output format
- `-v` - Verbose logging

**Parsers:**
- `ChatGPTPlatformParser` - ChatGPT tree structure JSON
- `ClaudePlatformParser` - Claude conversation JSON (with thinking extraction)
- `MarkdownParser` - Various markdown formats
- `HTMLParser` - ChatGPT share pages

---

### chat_chunker.py
**Location:** `bin/python/src/ai_utils/chat_processing/`  
**Stage:** 3 - Chunking

Splits large v2.0 YAML files into manageable chunks.

```bash
python -m chat_processing.chat_chunker INPUT [-o OUTPUT_DIR] [--target-size TOKENS]
```

**Options:**
- `--target-size` - Target chunk size in tokens (default: 4000)
- `--dry-run` - Preview without writing
- `--basename` - Custom output filename base

**Behavior:**
- Preserves Q&A coherence (chunks start with user messages)
- Validates schema version (2.0 or 2.1)
- Generates sequenced output files (.001.yml, .002.yml, etc.)

---

### process_stage4.py
**Location:** `ai_memories/scripts/`  
**Stage:** 4 - Orchestration and organization

Processes all files in `30_converted/` through chunking and organizes into `40_histories/`.

```bash
python scripts/process_stage4.py --platform PLATFORM
```

**Options:**
- `--platform` - chatgpt, claude, etc.

**Actions:**
1. Creates directory: `{platform}_{date}_{uuid}_{title}/`
2. Preserves provenance: `_source/` symlink, `_intermediate/` YAML
3. Chunks with title-based naming

---

## Utility Scripts

### rename_histories.py
**Location:** `ai_memories/scripts/`

Renames directories and chunk files to match naming convention.

```bash
python scripts/rename_histories.py [--dry-run]
```

**Transformations:**
- Swaps UUID and date positions (date-first for sorting)
- Truncates titles to 50 characters
- Renames chunks to match directory name

---

### validate_v2_json.py
**Location:** `ai_memories/scripts/`

Validates files against v2.0 schema.

```bash
python scripts/validate_v2_json.py FILE
```

---

### count_chunks.py
**Location:** `ai_memories/scripts/`

Counts chunks across history directories.

```bash
python scripts/count_chunks.py [DIRECTORY]
```

---

### archive_chatgpt_bulk.sh
**Location:** `ai_memories/scripts/`

Archives a ChatGPT bulk export to a dated directory.

```bash
./scripts/archive_chatgpt_bulk.sh /path/to/export.json
```

---

### reset_pipeline_test.sh
**Location:** `ai_memories/scripts/`

Resets pipeline directories for testing (clears 30_converted, 40_histories).

```bash
./scripts/reset_pipeline_test.sh
```

---

## Library Modules

### lib_parsers/
Platform-specific parsers:
- `json_parser.py` - ChatGPT and Claude JSON parsing
- `markdown_parser.py` - Markdown format parsing
- `yaml_parser.py` - YAML parsing
- `html_parser.py` - HTML parsing

### lib_converters/
- `conversion_framework.py` - Core conversion logic
- `chunker.py` - Chunking algorithm
- `lib_conversion_utils.py` - Utility functions

### lib_formatters/
- `markdown_formatter.py` - Output as Markdown
- `html_formatter.py` - Output as HTML
- `yaml_formatter.py` - Output as YAML

### schemas/
- `chat_history_v2.0.yml` - Schema definition
