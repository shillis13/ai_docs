# AI Workspace Architectural Layer Model

**Version:** 1.0.0 | **Status:** active | **Created:** 2025-11-16 | **Maintainer:** PianoMan

## Overview

The AI workspace uses a multi-layer architecture where different directory structures 
serve distinct purposes: communication/coordination, implementation/runtime, and 
shared knowledge. Each layer has clear boundaries and specific responsibilities.

## Architectural Layers

### Communication Layer

#### Directory

ai_comms/

#### Purpose

Communication and coordination between actors

#### Description

Official interface for task assignment, status tracking, and work product delivery.
This is the "public" coordination space where actors (Desktop Claude, CLI instances, 
ChatGPT, Codex, etc.) exchange messages and coordinate work.

#### Contains

- Official task status and assignments
- Messages between actors
- Delivered work products
- Structured coordination protocols

#### Subdirectories

- **Claude/**: Desktop Claude's mailbox
- **Claude Cli/**: CLI coordination hub
- **Codex Cli/**: Codex worker tasks
- **Chatgpt/**: ChatGPT mailbox
- **Tasks */**: Specialized task queues (AppleScript, orchestration, etc.)

#### Characteristics

- Clean, structured communication
- Official handoffs only
- What others see
- Protocol-driven state management

#### Example Workflow

Task appears in claude/tasks/to_execute/
→ Moved to in_progress/ when claimed
→ Final deliverable written to completed/task_xyz/response.md

### Implementation Layer

#### Directory

ai_claude/

#### Purpose

Configuration, runtime state, and working directories

#### Description

Actor-specific implementation details, working space, and runtime state.
This is the "private" workspace where actual work happens, distinct from
the clean communication interface.

#### Contains

- Persona model configuration
- Instructions and behavioral specs
- Working directories for active tasks
- Runtime operational state
- Draft/interim artifacts before delivery
- Skills and capabilities

#### Subdirectories

- **Persona Model/**: Two-loop persona configuration
- **Instructions/**: Operating instructions
- **Specs/**: Standards and protocols
- **Skills/**: Capabilities
- **Working Tasks/**: Active work in progress (runtime)
- **Ai Specific Memories/**: Learned patterns (configuration, not state)

#### Characteristics

- Messy, iterative reality
- Can be chaotic without affecting interface
- Private workspace
- Configuration AND runtime state

#### Distinction

ai_specific_memories = Configuration (learned patterns, techniques)
working_tasks/ = Runtime state (what I'm doing right now)

#### Example Workflow

Task claimed from ai_comms/claude/tasks/in_progress/
→ Working directory created: ai_claude/working_tasks/task_xyz/
→ Notes, drafts, experiments, iterations happen here
→ Final artifact delivered via ai_comms/claude/tasks/completed/

### Knowledge Layer

#### Directory

ai_memories/

#### Purpose

Shared knowledge base and collective learning

#### Description

Repository of shared knowledge, decisions, digests, and learnings that are
accessible to all actors. Not AI-specific by default - this is what we
collectively know and can reference.

#### Contains

- Conversation histories and exports
- Knowledge digests
- Decision records
- TODO backlog (ideas, research questions)
- Shared research and reports

#### Subdirectories

- **00 Incoming Chats/**: Raw conversation exports
- **10 Exported/**: Organized by platform
- **30 Chat Histories/**: Processed conversations
- **40 Digests/**: Extracted knowledge (todos/, decisions/, knowledge/)

#### Characteristics

- Shared across all actors
- Curated knowledge artifacts
- Can have privacy/isolation if needed later
- **Currently**: all memories are common

#### Example Workflow

Conversation completes
→ Export to 00_incoming_chats/
→ Process to 30_chat_histories/
→ Extract knowledge to 40_digests/knowledge/
→ If actionable: Create TODO in 40_digests/todos/

## Todo Vs Task Distinction

### Todos

#### Location

ai_memories/40_digests/todos/

#### Purpose

Backlog of ideas, needs, and research questions

#### Characteristics

- Not yet actionable
- Requires curation/refinement
- Shared across all actors
- May generate multiple tasks

#### Lifecycle

Idea captured → TODO created → Curated → Task(s) generated → Moved to execution queue

### Tasks

#### Location

ai_comms/{actor}/tasks/

#### Purpose

Actionable, prioritized, assigned work

#### Characteristics

- Ready to execute
- Clear definition and acceptance criteria
- Uses coordination protocol structure
- Actor-specific queues

#### Lifecycle

to_execute/ → in_progress/ → completed/ (or error/ or cancelled/)

#### Working State

##### Location

ai_claude/working_tasks/

##### Description

Runtime working directory separate from official task status.
Links to corresponding ai_comms task but contains messy reality
of doing the work.

## Complete Task Lifecycle Example

### Step 1 Assignment

#### Layer

communication

#### Location

ai_comms/claude/tasks/to_execute/

#### Action

Task appears in execution queue

#### State

Official assignment

### Step 2 Claim

#### Layer

communication

#### Location

ai_comms/claude/tasks/in_progress/

#### Action

Task moved when claimed

#### State

Official status update - "Started, working in progress"

### Step 3 Work

#### Layer

implementation

#### Location

ai_claude/working_tasks/task_xyz/

#### Action

Create working directory for actual work

#### State

- Notes, drafts, experiments
- Messy, iterative reality
- Links to ai_comms task

### Step 4 Deliver

#### Layer

communication

#### Location

ai_comms/claude/tasks/completed/task_xyz/response.md

#### Action

Final artifacts delivered

#### State

Official status - COMPLETED

### Step 5 Knowledge

#### Layer

knowledge

#### Location

ai_memories/40_digests/

#### Action

If reusable knowledge generated, extract to shared repository

#### State

Available to all actors

### Step 6 Cleanup

#### Layer

implementation

#### Location

ai_claude/working_tasks/

#### Action

Archive or cleanup working directory

#### State

Runtime state cleared

## Key Architectural Principles

### Separation Of Concerns

- Communication layer handles coordination interface
- Implementation layer handles actual work and runtime state
- Knowledge layer handles collective learning
- Each layer can change independently without affecting others

### Clear Boundaries

- Official status lives in ai_comms
- Working reality lives in ai_claude
- Shared knowledge lives in ai_memories
- No ambiguity about "where does this go?"

### Actor Agnostic

- Coordination protocol works for any actor/role
- Desktop Claude, CLI instances, ChatGPT, Codex all use same patterns
- Roles (orchestrator, worker, etc.) are flexible

### Natural Accommodation

- New use cases find logical homes without forcing
- No conflicts or awkward overlaps
- Architecture proved itself by handling TODO system naturally

## Validation Test

### Use Case

Desktop Claude autonomous TODO/task system

### Requirements

- Communication between actors
- Working state tracking
- Shared knowledge access

### Result

Everything found logical home:
- TODOs → ai_memories (shared backlog)
- Tasks → ai_comms (official coordination)
- Working dirs → ai_claude (runtime state)
- Dashboard → ai_claude (operational status)

### Conclusion

Architecture flexible enough to absorb new patterns without breaking
existing ones. Clear boundaries with understandable rationales.

## Cross References

- **Ai Root Summary.Md**: Directory structure overview
- **Coordination System V4 Digest.Md**: Task coordination protocol
- **Instr File Conventions V2.0.Yml**: File organization standards
