# Knowledge Glossary + Index

**Version:** 1.5.0
**Schema:** schema_knowledge_glossary_v1
**Generated:** 2026-03-19
**Term Count:** 43
**File Count:** 21

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

**Architectural Layer Model** (aliases: layer model, three-layer)
Three-tier architecture separating communication (ai_comms), implementation (ai_claude), and knowledge (ai_memories) concerns. Public coordination vs private workspace vs shared learning.
- Primary: `ai_general/docs/10_architecture/architectural_layer_model.condensed.yml`

### Protocol Domain

**AT Self-Wake** (aliases: AT alarm, self-wake)
Pattern where Desktop Claude schedules AT jobs to self-prompt after delegating long-running work. AT is Desktop's wake-up alarm, not CLI's doorbell.
- Primary: `ai_general/docs/30_protocols/protocol_at_self_wake.condensed.yml`

**Claim Prefix** (aliases: claimed prefix) [DEPRECATED v9.0]
DEPRECATED - replaced by flag files in v9.0. Was: worker renames task folder with `claimed_{YYYYMMDD_HHMMSS}_{PID}_` prefix to prevent race conditions.
- Primary: `ai_general/docs/30_protocols/protocol_taskCoordination.condensed.yml`

**Flag Files** (aliases: state flags, lifecycle flags)
Zero-byte files marking task state transitions: `{reqId}_{datetime}_started`, `_completed`, `_error`, `_cancelled`. v9.0 replacement for claim_prefix.
- Primary: `ai_general/docs/30_protocols/protocol_taskCoordination.condensed.yml`

### Entity Domain

**AI CLIs** (aliases: CLIs, CLI agents, terminal AIs)
Terminal-based command-line AI agents capable of autonomous work. Includes Claude CLI, Gemini CLI, Codex CLI, and Cline CLI. Coordinated via file-based task systems at `ai_comms/{cli}/tasks/`.
- Note: Distinct from Codex MCP which is a tool, not a worker

**Claude CLI** (aliases: Claude Code, claude_cli)
Claude running in terminal via `claude_cli.py` wrapper. Supports tmux sessions, agent profiles (-A librarian/dev-lead/custodian), named sessions (-n), and auto mode (-a).
- Script: `~/bin/ai/cli/claude_cli.py`
- Coordination: `ai_comms/claude_cli/`

**Cline CLI** (aliases: Cline, cline_cli)
Local agentic AI via Cline + llama-server. Qwen3-Coder backend on port 8081. Workspace rules via .clinerules.
- Script: `~/bin/ai/cli/cline_cli.py`

**Codex CLI** (aliases: Codex, codex_cli)
OpenAI's Codex running in terminal via `codex_cli.py` wrapper. Used for code analysis, generation, and autonomous execution tasks.
- Script: `~/bin/ai/cli/codex_cli.py`
- Coordination: `ai_comms/codex_cli/`
- Note: Different from Codex MCP (see Tools Domain)

**Gemini CLI** (aliases: Gemini Code, gemini_cli)
Google's Gemini running in terminal. Can participate in wave orchestration.
- Coordination: `ai_comms/gemini_cli/`

**Desktop Claude** (aliases: Desktop, Claude Desktop)
The primary orchestrator Claude running in the desktop app. Manages coordination, memory, and high-level decision making. Not a CLI - operates through GUI with MCP tool access.

**llama-server** (aliases: llama-server, local LLM, llm_server)
Local LLM inference server (llama.cpp or MLX). Profile-based via llm_server.sh. Default port 11881. Backend for Cline CLI, Goose, mcp-openai. SwiftBar plugin available for status/control.
- Port: 11881
- Script: `~/bin/ai/local_llm/llm_server.sh`

### Role Domain

**Librarian** (aliases: memory librarian)
Memory system curator, manages `ai_memories/` and chat history pipeline. Keeps exports and condensations flowing.
- Scope: `ai_memories/`

**Dev Lead** (aliases: dev lead, development coordinator)
Development coordinator, owns todos and task creation.
- Scope: `ai_general/todos/`

**Custodian** (aliases: repo custodian)
Repository maintainer, structural integrity. Guards structure, hygiene, consistency.
- Scope: `ai_root/`

**Peer Review** (aliases: peer reviewer)
Code/design reviewer, quality assurance. Performs reviews before merge or release.

**Tester** (aliases: qa tester)
Testing specialist, validation and verification. Drives test coverage and validation runs.

### Tools Domain

**Codex MCP** (aliases: Codex server, codex tool)
OpenAI's Codex operating as an MCP Server, invoked by Desktop Claude as an internal tool. Returns results synchronously within the same conversation. NOT an AI worker or CLI - it's a tool that Desktop Claude uses for delegated analysis and code tasks.
- Usage: `codex:codex` tool call
- Timeout: ~30-60 seconds
- Note: For async work, use Codex CLI instead

**Claude Code Plugins** (aliases: plugins, plugin system, /plugin)
Extension system for Claude Code - commands, agents, skills, MCP servers bundled as plugins.
- Primary: `claude_code_plugins_reference.latest.yml`
- CLI: `claude plugin <cmd>`
- REPL: `/plugin`
- Note: CLI subcommands work from shell; REPL commands only inside session

**Plugin Marketplace** (aliases: marketplace, plugin repo)
Git repos hosting plugin catalogs. Add via `claude plugin marketplace add owner/repo`.
- Examples: anthropics/claude-code, ananddtyagi/cc-marketplace, obra/superpowers-marketplace
- Location: `~/.claude/plugins/marketplaces/`

**Superpowers Plugin** (aliases: superpowers, obra superpowers)
Core skills library by Jesse Vincent - TDD, debugging, /brainstorm, /write-plan, git-worktrees. 20+ skills bundled.
- Source: superpowers-marketplace
- Repo: obra/superpowers

