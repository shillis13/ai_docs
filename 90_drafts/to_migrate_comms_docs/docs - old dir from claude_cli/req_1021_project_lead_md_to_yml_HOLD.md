# Request #1021: Project Lead - MD to YAML Conversion System

**Type:** Project Lead  
**Priority:** NORMAL  
**Posted:** [HOLD - Post after req_1020 plan approved]  
**Duration:** Multi-session (estimate: 4-6 days)

---

## Project Goal

Create comprehensive MD-to-YAML conversion system with schema generation, validation, and index creation tooling. This is a one-time batch conversion project that will:
1. Convert outstanding Markdown digests to YAML for better programmatic access
2. Generate schemas ensuring consistency across file types
3. Standardize metadata placement (title, description, TOC)
4. Create index generation tooling for directories
5. Build symlink gathering utilities for file collections

**Why:** YAML enables better parsing, validation, and programmatic access than Markdown. Schemas maintain consistency. Indexes provide context-cheap overviews.

---

## Scope & Constraints

### In Scope - Core (Must Complete)
1. **Schema recommendations:** Determine if each file type needs separate schema (chat, digests, instructions, specs, protocols, etc.)
2. **Schema creation:** Generate any missing schemas
3. **Conversion tool:** Use and extend `doc_converter.py`
4. **Metadata standardization:** Ensure each YAML has title, description, TOC in consistent location/element

### In Scope - Tooling (Must Complete)
5. **Index generation:** Script that creates `.index.yml` for files in a dir/group, using metadata from #4
6. **Symlink gathering:** Script that finds files of given type under a dir and creates symlinks in target dir

### Out of Scope
- Converting files outside digest directories
- Schema validation of existing YAML files (focus on new conversions)
- Automated re-conversion pipeline (one-time batch operation)

### Technical Constraints
- **Base tool:** `doc_converter.py` (extend/improve as needed)
- **Location:** Work within existing directory structure
- **Follow:** Development protocol (`instr_development_v2.0.yml`)
- **Schema format:** Your recommendation (JSON Schema, YAML schema, or structured comments)

