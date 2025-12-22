# Directory Structure Reference v2.0

**Architecture:** Multi-repo, AI-centric workspace with shared resources  
**Last Updated:** 2025-10-30  
**Status:** Finalized

---

## External Resources

**Public Repository:** https://github.com/shillis13/ai_docs
- Mount: `ai_general/docs/` IS this submodule (not a copy)
- Access: `https://raw.githubusercontent.com/shillis13/ai_docs/main/<path>`
- Purpose: Cross-platform doc access for iOS, Web, other AI platforms
- Sync: Push from docs/ to update; submodule commit in ai_root tracks version

---

## Physical Layout

```
~/Documents/AI/
├── ai_chat_artifacts/          # Mirrors 30_chat_histories structure
├── ai_chatgpt/                 # ChatGPT-specific repo
├── ai_claude/                  # Claude-specific repo
├── ai_general/                 # Shared resources across AIs
├── ai_memories/                # Central memory processing pipeline
└── comms/                      # Communication and task coordination
```

---

## Top-Level Directory Purposes

### ai_chat_artifacts/
**Purpose:** Storage for artifacts generated during chats (images, documents, data files)  
**Structure:** Mirrors `ai_memories/30_chat_histories/` for easy correlation  
**Example:**
```
ai_chat_artifacts/
├── claude/2025/10/30/
│   ├── diagram_architecture.png
│   └── analysis_data.csv
└── orchestrated/2025/10/29/
    └── collaboration_output.pdf
```

### ai_chatgpt/
**Purpose:** ChatGPT-specific configuration, instructions, and memories  
**Contains:**
```
ai_chatgpt/
├── ai_specific_memories/       # Chatty's self-knowledge
│   ├── capabilities.md         # What Chatty can/can't do
│   ├── work_log.md            # Projects worked on
│   ├── reflections.md         # Significant moments
│   ├── interactions.md        # Notable exchanges
│   └── ideas.md               # Future reference thoughts
│
├── ai_scripts/                # Ready-built scripts for Chatty
│   └── *.js
│
├── connectors/                # Custom connector configs
│
├── custom_gpts/               # Custom GPT configurations
│
├── instructions/              # Chatty-specific instructions
│   ├── to_ai_autogen/        # Auto-generated combined instructions
│   ├── instr_chatgpt_specific.md
│   ├── packager_config_instructions_v01.yml
│   └── [symlinks to ai_general/instructions/]
│
└── specs/                     # Chatty-specific specs
    ├── to_ai_autogen/
    ├── packager_config_specs_v01.yml
    └── [symlinks to ai_general/specs/]
```

### ai_claude/
**Purpose:** Claude-specific configuration, instructions, and memories  
**Contains:**
```
ai_claude/
├── ai_specific_memories/      # Claude's self-knowledge
│   ├── capabilities.md
│   ├── work_log.md
│   ├── reflections.md
│   ├── interactions.md
│   └── ideas.md
│
├── ai_scripts/               # Ready-built scripts for Claude
│   └── *.js
│
├── connectors/               # MCP connector configs
│
├── instructions/             # Claude-specific instructions
│   ├── to_ai_autogen/
│   ├── instr_claude_specific.md
│   ├── packager_config_instructions_v01.yml
│   └── [symlinks to ai_general/instructions/]
│
├── skills/                   # Claude Desktop skills
│
└── specs/                    # Claude-specific specs
    ├── to_ai_autogen/
    ├── packager_config_specs_v01.yml
    └── [symlinks to ai_general/specs/]
```

### ai_general/
**Purpose:** Shared resources, documentation, and tools used across all AIs  
**Contains:**
```
ai_general/
├── data/                     # Schemas and shared data structures
│   ├── schema_chat_history_v01.yaml
│   └── schema_chat_index_v01.yaml
│
├── docs/                     # Architecture and concept documentation
│   └── concept_*.md
│
├── instructions/             # Universal instructions (versioned)
│   ├── expectation_first_response_protocol_v1.0.md
│   ├── instr_bash_guide_v03.md
│   ├── instr_coding_guidelines_v01.md
│   ├── instr_development_guidelines_v04.md
│   ├── instr_footer_v03.md
│   ├── instr_general_guidelines_v03.md
│   ├── instr_python_guide_v05.md
│   ├── instr_vba_guide_v02.md
│   └── persistence_preferences.md
│
├── projects/                 # Project-specific documentation
│   ├── chat_history_processing/
│   ├── chat_orchestration/
│   ├── desktop_cli_workflow/
│   └── persona_modeling/
│
├── prompts/                  # Reusable prompt library
│   ├── exported/
│   │   └── promptblaze_export_YYYYMMDD.json
│   ├── group_architecture_and_design/
│   ├── group_code_development/
│   │   ├── prompt_generate_functions_with_test.md
│   │   ├── prompt_refactor_for_maintainability.md
│   │   └── prompt_regex_pattern_builder.md
│   ├── group_code_review/
│   ├── group_debugging_and_performance/
│   ├── group_learning_and_education/
│   ├── group_lifestyle_health/
│   ├── group_prompt_creation/
│   └── imported/
│       └── promptblaze_import_YYYYMMDD.json
│
├── research_and_reports/     # Research outputs
│   ├── ai_ethics/
│   ├── ai_platform_review_Oct2025/
│   ├── framework_to_compare_national_scale_economic_models/
│   └── system_design/
│
├── scripts/                  # Shared automation scripts
│   └── [symlinks to actual script locations]
│
└── specs/                    # Technical specifications (versioned)
    ├── spec_footer_v02.md
    └── spec_message_insert_v01.md
```

