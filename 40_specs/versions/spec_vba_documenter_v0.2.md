# VBA Documenter Design Specification

**Version:** 0.2.0
**Date:** 2025-12-17
**Status:** DRAFT - Ready for Review
**Purpose:** Extract VBA documentation, generate Confluence wiki markup with call graphs
**Target Location:** `~/bin/python/src/ai_utils/vba_documenter/`

## Scale Assessment

### Codebase Metrics
- **Files:** 66 (47 standard modules .bas, 19 class modules .cls)
- **Lines of Code:** 48,192
- **Procedures:** 951
- **Estimated Tokens:** 60-80K

### Existing Documentation
- **Style:** Structured block comments with delimiters (`'===`, `'***`)
- **Fields Present:** Function Name, Description, Parameters, Returns, Example Usage
- **Coverage:** Partial (many procedures have descriptors)

## Requirements

### Hard Requirements
1. **R1 - Ease of Use:** Single command generates ready-to-paste markup files
2. **R2 - No Confluence API:** Generate wiki markup files; user copies to Confluence
3. **R3 - Leverage Descriptors:** Parse and use existing function descriptors via regex

## Confluence Structure

### Module Level (H3)
- Module description (extract from header or generate)
- Dependencies callout (collapsible): modules called FROM this module
- Dependents call-in (collapsible): modules that call INTO this module
- Implementation (collapsible): full source code
- Testing (collapsible): test procedures if present

### Method Level (H4)
- Method descriptor (parsed from comment block)
- Implementation (collapsible): method source code

## Architecture

### Components

| Component | Purpose | Output |
|-----------|---------|--------|
| vba_parser | Parse VBA source files | List[Module] with nested Procedure objects |
| call_graph_analyzer | Build dependency graphs | Dict[module_name -> {calls_out, called_by}] |
| descriptor_parser | Parse structured comment blocks | MethodDescriptor dataclass |
| confluence_generator | Generate Confluence wiki markup | String (wiki markup) |
| description_generator | Generate missing descriptions (optional AI) | Enhanced descriptions |

## CLI Interface

### generate
```bash
vba-doc generate <source_path> [options]

Options:
  --output, -o       Output directory (default: ./confluence_docs/)
  --format           Output format [confluence|markdown] (default: confluence)
  --split            Split output [single|per-module] (default: single)
  --include-private  Include private members (default: false)
  --generate-missing AI-generate missing descriptions (default: false)
```

### analyze
```bash
vba-doc analyze <source_path> --report [summary|dependencies|undocumented]
```

### extract-calls
```bash
vba-doc extract-calls <source_path> --format [text|dot|mermaid]
```

## Implementation Plan

| Phase | Name | Effort | Deliverables |
|-------|------|--------|--------------|
| 1 | Core Parser & Data Model | 4-6 hrs | models.py, vba_parser.py, descriptor_parser.py |
| 2 | Call Graph Analysis | 2-3 hrs | call_analyzer.py |
| 3 | Confluence Generator | 3-4 hrs | confluence_generator.py, templates/*.j2 |
| 4 | CLI & Integration | 2-3 hrs | cli.py, __main__.py |
| 5 | Documentation & Polish | 2 hrs | README.md, examples |

**Total Estimated Effort:** 13-18 hours

## User Workflow

1. **Generate:** `vba-doc generate /path/to/VbaFileMethods_v02/ -o ./docs/`
2. **Copy to Confluence:** Open Confluence, create new page, switch to wiki markup editor, paste
3. **Update:** When code changes, re-run generate command

## Platform Recommendation

**Hybrid Approach (Recommended):**
1. Python tool handles: parsing, call graphs, structure extraction
2. Optional AI pass: Generate descriptions for undocumented procedures
3. Best of both worlds: Reliable structure + intelligent descriptions

## Dependencies

### Required
- click: CLI framework
- pyyaml: Config and data serialization
- jinja2: Template rendering

### No External Dependencies Needed
- VBA parsing: Pure regex/string parsing
- Call graph: Static analysis with regex

## Directory Structure

```
vba_documenter/
├── __init__.py
├── __main__.py           # Entry point
├── cli.py                # Click CLI
├── models.py             # Dataclasses
├── parsers/
│   ├── vba_parser.py     # Module/procedure parsing
│   └── descriptor_parser.py
├── analyzers/
│   └── call_graph.py     # Dependency analysis
├── generators/
│   ├── base.py           # Abstract generator
│   └── confluence.py     # Wiki markup output
├── templates/
│   ├── confluence_module.j2
│   ├── confluence_method.j2
│   └── confluence_index.j2
└── tests/
    └── fixtures/
```

## Open Questions

### For User
- Should we detect and document VBA References?
- Naming conventions for test procedures (e.g., Test_*)?
- Preference for call graph format: text list, Mermaid diagram, or both?
- Any modules to exclude from documentation?

### Technical
- Handle line continuations in signatures?
- Parse Attribute statements for additional metadata?
- Support for #Const compiler directives?
