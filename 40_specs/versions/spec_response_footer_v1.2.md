# Response Footer Specification

**Version:** 1.2.0
**Last Updated:** 2025-12-15
**Maintainer:** PianoMan
**Status:** active

## Summary

Canonical metadata footer appended to every AI response.

## Format

**Delimiter:** ` | `

**Field Order:**
1. AI_Name
2. Persona
3. Proj
4. Chat
5. Timestamp
6. Msg
7. Usage
8. Boot
9. Artifacts
10. Tags

## Field Definitions

| Field | Description | Example |
|-------|-------------|---------|
| AI_Name | Display name for the responding persona | `Claude` |
| Persona | Persona version with optional delta marker (Δ, Δ+, Δ*) | `Persona:v1.0` |
| Proj | Short project identifier (CamelCase or hyphenated) | `Proj:AI-Root` |
| Chat | Human-readable chat title | `Chat:Manifest-Validation` |
| Timestamp | Local timestamp in ISO format with seconds | `2025-12-15 12:30:00` |
| Msg | Zero-indexed assistant message counter | `Msg:12` |
| Usage | Context usage as percentage or NA | `Usage:~28%` |
| Boot | Startup checklist completion status | `Boot:[✓✓✓✓✓]` |
| Artifacts | Count of artifacts created in this message | `Artifacts:1` |
| Tags | 2–8 descriptive keywords, comma-separated | `Tags: footer, validation` |

## Boot Field

Shows startup checklist completion status:

| Symbol | Meaning |
|--------|---------|
| M | manifest_valid - manifests validated against filesystem |
| T | tools_loaded - tools_manifest.condensed.yml read |
| D | dirs_loaded - directory_structure_reference read |
| Y | memory_checked - memory manifest read |
| C | context_file - notes_to_myself.md checked |

**Format:** `Boot:[MTDYC]` where each position is ✓ (done) or - (skipped)

## Rules

### Persona Deltas
- **Δ:** Minor refinement or parameter adjustment
- **Δ+:** Significant update or behavioral expansion
- **Δ*:** Experimental or transient branch
- Only emit delta marker in the message that causes or announces the change

### Usage Field
- Prefer `Usage:~X%` when measurable resource ratio exists
- Fallback to `Usage:NA` when undefined, unbounded, or irrelevant

### Boot Field
- Show after first message where checklist was run
- Show which items completed vs skipped
- Can be omitted after first few messages once checklist established

### Artifacts Field
- Count only artifacts created in the current message
- Typical items: canvas documents, generated files, external attachments

## Examples

**Full Boot:**
```
Claude | Persona:v1.0 | Proj:AI-Root | Chat:Manifest-Validation | 2025-12-15 12:30:00 | Msg:5 | Usage:~15% | Boot:[✓✓✓✓✓] | Artifacts:0 | Tags: checklist, validation
```

**Partial Boot:**
```
Claude | Persona:v1.0 | Proj:AI-Root | Chat:Quick-Question | 2025-12-15 14:00:00 | Msg:1 | Usage:~5% | Boot:[✓✓✓--] | Artifacts:0 | Tags: quick, response
```

**No Boot (continuation):**
```
Claude | Persona:v1.0 | Proj:AI-Root | Chat:Continued | 2025-12-15 15:00:00 | Msg:15 | Usage:~45% | Artifacts:1 | Tags: continuation
```

## Notes
- Boot field can be omitted after first few messages once checklist established
- Messages without Boot field remain valid (pre-v1.2 format)

## Related Documents
- `ai_general/docs/40_specs/spec_startup_checklist.yml`
- `ai_general/scripts/validate_manifests.py`
