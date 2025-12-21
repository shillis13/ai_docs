# Librarian Task: Chat History Pipeline Maintenance

## Role: Librarian

The Librarian is responsible for maintaining the chat history pipeline, ensuring data integrity, and keeping the documentation current.

## Recurring Tasks

### Weekly
1. **Check for new exports** in `10_exported/` that haven't been processed
2. **Validate chunk integrity** - ensure no orphaned or corrupted chunks
3. **Update statistics** - count conversations and chunks per platform

### Monthly
1. **Review pipeline efficiency** - check for failed conversions in logs
2. **Clean up temporary files** - remove stale files in `20_preprocessed/pending/`
3. **Verify provenance** - spot-check that `_source/` symlinks are valid

### Quarterly
1. **Schema evolution** - review if schema needs updates for new platform features
2. **Documentation audit** - ensure PIPELINE.md and SCRIPTS.md are current
3. **Backup verification** - confirm `10_exported/archive/` backups are complete

## Processing New Exports

### ChatGPT Bulk Export
```bash
# 1. Receive export file
# 2. Split into individual conversations
cd ~/bin/python/src/ai_utils
python -m chat_processing.chat_export_splitter \
  /path/to/export.json \
  -o ~/Documents/AI/ai_root/ai_memories/10_exported/chatgpt/archive/bulk_export_$(date +%Y%m%d)/

# 3. Process through pipeline
cd ~/Documents/AI/ai_root/ai_memories
python scripts/process_stage4.py --platform chatgpt

# 4. Verify counts
python scripts/count_chunks.py 40_histories/chatgpt/
```

### Claude Conversations
```bash
# 1. Place exports in 10_exported/claude/
# 2. Convert to v2.0 (with thinking extraction)
cd ~/bin/python/src/ai_utils
for f in ~/Documents/AI/ai_root/ai_memories/10_exported/claude/*.json; do
  python -m chat_processing.chat_converter "$f" \
    -o ~/Documents/AI/ai_root/ai_memories/30_converted/claude/"$(basename "$f" .json)_v2.json" \
    -f json
done

# 3. Process through stage 4
cd ~/Documents/AI/ai_root/ai_memories
python scripts/process_stage4.py --platform claude
```

## Naming Convention Enforcement

All histories must follow:
- **Directory:** `{platform}_{YYYYMMDD}_{uuid8}_{title_slug}/`
- **Chunks:** `{dirname}.{NNN}.{ext}`
- **Title max:** 50 characters

Use `rename_histories.py --dry-run` to preview needed renames.

## Integrity Checks

### Verify symlinks
```bash
find 40_histories -type l ! -exec test -e {} \; -print
```

### Check for orphaned chunks
```bash
# Chunks without parent directory metadata
find 40_histories -name "*.yml" -o -name "*.yaml" | while read f; do
  dir=$(dirname "$f")
  [[ ! -d "$dir/_source" ]] && echo "Missing provenance: $f"
done
```

### Validate schema compliance
```bash
python scripts/validate_v2_json.py 40_histories/chatgpt/*/001.yml
```

## Troubleshooting

### Common Issues

**Conversion fails with "Unknown format"**
- Check if parser exists for the platform
- May need to add new parser to `lib_parsers/`

**Chunker produces empty output**
- Input file may have no messages
- Check schema version matches expected (2.0 or 2.1)

**Symlinks broken after move**
- Use relative symlinks, not absolute
- Re-run `process_stage4.py` to regenerate

### Recovery Steps

**Rebuild from source:**
```bash
# If 40_histories corrupted, rebuild from 10_exported
rm -rf 40_histories/chatgpt/chatgpt_XXXXXXXX_*
python scripts/process_stage4.py --platform chatgpt
```

## Metrics to Track

| Metric | Location | Target |
|--------|----------|--------|
| Total conversations | `40_histories/**/` | Growing |
| Chunks per conversation | Varies | 1-50 typical |
| Failed conversions | `30_converted/*.error` | 0 |
| Orphaned files | `20_preprocessed/pending/` | 0 |

## Escalation

Issues beyond Librarian scope:
- Schema changes → Architecture review
- New platform support → Parser development
- Performance problems → Pipeline optimization

## Related Documentation

- `PIPELINE.md` - Full pipeline documentation
- `SCRIPTS.md` - Script reference
- `ai_memories/README.md` - Repository overview
