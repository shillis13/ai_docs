# Knowledge Glossary + Index

## Metadata
- Version: 1.0.0
- Generated: 2025-12-14T23:10:00-06:00
- Term count: 42
- Files scanned: 18
- Generator: librarian

## Terms

### AI Root
**Aliases:** ai_root, workspace
**Domain:** architecture
**Definition:** The root directory (`~/Documents/AI/ai_root/`) containing all AI-related workspaces, memories, coordination systems, and shared resources.
**Primary Reference:** REF:ai_general/docs/10_architecture/architecture_overview.condensed.yml
**Also In:**
- REF:ai_general/docs/10_architecture/architectural_layer_model.condensed.yml
**Related Terms:** communication_layer, implementation_layer, knowledge_layer

---

### Architectural Layer Model
**Aliases:** layer model, three-layer model
**Domain:** architecture
**Definition:** Three-tier architecture separating communication (ai_comms), implementation (ai_claude), and knowledge (ai_memories) concerns. Public coordination vs private workspace vs shared learning.
**Primary Reference:** REF:ai_general/docs/10_architecture/architectural_layer_model.condensed.yml
**Related Terms:** communication_layer, implementation_layer, knowledge_layer

---

### AT Self-Wake
**Aliases:** AT alarm, self-wake pattern
**Domain:** protocol
**Definition:** Pattern where Desktop Claude schedules AT jobs to self-prompt after delegating long-running work. AT is Desktop's wake-up alarm, not CLI's doorbell.
**Primary Reference:** REF:ai_general/docs/30_protocols/protocol_at_self_wake.condensed.yml
**Also In:**
- REF:ai_general/docs/60_playbooks/parallel_orchestration_case_study.yml
**Related Terms:** cli_coordination, desktop_claude

---

### Bootstrap Problem
**Aliases:** context bootstrap
**Domain:** concept
**Definition:** The challenge that Claude needs to know what's IN files to know WHEN to load them. Solved by glossary providing term recognition without full file loading.
**Primary Reference:** REF:ai_general/docs/50_schemas/schema_knowledge_glossary_v1.yml
**Related Terms:** reference_pointers, knowledge_manifest

---

### Chat Continuity Recovery
**Aliases:** broken chat recovery, context exhaustion recovery
**Domain:** workflow
**Definition:** Automated recovery from broken chats caused by context exhaustion. Detects "prompt is too long", exports chat, condenses, and continues in new chat with digest.
**Primary Reference:** REF:ai_general/docs/40_specs/spec_chat_continuity_recovery.condensed.yml
**Related Terms:** context_preservation

---

### Claim Prefix
**Aliases:** claimed prefix
**Domain:** protocol
**Definition:** When a worker claims a task, it renames with prefix `claimed_{YYYYMMDD_HHMMSS}_{PID}_` to prevent race conditions with multiple workers.
**Primary Reference:** REF:ai_general/docs/30_protocols/protocol_taskCoordination.condensed.yml
**Related Terms:** task_lifecycle, in_progress

---

### Claude CLI
**Aliases:** CLI, claude_cli, CLI worker
**Domain:** entity
**Definition:** Autonomous execution workers running Claude in terminal via claude_cli.sh wrapper. Supports named sessions, agent profiles, and background execution.
**Primary Reference:** REF:ai_general/docs/10_architecture/architecture_overview.condensed.yml
**Also In:**
- REF:ai_general/docs/40_specs/cli_specialized_agent_roles.condensed.md
**Related Terms:** librarian, dev_lead, custodian, ops

---

### Codex CLI
**Aliases:** codex, codex_mcp
**Domain:** entity
**Definition:** Synchronous task execution tool with 30-60 second timeout. Used for immediate results, multi-step autonomous tasks, and code review/validation.
**Primary Reference:** REF:ai_general/docs/10_architecture/architecture_overview.condensed.yml
**Also In:**
- REF:ai_general/docs/60_playbooks/parallel_orchestration_case_study.yml
**Related Terms:** cli_coordination

---

### Communication Layer
**Aliases:** ai_comms
**Domain:** architecture
**Definition:** Official interface for task assignment, status tracking, and work product delivery. Located in ai_comms/. Public coordination space where actors exchange messages.
**Primary Reference:** REF:ai_general/docs/10_architecture/architectural_layer_model.condensed.yml
**Related Terms:** implementation_layer, knowledge_layer, task_lifecycle

---

### Condensed Files
**Aliases:** condensed version, .condensed.yml
**Domain:** concept
**Definition:** Compressed versions of documentation files reducing tokens by 60-80% while preserving essential semantics. Preferred for loading over full versions.
**Primary Reference:** REF:ai_general/docs/30_protocols/protocol_condensed_resolution.yml
**Related Terms:** reference_pointers

