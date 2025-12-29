# Task #XXXX: [Title]

**Type:** [Research | Development | Automation | Peer Review | ...]
**Priority:** [NORMAL | HIGH | CRITICAL]
**Posted:** YYYY-MM-DD HH:MM:00
**Author:** [Who created this task]
**Orchestrator:** [Who manages execution - default: Author]
**Claimed:** [Filled when work begins]
**Assigned:** [Filled when claimed]
**Completed:** [Filled when done]
**Status:** active

---

## Why

Why this task exists. What problem it solves. What it enables.

---

## Goal

What success looks like. The outcome, not the steps.

---

## Deliverables

Explicit outputs expected:

- [Report, script, documentation, assessment, etc.]
- [Format if it matters]

---

## Requirements

Non-negotiable constraints:

- [Must follow pattern X]
- [Must not break Y]
- [Must integrate with Z]

---

## Phases

*Optional. Use when work has natural stages or requires gates.*

### Phase 1: [Name]

**Gate:** [None | Report findings | Wait for approval before proceeding]

**Goal:** [Phase-specific goal]

### Phase 2: [Name]

**Depends on:** Phase 1 [findings | approval]

**Goal:** [Phase-specific goal]

---

## Considerations

Context, knowns, unknowns, preferences, guidance. Not requirements - judgment calls for the worker.

- [Known: ...]
- [Unknown: ...]
- [Preference: ...]
- [Reference: ...]

---

## Testing

How to verify the work is correct:

- [Acceptance criteria]
- [Test cases if applicable]

---

## Error Handling

What to do if things go wrong:

- [Retry? Escalate? Document and continue?]
- [Partial completion acceptable?]

---

## Impacts

What existing systems/docs/scripts may need updating as a result:

- [Or "None anticipated"]

---

## On Completion

- Create `report_final_YYYYMMDD_worker.md` with deliverables
- Notify Orchestrator [via IM or `~/bin/ai/notifications/desktop/notify_desktop_task_complete.sh`]
- [Other completion actions]

---

## Questions/Blockers

Contact Orchestrator via IM: `~/.claude/coordination/instant_messaging/active/task_XXXX/`

---

## Delegation

[Single worker | May subdivide with approval | Orchestrator decides after Phase N]

---

## Execution

*Worker creates `execution_log.md` in task folder to track:*

- Steps taken with estimates vs actuals
- Working notes and observations
- Time tracking

*Template: See protocol_taskCoordination_v5.0.yml for execution_log format.*

---

## Final Report Sections

*Include in `report_final_YYYYMMDD_worker.md`:*

### Results

[Primary deliverable content]

### Recommendations

Observations beyond scope. Potential improvements. Things noticed that warrant future consideration.

### Lessons Learned

Key insights, surprises, mistakes, patterns worth preserving.
