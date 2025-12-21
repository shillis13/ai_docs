# Codex MCP Delegation Guidelines Update

**Date:** 2025-11-23  
**Version:** 2.0 → 2.1  
**Reason:** Operational testing revealed Codex MCP works best with focused, single-task prompts

## Changes Made

### 1. Updated Project Files (Read-Only Context)
- `/mnt/project/instr_claude_use_of_ai_agents.md` (v1.0 → v1.1)
- `/mnt/project/claude_context_quick_notes.md`

### 2. Created New Version on Disk
- **New file:** `instr_claude_use_of_ai_agents_v2.1.yml`
- **Location:** `/Users/shawnhillis/Documents/AI/ai_root/ai_general/instructions/claude/`
- **Updated:** `claude_context_quick_notes.md` in same directory

### 3. Key Updates

#### Added: Codex Operational Constraints Section
```yaml
**Codex Operational Constraints:**
- Works best with focused, single-task prompts
- Avoid multi-step "do 1, 2, 3, and report" instructions
- Break complex workflows into discrete Codex calls
- Don't use approval-policy parameter (causes execution failures)
- For complex orchestration, use CLI Coordination instead
```

#### Updated: "Use Codex When" Section
**Before:**
- Complex multi-step operations (build, test, deploy)
- Task requires autonomous decision-making

**After:**
- Task is focused and single-purpose
- File system exploration or command execution
- Code generation or refactoring (discrete tasks)
- Simple data analysis or transformation

#### Updated: Codex Pattern Example
**Before:** Single complex multi-step call

**After:** Multiple focused calls:
```
codex("Step 1")
[Review results]
codex("Step 2")
[Review results]
codex("Step 3")
```

#### Updated: Quick Reference
**Delegation Hierarchy:**
- Codex MCP - Synchronous, **focused single tasks** (was: "immediate results")
- CLI Coordination - Async work, parallel execution, **complex orchestration** (added)

## Testing Results That Led to Changes

### What Worked ✅
- Simple, focused prompts: `codex("Echo test")`
- Single-task operations: `codex("List files in X")`
- Default parameters (no approval-policy)

### What Failed ❌
- Multi-step instructions: `codex("Do 1, 2, 3, report all")`
- Using `approval-policy` parameter
- Complex orchestration requests

### Root Cause
Codex MCP's architecture favors atomic operations over complex orchestration. For multi-step workflows, either:
1. Make multiple discrete Codex calls
2. Use CLI Coordination for async orchestration

## Migration Path

**For existing patterns:**
- Review any Codex delegation that involves multiple steps
- Break into discrete calls or move to CLI Coordination
- Remove any `approval-policy` parameters from Codex calls

**Future work:**
- May need to update `instr_claude_context_conservations.yml` if it references Codex
- Consider if userPreferences need formal update (currently inline in project context)

## Files to Review in Next Session

1. Check if `instr_claude_context_conservations.yml` needs updates
2. Verify if any other instruction files reference Codex patterns
3. Consider generating updated .md version of v2.1.yml

---

**Status:** ✅ Complete - Guidelines updated based on operational testing