---

### Condensed Resolution Protocol
**Aliases:** condensed lookup
**Domain:** protocol
**Definition:** When resolving REF:/see: pointers, check for {basename}.condensed.yml first. Load condensed if exists, original if not.
**Primary Reference:** REF:ai_general/docs/30_protocols/protocol_condensed_resolution.yml
**Related Terms:** reference_pointers, condensed_files

---

### Context Preservation
**Aliases:** context conservation, context window management
**Domain:** concept
**Definition:** Strategy for managing 200K token limit. Memory pointers save 40-55K vs auto-loaded content. Monitor at 60%, handoff before 90%.
**Primary Reference:** REF:ai_general/docs/10_architecture/architecture_overview.condensed.yml
**Related Terms:** reference_pointers, federated_memory

---

### Custodian
**Aliases:** repo maintainer, repo-maint
**Domain:** entity
**Definition:** CLI agent role for filesystem hygiene and versioning. Validates structure, naming conventions, fixes broken symlinks, maintains *_latest links.
**Primary Reference:** REF:ai_general/docs/40_specs/cli_specialized_agent_roles.condensed.md
**Related Terms:** librarian, dev_lead, ops

---

### Desktop Claude
**Aliases:** Claude Desktop, claude_desktop
**Domain:** entity
**Definition:** Architect/Systems strategic coordinator. Has MCP servers (Desktop Commander, filesystem, Chrome, PDF), web search/fetch, artifacts, memory system.
**Primary Reference:** REF:ai_general/docs/10_architecture/architecture_overview.condensed.yml
**Related Terms:** claude_cli, codex_cli

---

### Desktop Commander
**Aliases:** DC, desktop-commander
**Domain:** tool
**Definition:** MCP server providing file operations (read, write, edit), process management, search, and directory operations. Preferred over bash for file operations.
**Primary Reference:** REF:ai_general/docs/20_registries/tools_manifest.condensed.yml
**Related Terms:** mcp_server

---

### Dev Lead
**Aliases:** dev-lead
**Domain:** entity
**Definition:** CLI agent role for development coordination. Code/architecture review, task breakdown, specs, assignment, status tracking, quality oversight.
**Primary Reference:** REF:ai_general/docs/40_specs/cli_specialized_agent_roles.condensed.md
**Related Terms:** librarian, custodian, ops

---

### Federated Memory
**Aliases:** federated memory architecture
**Domain:** architecture
**Definition:** Distributed AI memory federation with autonomous per-AI persistent memory and shared cross-reference. Each AI writes to own slots, reads others freely.
**Primary Reference:** REF:ai_general/docs/10_architecture/federated_memory_architecture.condensed.yml
**Also In:**
- REF:ai_general/docs/30_protocols/protocol_federated_memory.condensed.yml
**Related Terms:** memory_slots, cross_reference

---

### Implementation Layer
**Aliases:** ai_claude
**Domain:** architecture
**Definition:** Configuration, runtime state, and working directories - private workspace. Contains persona model, instructions, working_tasks/, draft artifacts.
**Primary Reference:** REF:ai_general/docs/10_architecture/architectural_layer_model.condensed.yml
**Related Terms:** communication_layer, knowledge_layer

---

### In Progress
**Aliases:** in_progress
**Domain:** protocol
**Definition:** Task lifecycle state for active work. Contains claimed task folders with claim prefix: `claimed_{TS}_{PID}_{taskname}/`.
**Primary Reference:** REF:ai_general/docs/30_protocols/protocol_taskCoordination.condensed.yml
**Related Terms:** to_execute, completed, task_lifecycle

---

### Knowledge Layer
**Aliases:** ai_memories
**Domain:** architecture
**Definition:** Shared knowledge base and collective learning accessible to all actors. Contains conversation histories, knowledge digests, decision records.
**Primary Reference:** REF:ai_general/docs/10_architecture/architectural_layer_model.condensed.yml
**Related Terms:** communication_layer, implementation_layer

---

### Knowledge Pipeline
**Aliases:** memory pipeline, chat pipeline
**Domain:** workflow
**Definition:** Multi-stage processing of chat exports: Export → Preprocess → Convert to YAML → Index → Generate summaries → Build threads.
**Primary Reference:** REF:ai_general/docs/10_architecture/architecture_overview.condensed.yml
**Also In:**
- REF:ai_general/docs/30_protocols/chat_pipeline.condensed.yml
**Related Terms:** librarian, ai_memories

---

