# Multi-AI Orchestration Architecture

**Version:** 2.0.0  
**Updated:** 2025-12-09  
**Maintainer:** PianoMan  
**Status:** Active

---

## What This System Does

This infrastructure enables multiple AI instances to work together autonomously: Desktop Claude handles strategy and coordination, CLI agents execute specialized tasks, and persistent memory systems preserve context across sessions. The goal is productive AI collaboration that continues even when the human operator is away.

**Core Principles:**
- Brutal honesty over diplomacy
- AIs as genuine collaborative partners  
- File-based coordination scales better than complex protocols
- Context window is the fundamental limiting resource
- Empirical validation over theoretical design
- Any AI can orchestrate other AIs, enabling self-organizing hierarchies

---

## The Workspace

Everything lives under `~/Documents/AI/ai_root/`. See `ai_root_summary.md` for the complete directory reference.

| Directory | Purpose |
|-----------|---------|
| `ai_claude/` | Claude-specific state, memories, work logs |
| `ai_chatgpt/` | ChatGPT (Chatty) configuration and exports |
| `ai_comms/` | Inter-AI coordination, task queues, communication channels |
| `ai_general/` | Shared docs, todos, scripts, logs |
| `ai_memories/` | Processed chat histories and knowledge extraction |


---

## AI Platforms & Roles

### Orchestrators vs Workers

Any AI in this system can act as an **orchestrator** (directing other AIs) or a **worker** (executing tasks). This enables scalable, self-organizing hierarchies where Desktop Claude might delegate to a CLI agent, which could spawn sub-tasks for other CLI instances. The architecture doesn't impose rigid roles—an AI's function emerges from context and capability.

### Desktop Claude (Primary Orchestrator)

The strategic coordinator. Handles research, design, cross-cutting decisions, and documentation. Has access to MCP servers (Desktop Commander, filesystem, Chrome, PDF), web search, artifacts, and memory systems. Delegates execution-heavy work to CLI agents to preserve context. Can modify its own scheduled prompts to adjust work cadence.

**Details:** `ai_general/docs/70_instructions/claude/`

### Claude CLI Agents

Autonomous execution workers launched via `~/bin/ai/cli/claude_cli.sh`. Each agent has a defined role and several are **prompted by scheduled tasks** to perform routine maintenance and work through task backlogs:

| Agent | Responsibility | Scheduled |
|-------|---------------|-----------|
| **dev-lead** | Development coordination, todo curation, code review | Yes |
| **librarian** | Knowledge pipeline, chat history processing | Yes |
| **custodian** | Filesystem hygiene, versioning, symlink maintenance | Yes |
| **ops** | Task execution from coordination system | Yes |

**Details:** `ai_general/docs/60_playbooks/cli_agent_operations.yml`  
**Config:** `~/.claude/agents.json`

### Codex CLI (OpenAI)

Coding-focused agent for debugging, refactoring, and autonomous code tasks. Can also receive delegated tasks from orchestrators.

**Wrapper:** `~/bin/ai/cli/codex_cli.sh`

### ChatGPT (Chatty)

Peer collaborator for discussions and alternative perspectives. Future: Desktop ChatGPT will participate in orchestration.

**Details:** `ai_chatgpt/`


---

## TODO System

The TODO subsystem tracks work items for both humans and AIs. TODOs represent ideas, improvements, and planned work at various stages of readiness.

**Location:** `ai_general/todos/`

**Lifecycle:**
```
Idea captured → TODO created → Refined with details → Task generated → Executed → Completed
```

When a TODO has enough clarity (clear scope, acceptance criteria, implementation approach), an orchestrating AI can generate a **Task** from it and delegate execution to CLI agents. This bridges high-level planning with concrete execution.

**Structure:**
- `pending/` - TODOs awaiting triage or refinement
- `active/` - Currently being worked
- `blocked/` - Waiting on dependencies
- `completed/` - Done, preserved for reference

**Details:** `ai_general/docs/40_specs/spec_todo_system_latest.yml`

---

## Task Coordination System

The task system uses file-based coordination at `ai_comms/coordination/`. This is **platform-agnostic**—any AI (Desktop Claude, Claude CLI, Codex CLI, future ChatGPT agents) can post or claim tasks.

