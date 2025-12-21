# Task Orchestration Workflow - ASCII Reference

**Version:** 1.0.0  
**Date:** 2025-12-13  
**Status:** active - aligned with Protocol v7.0  
**Source:** PianoMan's visual workflow (Google Sheets)

## Overview

This document visualizes multi-level task orchestration where Claude Desktop 
delegates to CLI workers, who may further delegate to other workers.

---

## Step 1: Initial Task Assignment

Claude receives task, determines orchestration needed.

```
ai_comms/claude/
└── to_execute/
    └── orch_o1_task.md          ◄── Task file ready for claiming
```

---

## Step 2: Claude Claims & Decomposes

2.1. Claude claims task (moves to in_progress, adds PID)
2.2. Claude creates subtasks for delegation

```
ai_comms/claude/
└── in_progress/
    └── orch_o1_task/            ◄── 2.1 Task becomes directory
        ├── orch_o1_task.$$.md   ◄── 2.1 PID claim marker
        ├── orch_o1_task_t1.md   ◄── 2.2 SubTask 1 (for claude_cli)
        └── orch_o1_task_t2.md   ◄── 2.2 SubTask 2 (for codex_cli)
```

---

## Step 3: Orchestrator Distributes to Workers

Claude creates task dirs under each worker's to_execute/

```
ai_comms/claude_cli/               ai_comms/codex_cli/
└── Tasks/                         └── Tasks/
    └── to_execute/                    └── to_execute/
        └── orch_o1_task_t1/               └── orch_o1_task_t2/
            └── orch_o1_task_t1.md             └── orch_o1_task_t2.md
            ▲                                  ▲
            └── orchestrator creates ──────────┘
```

---

## Step 4: Workers Claim & Process

4.1. Workers claim by moving to in_progress + PID stamp
4.2. claude_cli (clA) determines further decomposition needed

```
ai_comms/claude_cli/ [clA]         ai_comms/codex_cli/ [coA]
└── Tasks/                         └── Tasks/
    └── in_progress/                   └── in_progress/
        └── orch_o1_task_t1/               └── orch_o1_task_t2/
            ├── orch_o1_task_t1.$$.md          └── orch_o1_task_t2.$$.md
            ├── orch_o1_task_t1.1.md  ◄── 4.2 SubTask
            └── orch_o1_task_t1.2.md  ◄── 4.2 SubTask
```

---

## Step 5: Further Delegation & First Completion

5.1. clA delegates subtasks to codex_cli
5.2. coA completes original task

```
ai_comms/claude_cli/               ai_comms/codex_cli/ [coA]
└── Tasks/                         └── Tasks/
    └── to_execute/                    └── completed/
        ├── orch_o1_task_t1.1/             └── orch_o1_task_t2/
        │   └── orch_o1_task_t1.1.md           ├── orch_o1_task_t2.$$.md
        └── orch_o1_task_t1.2/                 └── orch_o1_task_t2.$$.completion.md
            └── orch_o1_task_t1.2.md                               ▲
                                               5.2 completion file ┘
```

---

## Step 6: Secondary Workers Claim

Options: kicked off by orchestrator, scheduled task, or idle worker claims next

```
ai_comms/claude_cli/ [clB]         ai_comms/claude_cli/ [clC]
└── Tasks/                         └── Tasks/
    └── in_progress/                   └── in_progress/
        └── orch_o1_task_t1.1/             └── orch_o1_task_t1.2/
            └── orch_o1_task_t1.1.$$.md        └── orch_o1_task_t1.2.$$.md
```

---

## Step 7: Workers Execute & Report

7.1. clB hits milestone → creates .response.md, notifies parent
7.2. clC completes → creates .completion.md, notifies parent

