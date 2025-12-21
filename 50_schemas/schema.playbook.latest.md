# Playbook Schema

## Metadata

| Field | Value |
|-------|-------|
| Title | Playbook Schema |
| Version | 1.0.0 |
| Created | 2025-12-19 |
| Maintainer | PianoMan |
| Status | active |
| Category | schema |

## Purpose

Defines the standard structure for playbooks in the ai_root documentation system.
Playbooks are operational guides that document how to accomplish specific tasks or workflows.

## Schema Structure

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| metadata | object | Document metadata (title, version, created, maintainer, status, category) |
| summary | string | One-paragraph description of what the playbook covers |
| when_to_use | array | List of scenarios when this playbook applies |
| prerequisites | object/array | What must be in place before following the playbook |
| steps | object | Numbered steps with action, details, examples |

### Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| monitoring_commands | object | Commands for checking progress/status |
| common_issues | object | Known problems with symptom/fix patterns |
| example_deployment | object | Real-world example with results |
| related_docs | array | Links to related documentation |
| version_history | array | Changelog entries |

## Field Specifications

### metadata

```yaml
metadata:
  title: string          # Human-readable title
  version: string        # Semantic version (X.Y.Z)
  created: date          # ISO date (YYYY-MM-DD)
  maintainer: string     # Owner/maintainer name
  status: enum           # active | draft | deprecated
  category: string       # Always "playbook"
  tags: array            # Optional discovery tags
```

### summary

Single string or multiline block describing the playbook's purpose.
Should answer: "What does this playbook help me do?"

```yaml
summary: |
  How to deploy multiple Claude CLI workers in parallel with AT self-wake 
  monitoring. Desktop Claude orchestrates without doing the work.
```

### when_to_use

Array of scenarios or conditions when this playbook applies.

```yaml
when_to_use:
  - Large task divisible into independent subtasks
  - Same operation across multiple directories/files
  - Work that benefits from parallelism
```

### prerequisites

What must exist or be configured before using the playbook.
Can be object with categories or simple array.

```yaml
# Object form (preferred for complex prerequisites)
prerequisites:
  tools:
    - cliclick: "brew install cliclick"
    - Desktop Commander MCP
  scripts:
    location: ~/path/to/scripts/
    required:
      - script_a.js
      - script_b.sh
  
# Array form (simpler cases)
prerequisites:
  - Chrome with Gemini session
  - Desktop Commander MCP connected
```

### steps

Numbered steps for executing the playbook.
Each step has an action and supporting details.

```yaml
steps:
  1_step_name:
    action: Brief description of what to do
    example: Optional example command or code
    rule: Optional constraint or guideline
    wait: Optional timing/delay instruction
    verification: Optional way to confirm success
  
  2_next_step:
    action: Next action
    script: |
      # Code or commands
      command --flag value
    alternative: |
      # Alternative approach
```

### common_issues

Known problems with symptom/fix pattern.

```yaml
common_issues:
  issue_name:
    symptom: What you observe when this happens
    causes:
      - Possible cause 1
      - Possible cause 2
    fix: How to resolve it
    # or
    solutions:
      - Solution option 1
      - Solution option 2
```

### monitoring_commands

Commands for checking status during execution.

```yaml
monitoring_commands:
  list_sessions: "tmux list-sessions | grep $PREFIX"
  check_output: "tmux capture-pane -t $SESSION -p -S -50"
  count_progress: "ls $OUTPUT_DIR/*.yml | wc -l"
```

### example_deployment

Real-world example showing the playbook in action.

```yaml
example_deployment:
  task: "Condense 54 docs across 7 directories"
  workers: 7
  agent: librarian
  time: "~25 minutes"
  result: "72% reduction, all validated"
  see: "path/to/case_study.yml"
```

## Condensed Version Format

The .condensed.yml version should:

- Include header comment pointing to full version
- Preserve all step actions in compact form
- Collapse examples into inline format where possible
- Maintain essential troubleshooting info
- Target 60-80% token reduction

```yaml
---
# {Title} - Condensed
# Full: versions/{basename}_v{X.Y}.md

metadata:
  title: ...
  version: ...
  
summary: One-line or brief summary

when_to_use:
  - Scenario 1
  - Scenario 2

steps:
  1_name:
    action: What to do
    key_command: "example --flag"
  
  2_name:
    action: Next action

common_issues:
  issue: {symptom: "...", fix: "..."}
```

## Validation Checklist

- [ ] metadata includes all required fields
- [ ] summary clearly explains purpose
- [ ] when_to_use lists applicable scenarios
- [ ] prerequisites are complete and accurate
- [ ] steps are numbered and have action field
- [ ] common_issues use symptom/fix pattern
- [ ] Version in metadata matches filename
