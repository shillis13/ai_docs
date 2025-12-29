# Index File Schema Specification

**Slug:** schema_index_files_v1
**Current Version:** 1.2.0
**Last Updated:** 2025-11-01
**Canonical Example:** `instructions/index_instructions.yml`

## Overview

Defines the canonical YAML structure used by all `index_*.yml` manifests that describe deployable and reference assets inside this repository. This schema is authoritative for Codex CLI coordination and related automation.

## Top-Level Fields

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| category | yes | string | Lowercase identifier (e.g., `instructions`, `specs`). Must match directory name. Pattern: `^[a-z0-9_]+$` |
| version | yes | semver | Semantic version for the index document (MAJOR.MINOR.PATCH) |
| last_updated | yes | date | ISO-8601 date for most recent manual update (YYYY-MM-DD) |
| maintainer | yes | string | Maintainer name, username, or contact |
| files | yes | list | Ordered list of file metadata entries |
| subdirs_deployable | yes | boolean | Whether subdirectories contain deployable assets |

## File Entry Structure

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| name | yes | string | Filename relative to category root |
| description | yes | string | Concise human-readable synopsis |
| size_bytes | yes | integer | Exact file size in bytes |
| estimated_tokens | yes | integer | Approximate token count for LLM cost planning |
| status | yes | enum | Lifecycle state: `active`, `auxiliary`, `archived`, `draft` |
| audience | yes | enum | Intended audience: `all`, `internal`, `codex`, `claude`, `chatgpt`, `system` |
| deploy | yes | mapping | Platform-specific deployment toggles (boolean keys) |

### Deploy Keys
- claude_desktop
- chatgpt
- codex
- claude_cli
- gemini
- grok
- perplexity

## Status Values

| Status | Meaning |
|--------|---------|
| active | Current and in primary use |
| auxiliary | Supportive context, deploy selectively |
| archived | Retained for history, not deployed unless requested |
| draft | Work-in-progress awaiting promotion |

## Audience Values

| Audience | Meaning |
|----------|---------|
| all | Safe for every assistant surface |
| internal | Restrict to maintainers unless needed elsewhere |
| codex | Tailored for Codex agents and developers |
| claude | Optimized for Claude surfaces |
| chatgpt | Optimized for ChatGPT surfaces |
| system | Machine-generated artifacts, never deploy by default |

## Validation Rules

1. `estimated_tokens` should be derived from file size using consistent multiplier, rounded up
2. `status=archived` implies all deploy flags are false
3. At least one deploy flag must be true for `status=active` entries
4. `files` entries should be ordered by importance, then alphabetically

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.2.0 | 2025-11-01 | Expanded schema with explicit enumeration tables, added validation rules |
| 1.1.0 | 2025-04-12 | Clarified `subdirs_deployable`, introduced `draft` status |
| 1.0.0 | 2024-10-05 | Initial publication |
