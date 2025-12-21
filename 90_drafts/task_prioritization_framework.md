---
title: Task Prioritization Framework
version: 1.0.0
created: 2025-11-08
purpose: Decision framework for choosing what to work on when overwhelmed
---

# The "Too Many Ideas" Problem - Solution Framework

## Core Principle: Time-Boxing Over Perfection

You have limited attention. Focus on **impact per hour** rather than completeness.

---

## Three-Tier Priority System

### Tier 1: BLOCKING (Do First)
**Definition:** Prevents other work or causes daily friction.

**Current examples:**
- req_1020 already in progress - FINISH IT
- Any task with "BLOCKED_BY" tags referencing incomplete work
- Infrastructure failures affecting daily workflow

**Decision rule:** If it's blocking, it gets done TODAY or explicitly cancelled.

---

### Tier 2: MULTIPLIER (Do Next)
**Definition:** Unlocks multiple future tasks or reduces future friction significantly.

**Current examples:**
- req_1024 (YAML validation) - affects all future YAML work
- req_2111 (terminal-notifier) - enables better automation feedback
- req_2112 (notification fallback design) - foundational for user experience

**Decision rule:** Pick ONE multiplier per week. Complete it before taking on more.

**Warning signs you're avoiding multipliers:**
- Lots of "almost done" tasks
- Same types of problems recurring
- Building workarounds instead of foundations

---

### Tier 3: INTERESTING (Do Later / Batch / Delegate)
**Definition:** Valuable but not urgent. Can wait or be batched.

**Current examples:**
- req_2106, 2107 (packaging questions/summaries) - research tasks
- req_1021 (CLI comms perspective) - documentation/analysis
- Most design/exploration tasks

**Decision rules:**
- Batch similar work (all documentation together, all research together)
- Delegate to CLI/Codex when possible
- Schedule specific "exploration time" rather than mixing with execution work
- Keep a "someday/maybe" list and review monthly

---

## Decision Flowchart

```
New idea arrives:
├─> Does it block current work?
│   ├─> YES → Tier 1 (do now)
│   └─> NO → continue
├─> Does it unblock 3+ future tasks?
│   ├─> YES → Tier 2 (schedule this week)
│   └─> NO → continue
└─> Interesting but not urgent?
    └─> YES → Tier 3 (capture and batch)
```

---

## Applied to Current Backlog

### BLOCKING (Tier 1):
- **req_1020** (in progress) - FINISH THIS FIRST

### MULTIPLIER (Tier 2) - Pick ONE:
- **req_1024**: YAML validation → Affects all YAML workflows
- **req_2112**: Notification fallback design → Foundation for automation UX
- **req_2111**: Terminal-notifier install → Enables better feedback loops

**Recommendation:** req_2112 (notification design) because it's pure design (fast) and unblocks the install task (2111) naturally.

### INTERESTING (Tier 3) - Batch or Delegate:
**Installation cluster** (delegate to CLI as single coordinated task):
- req_2100: Orchestrate parallel installation
- req_2103: Launch codex instances  
- req_2106: Packaging questions
- req_2107: Summarize packaging questions

**Documentation/Design** (batch into one session):
- req_1021: CLI comms perspective
- Various README/doc tasks

---

## Escape Hatches

### "I'm stuck on everything"
→ Pick the SMALLEST task (<1 hour) and complete it fully. Momentum > optimization.

### "New urgent thing appeared"
→ Ask: "Does this block me TODAY?" If no, add to Tier 2/3. If yes, pause current Tier 1 and handle it.

### "Everything seems equally important"
→ Flip a coin between top 2 Tier 2 items. Analysis paralysis is worse than suboptimal choice.

### "I want to work on X but should work on Y"
→ Time-box: "I'll give X 30 minutes, then switch to Y regardless." Often the pull toward X is just exploration itch, satisfied quickly.

---

## Weekly Review Cadence

**Monday:** Review backlog, pick ONE Tier 2 task for the week
**Wednesday:** Check-in - still on track? Adjust if blocked
**Friday:** Close out week - what shipped? What moves to next week?

**Monthly:** Prune Tier 3 - Archive ideas that haven't moved in 30 days

---

## Metrics That Matter

**Good indicators:**
- Completed tasks in last 7 days: >3
- Tasks in "in_progress" simultaneously: ≤2  
- Age of oldest "to_execute" task: <14 days

**Warning indicators:**
- Completed tasks in last 7 days: <2
- Tasks in "in_progress" simultaneously: >3
- Age of oldest "to_execute" task: >30 days
- New tasks added > tasks completed (sustained)

---

## Emergency Reset Protocol

If backlog exceeds 20 items or nothing ships for 2 weeks:

1. **CANCEL everything in Tier 3** - Move to archive/someday
2. **Pick ONE Tier 2** - Only this matters this week
3. **Finish ALL Tier 1** - Or explicitly cancel them
4. **Start fresh** - Only add new tasks after clearing to <5 total

**Remember:** Idea generation is infinite. Execution time is finite. The backlog will ALWAYS be full. That's not a bug, it's a feature. The goal is FLOW, not EMPTY.

---

## Immediate Action (Right Now)

Based on current state:

**NOW:** 
**NEXT:** 
**THEN:** 

**PARK:** Everything else until above three are DONE.

