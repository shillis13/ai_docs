# AI Workspace Architectural Layer Model (Condensed)

**Source:** `../architectural_layer_model.md` | **Version:** 1.0.0

## Three-Layer Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│ COMMUNICATION LAYER (ai_comms/)                                 │
│   Official coordination interface - task assignment, delivery   │
│   claude/ │ claude_cli/ │ codex_cli/ │ chatgpt/ │ tasks_*/      │
├─────────────────────────────────────────────────────────────────┤
│ IMPLEMENTATION LAYER (ai_claude/)                               │
│   Private workspace - config, runtime state, working dirs       │
│   persona_model/ │ instructions/ │ working_tasks/               │
├─────────────────────────────────────────────────────────────────┤
│ KNOWLEDGE LAYER (ai_memories/)                                  │
│   Shared knowledge - histories, digests, collective learning    │
│   00_incoming_chats/ │ 10_exported/ │ 30_histories/ │ 40_digests/│
└─────────────────────────────────────────────────────────────────┘
```

## Communication Layer (`ai_comms/`)

**Purpose:** Official interface for task assignment, status tracking, work product delivery

| Subdirectory | Purpose |
|--------------|---------|
| `claude/` | Desktop Claude's mailbox |
| `claude_cli/` | CLI coordination hub |
| `codex_cli/` | Codex worker tasks |
| `chatgpt/` | ChatGPT mailbox |
| `tasks_*/` | Specialized task queues |

**Characteristics:** Clean/structured, official handoffs only, protocol-driven

## Implementation Layer (`ai_claude/`)

**Purpose:** Configuration, runtime state, working directories - **private workspace**

| Subdirectory | Purpose |
|--------------|---------|
| `persona_model/` | Two-loop persona configuration |
| `instructions/` | Operating instructions |
| `working_tasks/` | Active work in progress (runtime) |
| `ai_specific_memories/` | Learned patterns (config, not state) |

**Key Distinction:**
- `ai_specific_memories/` = Configuration (learned patterns)
- `working_tasks/` = Runtime state (what I'm doing now)

## Knowledge Layer (`ai_memories/`)

**Purpose:** Shared knowledge base - accessible to all actors

| Subdirectory | Purpose |
|--------------|---------|
| `00_incoming_chats/` | Raw conversation exports |
| `10_exported/` | Organized by platform |
| `30_chat_histories/` | Processed conversations |
| `40_digests/` | Extracted knowledge (todos/, decisions/, knowledge/) |

## TODO vs Task Distinction

| Aspect | TODO | Task |
|--------|------|------|
| Location | `ai_general/todos/` | `ai_comms/{actor}/tasks/` |
| Purpose | Backlog - ideas, needs, questions | Actionable, prioritized, assigned |
| State | Not yet actionable | Ready to execute |
| Lifecycle | Idea → Curated → Task(s) | to_execute → in_progress → completed |

**Working State:** `ai_claude/working_tasks/` - runtime dir linked to ai_comms task

## Complete Task Lifecycle

```
1. ASSIGNMENT   ai_comms/claude/tasks/to_execute/      Official assignment
2. CLAIM        ai_comms/claude/tasks/in_progress/     Status: Started
3. WORK         ai_claude/working_tasks/task_xyz/      Notes, drafts, experiments
4. DELIVER      ai_comms/.../completed/response.md     COMPLETED
5. KNOWLEDGE    ai_memories/40_digests/                Extract reusable knowledge
6. CLEANUP      ai_claude/working_tasks/               Archive/cleanup
```

## Key Principles

1. **Separation of Concerns**
   - Communication layer → coordination interface
   - Implementation layer → actual work and runtime state
   - Knowledge layer → collective learning

2. **Clear Boundaries**
   - Official status → ai_comms
   - Working reality → ai_claude
   - Shared knowledge → ai_memories

3. **Actor Agnostic** - Protocol works for any actor (Desktop Claude, CLI, ChatGPT, Codex)

## Cross-References

- `ai_root_summary.md` - Directory structure overview
- `coordination_system_v4_digest.md` - Task coordination protocol
- `instr_file_conventions_v2.0.yml` - File organization standards