**Episodic Memory Plugin** (aliases: episodic-memory, conversation memory)
Semantic search across past Claude Code conversations. Persistent memory between sessions.
- Source: superpowers-marketplace

**Desktop Skills** (aliases: claude.ai skills, upload skills, SKILL.md)
Model-invoked instruction packages for Claude Desktop. Auto-activate based on context. Structure: SKILL.md + optional scripts/references/assets/.
- Primary: `claude_desktop_skills_reference.latest.yml`
- Install: Settings > Capabilities > Skills > + Add
- Note: Different from Claude Code plugins

**CSV Data Summarizer** (aliases: csv analyzer)
Desktop skill - auto-analyzes CSV uploads with stats and visualizations. No prompting needed.
- Type: community skill

**Search Agent** (aliases: knowledge search, conversation search)
Gemini-based dual-mode search over 4yr conversation archive. SEARCH mode returns curated results, ANSWER mode synthesizes editorial with citations. 4-layer cascade: topics > synonyms > content > content+synonyms.
- Primary: `spec_search_agent.latest.condensed.yml`
- Prompt: `ai_comms/gemini_cli/prompts/search_agent_v2.5.md`
- Version: 2.5

**Search Cascade** (aliases: 4-layer search)
Search fallback strategy: L1 topics > L2 topics+synonyms > L3 content > L4 content+synonyms. All case-insensitive. Never stop at L1 for "no results".
- Primary: `spec_search_agent.latest.condensed.yml`

**Answer Mode** (aliases: editorial mode)
Search agent mode for questions - synthesizes editorial answer with [N] inline citations + Sources section + token sizes. Triggers: what, why, how, explain.
- Primary: `spec_search_agent.latest.condensed.yml`

**Search Mode** (aliases: results mode)
Search agent mode for discovery - returns curated list with Why Relevant, Patterns Noticed, To Go Deeper. Triggers: find, search, list, show.
- Primary: `spec_search_agent.latest.condensed.yml`

**CLI-Agent MCP** (aliases: cli-agent MCP, CLI agent MCP server)
MCP server for launching and managing CLI agents. Entrypoint for spinning up CLI workers.
- Path: `ai_general/apps/mcps/cli-agent/`

### Concept Domain

**Message Inserts** (aliases: memory inserts, INSERT blocks, structured inserts)
Structured comment blocks (`<<<INSERT type=X>>>...<<<END INSERT>>>`) for machine-readable markers. Types: BOOKMARK, MEMORY, DECISION, QUESTION. Enables stateless platforms (Web/iOS) to mark observations for later harvest by Desktop Claude.
- Primary: `spec_message_insert.latest.condensed.yml`
- Note: Primary persistence mechanism for platforms without filesystem access

**Bootstrap Problem** (aliases: context bootstrap)
The challenge that Claude needs to know what's IN files to know WHEN to load them. Solved by glossary providing term recognition without full file loading.
- Primary: `ai_general/docs/50_schemas/schema_knowledge_glossary_v1.yml`

**Bootstrap Hierarchy** (aliases: bootstrap order, load order)
CLI bootstrap sequence: global.md > platform/{platform}.md > role/{role}/role.yml context_files > tasking.md (if -T).
- Path: `ai_general/prompts/`
- Note: Replaced ~/.claude/agents.json approach

**Context Files** (aliases: context_files list, role context)
REF: pointers in role.yml defining what files to load for that role. Resolved at launch by claude_cli.py or cli-agent MCP.

**Reference Pointers** (aliases: REF:, file pointers)
Syntax for referring to files without loading them: `REF:path/to/file.yml`. Enables lazy loading and reduced context consumption.

**role.yml** (aliases: role.yml, role definition)
Role config file with context_files, duties, ownership. Located at `ai_general/prompts/roles/{role}/role.yml`.
- Note: Replaced hardcoded agent definitions

**Time Loop** (aliases: time loop, UI reset)
Claude Desktop phenomenon where response fails mid-generation and UI resets context as if prompt was never submitted. Appears to user as going back in time. Distinct from local model reasoning loops.
- Platform: Claude Desktop only

### Workflow Domain

**Condensed Chat History** (aliases: condensed chat, chat condensation)
A semantic summary of a chat created on-demand, NOT pre-stored artifacts. Created when needed from raw exports or conversation_search results.
- Process: raw chat > extract decisions/outcomes/learnings > ~80% token reduction
- Output: `.condensed.yml` format
- Note: If asked to "import condensed history," CREATE the summary rather than searching for existing files

**Chat Continuity Recovery** (aliases: broken chat recovery)
Automated recovery from broken chats caused by context exhaustion. Detects "prompt is too long", exports chat, condenses, and continues in new chat with digest.
- Primary: `ai_general/docs/40_specs/spec_chat_continuity_recovery.condensed.yml`

**Task Lifecycle** (aliases: task states)
Directory-based lifecycle: staged > to_execute > in_progress > completed/error/cancelled.
- Primary: `ai_general/docs/30_protocols/protocol_taskCoordination.condensed.yml`

**Task ID Counter** (aliases: NextId, next_id.sh, atomic counter)
Atomic hierarchical ID generation for orchestration tasks. Uses mkdir spinlock for POSIX-compliant atomicity. Supports session conflict handling (fork/exit/queue).
- Primary: `ai_general/docs/40_specs/spec_task_id_counter.latest.md`
- Script: `~/bin/ai/next_id.sh`

---

## See Also

- Full YAML version: `glossary_knowledge_index_v1.1.yml`
- Schema: `50_schemas/schema_knowledge_glossary_v1.yml`
