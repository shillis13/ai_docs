# CLI Agent Instruction Layers

## Overview

All CLI agents (Claude, Gemini, Codex) use a consistent 4-layer instruction model. Layers are concatenated in order, with later layers able to override or extend earlier ones.

## Layer Architecture

```
┌─────────────────────────────────────────┐
│ Layer 4: Prompt (immediate task)        │  ← Most specific
├─────────────────────────────────────────┤
│ Layer 3: Task File (coordination task)  │
├─────────────────────────────────────────┤
│ Layer 2: Agent Role (librarian, etc.)   │
├─────────────────────────────────────────┤
│ Layer 1: Global (all agents)            │  ← Most general
└─────────────────────────────────────────┘
```

## File Locations

| Layer | Location | Loaded Via |
|-------|----------|------------|
| 1. Global | `ai_general/instructions/global/all_cli_agents.md` | Auto (always) |
| 2. Agent | `ai_general/prompts/agents/{agent}_instructions.md` | `-A <agent>` flag |
| 3. Task | `ai_comms/{cli}/tasks/{task_file}.md` | `-t <task>` flag |
| 4. Prompt | Command line argument | Positional arg |

## Layer Contents

### Layer 1: Global (all_cli_agents.md)

Applies to ALL CLI agents regardless of role:
- User preferences (brutal honesty, no moral judgment)
- Workspace conventions (file paths, naming)
- Communication protocols (how to report, coordinate)
- Quality standards (validation, documentation)
- Tool usage patterns (when to delegate)

### Layer 2: Agent Role

Role-specific expertise and responsibilities:
- Primary tasks and scope
- Key scripts and tools for this role
- Domain knowledge
- Workflow patterns
- Output expectations

Example roles: `librarian`, `dev_lead`, `custodian`, `analyst`

### Layer 3: Task File

Specific work item from coordination system:
- Task ID and metadata
- Background/context
- Goal and deliverables
- Requirements and constraints
- Success criteria

### Layer 4: Prompt

Immediate instruction:
- Specific action to take
- Parameters or variations
- Priority overrides

## CLI Flag Reference

### Common Flags (all wrappers)

```bash
-A <agent>    # Load Layer 2 (agent role)
-t <task>     # Load Layer 3 (task file)
-y            # Auto-approve (YOLO mode)
-p            # Print/headless mode
-r            # Resume previous session
```

### Examples

```bash
# All 4 layers
gemini_cli.sh -y -A librarian -t req_2205 "Process Claude exports first"

# Layers 1, 2, 4 only (no task file)
claude_cli.sh -A dev_lead "Review the PR"

# Layers 1, 4 only (ad-hoc)
codex_cli.sh -p "Count files in ai_memories"
```

## Instruction Assembly

Scripts assemble instructions in order:

```bash
INSTRUCTIONS=""

# Layer 1: Global (always)
if [[ -f "$GLOBAL_INSTRUCTIONS" ]]; then
    INSTRUCTIONS+="$(cat "$GLOBAL_INSTRUCTIONS")\n\n---\n\n"
fi

# Layer 2: Agent (if -A specified)
if [[ -n "$AGENT" && -f "$AGENT_FILE" ]]; then
    INSTRUCTIONS+="## Agent: $AGENT\n\n$(cat "$AGENT_FILE")\n\n---\n\n"
fi

# Layer 3: Task (if -t specified)
if [[ -n "$TASK" && -f "$TASK_FILE" ]]; then
    INSTRUCTIONS+="## Task\n\n$(cat "$TASK_FILE")\n\n---\n\n"
fi

# Layer 4: Prompt (positional arg)
INSTRUCTIONS+="## Immediate Instruction\n\n$PROMPT"
```

## Platform-Specific Notes

### Claude CLI
- Uses `CLAUDE.md` for per-directory context (separate from layers)
- `--dangerously-skip-permissions` for YOLO mode

### Gemini CLI
- Uses `GEMINI.md` for per-directory context
- `--yolo` or `-y` for auto-approve

### Codex CLI
- Inherits from calling environment
- Typically short-lived, task-focused

## Migration

Existing agent-specific scripts (e.g., `gemini_librarian.sh`) become convenience wrappers:

```bash
# gemini_librarian.sh becomes:
gemini_cli.sh -A librarian "$@"
```