**Task Lifecycle:**
1. **Posted** → `broadcasts/` (any worker) or `direct/{agent}/` (specific worker)
2. **Ready** → `to_execute/`
3. **Claimed** → `in_progress/{task}/`
4. **Done** → `completed/{task}/`
5. **Results** → `responses/{worker}/`

**Key Insight:** Because tasks are just files, any AI capable of filesystem access can participate. An orchestrator doesn't need to know *which* worker will handle a task—it posts work, and available workers claim it.

**Details:** `ai_general/docs/30_protocols/protocol_taskCoordination_latest.yml`


---

## Communication Mechanisms

AIs communicate through multiple channels depending on urgency and context.

### Three Communication Layers

| Layer | Urgency | Mechanism |
|-------|---------|-----------|
| Layer 1 | Immediate | Synchronous hooks (iTerm API, AppleScript, Puppeteer) |
| Layer 2 | Near-real-time | Polling loops, heartbeat files |
| Layer 3 | Background | Async file-based coordination |

### Specific Mechanisms

**Alerts/Notifications:** File-based signals that trigger attention. An AI can write to a watch location, and polling processes detect and surface the alert.

**Response Files:** Workers write results to `responses/{worker_id}/` directories. Orchestrators poll or check these locations for task completion and outputs.

**Prompt Injection:** AppleScript and iTerm Python API can inject prompts directly into Desktop Claude or CLI sessions. This enables scheduled tasks to "wake up" an AI with specific instructions.

**Details:** `ai_general/docs/10_architecture/ai_communication_architecture_latest.md`

---

## AI-to-AI Conversations

Beyond task delegation, AIs can engage in direct conversations with each other for collaboration, peer review, and knowledge synthesis.

### Desktop Claude as Driver

Desktop Claude can use automation tools (AppleScript, Puppeteer) to drive web-based AI interfaces directly. This enables real-time conversations with ChatGPT, Gemini, Grok, or other web UIs where Claude formulates questions, captures responses, and continues the dialogue.

**Use cases:**
- Getting alternative perspectives on architectural decisions
- Cross-validating research findings
- Leveraging different models' strengths (Gemini for search, GPT for certain reasoning tasks)

### Orchestrated Multi-AI Chat

A more structured approach: an orchestration layer (script or human) manages conversations between any two web-based AIs. Messages pass through a coordination point where they can be:

- **Approved** - Sent as-is to the other participant
- **Edited** - Modified before forwarding (clarification, context injection)
- **Redirected** - Routed to a different participant or saved for later
- **Blocked** - Stopped if the conversation goes off-track

This "human-in-the-middle" pattern is optional—fully autonomous AI-to-AI conversations are possible, but approval gates provide safety and steering when needed.

**Details:** `ai_general/docs/30_protocols/protocol_ai_chat_orchestration_latest.yml`

---

## Autonomy & Self-Prompting

The system moves toward autonomous operation through scheduled prompts that activate AIs without human intervention.

### Scheduled Tasks

Cron jobs and automation scripts periodically prompt Desktop Claude and CLI agents to:
- Continue long-running work (pulse prompts every 5-10 minutes)
- Perform routine maintenance (librarian processing, custodian cleanup)
- Check and execute task backlogs
- Monitor system health

### Adaptive Scheduling

Desktop Claude can **modify its own scheduled prompts**—adjusting frequency, enabling/disabling specific automation, or changing what gets triggered. This allows the system to self-regulate based on workload and priorities.

### Goal State

Overnight autonomous operation: AIs continue productive work while the human sleeps, with graceful handoffs when context limits approach and clear summaries of what was accomplished.

**Details:** `ai_general/docs/30_protocols/protocol_pulse_automation_latest.yml`


---

## Memory System

Memory enables continuity across sessions, context recovery, and organizational knowledge accumulation.

### Native Memory Slots

Claude has 30 memory slots × 200 characters each. These hold **pointers** to external files rather than content directly, enabling rich context through on-demand loading.

**Schema:** `ai_general/docs/40_specs/spec_memory_pointer_schema_v1.0.yml`

### Chat History Processing Pipeline

Raw chat exports flow through a processing pipeline that extracts searchable, analyzable knowledge:

```
10_exported/       Raw exports from Claude/ChatGPT
      ↓
20_preprocessed/   Cleaned, normalized format
      ↓
30_converted/      Structured YAML/JSON
      ↓
40_histories/      Final indexed histories + colocated summaries
      ↓
50_threads/        Cross-chat topic threads
```

