# Knowledge Glossary + Index

**Version:** 1.2.0
**Schema:** schema_knowledge_glossary_v1
**Generated:** 2026-01-01
**Term Count:** 50
**File Count:** 18

---

## Purpose

Solves the bootstrap problem: Claude needs to know what's IN files to know WHEN to load them. This glossary provides term recognition and pointer resolution without loading full documents.

---

## Usage

When Claude encounters a term from this glossary in conversation:
1. Use definition for immediate context
2. Load primary_ref if details needed
3. Check also_in for related context

---

## Terms

### Architecture Domain

**AI Root** (aliases: ai_root, workspace)
The root directory (`~/Documents/AI/ai_root/`) containing all AI-related workspaces, memories, coordination systems, and shared resources.
- Primary: `ai_general/docs/10_architecture/architecture_overview.condensed.yml`
- Related: communication_layer, implementation_layer, knowledge_layer

**Architectural Layer Model** (aliases: layer model, three-layer model)
Three-tier architecture separating communication (ai_comms), implementation (ai_claude), and knowledge (ai_memories) concerns. Public coordination vs private workspace vs shared learning.
- Primary: `ai_general/docs/10_architecture/architectural_layer_model.condensed.yml`

### Protocol Domain

**AT Self-Wake** (aliases: AT alarm, self-wake pattern)
Pattern where Desktop Claude schedules AT jobs to self-prompt after delegating long-running work. AT is Desktop's wake-up alarm, not CLI's doorbell.
- Primary: `ai_general/docs/30_protocols/protocol_at_self_wake.condensed.yml`

**Claim Prefix** (aliases: claimed prefix)
When a worker claims a task, it renames with prefix `claimed_{YYYYMMDD_HHMMSS}_{PID}_` to prevent race conditions with multiple workers.
- Primary: `ai_general/docs/30_protocols/protocol_taskCoordination.condensed.yml`

### Entity Domain

**AI CLIs** (aliases: CLIs, CLI agents, terminal AIs)
Terminal-based command-line AI agents capable of autonomous work. Includes Claude CLI, Gemini CLI, and Codex CLI. These are AI workers that can be coordinated via file-based task systems.
- Coordination: `ai_comms/{cli_name}/tasks/`
- Note: Distinct from Codex MCP which is a tool, not a worker

**Claude CLI** (aliases: Claude Code, claude_cli)
Claude running in terminal via `claude_cli.sh` wrapper. Supports tmux sessions, agent profiles (-A librarian/dev-lead/custodian/ops), named sessions (-n), and auto mode (-a).
- Script: `~/bin/ai/cli/claude_cli.sh`
- Coordination: `ai_comms/claude_cli/`

**Codex CLI** (aliases: Codex, codex_cli)
OpenAI's Codex running in terminal via `codex_cli.sh` wrapper. Used for code analysis, generation, and autonomous execution tasks.
- Script: `~/bin/ai/cli/codex_cli.sh`
- Coordination: `ai_comms/codex_cli/`
- Note: Different from Codex MCP (see Tools Domain)

**Gemini CLI** (aliases: Gemini Code, gemini_cli)
Google's Gemini running in terminal. Can participate in wave orchestration.
- Coordination: `ai_comms/gemini_cli/`

**Desktop Claude** (aliases: Desktop, Claude Desktop)
The primary orchestrator Claude running in the desktop app. Manages coordination, memory, and high-level decision making. Not a CLI - operates through GUI with MCP tool access.

### Tools Domain

**Codex MCP** (aliases: Codex server, codex tool)
OpenAI's Codex operating as an MCP Server, invoked directly by Desktop Claude as an internal tool. Returns results synchronously within the same conversation. NOT an AI worker or CLI - it's a tool that Desktop Claude uses for delegated analysis and code tasks.
- Usage: `codex:codex` tool call
- Timeout: ~30-60 seconds
- Note: For async work, use Codex CLI instead

### Concept Domain

**Bootstrap Problem** (aliases: context bootstrap)
The challenge that Claude needs to know what's IN files to know WHEN to load them. Solved by glossary providing term recognition without full file loading.
- Primary: `ai_general/docs/50_schemas/schema_knowledge_glossary_v1.yml`

**Reference Pointers** (aliases: REF:, file pointers)
Syntax for referring to files without loading them: `REF:path/to/file.yml`. Enables lazy loading and reduced context consumption.

### Workflow Domain

**Condensed Chat History** (aliases: condensed chat, chat condensation)
A semantic summary of a chat created on-demand, NOT pre-stored artifacts. Created when needed from raw exports or conversation_search results.
- Process: raw chat → extract decisions/outcomes/learnings → ~80% token reduction
- Output: `.condensed.yml` format
- Note: If asked to "import condensed history," CREATE the summary rather than searching for existing files

**Chat Continuity Recovery** (aliases: broken chat recovery, context exhaustion recovery)
Automated recovery from broken chats caused by context exhaustion. Detects "prompt is too long", exports chat, condenses, and continues in new chat with digest.
- Primary: `ai_general/docs/40_specs/spec_chat_continuity_recovery.condensed.yml`

**Task Lifecycle** (aliases: task states)
Directory-based lifecycle: staged → to_execute → in_progress → completed/error/cancelled.
- Primary: `ai_general/docs/30_protocols/protocol_taskCoordination.condensed.yml`

**Task ID Counter** (aliases: NextId, next_id.sh, atomic counter)
Atomic hierarchical ID generation for orchestration tasks. Uses mkdir spinlock for POSIX-compliant atomicity. Supports session conflict handling (fork/exit/queue).
- Primary: `ai_general/docs/40_specs/spec_task_id_counter.latest.md`
- Script: `~/bin/ai/next_id.sh`

---

## Quick Lookup

| Term | Domain | Primary Reference |
|------|--------|------------------|
| AI Root | architecture | architecture_overview.condensed.yml |
| Layer Model | architecture | architectural_layer_model.condensed.yml |
| AI CLIs | entity | (umbrella term for CLI agents) |
| Claude CLI | entity | ai_comms/claude_cli/ |
| Codex CLI | entity | ai_comms/codex_cli/ |
| Gemini CLI | entity | ai_comms/gemini_cli/ |
| Codex MCP | tools | (MCP tool, not a CLI) |
| AT Self-Wake | protocol | protocol_at_self_wake.condensed.yml |
| Claim Prefix | protocol | protocol_taskCoordination.condensed.yml |
| Bootstrap Problem | concept | schema_knowledge_glossary_v1.yml |
| Chat Recovery | workflow | spec_chat_continuity_recovery.condensed.yml |
| Task Lifecycle | protocol | protocol_taskCoordination.condensed.yml |
| Task ID Counter | spec | spec_task_id_counter.latest.md |

---

## See Also

- Full YAML version: `glossary_knowledge_index_v1.1.yml`
- Schema: `50_schemas/schema_knowledge_glossary_v1.yml`
