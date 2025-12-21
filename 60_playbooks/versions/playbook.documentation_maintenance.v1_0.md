# Documentation Maintenance Playbook

## Metadata

| Field | Value |
|-------|-------|
| Title | Documentation Maintenance Playbook |
| Version | 1.0.0 |
| Created | 2025-12-19 |
| Maintainer | PianoMan |
| Status | active |
| Category | playbook |

## Summary

How to create, update, and properly register documentation in the ai_root workspace.
Ensures consistent file structure, versioning, symlinks, and cross-reference updates.

## When to Use

- Creating new documentation (playbooks, specs, schemas, protocols, instructions)
- Updating existing documentation with significant changes
- Adding new terms or concepts that need discovery via glossary
- Refactoring documentation structure

## Prerequisites

- Write access to ai_root workspace
- Understanding of doc type directories (10_architecture through 70_instructions)
- Familiarity with YAML condensation patterns

## Directory Structure

```
ai_general/docs/{doc_type}/
├── versions/
│   ├── {basename}_v1.0.md           # Canonical source (versioned)
│   ├── {basename}_v1.0.condensed.yml # Condensed version (versioned)
│   ├── {basename}_v1.1.md           # Next version
│   └── {basename}_v1.1.condensed.yml
├── {basename}_latest.md → versions/{basename}_v1.1.md              # Symlink to latest
├── {basename}_latest.condensed.yml → versions/{basename}_v1.1.condensed.yml  # Symlink to latest condensed
└── ... other files ...
```

## Steps

### Step 1: Create Canonical Source

**Action:** Write the full documentation in Markdown format.

**Location:** `ai_general/docs/{doc_type}/versions/{basename}_v{X.Y}.md`

**Requirements:**
- Include metadata section (title, version, created, maintainer, status)
- Follow established structure for doc type (see existing examples)
- Use ASCII tables per instr_writing.condensed.yml
- Version number in filename matches metadata version

**Example:**
```bash
# New playbook
ai_general/docs/60_playbooks/versions/playbook_my_workflow_v1.0.md

# New schema
ai_general/docs/50_schemas/versions/schema_my_format_v1.0.yml
```

### Step 2: Create Condensed Version

**Action:** Create a condensed YAML version (~60-80% smaller).

**Location:** `ai_general/docs/{doc_type}/versions/{basename}_v{X.Y}.condensed.yml`

**Requirements:**
- Preserve all essential information
- Use YAML structure for efficient parsing
- Include pointer to full version in header
- Target 60-80% token reduction

**Condensation patterns:**
- Convert prose to structured YAML
- Use abbreviations where clear (e.g., "desc" for "description")
- Collapse examples into compact format
- Remove redundant explanations

### Step 3: Create/Update Symlinks

**Action:** Create symlinks at doc_type level pointing to latest versions.

**Commands:**
```bash
cd ai_general/docs/{doc_type}/

# For markdown
ln -sf versions/{basename}_v{X.Y}.md {basename}_latest.md

# For condensed
ln -sf versions/{basename}_v{X.Y}.condensed.yml {basename}_latest.condensed.yml
```

**Why symlinks:**
- References use `{basename}_latest.md` or `{basename}.condensed.yml`
- Updating version only requires changing symlink target
- No need to update all references when version changes

### Step 4: Update Reference Files

**Action:** Evaluate and update cross-reference files as needed.

**Files to consider:**

1. **Knowledge Manifest** (`docs/70_instructions/claude/_knowledge_manifest.yml`)
   - Add entry if doc should be discoverable
   - Set appropriate load_sequence (auto/topic/demand/execution)
   - Add triggers and abstract for topic-loaded docs

2. **Document Registry** (`docs/20_registries/document_registry.yml`)
   - Add entry with id, filename, title, version, status, path
   - Include tags and audience

3. **Glossary** (`docs/20_registries/glossary_knowledge_index.condensed.yml`)
   - Add terms that should trigger loading this doc
   - Format: `term: {d: "brief description", r: "path/to/doc.condensed.yml"}`
   - Update term count in header comment
   - Add to appropriate domain list

**Decision criteria:**
- New concept/term? → Add to glossary
- Should Claude discover this? → Add to manifest with triggers
- Formal documentation? → Add to registry

### Step 5: Verify

**Action:** Confirm all pieces are in place.

**Checklist:**
- [ ] Canonical .md exists in versions/ with version number
- [ ] Condensed .yml exists in versions/ with matching version
- [ ] Symlinks exist at doc_type level and point correctly
- [ ] Manifest entry added (if applicable) with correct path
- [ ] Glossary terms added (if applicable)
- [ ] Registry entry added (if applicable)

**Verify symlinks:**
```bash
ls -la ai_general/docs/{doc_type}/{basename}*
# Should show symlinks pointing to versions/
```

## Common Issues

### Symlink points to wrong version

**Symptom:** References load outdated content.

**Fix:** Update symlink target:
```bash
ln -sf versions/{basename}_v{NEW}.md {basename}_latest.md
```

### Glossary term doesn't trigger load

**Symptom:** Claude doesn't find doc when topic mentioned.

**Fix:** 
- Check term is in glossary with correct path
- Verify path in `r:` field is correct relative path
- Ensure condensed file exists at that path

### Version mismatch

**Symptom:** Metadata version doesn't match filename version.

**Fix:** Keep filename version and metadata.version synchronized.

## Example: Creating a New Playbook

```bash
# 1. Create canonical source
vim ai_general/docs/60_playbooks/versions/playbook_my_workflow_v1.0.md

# 2. Create condensed version
vim ai_general/docs/60_playbooks/versions/playbook_my_workflow_v1.0.condensed.yml

# 3. Create symlinks
cd ai_general/docs/60_playbooks/
ln -sf versions/playbook_my_workflow_v1.0.md playbook_my_workflow_latest.md
ln -sf versions/playbook_my_workflow_v1.0.condensed.yml playbook_my_workflow_latest.condensed.yml

# 4. Update glossary (if new terms)
vim ai_general/docs/20_registries/glossary_knowledge_index.condensed.yml
# Add: my_workflow: {d: "Brief description", r: "60_playbooks/playbook_my_workflow_latest.condensed.yml"}

# 5. Update manifest (if should be discoverable)
vim ai_general/docs/70_instructions/claude/_knowledge_manifest.yml
# Add entry under appropriate load_sequence
```

## Related Documentation

- `instr_writing.condensed.yml` - Writing standards (tables, formatting)
- `SCHEMA_EVOLUTION_GUIDE.md` - Schema versioning patterns
- `instr_file_conventions.condensed.yml` - General file conventions