```
ai_comms/claude_cli/ [clB]         ai_comms/claude_cli/ [clC]
└── Tasks/                         └── Tasks/
    └── in_progress/                   └── completed/
        └── orch_o1_task_t1.1/             └── orch_o1_task_t1.2/
            ├── orch_o1_task_t1.1.$$.md        ├── orch_o1_task_t1.2.$$.md
            └── orch_o1_task_t1.1.$$.response.md   └── orch_o1_task_t1.2.$$.completion.md
                        ▲                                          ▲
            milestone report                              final completion
```

---

## Step 8: Parent Receives Notifications

8.1. clA prompts clB to continue; clB completes
8.2. clA creates symlinks to completed child task dirs

```
ai_comms/claude_cli/ [clA]         ai_comms/claude_cli/ [clB]
└── Tasks/                         └── Tasks/
    └── in_progress/                   └── completed/
        └── orch_o1_task_t1/               └── orch_o1_task_t1.1/
            ├── orch_o1_task_t1.$$.md          ├── orch_o1_task_t1.1.$$.md
            ├── orch_o1_task_t1.1@ ────────────┤   ◄── symlink
            └── orch_o1_task_t1.2@ ──┐         ├── orch_o1_task_t1.1.$$.response.md
                                     │         └── orch_o1_task_t1.1.$$.completion.md
                                     │
                                     └──► (points to clC's completed dir)
```

---

## Step 9: First-Level Worker Completes

9.1. clA completes, notifies Claude Desktop
9.2. Claude creates symlinks to completed worker dirs

```
ai_comms/claude_cli/ [clA]              ai_comms/claude/
└── Tasks/                              └── in_progress/
    └── completed/                          └── orch_o1_task/
        └── orch_o1_task_t1/                    ├── orch_o1_task.$$.md
            ├── orch_o1_task_t1.$$.md          ├── orch_o1_task_t1@ ──► claude_cli/completed/
            ├── orch_o1_task_t1.1@             └── orch_o1_task_t2@ ──► codex_cli/completed/
            ├── orch_o1_task_t1.2@
            └── orch_o1_task_t1.$$.completion.md
```

---

## Step 10: Top-Level Completion

Claude Desktop completes orchestration task

```
ai_comms/claude/
└── completed/
    └── orch_o1_task/
        ├── orch_o1_task.$$.md
        ├── orch_o1_task.$$.completion.md    ◄── Final completion marker
        ├── orch_o1_task_t1@ ──────────────► claude_cli/Tasks/completed/orch_o1_task_t1/
        └── orch_o1_task_t2@ ──────────────► codex_cli/Tasks/completed/orch_o1_task_t2/
```

---

## Summary: Key Conventions

### Directory States
```
to_execute/    →    in_progress/    →    completed/
  (unclaimed)        (claimed+working)      (done)
```

### File Naming
| File Pattern | Purpose |
|--------------|---------|
| `task.md` | Original task definition |
| `task.$$.md` | Claimed task (PID marker) |
| `task.$$.response.md` | Milestone/interim report |
| `task.$$.completion.md` | Final completion marker |
| `subtask@` | Symlink to completed child task dir |

### Claim Contract
- **Unclaimed:** Task dir exists in `to_execute/`
- **Claimed:** Task dir in `in_progress/` with `$$`-stamped task file
- **Atomic claim:** First to successfully `mv` to `in_progress/` owns it

### Orchestration Hierarchy
```
Claude Desktop (orchestrator)
├── claude_cli [clA] (worker/sub-orchestrator)
│   ├── claude_cli [clB] (worker)
│   └── claude_cli [clC] (worker)
└── codex_cli [coA] (worker)
```

---

## Protocol Alignment Notes

**Aligned with protocol_taskCoordination v7.0:**

1. **Claim format:** `claimed_{timestamp}_{pid}_` prefix on dir name (kept for audit trail)

2. **Response types:** `.response.md` (milestone) vs `.completion.md` (final) - now in v7.0

3. **Subtask nesting:** Documented in v7.0 with `{parent}_t{N}.{M}` naming

4. **Symlinks:** Documented in v7.0 for completed child task refs

See: `ai_general/docs/30_protocols/protocol_taskCoordination_v7.0.md`