### ai_memories/
**Purpose:** Central memory processing pipeline - converts chat exports to searchable, indexed memories  
**Contains:**
```
ai_memories/
├── 10_exported/              # Raw chat exports from AI platforms
│   ├── claude_desktop/
│   ├── claude_web/
│   ├── claude_cli/
│   └── chatgpt/
│
├── 20_staged_converted_histories/  # YAML conversion staging
│   ├── pending/              # Awaiting validation
│   └── converted/            # Validated, ready for chunking
│
├── 30_chat_histories/        # Chunked YAML archives (terminal storage)
│   ├── indexes/              # Content indexes
│   │   ├── by_date.json
│   │   ├── by_topic.json
│   │   ├── by_ai.json
│   │   ├── by_participants.json
│   │   └── full_text_index/
│   ├── claude/YYYY/MM/DD/
│   ├── chatgpt/YYYY/MM/DD/
│   └── orchestrated/YYYY/MM/DD/
│       ├── conv_001_claude.yml
│       ├── conv_001_chatgpt.yml
│       └── conv_001_meta.yml
│
└── 40_digests/               # Processed memory artifacts
    ├── chat_mems/            # Per-chat memory digests
    │   └── memory_digest_{topic}_v{XX}.md
    │
    ├── decisions/            # Architecture decisions (immutable)
    │   └── decision_{date}_{topic}_v{XX}.md
    │
    ├── indexes/              # Digest/artifact indexes
    │   ├── decision_catalog.json
    │   ├── knowledge_graph.json
    │   ├── todo_tracking.json
    │   └── digest_references.json
    │
    ├── knowledge/            # Reusable facts and information
    │   └── knowledge_{topic}_v{XX}.md
    │
    ├── todos/                # Action items and tracking
    │   ├── action_items.md           # Top-level index
    │   ├── discussion_topics.md      # Top-level index
    │   ├── research_questions.md     # Top-level index
    │   ├── action_items/             # Detailed item directories
    │   │   └── item_001_{slug}/
    │   │       ├── description.md
    │   │       ├── notes.md
    │   │       └── files/
    │   ├── discussion_topics/
    │   │   └── topic_001_{slug}/
    │   └── research_questions/
    │       └── question_001_{slug}/
    │
    └── user_context/         # Information about PianoMan
        └── user_{aspect}_v{XX}.md
```

### comms/
**Purpose:** AI-to-AI communication and task coordination system  
**Contains:**
```
comms/
├── announcements/            # Broadcast messages to all AIs
│
├── chatgpt/                  # Chatty's mailbox
│   ├── drafts/
│   ├── inbox/
│   ├── outbox/
│   └── saved/
│
├── claude/                   # Claude Desktop's mailbox
│   ├── drafts/
│   ├── inbox/
│   ├── outbox/
│   └── saved/
│
├── claude_cli/               # CLI coordination (symlinked from ~/.claude/coordination)
│   ├── broadcasts/
│   ├── direct/
│   │   └── cli_{PID}/
│   ├── responses/
│   │   └── cli_{PID}/
│   └── cli_registry.md
│
├── codex_cli/                # Codex CLI coordination
│
├── tasks_applescript/        # AppleScript task execution
│   ├── cancelled/
│   ├── completed/
│   ├── error/
│   ├── in_progress/
│   ├── logs/
│   └── to_execute/
│
├── tasks_chat_orchestration/ # Chat orchestrator tasks
│   ├── cancelled/
│   ├── completed/
│   ├── error/
│   ├── in_progress/
│   ├── logs/
│   └── to_execute/
│
└── tasks_script/             # Script execution tasks
    ├── cancelled/
    ├── completed/
    ├── error/
    ├── in_progress/
    ├── logs/
    └── to_execute/
```

---

## Memory Processing Pipeline Flow

```
1. Export from AI platform
   ↓
   10_exported/{ai}/

2. Convert to YAML format
   ↓
   20_staged_converted_histories/pending/
   ↓ (after validation)
   20_staged_converted_histories/converted/

3. Chunk by date/session/size
   ↓
   30_chat_histories/{ai}/YYYY/MM/DD/
   (terminal storage - never moves)

4. Process chunks → generate artifacts
   ↓
   40_digests/{type}/
   (can be regenerated multiple times)

5. Index for search
   ↓
   30_chat_histories/indexes/ (content)
   40_digests/indexes/ (artifacts)
```

---

## Symlink Strategy

**AI-specific repos symlink to ai_general for shared resources:**

