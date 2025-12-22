# AI Documentation Registry

**Version:** 1.0.0
**Created:** 2025-11-13
**Purpose:** Central source of truth for all canonical documentation

---

## Overview

This registry tracks all canonical AI documentation across workspaces, providing a single point of reference for document locations, versions, and migration status.

---

## Active Documents

### Instructions

| ID | Title | Version | Location |
|----|-------|---------|----------|
| instr_operating_principles_v2.0 | Operating Principles | 2.0.0 | ai_general/instructions/ |
| instr_development_v2.0 | Development Protocol | 2.0.0 | ai_general/instructions/ |
| instr_file_conventions_v2.0 | File Format Conventions | 2.0.0 | ai_general/instructions/ |
| instr_claude_use_of_ai_agents_v2.0 | CLI and Codex Delegation | 2.0.0 | instructions/claude/ |

### Protocols

| ID | Title | Version | Location |
|----|-------|---------|----------|
| protocol_coordination_v4.0 | CLI Coordination Protocol | 4.0.0 | knowledge/ |
| protocol_taskCoordination_v5.0 | Task Coordination Protocol | 5.0.0 | specs_and_protocols/ |

### Specifications

| ID | Title | Version | Location |
|----|-------|---------|----------|
| spec_response_footer_v1.1 | Response Footer Specification | 1.1.0 | specs_and_protocols/ |

### Indexes

| ID | Title | Version | Location |
|----|-------|---------|----------|
| index_instructions_v2.0 | AI Instruction Index | 2.0.0 | instructions/ |

---

## Document Fields

Each registered document includes:
- **id**: Unique identifier
- **filename**: Physical filename
- **title**: Human-readable title
- **version**: Semantic version
- **status**: active, deprecated, draft
- **canonical_path**: Intended final location
- **current_location**: Where it currently resides
- **description**: Brief summary
- **tags**: Classification tags
- **last_updated**: Date of last update
- **audience**: Which AI entities should use it

---

## Migration Status

### Pending Moves
- `ai_general/instructions/instr_operating_principles_v2.0.yml` → `ai_general/canonical/instructions/`
- `ai_general/instructions/instr_development_v2.0.yml` → `ai_general/canonical/instructions/`
- `ai_general/knowledge/coordination_system_v4_digest.md` → `ai_general/canonical/protocols/`

### Completed Moves
(none yet)

---

## Usage

AIs should reference this registry to:
1. Locate canonical versions of documents
2. Avoid outdated/duplicate files
3. Check document status before relying on content
4. Track migration progress
