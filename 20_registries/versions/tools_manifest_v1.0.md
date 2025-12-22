# Tools Manifest

**Version:** 1.0.0
**Updated:** 2025-12-14
**Purpose:** Canonical paths for all AI-accessible tools

---

## Overview

This manifest provides the single source of truth for AI tool locations. All AI instances (Desktop Claude, CLI agents, Codex) should reference this file before invoking tools by name.

---

## CLI Wrappers

**Base Path:** `~/bin/ai/cli`

### claude_cli.sh
**Path:** `~/bin/ai/cli/claude_cli.sh`
**Description:** Claude CLI wrapper with tmux, agents, session management

**Usage:**
```
claude_cli.sh [options] "prompt"
-t, --tmux        Run in tmux session
-a, --auto        Skip permission prompts
-A, --agent TYPE  Use agent (librarian, dev-lead, custodian, ops)
-n, --named NAME  Resume named session
-c, --continue    Continue most recent session
```

**Examples:**
- `claude_cli.sh -a -A librarian -t "Process exports"`
- `claude_cli.sh -t -a -c "Continue refactoring"`

### codex_cli.sh
**Path:** `~/bin/ai/cli/codex_cli.sh`
**Description:** Codex CLI wrapper with tmux support

**Usage:**
```
codex_cli.sh [options] "prompt"
-t, --tmux        Run in tmux session
-a, --auto        Full auto mode (--dangerously-bypass-approvals-and-sandbox)
-s, --session     Tmux session name
-w, --workdir     Working directory
```

---

## CLI Utilities

**Base Path:** `~/bin/ai/cli`

### Lifecycle Scripts
**Path:** `~/bin/ai/cli/lifecycle/`
- `cleanup_cli_sessions.sh`
- `list_cli_sessions.sh`
- `launch_claude_cli_task.scpt`
- `launch_codex_cli_task.scpt`

### Wrappers
**Path:** `~/bin/ai/cli/wrappers/`
- `cli_script_wrapper.sh`
- `cli_tmux_wrapper.sh`
- `check_lock.sh`
- `monitor_cli_sessions.py`

---

## Python Tools

**Base Path:** `~/bin/all_languages/python/src`

### todo_mgr
**Path:** `~/bin/all_languages/python/src/todo_mgr/todo_mgr.py`
**Description:** TODO system manager - kanban, status changes, queries

**Usage:**
```
python3 todo_mgr.py kanban
python3 todo_mgr.py status <todo_dir> <new_status>
python3 todo_mgr.py query --status Ready
```

### Other Python Tools
- `ai_utils/` - AI-specific utilities (chat processing, memory ops)
- `doc_utils/` - Document processing utilities
- `yaml_utils/` - YAML manipulation utilities

---

## Chat Pipeline

**Base Path:** `~/bin/ai/chat_pipeline`

Main scripts:
- `export_chats.py`
- `preprocess_exports.py`
- `convert_to_yaml.py`

---

## Automation

**Base Path:** `~/bin/ai`

- `desktop_automation/` - AppleScript and macOS automation
- `claude_automation/` - Claude Desktop UI automation

---

## Quick Reference

| Operation | Path |
|-----------|------|
| Invoke Claude CLI | `~/bin/ai/cli/claude_cli.sh` |
| Invoke Codex CLI | `~/bin/ai/cli/codex_cli.sh` |
| Run todo_mgr | `python3 ~/bin/all_languages/python/src/todo_mgr/todo_mgr.py` |
| List sessions | `~/bin/ai/cli/lifecycle/list_cli_sessions.sh` |

---

## Notes

- All paths use `~` for portability (expands to `/Users/shawnhillis`)
- CLI wrappers handle PATH setup internally
- Python tools may need explicit `python3` invocation
- Codex CLI `-a` flag uses `--dangerously-bypass-approvals-and-sandbox`
