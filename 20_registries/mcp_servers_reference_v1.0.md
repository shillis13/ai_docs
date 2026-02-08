# MCP Servers & Tools Reference

**Version:** 1.0.0  
**Status:** Active  
**Location:** `ai_general/docs/20_registries/`  
**Created:** 2026-02-04  

---

## Overview

MCP (Model Context Protocol) servers extend Desktop Claude's capabilities by providing
structured tool access to filesystem operations, AI execution, agent management, messaging,
and more. Each server exposes a set of tools callable directly from Desktop Claude's context.

## Server Inventory

### Desktop Commander

**Purpose:** Filesystem operations, process management, file search, and system interaction.  
**Type:** Third-party MCP server  
**Used by:** Desktop Claude (primary), Codex MCP (as connector)

| Tool Category | Functions |
|:---|:---|
| File I/O | `read_file`, `read_multiple_files`, `write_file`, `write_pdf`, `edit_block` |
| Directory | `list_directory`, `create_directory`, `move_file`, `get_file_info` |
| Search | `start_search`, `get_more_search_results`, `stop_search`, `list_searches` |
| Processes | `start_process`, `interact_with_process`, `read_process_output`, `force_terminate` |
| Sessions | `list_sessions`, `list_processes`, `kill_process` |
| Config | `get_config`, `set_config_value` |
| Meta | `get_usage_stats`, `get_recent_tool_calls` |


### Codex MCP

**Purpose:** Synchronous AI execution — invoke OpenAI Codex as an internal tool for bounded tasks.  
**Type:** Custom MCP server (codex:)  
**Used by:** Desktop Claude only  
**Timeout:** 30–60 seconds  
**Key distinction:** This is a tool, NOT a worker. Stateless, synchronous, returns results inline.

| Tool | Description |
|:---|:---|
| `codex:codex` | Start a new Codex session with prompt, optional config overrides |
| `codex:codex-reply` | Continue an existing Codex conversation by thread ID |

**Connectors available to Codex:** Desktop Commander, filesystem, shell/processes, web search.

---

### CLI Agent MCP

**Purpose:** Launch and manage CLI agents with role-based bootstrapping.  
**Type:** Custom MCP server (cli-agent:)  
**Source:** `ai_general/apps/mcps/cli-agent/server.py`

| Tool | Description |
|:---|:---|
| `launch_agent` | Generic launcher — specify platform + role |
| `launch_librarian` | Memory system curator |
| `launch_dev_lead` | Development coordinator |
| `launch_custodian` | Repository maintainer |
| `launch_ops` | Task execution coordinator |
| `launch_peer_review` | Code/design reviewer |
| `launch_tester` | Testing and validation |
| `launch_researcher` | Corpus search orchestrator (Claude CLI, coordinates Gemini shards) |
| `launch_validator` | Adversarial cross-checker (must run on different model than synthesizer) |
| `kill` | Terminate a running agent session |
| `attach` | Get command to attach to agent tmux session |
| `send_keys` | Send keystrokes to agent session |
| `list_sessions` | List all active CLI agent sessions |
| `get_status` | Detailed status including recent output |

**Platforms:** claude_cli, codex_cli, gemini_cli  
**Roles:** librarian, dev_lead, custodian, ops, peer_review, tester, researcher, validator

---

### Task Coordination MCP

**Purpose:** Playbook-based orchestration and task lifecycle management.  
**Type:** Custom MCP server (task-coord:)  
**Source:** `ai_general/apps/mcps/task-coord/server.py`

| Tool | Description |
|:---|:---|
| `list_playbooks` | Available orchestration patterns |
| `get_playbook` | Full playbook definition |
| `start_playbook` | Create initial task or run start action |
| `stop_playbook` | Cancel a running playbook |
| `list_templates` | Available task templates |
| `gen_task` | Generate task from template |
| `list_tasks` | List tasks, filter by platform/status |
| `get_task` | Get task content by ID |
| `move_task` | Change task status (staged → to_execute → in_progress → completed) |
| `list_platforms` | Available execution platforms |

---

### Knowledge Search MCP

**Purpose:** Dual-mode search over 4+ year conversation archive (500+ chats, 4700+ chunks).  
**Type:** Custom MCP server (knowledge-search:)

| Tool | Description |
|:---|:---|
| `search` | Topic/keyword search with 4-layer cascade. Modes: SEARCH (curated results) or ANSWER (editorial synthesis with citations) |
| `grep_search` | Regex over full chunk file contents — slower but catches terms not in topic index |
| `stats` | Knowledge base index statistics |

**4-Layer Cascade:** L1 topic index → L2 topics + synonyms → L3 full content → L4 content + synonyms

---

### Chat Pipeline MCP

