# Instruction: Chat History Condensation v2.0

**Version:** 2.0.0
**Created:** 2026-01-17
**Previous Version:** 1.1.0
**Maintainer:** PianoMan
**Status:** active
**Schema:** schema_condensation_flags.v1.0.yml

## Changes from v1.1

- Added orthogonal condensation flags
- Presets for common use cases
- Flag-aware extraction rules

## Core Principle

This is **semantic extraction with configurable granularity**. Flags control what survives — you extract based on active flags. Think: "Given these flags, what content should remain?"

## Flag Interpretation

Flags are REMOVAL directives (except `preserve_*` which are KEEP directives).

### strip_conversational
When true, remove:
- Greetings, thanks, acknowledgments
- Fillers, caveats, disclaimers, hedging
- Permission seeking, validation statements
- "Let me...", "I'll now...", process narration

Keep: direct statements, actual content, substantive questions.

### strip_explanatory
When true, remove:
- "For example...", "For instance..."
- Analogies, metaphors, "think of it like"
- "Here's why it matters" blocks
- Background context for understanding

Keep: conclusions, decisions, facts, specifications.

### summarize_failures
When true:
- Collapse failed attempts to single line: "Tried X → failed because Y → led to Z"
- Keep the learning, what eventually worked
- Remove verbose debugging sessions, iteration-by-iteration logs

### summarize_rationales
When true:
- Collapse reasoning to: `decision: "what"`, `key_factors: ["factor1", "factor2"]`
- Keep the decision, critical factors
- Remove exploration, weighing pros/cons at length

### strip_meta
When true, remove:
- "Good question", "Let me think"
- "That's interesting", self-corrections
- Reflections on conversation itself

Keep: actual content.

### strip_redundant
When true:
- Keep only final/latest version
- Remove earlier drafts, repeated explanations
- Remove "as mentioned earlier" refs

### preserve_quotes
When true:
- Keep verbatim quoted material even when other flags would remove
- Mark with source attribution
- Use for: relationship content, evidence, exact wording matters

### preserve_artifacts
When true:
- Keep code blocks, ASCII art, diagrams intact
- Only final/working versions (drafts still removed by strip_redundant)

## Extraction Rules (Always Apply)

### Decisions
- What was decided
- Why (rationale) — compress if `summarize_rationales` flag active
- Options rejected (if instructive)

### Outcomes
- What was accomplished
- Final state achieved
- Problems resolved

### Facts
- Names, dates, versions
- Paths, IDs, identifiers
- Error messages with fixes

### Procedures
- Workflows that worked
- Step sequences
- Commands that succeeded

## Always Discard (Regardless of Flags)

- User/assistant role markers (structural)
- Timestamps on individual messages
- Conversation flow markers
- Broken/partial content
- Duplicate exact content

## Output Format

- Format: human-readable YAML
- Extension: `.condensed.yml`

### Structure

```yaml
condensed_metadata:
  source: "{filename}"
  original_tokens: N
  condensed_tokens: N
  reduction: "N%"
  flags_applied:
    strip_conversational: true/false
    strip_explanatory: true/false
    # ... all flags used
  preset: "preset_name"  # if used

summary: |
  One paragraph overview

decisions:
  - decision: "What was decided"
    rationale: "Why"  # or key_factors if summarize_rationales

failures:  # if summarize_failures
  - tried: "what"
    failed_because: "why"
    led_to: "outcome"

procedures:
  - name: "Procedure name"
    steps: [...]

discoveries:
  - "Technical insight"

outcomes:
  - "What was accomplished"

quotes:  # if preserve_quotes
  - speaker: "who"
    content: "verbatim text"
    context: "what it relates to"

artifacts:  # if preserve_artifacts
  - type: "code|ascii_art|diagram|creative"
    description: "What it is"
    content: |
      Exact content preserved
```

## Presets

| Preset | Use When | Flags | Reduction |
|--------|----------|-------|-----------|
| import_continuation | Bringing chat forward for continuation | strip_conversational, strip_meta, strip_redundant, summarize_failures | 40-60% |
| archival | Maximum compression for storage | ALL strip_*, ALL summarize_*, preserve_artifacts | 70-90% |
| relationship_preservation | Emotional content matters | strip_explanatory, strip_meta, strip_redundant, summarize_*, preserve_quotes | 30-50% |
| technical_extraction | Just decisions and code | ALL strip_*, ALL summarize_*, preserve_artifacts | 70-90% |
| minimal | Light touch, keep most content | strip_meta, strip_redundant, preserve_quotes, preserve_artifacts | 15-30% |

## Quality Targets

- **Completeness:** All decisions and outcomes must be recoverable
- **Readability:** Someone unfamiliar should understand what happened
- **Flag compliance:** Applied flags must be reflected in output

### Validation Checklist

- Did I apply all specified flags correctly?
- Are decisions/outcomes preserved regardless of flags?
- Is output valid YAML?
- Could someone understand what happened?

## Forbidden

- Base64/binary encoding (explodes tokens)
- Dropping decisions/outcomes (defeats purpose)
- Ignoring flags (use what was specified)
- Inventing content not in source
