# Directory Structure Reference (Condensed)
**Source:** directory_structure_reference_v02.md | v2.0 | 2025-10-30

## Top-Level Layout
```
~/Documents/AI/
├── ai_chat_artifacts/   # Mirrors 30_chat_histories (images, docs, data)
├── ai_chatgpt/          # ChatGPT: memories, instructions, specs, custom_gpts
├── ai_claude/           # Claude: memories, instructions, skills, connectors
├── ai_general/          # Shared: docs, instructions, prompts, specs, scripts
├── ai_memories/         # Memory pipeline (export→convert→store→digest)
└── comms/               # Coordination: mailboxes, tasks, CLI sync
```

## AI-Specific Repos (ai_chatgpt/, ai_claude/)
```
ai_{ai}/
├── ai_specific_memories/    # Self-knowledge: capabilities, work_log, reflections
├── ai_scripts/              # Ready-built scripts
├── connectors/              # MCP/custom connectors
├── custom_gpts/             # (ChatGPT only)
├── skills/                  # (Claude only)
├── instructions/
│   ├── to_ai_autogen/       # Auto-generated combined instructions
│   ├── instr_{ai}_specific.md
│   ├── packager_config_instructions_v01.yml
│   └── [symlinks to ai_general/instructions/]
└── specs/
    ├── to_ai_autogen/
    ├── packager_config_specs_v01.yml
    └── [symlinks to ai_general/specs/]
```

## Shared Resources (ai_general/)
```
ai_general/
├── data/             # Schemas (chat_history, chat_index)
├── docs/             # Architecture/concept docs
├── instructions/     # Universal versioned instructions (v01-v05)
├── projects/         # chat_history_processing, chat_orchestration, etc.
├── prompts/          # Prompt library (by group: code_dev, review, etc.)
│   ├── exported/     # promptblaze_export_YYYYMMDD.json
│   └── imported/
├── research_and_reports/
├── scripts/          # Symlinks to actual script locations
└── specs/            # Technical specifications (versioned)
```

## Memory Pipeline (ai_memories/)
```
10_exported/                 → Raw exports: claude_desktop/, claude_web/, claude_cli/, chatgpt/
20_staged_converted_histories/ → pending/ → converted/ (YAML staging)
30_chat_histories/           → Terminal storage + indexes/
    ├── indexes/             # by_date.json, by_topic.json, by_ai.json, by_participants.json, full_text_index/
    ├── {ai}/YYYY/MM/DD/
    └── orchestrated/  # conv_{XXX}_claude.yml, _chatgpt.yml, _meta.yml
40_digests/            → Processed artifacts
    ├── chat_mems/     # memory_digest_{topic}_v{XX}.md
    ├── decisions/     # decision_{date}_{topic}_v{XX}.md (immutable)
    ├── indexes/       # decision_catalog.json, knowledge_graph.json, todo_tracking.json, digest_references.json
    ├── knowledge/     # knowledge_{topic}_v{XX}.md
    ├── todos/
    │   ├── action_items.md, discussion_topics.md, research_questions.md  # Top-level indexes
    │   ├── action_items/, discussion_topics/, research_questions/
    │   └── item_XXX_{slug}/ → description.md, notes.md, files/
    └── user_context/  # user_{aspect}_v{XX}.md
```

## Communication (comms/)
```
comms/
├── announcements/           # Broadcast to all AIs
├── chatgpt/, claude/        # Per-AI mailboxes (drafts/inbox/outbox/saved)
├── claude_cli/              # symlink: ~/.claude/coordination
│   ├── broadcasts/, direct/cli_{PID}/, responses/cli_{PID}/
│   └── cli_registry.md
├── codex_cli/
└── tasks_{type}/            # applescript, chat_orchestration, script
    ├── to_execute/, in_progress/, completed/, error/, cancelled/
    └── logs/
```

## Symlink Strategy
**AI repos → ai_general:** Versioned files in ai_general, symlinks without version (always get latest)
```bash
ai_claude/instructions/
└── instr_general_guidelines.md → ../../ai_general/instructions/instr_general_guidelines_v03.md

ai_claude/ai_scripts/
└── schema_chat_history.yaml → ../../ai_general/data/schema_chat_history_v01.yaml
```
**CLI coordination:** `~/.claude/coordination → ~/Documents/AI/comms/claude_cli/`
**Breaking changes:** New filename in ai_general, update symlinks when ready

## File Naming Conventions
- **Chats:** `{chat_id}_{YYYY-MM-DD}_{title_slug}.yml`
- **Digests:** `{type}_digest_{topic}_v{XX}.md` or `{type}_{date}_{topic}_v{XX}.md`
- **Orchestrated:** `conv_{XXX}_claude.yml`, `conv_{XXX}_chatgpt.yml`, `conv_{XXX}_meta.yml`

## Key Principles
1. **Repo isolation** - Each AI self-contained with own memories/instructions
2. **Shared resources** - Version-controlled in ai_general/
3. **Central memory** - ai_memories/ processes all AI conversations uniformly
4. **Coordination** - comms/ enables async messaging and task management

## Related Documentation
- Memory Digest: `memory_digest_workspace_architecture_v02.md`
- Implementation Guide: `workspace_implementation_guide_v02.md`
- Chat Processing: `ai_general/projects/chat_history_processing/README.md`
- CLI Coordination: `ai_general/projects/desktop_cli_workflow/protocol_v3.1.md`
