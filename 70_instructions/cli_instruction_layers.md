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
| 1. Global | `ai_general/prompts/global/cli_agents.md` | Auto (always) |
| 2. Agent | `ai_general/prompts/agents/{agent}.md` | `-A <agent>` flag |
| 3. Task | `ai_comms/{platform}_cli/tasks/{task}.md` | `-T <task>` flag |
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
-T <task>     # Load Layer 3 (task file) - Claude uses -T, Gemini uses -t
-y / -a       # Auto-approve (-y for Gemini, -a for Claude)
-p            # Print/headless mode
-r            # Resume previous session
```

### Examples

```bash
# All 4 layers
claude_cli.sh -a -A librarian -T req_2205 "Process Claude exports first"
gemini_cli.sh -y -A librarian -t req_2205 "Process Claude exports first"

# Layers 1, 2, 4 only (no task file)
claude_cli.sh -A dev-lead "Review the PR"

# Layers 1, 4 only (ad-hoc)
gemini_cli.sh -y -p "Count files in ai_memories"
```

## Instruction Assembly

Scripts assemble instructions in order:

```bash
GLOBAL="$PROJECT_DIR/ai_general/prompts/global/cli_agents.md"
AGENTS_DIR="$PROJECT_DIR/ai_general/prompts/agents"
TASKS_DIR="$PROJECT_DIR/ai_comms/${platform}_cli/tasks"

INSTRUCTIONS=""

# Layer 1: Global (always)
if [[ -f "$GLOBAL" ]]; then
    INSTRUCTIONS+="$(cat "$GLOBAL")\n\n---\n\n"
fi

# Layer 2: Agent (if -A specified)
if [[ -n "$AGENT" && -f "$AGENTS_DIR/${AGENT}.md" ]]; then
    INSTRUCTIONS+="## Agent: $AGENT\n\n$(cat "$AGENTS_DIR/${AGENT}.md")\n\n---\n\n"
fi

# Layer 3: Task (if -T specified)
if [[ -n "$TASK" && -f "$TASKS_DIR/${TASK}.md" ]]; then
    INSTRUCTIONS+="## Task\n\n$(cat "$TASKS_DIR/${TASK}.md")\n\n---\n\n"
fi

# Layer 4: Prompt (positional arg)
INSTRUCTIONS+="## Immediate Instruction\n\n$PROMPT"
```

## Platform-Specific Notes

### Claude CLI
- Uses `CLAUDE.md` for per-directory context (separate from layers)
- `-a` or `--auto-approve` for auto-approve (maps to `--dangerously-skip-permissions`)
- `-T` or `--task-file` for task layer
- Session persistence via `--resume`, `--continue`, `--named`

### Gemini CLI
- Uses `GEMINI.md` for per-directory context
- `-y` or `--yolo` for auto-approve
- `-t` for task layer
- Session persistence via `--resume` and `/chat save`/`/chat resume`

### Codex CLI
- Inherits from calling environment
- Typically short-lived, task-focused
- May use simplified layer loading

## Implementation Status

| Wrapper | Layered Loading | Auto-Approve Flag | Task Flag |
|---------|-----------------|-------------------|-----------|
| `claude_cli.sh` | ✅ | `-a` | `-T` |
| `gemini_cli.sh` | ✅ | `-y` | `-t` |
| `codex_cli.sh` | ✅ | `-a` | `-T` |

Agent-specific convenience scripts delegate to the "best" platform for that role:

| Wrapper | Agent | Platform | Reason |
|---------|-------|----------|--------|
| `librarian_gemini.sh` | librarian | Gemini | 1M context for corpus |
| `dev_lead_claude.sh` | dev-lead | Claude | Session persistence |
| `custodian_claude.sh` | custodian | Claude | Structure validation |
| `ops_codex.sh` | ops | Codex | Quick synchronous tasks |

```bash
# These are equivalent:
librarian_gemini.sh -y "Process exports"
gemini_cli.sh -y -A librarian "Process exports"
```
