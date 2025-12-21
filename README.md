# AI Documentation System

## Overview

This documentation tree uses a **numbered directory hierarchy** (10-70) representing abstraction levels, with **condensed versions** for token-efficient loading.

## Directory Structure

| Dir | Purpose | Loads |
|-----|---------|-------|
| 10_architecture/ | Foundational concepts | Standalone |
| 20_registries/ | Indexes, inventories | References 10 |
| 30_protocols/ | Communication patterns | References 10-20 |
| 40_specs/ | Detailed specifications | References 10-30 |
| 50_schemas/ | Data structures | References 10-40 |
| 60_playbooks/ | How-to guides | References 10-50 |
| 70_instructions/ | Operational rules | References 10-60 |

**Rule:** Higher numbers may depend on lower numbers, never reverse.

## Condensed Files

Every documentation file has (or can have) a condensed version:

```
spec_foo_latest.yml           # Full version (~500 tokens)
spec_foo_latest.condensed.yml # Condensed (~150 tokens)
```

### What Condensation Preserves
- ALL rules, constraints, thresholds
- Paths, patterns, naming conventions
- Decision logic and criteria
- Edge cases and warnings

### What Condensation Strips
- Verbose prose explanations
- Redundant examples
- Table of contents
- Migration/historical notes

### Resolution Protocol

When loading a `REF:` or `see:` pointer:
1. Parse target path
2. Check if `.condensed.yml` version exists
3. If yes → load condensed
4. If no → load original

To force full version: `REF:full:path/to/file.yml`

See: `30_protocols/protocol_condensed_resolution_latest.yml`

## Token Budget

| Version | Total Tokens | % of 200K Context |
|---------|--------------|-------------------|
| All originals | ~111,000 | 55% |
| All condensed | ~40,000 | 20% |
| **Savings** | **~71,000** | **35%** |

## Load Tiers

Defined in `70_instructions/claude/_knowledge_manifest.yml`:

- **auto**: Load at conversation start (~2-3K tokens)
- **topic**: Load when related topic arises
- **demand**: Load only on explicit request

## Creating New Documentation

1. Create full version: `name_latest.yml` or `name_latest.md`
2. Add metadata with `load_priority: auto|topic|demand`
3. Create condensed version: `name_latest.condensed.yml`
4. Run manifest generator to update indexes

## Maintenance

- **Custodian agent**: Structure, naming, symlinks
- **Librarian agent**: Content, condensation, knowledge curation

## Key Files

- `_knowledge_manifest.yml` - Master index of all documentation
- `30_protocols/protocol_condensed_resolution_latest.yml` - Resolution rules
- `60_playbooks/playbook_parallel_orchestration.yml` - How we condensed 54 files