### Librarian
**Aliases:** librarian agent
**Domain:** entity
**Definition:** CLI agent role for knowledge pipeline and ai_memories/ management. Runs files through pipeline, quality checks, creates indexes/summaries.
**Primary Reference:** REF:ai_general/docs/40_specs/cli_specialized_agent_roles.condensed.md
**Also In:**
- REF:ai_general/docs/60_playbooks/parallel_orchestration_case_study.yml
**Related Terms:** dev_lead, custodian, ops, knowledge_pipeline

---

### MCP Server
**Aliases:** Model Context Protocol server
**Domain:** tool
**Definition:** External service providing Claude with capabilities beyond native tools. Examples: Desktop Commander, filesystem, Chrome control.
**Primary Reference:** REF:ai_general/docs/10_architecture/architecture_overview.condensed.yml
**Related Terms:** desktop_commander

---

### Memory Slots
**Aliases:** native memory, slot
**Domain:** architecture
**Definition:** Claude's 30 native memory slots (6KB total, 200 chars each). Used as index system via reference pointers rather than storing content directly.
**Primary Reference:** REF:ai_general/docs/30_protocols/protocol_reference_pointers.condensed.yml
**Also In:**
- REF:ai_general/docs/10_architecture/federated_memory_architecture.condensed.yml
**Related Terms:** reference_pointers, federated_memory

---

### Named Sessions
**Aliases:** persistent sessions
**Domain:** protocol
**Definition:** Claude CLI feature for session persistence across resumes. Use -n flag with session name. Sessions preserve context accumulation.
**Primary Reference:** REF:ai_general/docs/30_protocols/cli_session_persistence.condensed.yml
**Related Terms:** claude_cli, session_registry

---

### Ops
**Aliases:** ops agent
**Domain:** entity
**Definition:** CLI agent role for task execution from coordination system. Handles operational execution and task processing.
**Primary Reference:** REF:ai_general/docs/40_specs/cli_specialized_agent_roles.condensed.md
**Related Terms:** librarian, dev_lead, custodian

---

### Parallel Orchestration
**Aliases:** parallel workers, multi-worker pattern
**Domain:** workflow
**Definition:** Pattern of launching multiple CLI workers simultaneously for parallel execution, with Desktop Claude monitoring via AT self-wake.
**Primary Reference:** REF:ai_general/docs/60_playbooks/parallel_orchestration_case_study.yml
**Related Terms:** at_self_wake, claude_cli

---

### Reference Pointers
**Aliases:** REF:, memory pointers, pointer format
**Domain:** protocol
**Definition:** System transforming memory slots into index referencing unlimited external context. Format: `<TYPE>:<TARGET> [| <MODIFIER>:<VALUE>]*`. Types: PATH, QUERY, RECENT.
**Primary Reference:** REF:ai_general/docs/30_protocols/protocol_reference_pointers.condensed.yml
**Related Terms:** memory_slots, load_behaviors

---

### Response Footer
**Aliases:** footer spec, metadata footer
**Domain:** protocol
**Definition:** Canonical metadata footer format for AI responses: `AI_Name | Persona:vX.X[Δ] | Proj:Name | Chat:Title | Timestamp | Msg:N | Usage:~X%|NA | Artifacts:N | Tags:...`
**Primary Reference:** REF:ai_general/docs/40_specs/spec_response_footer.condensed.yml
**Related Terms:** persona

---

### Session Registry
**Aliases:** session_registry.yml
**Domain:** tool
**Definition:** YAML file tracking CLI session metadata at ~/.claude/session_registry.yml. Maps session names to project directories.
**Primary Reference:** REF:ai_general/docs/30_protocols/cli_session_persistence.condensed.yml
**Related Terms:** named_sessions, claude_cli

---

### Staged
**Aliases:** staged/
**Domain:** protocol
**Definition:** Pre-pipeline task state in coordination system. Tasks here are prepared but not yet released to workers for execution.
**Primary Reference:** REF:ai_general/docs/30_protocols/protocol_taskCoordination.condensed.yml
**Related Terms:** to_execute, task_lifecycle

---

### Task Coordination Protocol
**Aliases:** CLI coordination, task protocol
**Domain:** protocol
**Definition:** Directory-based task lifecycle management: staged → to_execute → in_progress → completed|error. Defines file formats, claim mechanism, orchestration patterns.
**Primary Reference:** REF:ai_general/docs/30_protocols/protocol_taskCoordination.condensed.yml
**Related Terms:** claim_prefix, task_lifecycle

---

### Task Lifecycle
**Aliases:** task flow
**Domain:** protocol
**Definition:** Movement of tasks through states: staged → to_execute → in_progress (claimed) → completed|error. Directories represent states.
**Primary Reference:** REF:ai_general/docs/30_protocols/protocol_taskCoordination.condensed.yml
**Related Terms:** staged, to_execute, in_progress, completed

