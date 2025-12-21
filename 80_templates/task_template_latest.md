# Task Template v1.1

---

## Philosophy: Goal-Oriented, Not Prescriptive

This template embodies a fundamental shift in how we delegate work to AI agents.

### The Old Way (Prescriptive)
```
1. Run validate.sh on each file
2. Move valid files to output/
3. Run chunker.py with --size 4000
4. Report count of processed files
```

This assumes the author knows the best path. The worker executes steps without understanding why. If something unexpected happens, they have no basis for judgment.

### The New Way (Goal-Oriented)
```
Goal: All conversations validated, chunked, and organized for retrieval.

Requirements:
- Chunks must not exceed 4000 tokens
- Invalid files quarantined with documented reason

Considerations:
- Chunker should break on assistant messages, not user messages
- Watch for escaped unicode - indicates upstream parsing failure
```

This trusts the worker's intelligence. They understand the outcome needed and the constraints. They can adapt when reality doesn't match expectations. They can catch problems the author didn't anticipate.

### Why This Matters

An intelligent agent with good context will often find better solutions than rigid procedures allow. More importantly, they can **notice when something is wrong** - but only if they understand what "right" looks like.

A worker following steps will execute them perfectly and produce garbage if the steps are flawed. A worker pursuing a goal with understanding will stop and say "this output doesn't look right."

### Key Principles

1. **Goal describes WHAT, not HOW** - The outcome, not the steps to get there
2. **Requirements are hard constraints** - Non-negotiable. Must be true for success.
3. **Considerations are soft guidance** - Context for judgment. "Prefer X" not "Always X"
4. **Examples show good AND bad** - Workers need to recognize failure, not just success
5. **Trust intelligence, provide context** - Don't script; inform and empower

### Anti-Patterns to Avoid

| Don't | Do Instead |
|:------|:-----------|
| List steps in Goal section | Describe the end state |
| Make everything a Requirement | Use Considerations for preferences |
| Assume worker knows what "good" looks like | Provide examples of success and failure |
| Write procedures disguised as goals | Ask: "Could a smart person achieve this differently?" |

---

# Task #XXXX: [Title]

**Type:** [Research | Development | Automation | Peer Review | ...]  
**Priority:** [NORMAL | HIGH | CRITICAL]  
**Posted:** YYYY-MM-DD HH:MM  
**Author:** [Who created this task]  
**Orchestrator:** [Who manages execution - default: Author]  
**Claimed:** [Filled when work begins]  
**Assigned:** [Filled when claimed]  
**Completed:** [Filled when done]

---

## Why

Why this task exists. What problem it solves. What it enables.

*This gives the worker motivation and context for judgment calls.*

---

## Goal

What success looks like. The outcome, not the steps.

*Test: Can you verify success without knowing HOW it was achieved? If you're describing process, move it to Considerations.*

**Good:** "All chat exports converted to v2 schema, validated, and organized by platform/date."

**Bad:** "Run converter.py on each file, then run validate.sh, then move to output/."

---

## Deliverables

Explicit outputs expected:
- [Report, script, documentation, assessment, etc.]
- [Format if it matters]

---

## Requirements

Non-negotiable constraints. These MUST be true for success.

- [Must follow pattern X]
- [Must not break Y]  
- [Output must include Z]

*Keep this list short. If everything is required, nothing is prioritized.*

---

## Success Looks Like

Examples of correct output. What should the worker see if things are working?

```
# Example: A well-formed chunk file
- Filename: {title_slug}_chunk_001.yml
- Content has no escaped unicode (\uXXXX)
- Ends on assistant message
- Size between 3000-5000 tokens
```

---

## Failure Looks Like

Examples of incorrect output. What patterns indicate problems?

```
# Red flags to watch for:
- Filename contains source artifacts: 0181_, _v2, ChatGPT-
- Content has literal \n strings instead of newlines
- Chunk breaks after user message
- Wild size variance (2KB next to 45KB)
```

*This section is critical. Workers can't catch problems they don't know to look for.*

---

## Considerations

Context, guidance, preferences. Not requirements - judgment calls for the worker.

- [Known: ...]
- [Unknown: ...]
- [Preference: ...]
- [If you encounter X, consider Y...]
- [Reference: ...]

*These inform decisions but don't mandate them.*

---

## Phases

*Optional. Use when work has natural stages or requires gates.*

### Phase 1: [Name]

**Gate:** [None | Report findings | Wait for approval before proceeding]

**Goal:** [Phase-specific goal - still outcome, not steps]

### Phase 2: [Name]

**Depends on:** Phase 1 [findings | approval]

**Goal:** [Phase-specific goal]

---

## Testing

How to verify the work is correct:
- [Acceptance criteria]
- [Spot-check procedures]

---

## Error Handling

What to do when things go wrong:
- [Retry? Escalate? Document and continue?]
- [Partial completion acceptable?]
- [When to stop and ask vs. proceed with judgment?]

---

## On Completion

- Create `report_final_YYYYMMDD_worker.md` with deliverables
- Notify Orchestrator
- [Other completion actions]

---

## Questions/Blockers

Contact Orchestrator via IM or documented channel.

---

## Execution Log

*Worker creates execution notes tracking:*
- What was attempted and what happened
- Observations and anomalies noticed
- Decisions made and rationale

---

## Final Report Sections

### Results
[Primary deliverable content]

### Anomalies Noticed
[Things that seemed wrong or unexpected - even if handled]

### Recommendations  
[Observations beyond scope. Improvements for next time.]

### Lessons Learned
[Key insights worth preserving for future similar tasks]
