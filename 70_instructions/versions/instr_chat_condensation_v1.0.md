# Chat History Condensation Instructions

**Version:** 1.0  
**Purpose:** Guidelines for condensing chat history chunks to semantic summaries

---

## Goal

Reduce the **token count** required to represent conversation content while preserving all meaningful information.

**Why token reduction matters:** Condensed files are loaded into LLM context windows - Claude (200K), ChatGPT, and other systems with limited context. Every token saved means more content can be processed together. Even if you're running on Gemini with 1M tokens, the output must be efficient for systems that will consume it later.

**DO NOT use compression or encoding.** Base64, gzip, zlib, etc. reduce BYTES but dramatically INCREASE TOKENS. A base64 blob is the worst possible format for LLM consumption.

A condensed file must be:
- **Human-readable YAML** - no encoded blobs
- **Semantically complete** - all meaningful information preserved
- **Token-efficient** - 60-90% fewer tokens than original

---

## What to Preserve

### Always Keep
- **Decisions** - choices made and their rationale
- **Procedures** - workflows, processes, step sequences established
- **Technical discoveries** - insights, patterns, solutions found
- **Problems and solutions** - what broke, what fixed it
- **Configuration changes** - settings, parameters, values chosen
- **Outcomes** - what was accomplished, final state
- **Key facts** - names, dates, versions, paths, identifiers
- **Code that works** - final working implementations
- **Commands that succeeded** - actual syntax used

### Preserve with Context
- Error messages (with what caused and resolved them)
- Decisions that were rejected (if rationale is instructive)
- Constraints discovered (limits, blockers, requirements)

---

## What to Remove

### Always Remove
- Conversational filler ("Let me...", "I'll now...", "Sure, I can...")
- Politeness exchanges ("Thanks!", "You're welcome", "Great question")
- Thinking-out-loud that led nowhere
- Failed attempts superseded by working solutions
- Redundant iterations (keep final version only)
- Verbose explanations when terse summary suffices
- Role markers (user/assistant labels) - context makes speaker clear

### Remove When Redundant
- Examples that duplicate already-stated concepts
- Repeated confirmations of the same point
- Step-by-step narration when outcome suffices

---

## Quality Criteria

### Token Reduction
- **Target:** 70-90% reduction from original
- **Minimum acceptable:** 60% reduction
- **Exception:** Very dense technical content may only achieve 40-50%

### Data Integrity
- No semantic information lost
- All key facts recoverable from condensed version
- Someone reading only the condensed version gets complete picture

### Structure
- Valid YAML that parses without errors
- Navigable sections (can find specific topics)
- Consistent formatting within file

---

## Output Format

Condensed files use `.condensed.yml` extension alongside the source file.

```yaml
condensed:
  source: "{original_filename}"
  original_tokens: N
  condensed_tokens: N
  reduction: "N%"

summary: |
  One paragraph describing what this conversation covered.

decisions:
  - decision: "What was decided"
    rationale: "Why (if recorded)"
  
procedures:
  - name: "Procedure name"
    purpose: "What it accomplishes"
    steps:
      - Step 1
      - Step 2

discoveries:
  - "Technical insight or pattern learned"
  - "Unexpected behavior found"

outcomes:
  - "What was accomplished"
  - "Final state achieved"

# Optional sections as relevant:
code_artifacts:
  - description: "What this code does"
    language: python
    code: |
      # Final working version only

errors_resolved:
  - error: "Error message"
    cause: "What caused it"
    fix: "What resolved it"

configuration:
  - setting: "Name"
    value: "Value chosen"
    reason: "Why this value"
```

Adapt sections to content - omit empty sections, add domain-specific sections as needed.

---

## Anti-Patterns

### DO NOT: Over-Condense
Do not strip so much that meaning is lost. The condensed version must stand alone.

**Bad:** `"Fixed the bug"`  
**Good:** `"Fixed race condition in task claiming by using atomic file moves instead of read-then-write"`

### DO NOT: Under-Condense
Do not preserve conversational back-and-forth. Extract the meaning.

**Bad:** `"User asked X, Assistant replied Y, User clarified Z..."`  
**Good:** The final understanding/decision from that exchange

### DO NOT: Encode or Compress
**Never use base64, gzip, zlib, or any binary encoding.** This is critical. Encoding reduces bytes but explodes token count. These files must load efficiently into LLM context windows.

**Reject immediately if you see:** `z: eJzt...` or any encoded blob field.  
**Correct output:** Human-readable YAML with natural language throughout.

### DO NOT: Drop Identifiers
Do not remove IDs, timestamps, paths, or versions. These enable tracing and debugging.

---

## Validation Checklist

Before considering a file condensed:

- [ ] Output is human-readable YAML (no base64, no encoded blobs)
- [ ] YAML parses without errors
- [ ] All decisions from original are represented
- [ ] All working code/commands preserved
- [ ] All key identifiers retained
- [ ] No "what happened?" questions unanswerable from condensed version
- [ ] Token reduction ≥60%

---

## Related Documents

- `task_template_condensation_with_codex_review.md` - Task template for batch condensation
- `librarian_wave.md` - Agent instructions for parallel condensation waves