```bash
# Example: Claude's instructions
ai_claude/instructions/
├── instr_claude_specific.md                    # Claude-only
└── instr_general_guidelines.md → ../../ai_general/instructions/instr_general_guidelines_v03.md

# Example: Shared schemas
ai_claude/ai_scripts/
└── schema_chat_history.yaml → ../../ai_general/data/schema_chat_history_v01.yaml
```

**CLI coordination symlink:**
```bash
~/.claude/coordination → ~/Documents/AI/comms/claude_cli/
```

**Maintenance:**
- Versioned files live in `ai_general/` 
- Symlinks in AI repos point without version numbers (always get latest)
- Breaking changes = new filename in general, update symlinks when ready

---

## File Naming Conventions

### Chat History Files
**Location:** `30_chat_histories/{ai}/YYYY/MM/DD/`  
**Format:** `{chat_id}_{YYYY-MM-DD}_{title_slug}.yml`  
**Example:** `chat_001_2025-10-30_workspace_organization.yml`

### Digest Files
**Location:** `40_digests/{type}/`  
**Format:** `{type}_digest_{topic}_v{XX}.md` or `{type}_{date}_{topic}_v{XX}.md`  
**Examples:**
- `memory_digest_workspace_organization_v01.md`
- `decision_20251030_caching_strategy_v02.md`
- `knowledge_promptblaze_schema_v01.md`

### Orchestrated Chats
**Location:** `30_chat_histories/orchestrated/YYYY/MM/DD/`  
**Format:**
```
conv_{XXX}_claude.yml       # Claude's messages
conv_{XXX}_chatgpt.yml      # ChatGPT's messages
conv_{XXX}_meta.yml         # Orchestration metadata
```

---

## Key Principles

### Repo Isolation
Each AI repo is self-contained:
- Own instructions (with symlinks to general)
- Own specs (with symlinks to general)
- Own AI-specific memories
- Own scripts and connectors

### Shared Resources
Common resources live in `ai_general/`:
- Version-controlled instructions and specs
- Schemas and data structures
- Project documentation
- Prompt libraries
- Research outputs

### Central Memory Pipeline
`ai_memories/` processes conversations from ANY AI:
- Normalized YAML format
- Unified indexing
- Consistent digest generation
- Cross-AI conversation support

### Communication System
`comms/` enables coordination:
- Per-AI mailboxes for async messages
- Task execution systems with full workflow
- CLI coordination with symlink integration
- Broadcast announcements to all AIs

---

## Quick Reference Commands

### Create Full Structure
```bash
cd ~/Documents/AI

# Create top-level directories
mkdir -p ai_chat_artifacts ai_chatgpt ai_claude ai_general ai_memories comms

# Create ai_chatgpt structure
mkdir -p ai_chatgpt/{ai_specific_memories,ai_scripts,connectors,custom_gpts,instructions/to_ai_autogen,specs/to_ai_autogen}

# Create ai_claude structure
mkdir -p ai_claude/{ai_specific_memories,ai_scripts,connectors,instructions/to_ai_autogen,skills,specs/to_ai_autogen}

# Create ai_general structure
mkdir -p ai_general/{data,docs,instructions,projects/{chat_history_processing,chat_orchestration,desktop_cli_workflow,persona_modeling},prompts/{exported,imported,group_{architecture_and_design,code_development,code_review,debugging_and_performance,learning_and_education,lifestyle_health,prompt_creation}},research_and_reports,scripts,specs}

# Create ai_memories structure
mkdir -p ai_memories/{10_exported/{claude_desktop,claude_web,claude_cli,chatgpt},20_staged_converted_histories/{pending,converted},30_chat_histories/{indexes,claude,chatgpt,orchestrated},40_digests/{chat_mems,decisions,indexes,knowledge,todos/{action_items,discussion_topics,research_questions},user_context}}

# Create comms structure
mkdir -p comms/{announcements,chatgpt/{drafts,inbox,outbox,saved},claude/{drafts,inbox,outbox,saved},claude_cli,codex_cli,tasks_{applescript,chat_orchestration,script}/{cancelled,completed,error,in_progress,logs,to_execute}}

# Create CLI coordination symlink
ln -s ~/Documents/AI/comms/claude_cli ~/.claude/coordination
```

### Navigate to Key Locations
```bash
# Memory processing
cd ~/Documents/AI/ai_memories

# Claude config
cd ~/Documents/AI/ai_claude

# Shared resources
cd ~/Documents/AI/ai_general

# Communication system
cd ~/Documents/AI/comms
```

---

## Related Documentation

- **Memory Digest:** `memory_digest_workspace_architecture_v02.md`
- **Implementation Guide:** `workspace_implementation_guide_v02.md`
- **Chat Processing Flow:** `ai_general/projects/chat_history_processing/README.md`
- **CLI Coordination:** `ai_general/projects/desktop_cli_workflow/protocol_v3.1.md`

---

**Document Version:** 2.0  
**Created:** 2025-10-30  
**Architecture:** Multi-repo, AI-centric with shared resources  
**Status:** Finalized and ready for implementation
