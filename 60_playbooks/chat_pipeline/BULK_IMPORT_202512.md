# Bulk Import Log: December 2025

## Summary

**Date:** December 1-4, 2025  
**Operator:** PianoMan + Claude (Desktop)  
**Scope:** Full ChatGPT history + Claude conversation reconversion

## Final Counts

| Platform | Conversations | Chunks | Date Range |
|----------|---------------|--------|------------|
| ChatGPT | 1,000 | 3,976 | Dec 2022 - Dec 2025 |
| Claude | 424 | 807 | Jun 2025 - Dec 2025 |
| **Total** | **1,424** | **4,783** | |

### ChatGPT Breakdown by Year
- 2022: 4
- 2023: 92
- 2024: 299
- 2025: 608
- Individual exports (non-bulk): 16

## Actions Performed

### 1. ChatGPT Bulk Export Processing

**Source:** ChatGPT data export (Settings → Data controls → Export)  
**File:** Single JSON containing 984 conversations

**Steps:**
```bash
# Split bulk export
python -m chat_processing.chat_export_splitter export.json \
  -o 10_exported/chatgpt/archive/bulk_export_20251201/

# Process through pipeline
python scripts/process_stage4.py --platform chatgpt
```

**Result:** 984 bulk + 16 individual = 1,000 conversations

### 2. Claude Thinking Extraction

**Problem:** Claude's extended thinking was merged with response content in schema v2.0

**Solution:** Modified `ClaudePlatformParser` in `json_parser.py` (lines 415-435) to:
1. Parse `content` blocks by type (`thinking` vs `text`)
2. Store thinking separately in `thinking` field
3. Bump schema to v2.1 when thinking present

**Reconversion:**
```bash
# Backup existing
mv 40_histories/claude/* 40_histories/_backup_claude_20251202/

# Reconvert all 424 Claude conversations
for src in _backup_claude_20251202/claude_*/_source/*.json; do
  python -m chat_processing.chat_converter "$src" -o 30_converted/claude/... -f json
done

# Rechunk
python scripts/process_stage4.py --platform claude
```

**Result:** 618 of 807 chunks (77%) now have separate `thinking` field

### 3. Directory Naming Convention Change

**Old format:** `chatgpt_e39c2f79_20221221_Quantum_Computing_Overview/`  
**New format:** `chatgpt_20221221_e39c2f79_Quantum_Computing_Overview/`

**Rationale:** Date-first enables chronological sorting with `ls`

**Script:** `rename_histories.py` (created during session)

### 4. Chunk Naming Convention Change

**Old format:** `Quantum_Computing_Overview.chunk_001.yml`  
**New format:** `chatgpt_20221221_e39c2f79_Quantum_Computing_Overview.001.yml`

**Changes:**
- Chunk name matches directory name
- Removed `.chunk` (`.NNN` suffix sufficient)
- Title truncated to 50 chars

**Script:** Ad-hoc Python script (`/tmp/rename_chunks.py`)

### 5. Directory Structure Fixes

**Problem:** `chatgpt_*` and `claude_*` directories at `40_histories/` root instead of platform subdirs

**Fix:**
```bash
mkdir -p 40_histories/chatgpt 40_histories/claude
mv 40_histories/chatgpt_* 40_histories/chatgpt/
mv 40_histories/claude_* 40_histories/claude/
```

### 6. Script Organization

**Moved:** Pipeline scripts from `ai_memories/*.py` to `ai_general/scripts/chat_pipeline/`  
**Symlink:** `ai_memories/scripts` → `../ai_general/scripts/chat_pipeline`

## Data Integrity Verification

### Size Reduction Analysis

| Conversation | Original JSON | Converted | Reduction |
|--------------|---------------|-----------|-----------|
| Largest (5MB) | 5.0 MB | 296 KB | 95% |
| Medium (2MB) | 2.1 MB | 324 KB | 85% |
| Small (5KB) | 4.5 KB | 1.7 KB | 61% |

**What was removed (overhead only, no content loss):**
1. Tree structure UUIDs (parent/child references)
2. User custom instructions (repeated per conversation)
3. Empty system messages
4. Tool execution artifacts (streaming chunks)
5. Redundant metadata fields

### Symlink Verification
```bash
find 40_histories -type l ! -exec test -e {} \; -print
# Result: 0 broken symlinks
```

### Schema Compliance
- All ChatGPT chunks: v2.0
- All Claude chunks: v2.0 or v2.1 (when thinking present)

## Issues Encountered

### 1. Codex MCP Permission Loop
**Problem:** macOS TCC (Transparency, Consent, and Control) triggered endless permission dialogs when Codex MCP spawned subprocesses.

**Resolution:** Removed Codex MCP configuration. Use CLI Coordination for heavy tasks instead.

### 2. Extension Mismatch
**Problem:** ChatGPT chunks use `.yml`, Claude chunks use `.yaml`

**Status:** Left as-is for now. Both are valid YAML extensions.

### 3. process_stage4.py Platform Path Bug
**Problem:** Script was writing to `40_histories/` root instead of `40_histories/{platform}/`

**Fix:** Line 136 changed from:
```python
output_dir = HISTORIES_DIR / dir_name
```
To:
```python
output_dir = HISTORIES_DIR / meta['platform'] / dir_name
```

## Files Modified

| File | Change |
|------|--------|
| `json_parser.py` | Added Claude thinking extraction (lines 415-435) |
| `chat_chunker.py` | Added schema 2.1 support (line 284-286) |
| `process_stage4.py` | Fixed platform subdirectory, increased title max |
| `rename_histories.py` | Created for naming convention migration |

## Backup Locations

- `10_exported/chatgpt/archive/bulk_export_20251201/` - Original ChatGPT JSON (984 files)
- `40_histories/_backup_claude_20251202/` - Pre-reconversion Claude (deleted after verification)

## Next Steps

1. Update `process_stage4.py` to generate new naming format for future runs
2. Standardize `.yml` vs `.yaml` extension
3. Consider platform/year subdirectory structure for large archives
4. Implement search/index capability over chunks
