# Instruction: TODO System

**Version:** 1.0.0  
**Last Updated:** 2026-01-02  
**Maintainer:** PianoMan  
**Status:** active  
**Load Priority:** topic

## Purpose

Document how AI agents should create TODO items for discovered work that falls outside the scope of their current task or prompt.

## When to Create a TODO

Create a TODO when you discover:
- **Required work** that blocks or enables other goals but isn't your current assignment
- **Technical debt** that should be addressed but not now
- **Missing infrastructure** needed for future capabilities
- **Follow-up work** from completed tasks that wasn't in original scope

Do NOT create TODOs for:
- Observations without clear action (use feedback log)
- Work within your current task scope (just do it)
- Questions needing user input (ask directly)

## TODO Location

All TODOs live in: `ai_general/todos/`

Structure:
```
ai_general/todos/
├── todo_NNNN_short_description/    # Active TODOs
│   ├── notes.md                    # Main content
│   └── scripts/                    # Supporting files
├── completed/                       # Done TODOs
├── trash/                          # Abandoned TODOs
└── template_todo_item/             # Template for new items
```

## Creating a TODO

### Option 1: Using todo_mgr (preferred)

```bash
todo_mgr
# Interactive mode - follow prompts to create new item
```

### Option 2: Manual Creation

1. Determine next number: `ls ai_general/todos/ | grep -E '^todo_[0-9]+' | sort -t_ -k2 -n | tail -1`
2. Create directory: `mkdir ai_general/todos/todo_NNNN_short_description`
3. Copy template: `cp ai_general/todos/template_todo_item/notes.md ai_general/todos/todo_NNNN_short_description/`
4. Fill in the template fields

## TODO Template Fields

```markdown
# todo_NNNN_description

**Created:** YYYY-MM-DD  
**Updated:** YYYY-MM-DD

## Description
What this TODO accomplishes - be specific

## Requirements
- Concrete requirement 1
- Concrete requirement 2

## Dependencies
**Parent todo:** [path or "none"]
**Depends on:** [other todos/tasks or "none"]
**Related:** [reference links]

## Outputs
- Expected deliverable 1
- Expected deliverable 2

## Done When
- [ ] Specific completion criterion 1
- [ ] Specific completion criterion 2

## Notes
YYYY-MM-DD: Created during [context]. Discovered while [what you were doing].
```

## Guidelines

1. **Clear title** - `todo_NNNN_verb_noun` format (e.g., `todo_0045_update_glossary`)
2. **Actionable description** - What needs to happen, not just what's wrong
3. **Define done** - Specific, checkable completion criteria
4. **Note context** - Why this matters, where it came from
5. **Link related work** - Reference tasks, chats, or other TODOs

## TODO Lifecycle

```
Created (Triaging)
    ↓
Needs Research / Needs Derivation / Ready
    ↓
In Progress
    ↓
Completed → moved to completed/
    or
Abandoned → moved to trash/
```

## From Agent's Perspective

When you discover out-of-scope work:

1. **Assess urgency** - Does this block current work? If yes, report to orchestrator
2. **Check existing TODOs** - Might already be tracked
3. **Create TODO** - Use todo_mgr or manual process
4. **Log briefly** - Note in your session output that you created a TODO
5. **Continue work** - Don't get derailed by the discovery

Example session note:
```
[NOTE] Created todo_0045_update_glossary - discovered missing "pulse automation" term while condensing December exports.
```
