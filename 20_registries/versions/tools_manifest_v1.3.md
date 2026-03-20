# Tools Manifest

**Version:** 1.3.0  
**Updated:** 2026-02-14  
**Purpose:** Canonical paths for all AI-accessible tools — single source of truth

---

## CLI Wrappers

**Base:** [~/bin/ai/cli](file:///~/bin/ai/cli)

### claude_cli.py
- **Path:** `~/bin/ai/cli/claude_cli.py  [~/bin/ai/cli/claude_cli.py](file:///~/bin/ai/cli/claude_cli.py)
- **Invoke:** `python3 ~/bin/ai/cli/claude_cli.py`
- **Description:** Claude CLI with tmux, agents, session management. Bootstrap: `global.md → platform/claude.md → roles/{type}/role.yml`
- **Flags:**
  - `-t` — tmux
  - `-a` — auto
  - `-A TYPE` — agent type (librarian, dev_lead, custodian, ops, peer_review, tester, researcher, validator)
  - `-n NAME` — named session
  - `-c` — continue
  - `-T` — task mode (loads tasking.md)
- **Examples:**
  - `python3 claude_cli.py -a -A librarian -t "prompt"`
  - `python3 claude_cli.py -t -a -c "continue"`

### codex_cli.py
- **Path:** `~/bin/ai/cli/codex_cli.py`
- **Invoke:** `python3 ~/bin/ai/cli/codex_cli.py`
- **Description:** OpenAI Codex CLI with tmux
- **Flags:**
  - `-t` — tmux
  - `-a` — full auto (`--dangerously-bypass-approvals-and-sandbox`)
  - `-s` — session name
  - `-w` — workdir

### gemini_cli.py
- **Path:** `~/bin/ai/cli/gemini_cli.py`
- **Invoke:** `python3 ~/bin/ai/cli/gemini_cli.py`
- **Description:** Gemini CLI with tmux, wave orchestration, shard support
- **Flags:**
  - `-t` — tmux
  - `-a` — auto
  - `-s` — session name

### cline_cli.py
- **Path:** `~/bin/ai/cli/cline_cli.py`
- **Invoke:** `python3 ~/bin/ai/cli/cline_cli.py`
- **Description:** Cline CLI wrapper with tmux support

### lib_cli_common.py
- **Path:** `~/bin/ai/cli/lib_cli_common.py`
- **Description:** Shared library for all CLI wrappers — tmux, session management, common flags

---

## Agent Shortcuts

Convenience shell wrappers for specific agent profiles. Role configs live in `ai_general/roles/`.

| Script | Description |
|--------|-------------|
| `librarian_claude.sh` | Claude librarian agent |
| `librarian_gemini.sh` | Gemini librarian agent |
| `custodian_claude.sh` | Claude custodian agent |
| `dev_lead_claude.sh` | Claude dev-lead agent |
| `ops_codex.sh` | Codex ops agent |
| `gemini_session.sh` | Generic Gemini session launcher |

---

## Agent Roles

**Base:** `ai_general/roles/`  
**Manifest:** `ai_general/roles/manifest.yml`  
**Available:** librarian, dev_lead, custodian, ops, peer_review, tester, researcher, validator  
**Structure:** Each role has `role.yml` + `prompt.md` (or `instructions.md`)  
**Memory paths:** `ai_claude_cli/memories/{role}/`

---

## CLI Utilities

### Lifecycle
**Base:** `~/bin/ai/cli/lifecycle/`
- `cleanup_cli_sessions.sh`
- `list_cli_sessions.sh`
- `launch_claude_cli_task.scpt`
- `launch_codex_cli_task.scpt`

### Wrappers
**Base:** `~/bin/ai/cli/wrappers/`
- `cli_script_wrapper.sh`
- `cli_tmux_wrapper.sh`
- `check_lock.sh`
- `monitor_cli_sessions.py`

### Wave
**Base:** `~/bin/ai/cli/`
- `spawn_wave_librarians.sh`
- `wave_launcher.sh`
- `wave_monitor.sh`

### Shard Tools
**Base:** `~/bin/ai/cli/`  
Gemini shard creation and management for research orchestration.
- `create_shards.py`
- `rebuild_shards.py`
- `transform_shard_roles.py`

### Monitoring
**Base:** `~/bin/ai/cli/`
- `agent_monitor.sh`
- `tail_agent_log.sh`
- `spawn_subtasks.sh`

---

## Python Tools

**Base:** `~/bin/all_languages/python/src`

### todo_mgr
- **Path:** `~/bin/all_languages/python/src/todo_mgr/todo_mgr.py`
- **Description:** TODO system — kanban, status, queries
- **Commands:** `kanban`, `status <dir> <status>`, `query --status Ready`

### Other Python Packages
`ai_utils/`, `doc_utils/`, `yaml_utils/`, `json_utils/`, `file_utils/`, `archive_utils/`, `dev_utils/`, `metadata_utils/`, `repo_tools/`, `email_tools/`, `gdir/`

---

## Chat Pipeline

**Base:** `~/bin/ai/chat_pipeline`

### Numbered Steps (sequential pipeline)
1. `01_split_claude_bulk.py`
2. `02_convert_claude_json_to_v2.py`
3. `03_dedupe_conversations.py`
4. `04_extract_embedded_step.py`
5. `05_chunk_conversation.py`
6. `06_organize_histories.py`

### Utilities
`build_chat_index.py`, `curate_topics.py`, `extract_artifacts.py`, `extract_embedded_content.py`, `remediate_oversized_chunks.py`, `process_claude_bulk.py`, `pipeline_chatgpt_full.py`, `batch_remediate.py`

### YAML Fixers
`fix_yaml_errors.py`, `fix_yaml_errors_v2.py`, `fix_yaml_errors_v3.py`, `fix_yaml_errors_v4.py`, `fix_yaml_aggressive.py`, `fix_yaml_final.py`

---

## Chat Processing

**Base:** `~/bin/ai/chat_processing`  
Higher-level chat conversion and chunking library (pip-installable).

**Key files:** `chat_converter.py`, `chat_chunker.py`, `chat_export_splitter.py`, `doc_converter.py`, `md_structure_parser.py`  
**Sub-libraries:** `lib_converters/`, `lib_formatters/`, `lib_parsers/`

---

## Orchestration

**Base:** `~/bin/ai/orchestration`  
Multi-AI orchestration, query dispatch, callback handling.

- `dispatch_query.py` — route queries to appropriate AI
- `fire_callback.py` — trigger callbacks on task completion
- `research_orchestrator.sh` — coordinate research across platforms
- `orchestrate-chat.sh` — cross-platform chat orchestration
- `watch_dir.sh` — directory watcher for file-based coordination

---

## Prompting

**Base:** `~/bin/ai/prompting`  
Cross-platform prompt sending and scheduling.

- `send_prompt.sh` — unified prompt sender
- `lib_send_prompt_cli.sh` — CLI target delivery
- `lib_send_prompt_desktop.sh` — Desktop app delivery
- `lib_send_prompt_webui.sh` — Web UI delivery
- `scheduled_prompts_daemon.sh` — daemon for scheduled delivery
- `send_scheduled_prompt.sh` — execute a scheduled prompt
- `set_scheduled_prompt.sh` — configure a scheduled prompt
- `ai_isBusy.sh` — check if AI target is processing

---

## Pulse

**Base:** `~/bin/ai/pulse`  
Automated timed prompting for autonomous AI operation.

- `pulse_mgr.sh` — manage pulse configurations
- `send_pulse.sh` — send a pulse prompt
- `pulse_config.sh` — pulse configuration

---

## Desktop Automation

**Base:** `~/bin/ai/desktop_automation`  
AppleScript and browser-based Claude Desktop automation.

- `create_project_chat.applescript` — create new project chat
- `get_desktop_chat_url.sh` — get current Desktop app chat URL
- `new_project_chat.sh` — new project chat wrapper
- `export-claude-chat.py` — export chat from Desktop
- `chrome_dom_message_extractor.py` — extract messages via Chrome DOM
- `chat_url_monitor.py` — monitor chat URL changes

---

## Claude Automation

**Base:** `~/bin/ai/claude_automation`  
Claude-specific chat creation and management (Node.js).

- `create_claude_chat.js`
- `new_chat.sh`
- `open_claude_chat.sh`

---

## Gemini Chat

**Base:** `~/bin/ai/gemini_chat`  
Gemini browser automation — canvas downloads, file upload, chat control.

- `gemini_chat_automation.applescript` — chat interaction
- `gemini_download_canvas.js` / `gemini_download_canvas_filtered.js` — download canvas content
- `gemini_upload_file.js` — upload files to Gemini
- `gemini_list_files.js` — list files in conversation
- `gemini_smart_canvas_toggle.applescript` / `gemini_ensure_canvas.applescript` — canvas state management

---

## Chat Continuity

**Base:** `~/bin/ai/chat_continuity`  
Broken chat detection and auto-continuation.

- `chat_continuity.sh` — main continuity handler
- `continue-chat.sh` — continue a broken chat
- `detectBrokenChat.sh` — detect broken state
- `brokenChatDaemon.sh` — daemon for auto-detection
- `ai_export_chat.sh` — export for continuation

---

## Chats (Node.js Library)

**Base:** `~/bin/ai/chats`  
Node.js chat interaction library — extract, create, send, wait.

- `chat-extract.js`, `chat-new.js`, `chat-open.js`, `chat-send.js`, `chat-wait.js`
- `export_chats_since.sh`, `ai_export_chat.sh`, `ai_isBusy.sh`

---

## MCP Servers

All MCP servers live under `ai_general/apps/mcps/`.

### task-coord
- **Path:** `ai_general/apps/mcps/task-coord/server.py`
- **Type:** Python
- **Purpose:** Task coordination — playbooks, task lifecycle, file watchers
- **Tools:** `list_playbooks`, `get_playbook`, `start_playbook`, `stop_playbook`, `list_templates`, `gen_task`, `list_tasks`, `get_task`, `move_task`, `list_platforms`, `start_watcher`, `list_watchers`, `check_watcher`, `stop_watcher`, `cleanup_watchers`

### cli-agent
- **Path:** `ai_general/apps/mcps/cli-agent/server.py`
- **Type:** Python
- **Purpose:** Launch and manage CLI agents with role-based bootstrapping
- **Tools:** `launch_agent`, `launch_librarian`, `launch_dev_lead`, `launch_custodian`, `launch_ops`, `launch_peer_review`, `launch_tester`, `launch_researcher`, `launch_validator`, `kill`, `attach`, `send_keys`, `list_sessions`, `get_status`

### knowledge-search
- **Path:** `ai_general/apps/mcps/knowledge-search/server.py`
- **Type:** Python
- **Purpose:** Search 4+ year AI conversation archive (500+ chats, 4700+ chunks)
- **Tools:** `search`, `stats`, `grep_search`

### memory
- **Path:** `ai_general/apps/mcps/memory/server.py`
- **Type:** Python
- **Purpose:** Federated memory slot system — read/write/search across AI instances
- **Tools:** `get_manifest`, `read`, `append`, `update`, `delete`, `search`, `set_slot_config`, `stats`

### messages
- **Path:** `ai_general/apps/mcps/messages/server.py`
- **Type:** Python
- **Purpose:** Inter-agent messaging — broadcasts, direct messages, threading
- **Tools:** `broadcast`, `send_direct`, `list_broadcasts`, `list_direct`, `check_responses`, `acknowledge`

### prompting
- **Path:** `ai_general/apps/mcps/prompting/server.py`
- **Type:** Python
- **Purpose:** Cross-platform prompt delivery — CLI, Desktop, Web targets
- **Tools:** `send_prompt`, `is_busy`, `list_sessions`, `observe_session`, `wait_response`, `send_to_session`

### todo
- **Path:** `ai_general/apps/mcps/todo/server.py`
- **Type:** Python
- **Purpose:** TODO management — create, status, flags, tags, kanban
- **Tools:** `list`, `get`, `create`, `set_status`, `add_flag`, `remove_flag`, `add_tag`, `remove_tag`, `complete`, `trash`, `kanban`

### chat-pipeline
- **Path:** `ai_general/apps/mcps/chat-pipeline/server.py`
- **Type:** Python
- **Purpose:** Chat history processing pipeline — normalize, chunk, condense
- **Tools:** `pipeline_status`, `normalize`, `chunk_file`, `prepare_for_condensation`, `condense_history`, `review_quarantine`, `retry_quarantine`, `get_pipeline_config`

### chat (deprecated)
- **Path:** `ai_general/apps/mcps/chat/server.js`
- **Type:** Node.js
- **Purpose:** Chat archive access and search (deprecated in favor of knowledge-search)

---

## Utilities

**Base:** `~/bin/ai/utils`

- `file_size_stats.py` — file size analysis
- `manageCronJobs.sh` — cron job management
- `new_iterm_tab.sh` — open new iTerm tab
- `next_id.sh` — generate next sequential ID
- `repo_status.py` — git repo status checker
- `restartClaudeApp.sh` — restart Claude Desktop app
- `validate_tools_manifest.py` — validate this manifest against filesystem

---

## Quick Reference

| Tool | Command |
|------|---------|
| Claude CLI | `python3 ~/bin/ai/cli/claude_cli.py` |
| Codex CLI | `python3 ~/bin/ai/cli/codex_cli.py` |
| Gemini CLI | `python3 ~/bin/ai/cli/gemini_cli.py` |
| Cline CLI | `python3 ~/bin/ai/cli/cline_cli.py` |
| TODO Manager | `python3 ~/bin/all_languages/python/src/todo_mgr/todo_mgr.py` |
| List Sessions | `~/bin/ai/cli/lifecycle/list_cli_sessions.sh` |
| Send Prompt | `~/bin/ai/prompting/send_prompt.sh` |
| Pulse Manager | `~/bin/ai/pulse/pulse_mgr.sh` |

---

## Notes

- `~` = `/Users/shawnhillis`
- CLI wrappers are Python (migrated from `.sh` — 2026-01)
- All python scripts need explicit `python3`
- Roles live in `ai_general/roles/` with `role.yml` + `prompt.md` per role
- 8 roles: librarian, dev_lead, custodian, ops, peer_review, tester, researcher, validator
- `cline_cli.py` added for Cline integration (2026-02)
- 10 MCP servers: task-coord, cli-agent, knowledge-search, memory, messages, prompting, todo, chat-pipeline, chat
- Shard tools (`create_shards.py`, `rebuild_shards.py`) support Gemini research orchestration
- `validate_tools_manifest.py` in `~/bin/ai/utils/` for self-validation
