# Documentation Update Workflow

**Version:** 1.0.0
**Created:** 2025-12-19
**Source Chat:** ai_general/tmp/Claude-Gemini_How_to_Update_Docs.md
**Status:** validated

## Summary

End-to-end workflow for creating/updating documentation in ai_root. Covers file naming convention, versioning, symlinks, and cross-reference updates.

## Critical Meta-Lesson

**Problem:** Created playbook without consulting existing examples.
**Root Cause:** Assumed structure without loading existing playbooks first.

**Lesson:** BEFORE creating any doc type:
1. Find existing examples: `ls ai_general/docs/{doc_type}/`
2. Read at least one to understand structure
3. Check for schema in 50_schemas/
4. THEN create new doc following established pattern

## File Naming Convention

**Pattern:** `{type}.{slug}.{version}.{variant?}.{ext}`

**Separators:**
- Dot: Field separator (parseable with `split('.')`)
- Underscore: Word separator within a field

**Fields:**
- type: playbook, schema, protocol, spec, instr
- slug: descriptive_name_with_underscores
- version: v1_0, v2_1, or "latest" (underscore replaces dot)
- variant: condensed (optional)
- ext: md, yml

**Examples:**
- `playbook.gemini_canvas_automation.v1_1.md`
- `playbook.gemini_canvas_automation.v1_1.condensed.yml`
- `playbook.gemini_canvas_automation.latest.md`

## Directory Structure

```
ai_general/docs/{doc_type}/
├── versions/
│   ├── {type}.{slug}.v1_0.md           # canonical source
│   ├── {type}.{slug}.v1_0.condensed.yml # condensed version
│   ├── {type}.{slug}.v1_1.md           # next version
│   └── {type}.{slug}.v1_1.condensed.yml
├── {type}.{slug}.latest.md → versions/{type}.{slug}.v1_1.md
└── {type}.{slug}.latest.condensed.yml → versions/{type}.{slug}.v1_1.condensed.yml
```

## Workflow Steps

### 1. Check Existing
```bash
ls ai_general/docs/{doc_type}/
cat ai_general/docs/{doc_type}/some_existing_example.md
```
**Rule:** Never create without understanding established patterns.

### 2. Create Canonical
- Location: `{doc_type}/versions/{type}.{slug}.v{X_Y}.md`
- Include metadata section (title, version, created, maintainer, status)
- Follow schema structure for doc type
- Use ASCII tables per instr_writing.condensed.yml

### 3. Create Condensed
- Location: `{doc_type}/versions/{type}.{slug}.v{X_Y}.condensed.yml`
- Target: 60-80% token reduction
- Include header comment pointing to full version
- Filename version must match metadata.version

### 4. Create Symlinks
```bash
cd ai_general/docs/{doc_type}/
ln -sf versions/{type}.{slug}.v{X_Y}.md {type}.{slug}.latest.md
ln -sf versions/{type}.{slug}.v{X_Y}.condensed.yml {type}.{slug}.latest.condensed.yml
```

### 5. Update References
- **Glossary:** `docs/20_registries/glossary_knowledge_index.condensed.yml`
- **Knowledge Manifest:** `docs/70_instructions/claude/_knowledge_manifest.yml`
- **Document Registry:** `docs/20_registries/document_registry.yml`

### 6. Verify
- [ ] Canonical .md exists in versions/
- [ ] Condensed .yml exists in versions/
- [ ] Symlinks point correctly (`ls -la {type}.{slug}.*`)
- [ ] References updated

## Common Issues

| Issue | Symptom | Fix |
|-------|---------|-----|
| Created without checking | Doesn't match patterns | Read existing examples first |
| Symlink naming inconsistent | One uses _latest, other doesn't | Both should be .latest. |
| Version in filename has dot | Parsing fails | Use underscore (v1_0) |
| Version mismatch | Filename vs metadata differ | Keep synchronized |

## Migration for Existing Files

**Old Pattern:** `playbook_gemini_canvas_automation_v1.0.md`
**New Pattern:** `playbook.gemini_canvas_automation.v1_0.md`

**Steps:**
1. Move to versions/ with new name
2. Create condensed version
3. Create symlinks with .latest. pattern
4. Update glossary and manifest paths
5. Remove old files