### Directory Context
Target directories for conversion (you'll identify specific files):
- `/Users/shawnhillis/Documents/AI/ai_root/ai_memories/40_digests/`
- `/Users/shawnhillis/Documents/AI/ai_root/ai_general/instructions/`
- `/Users/shawnhillis/Documents/AI/ai_root/ai_general/specs_and_protocols/`
- Other locations TBD during planning

---

## Your Authority

### You Can Decide
- Schema format (JSON Schema, YAML schema, structured comments)
- Whether each file type needs separate schema or shared schema works
- Implementation approach for conversion, indexing, symlink tools
- File organization for schemas and tooling
- Metadata structure within YAML files (as long as title/description/TOC are consistent)
- Index file format and structure

### You Must Follow
- Development protocol standards
- Extend `doc_converter.py` (don't replace it)
- Preserve all content during conversion (no data loss)
- Maintain metadata (dates, versions, tags) from original files

### You Should Get Peer Review
- Schema design decisions (Codex review)
- Conversion algorithm (especially edge case handling)
- Index generation approach
- Metadata structure design

---

## Checkpoints

### Checkpoint 1: Implementation Plan Review (REQUIRED)
**When:** After task breakdown, before implementation starts  
**Deliverable:** Implementation plan document covering:

**Schema Analysis:**
- Which file types exist and their differences
- Schema recommendation (one vs multiple, format choice)
- Rationale for decisions

**Metadata Design:**
- Proposed structure for title/description/TOC
- How it supports search, indexing, context-cheap overview
- Examples from different file types

**Task Breakdown:**
- Core conversion tasks
- Tooling tasks (indexing, symlinks)
- Testing and validation approach
- Batch conversion strategy

**Options Considered:**
- Alternatives evaluated and rejected
- Trade-offs in design decisions

**Success Criteria:**
- How we'll know each task is complete
- Validation approach
- Integration testing plan

**Action:** Post plan to `in_progress/req_1021_*/implementation_plan.md`  
**We will:** Review and approve before you proceed

### Checkpoint 2: Peer Reviews (ONGOING)
**When:** During implementation, major components  
**Deliverable:** Peer review results from Codex

**Focus areas for review:**
- Schema design
- Conversion algorithm (edge cases)
- Metadata structure
- Tooling scripts (index generation, symlinks)

### Checkpoint 3: Final Demo (REQUIRED)
**When:** All implementation complete  
**Deliverable:** Demo package including:

**Implementation Review:**
- Plan adherence (what changed and why)
- Each task completion evidence
- Edge cases handled

**Working Demo:**
- Convert sample .md files (different types)
- Show schema validation working
- Generate index for sample directory
- Create symlink collection
- Show metadata consistency across file types

**Outputs:**
- All converted .yml files
- Schema files
- Index generation script
- Symlink gathering script
- Documentation

**Action:** Post to `in_progress/req_1021_*/final_demo.md`

---

## Success Criteria

### Schema Requirements
- [ ] Schema recommendation document (one vs multiple, format choice)
- [ ] Schemas created for all file types needing conversion
- [ ] Schemas validated against sample files
- [ ] Schema documentation (structure, validation rules, examples)

### Conversion Requirements
- [ ] `doc_converter.py` extended to handle all .md digest formats
- [ ] Handles edge cases (embedded code, special characters, nested structure)
- [ ] Preserves all metadata (dates, versions, tags)
- [ ] No data loss during conversion
- [ ] Batch processing capability
- [ ] All outstanding .md files converted successfully

### Metadata Requirements
- [ ] Title/description/TOC in consistent location across all file types
- [ ] Structure supports search and indexing
- [ ] Provides context-cheap overview without reading full content
- [ ] Examples demonstrate pattern across different file types

### Tooling Requirements
- [ ] Index generation script works for any directory/group
- [ ] Indexes use metadata from converted files (#4)
- [ ] Symlink gathering script finds files by type
- [ ] Config file support for custom groupings
- [ ] Documentation and usage examples

### Quality Requirements
- [ ] Code follows development protocol
- [ ] Edge case testing complete
- [ ] Peer review by Codex completed
- [ ] Documentation complete

---

## Context Files & Resources

**Project files available:**
- `/Users/shawnhillis/Documents/AI/ai_root/ai_general/instructions/instr_development_v2.0.yml` - Development protocol
- `doc_converter.py` - Base conversion tool (location TBD during planning)
- `/Users/shawnhillis/Documents/AI/ai_root/ai_memories/40_digests/todos/task_md_to_yml_conversion/notes.md` - Original task notes

**Target directories to survey:**
- `ai_memories/40_digests/` - Memory digests
- `ai_general/instructions/` - Instruction files
- `ai_general/specs_and_protocols/` - Specifications
- Other locations you identify during analysis

**Related files:**
- `instr_file_conventions_v2.0.yml` - File format conventions (in project context)
- Existing YAML files as examples of target format

---

## Subtask Management

You are authorized to:
- Break this into subtasks (schema, conversion, tooling, testing)
- Post subtasks to coordination system if delegation needed
- Specify which CLI instance handles specific tasks
- Manage internal task list if work stays within your instance

**Suggested subtask areas:**
1. File type analysis and schema design
2. Conversion tool enhancement
3. Metadata standardization
4. Index generation tooling
5. Symlink gathering tooling
6. Batch conversion execution
7. Testing and validation

Report progress at natural milestones.

---

## Getting Help

**If blocked:**
1. Use Codex (with `--dangerous` flag) for:
   - Schema design review
   - Conversion algorithm validation
   - Tooling approach review
   - Edge case debugging

2. Ask us via coordination response if:
   - Scope ambiguity (which files to convert)
   - Design decisions need user input
   - Trade-offs need prioritization

**Codex usage:** Spawn as needed for peer review and unblocking. Document consultations.

---

## Notes

**Why this matters:**
- YAML enables programmatic access and validation
- Schemas ensure consistency across large file corpus
- Standardized metadata supports search and indexing
- Index generation provides context-cheap navigation
- Symlink tooling enables flexible file groupings

**Key design goals:**
- No data loss during conversion
- Maintain backward compatibility (original .md files stay until validated)
- Tooling should be reusable for future file organization
- Metadata structure should be extensible

**Potential added scope decision points:**
- Should converted files replace originals or live alongside?
- Should index files use symlinks or direct references?
- Config file format for custom groupings?

These can be addressed during implementation planning.

---

**Expected Timeline:** 4-6 days  
**Project Lead:** CLI instance claiming this task  
**Oversight:** PianoMan + Desktop Claude  
**Peer Review:** Codex  
**Launch:** After req_1020 (Chat Chunker) plan approved (staggered launch)