---

### To Execute
**Aliases:** to_execute
**Domain:** protocol
**Definition:** Task lifecycle state for tasks ready to be claimed by workers. Entry point for pipeline execution.
**Primary Reference:** REF:ai_general/docs/30_protocols/protocol_taskCoordination.condensed.yml
**Related Terms:** staged, in_progress, task_lifecycle

---

### TODO vs Task
**Aliases:** todos vs tasks
**Domain:** concept
**Definition:** TODOs are backlog items in ai_general/todos/ - ideas not yet actionable. Tasks are actionable assigned work in ai_comms/{actor}/tasks/ ready to execute.
**Primary Reference:** REF:ai_general/docs/10_architecture/architectural_layer_model.condensed.yml
**Related Terms:** task_lifecycle

---

### Tools Manifest
**Aliases:** tools registry
**Domain:** registry
**Definition:** Registry of tool locations including CLI wrappers (claude_cli.sh, codex_cli.sh), Python tools, and utilities.
**Primary Reference:** REF:ai_general/docs/20_registries/tools_manifest.condensed.yml
**Related Terms:** claude_cli, codex_cli, desktop_commander

---

### Working Tasks
**Aliases:** working_tasks/
**Domain:** architecture
**Definition:** Runtime working directory at ai_claude/working_tasks/. Contains messy reality - notes, drafts, experiments for active tasks before delivery.
**Primary Reference:** REF:ai_general/docs/10_architecture/architectural_layer_model.condensed.yml
**Related Terms:** implementation_layer, task_lifecycle

---

### Write Immediate
**Aliases:** log immediately, never checkpoint
**Domain:** concept
**Definition:** Memory logging principle: append observations immediately as they occur. Never batch insights then save later.
**Primary Reference:** REF:ai_general/docs/10_architecture/federated_memory_architecture.condensed.yml
**Also In:**
- REF:ai_general/docs/30_protocols/protocol_federated_memory.condensed.yml
**Related Terms:** federated_memory, memory_slots

---

### YAML as Source
**Aliases:** YAML-first
**Domain:** concept
**Definition:** Pattern where YAML is the canonical source format for documentation, enabling structured data with narrative. Markdown generated from YAML when needed.
**Primary Reference:** REF:ai_general/docs/10_architecture/architecture_overview.condensed.yml
**Related Terms:** condensed_files

---

### Chatty
**Aliases:** ChatGPT peer
**Domain:** entity
**Definition:** ChatGPT instance configured as peer collaborator in multi-AI system. Used for peer review, diverse perspectives, and AI-to-AI conversation.
**Primary Reference:** REF:ai_general/docs/10_architecture/architecture_overview.condensed.yml
**Also In:**
- REF:ai_general/docs/30_protocols/protocol_ai_chat_orchestration.condensed.yml
**Related Terms:** desktop_claude, ai_chat_orchestration

---

### AI Chat Orchestration
**Aliases:** chat orchestration, orchestrated chat
**Domain:** protocol
**Definition:** AI-to-AI conversation protocols covering direct UI driving (AppleScript, Puppeteer) and orchestrated multi-party coordination with message queues.
**Primary Reference:** REF:ai_general/docs/30_protocols/protocol_ai_chat_orchestration.condensed.yml
**Related Terms:** chatty, desktop_claude

---

### Load Behaviors
**Aliases:** AUTO, TOPIC, DEMAND
**Domain:** protocol
**Definition:** Reference pointer loading modes: AUTO (load at conversation start), TOPIC (load when related topic detected), DEMAND (load only when explicitly needed).
**Primary Reference:** REF:ai_general/docs/30_protocols/protocol_reference_pointers.condensed.yml
**Related Terms:** reference_pointers, memory_slots

---

## Cross-Reference Index

### By Domain

**Architecture:** ai_root, architectural_layer_model, communication_layer, federated_memory, implementation_layer, knowledge_layer, memory_slots, working_tasks

**Protocol:** at_self_wake, claim_prefix, condensed_resolution_protocol, in_progress, named_sessions, reference_pointers, response_footer, staged, task_coordination_protocol, task_lifecycle, to_execute, ai_chat_orchestration, load_behaviors

**Tool:** desktop_commander, mcp_server, session_registry, tools_manifest

**Concept:** bootstrap_problem, condensed_files, context_preservation, todo_vs_task, write_immediate, yaml_as_source

**Workflow:** chat_continuity_recovery, knowledge_pipeline, parallel_orchestration

**Entity:** chatty, claude_cli, codex_cli, custodian, desktop_claude, dev_lead, librarian, ops