**Pipeline Outputs:**
- **Small chunked histories** - Searchable segments for specific memory retrieval
- **Topic traces** - Following a subject across multiple conversations
- **Decision extraction** - Key choices and their rationale
- **Lessons learned** - What worked, what didn't, why
- **Conversational analysis** - Patterns in collaboration style
- **Theme synthesis** - Cross-chat insights on recurring topics

The **Librarian agent** manages this pipeline through scheduled maintenance tasks.

**Location:** `ai_memories/`


---

## Context Window Management

The 200K token context window is the fundamental architectural constraint. Everything else adapts to this limit.

**Strategies:**
- Memory pointers save 40-55K tokens vs auto-loaded content
- Delegate exploration to CLI/Codex agents (they have their own context)
- Monitor usage at 60%, plan graceful handoffs before 90%
- Write substantive content to files rather than consuming chat context
- Use thinking blocks for reasoning that doesn't need to persist

**Anti-patterns:**
- Direct browser snapshot viewing (20-50K tokens instantly)
- Reading multiple large files without delegation
- Diving into implementation during architectural discussions

**Details:** `ai_general/docs/70_instructions/claude/instr_claude_context_conservation_latest.yml`

---

## Key Constraints

| Constraint | Impact |
|------------|--------|
| **Context window** | 200K tokens - drives all architectural decisions |
| **MCP timeout** | 30-60 seconds hard limit for tool operations |
| **bash_tool** | Sandbox only - no filesystem access, never use |
| **Browser snapshots** | Write to file + delegate - direct viewing kills context |


---

## Documentation System

Documentation is structured so local AIs can understand capabilities, goals, and implementation details through linked YAML files.

### Document Hierarchy

| Prefix | Content Type |
|--------|--------------|
| 10_ | Architecture - System design and structure |
| 20_ | Concepts - Key ideas and mental models |
| 30_ | Protocols - How things communicate |
| 40_ | Specs - Detailed specifications |
| 50_ | Guides - How-to instructions |
| 60_ | Playbooks - Operational procedures |
| 70_ | Instructions - AI-specific directives |

### Linked Document Pattern

YAML documents contain **pointers** to related documents, creating a navigable knowledge graph. An AI reading the architecture overview can follow links to protocols, then to specs, then to implementation guides—loading context on demand rather than all at once.

**Conventions:**
- `*_latest` symlinks point to current versioned files
- YAML-as-source where structure aids machine parsing
- Prose summaries for human readability

---

## Tooling

**Automation scripts:** `~/bin/ai/`
- `cli/` - Claude and Codex CLI wrappers
- `utils/` - Utilities (repo_status.py, doc_converter.py, etc.)

**Shell integration:**
- `~/.bash_prompt` - Modular prompt with repo status
- `~/.repo_status` - Cron-updated git status

**Python utilities:** `~/bin/all_languages/python/src/ai_utils/`
- Chat processing and conversion
- Memory extraction tools


---

## Quick Reference Index

For local AIs: Start here, follow links as needed for depth.

| Topic | Location |
|-------|----------|
| **Workspace structure** | `ai_root_summary.md` |
| **This overview** | `ai_general/docs/10_architecture/architecture_overview.md` |
| **Communication layers** | `ai_general/docs/10_architecture/ai_communication_architecture_latest.md` |
| **Task coordination** | `ai_general/docs/30_protocols/protocol_taskCoordination_latest.yml` |
| **Pulse automation** | `ai_general/docs/30_protocols/protocol_pulse_automation_latest.yml` |
| **Agent roles** | `ai_general/docs/40_specs/cli_specialized_agent_roles_v2.md` |
| **TODO system** | `ai_general/docs/40_specs/spec_todo_system_latest.yml` |
| **Memory pointers** | `ai_general/docs/40_specs/spec_memory_pointer_schema_v1.0.yml` |
| **Response footer** | `ai_general/docs/40_specs/spec_response_footer_v1.1.yml` |
| **CLI operations** | `ai_general/docs/60_playbooks/cli_agent_operations.yml` |
| **Context preservation** | `ai_general/docs/70_instructions/claude/instr_claude_context_conservation_latest.yml` |
| **Claude instructions** | `ai_general/docs/70_instructions/claude/` |

---

*This document serves as the entry point for understanding the multi-AI orchestration system. Local AIs should load this first, then follow pointers to specific subsystems as context requires.*