**Purpose:** Processing pipeline for chat history: export → normalize → chunk → condense.  
**Type:** Custom MCP server (chat-pipeline:)

| Tool | Description |
|:---|:---|
| `pipeline_status` | Counts and status for each pipeline stage |
| `normalize` | Convert raw exports to normalized YAML |
| `chunk_file` | Chunk a specific conversation file |
| `prepare_for_condensation` | Full pipeline: JSON → YAML → chunk → organize into YYYY/MM |
| `condense_history` | Condense a chat history file (async via librarian or sync via Codex) |
| `review_quarantine` | List quarantined files with failure reasons |
| `retry_quarantine` | Retry processing a quarantined file |
| `get_pipeline_config` | Get or update pipeline configuration |

---

### Messages MCP

**Purpose:** Inter-AI messaging — broadcasts and direct messages between agents.  
**Type:** Custom MCP server (messages:)

| Tool | Description |
|:---|:---|
| `broadcast` | Send message to all agents |
| `send_direct` | Send message to specific agent or session |
| `list_broadcasts` | List recent broadcast messages |
| `list_direct` | List direct messages (inbox) for a recipient |
| `check_responses` | Check for responses to a specific message |
| `acknowledge` | Mark a message as read/processed |

---

### Prompting MCP

**Purpose:** Deliver prompts to AI targets with proper timing, busy detection, and verification.  
**Type:** Custom MCP server (prompting:)

| Tool | Description |
|:---|:---|
| `send_prompt` | Send prompt to any AI target (Desktop, Web, CLI, Codex, Gemini, ChatGPT) |
| `is_busy` | Check if AI target is currently processing |
| `list_sessions` | List active tmux sessions |
| `observe_session` | Capture current state of a CLI session |
| `wait_response` | Wait for response pattern in CLI session |
| `send_to_session` | Low-level direct tmux send with timing |

**Targets:** claude-desktop, claude-web, claude-cli, codex-cli, gemini-cli, chatgpt-web, chatgpt-app

---

### Todo MCP

**Purpose:** Todo/task management with kanban views, status tracking, tags, and flags.  
**Type:** Custom MCP server (todo:)

| Tool | Description |
|:---|:---|
| `list` | List todos with optional filtering |
| `get` | Get details by ID, number, or reference code |
| `create` | Create new todo |
| `set_status` | Update status (Triaging → Ready → In_Progress → Done) |
| `add_flag` / `remove_flag` | Manage flags (e.g., high_priority, needs_testing) |
| `add_tag` / `remove_tag` | Manage tags |
| `complete` | Mark done and move to completed/ |
| `trash` | Soft delete |
| `kanban` | Kanban board view (JSON or text) |

---

### Chat Tools MCP

**Purpose:** Chat navigation, export, import, and browser-based chat interaction.  
**Type:** Custom MCP server (chat:)

| Tool | Description |
|:---|:---|
| `open_chat` | Navigate to specific chat by URL or ID |
| `open_project` | Navigate to a Claude project |
| `new_chat` | Open new chat, optionally in project |
| `get_messages` | Extract messages from current chat |
| `send_message` | Type and send message to Claude |
| `wait_response` | Wait for Claude to finish responding |
| `export_chats` | Export chats to JSON via Playwright DOM extraction |
| `import_chat` | Export → convert → condense pipeline |
| `continue_in_new_chat` | Fork current chat with condensed context |
| `attach_file` | Attach file to current chat |
| `detect_active` | Detect which AI chat is active in browser |
| `restart_desktop_claude` | Restart app and reopen current chat |

---

### Chrome Control MCP

**Purpose:** Browser tab management and JavaScript execution.  
**Type:** Third-party MCP server

| Tool | Description |
|:---|:---|
| `open_url` | Open URL in Chrome |
| `get_current_tab` / `list_tabs` | Tab information |
| `switch_to_tab` / `close_tab` / `reload_tab` | Tab management |
| `go_back` / `go_forward` | Browser navigation |
| `execute_javascript` | Run JS in current tab |
| `get_page_content` | Extract page text content |

---

## Tool Usage by Actor

| MCP Server | Desktop Claude | CLI Agents | Codex MCP |
|:---|:---:|:---:|:---:|
| Desktop Commander | ✓ primary | — | ✓ connector |
| Codex MCP | ✓ invokes | — | — |
| CLI Agent | ✓ launches | — | — |
| Task Coordination | ✓ | — | — |
| Knowledge Search | ✓ | ✓ (direct) | — |
| Chat Pipeline | ✓ | — | — |
| Messages | ✓ | ✓ (file-based) | — |
| Prompting | ✓ | — | — |
| Todo | ✓ | — | — |
| Chat Tools | ✓ | — | — |
| Chrome Control | ✓ | — | — |
