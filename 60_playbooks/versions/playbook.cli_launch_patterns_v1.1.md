# CLI Launch Patterns for Desktop Claude

**Version:** 1.1.0
**Created:** 2025-11-30
**Updated:** 2026-01-06
**Purpose:** Standard patterns for Desktop Claude to launch and monitor Claude CLI and Codex CLI

---

## Overview

Desktop Claude cannot launch CLI processes directly via Desktop Commander - the subprocess environment lacks a proper TTY and hangs indefinitely. Instead, use **AppleScript + iTerm2 + tmux** for reliable execution with monitoring capability.

---

## The Standard Pattern

### Launch Command (AppleScript)

```applescript
tell application "iTerm2"
    activate
    set newWindow to (create window with default profile)
    tell current session of newWindow
        write text "~/bin/ai/cli/claude_cli.sh -t -s SESSION_NAME -p -a 'YOUR PROMPT HERE'"
    end tell
    set miniaturized of newWindow to true
end tell
```

**IMPORTANT:** Use `write text` to send the command. The `create window with ... command "..."` syntax leaves blank windows.

### Monitor via Desktop Commander

```bash
# Check if running
tmux list-sessions

# Capture output
tmux capture-pane -t SESSION_NAME -p -S -500

# Send input if stuck
tmux send-keys -t SESSION_NAME "y" Enter

# Kill when done
tmux kill-session -t SESSION_NAME
```

---

## Script Flags

### claude_cli.sh / codex_cli.sh

| Flag | Purpose |
|:-----|:--------|
| `-t, --tmux` | **REQUIRED** - Run in tmux (enables monitoring) |
| `-s, --session NAME` | Session name for tracking (default: cli_<pid>) |
| `-p, --print` | Print mode - non-interactive |
| `-a, --auto` | Auto-approve tool calls |
| `-w, --workdir DIR` | Working directory |
| `--no-mcp` | Skip MCP configuration |

### Typical Usage

```bash
# Claude CLI - full automation
~/bin/ai/cli/claude_cli.sh -t -s mywork -p -a "your prompt"

# Codex CLI - full automation  
~/bin/ai/cli/codex_cli.sh -t -s codex_task -p -a "your prompt"

# Cline CLI - local LLM (requires llm_serve.sh running first)
~/bin/ai/cli/cline_cli.py --queue "your task"
```

### Cline CLI (Local LLM)

Cline uses a local llama-server backend instead of cloud APIs.

**Prerequisites:**
```bash
# Start local LLM server FIRST
ai_general/scripts/local_llm/llm_serve.sh code   # Port 8081
```

**Usage:**
```bash
# Queue a task (async)
~/bin/ai/cli/cline_cli.py --queue "Analyze this file"

# Run directly (sync)
~/bin/ai/cli/cline_cli.py --run "Simple task"

# Raw mode (skip context injection)
~/bin/ai/cli/cline_cli.py --queue "Quick fix" --raw
```

**Key Differences:**
- No tmux required (Python wrapper handles execution)
- Workspace context from `.clinerules` auto-injected
- Backend: Qwen3-Coder-30B via llama-server/MLX

---

## Complete Desktop Claude Workflow

### 1. Launch via AppleScript

```python
# In Desktop Claude's tool call:
Control your Mac:osascript with script:
'''
tell application "iTerm2"
    create window with default profile command "~/bin/ai/cli/claude_cli.sh -t -s librarian -p -a 'Process the pipeline'"
    tell current window
        set miniaturized to true
    end tell
end tell
'''
```

### 2. Wait and Monitor

```python
# Check if running (via Desktop Commander)
start_process("sleep 30 && tmux list-sessions")

# Capture output
start_process("tmux capture-pane -t librarian -p -S -500")
```

### 3. Handle Stuck Prompts

```python
# If CLI asks for input
start_process("tmux send-keys -t librarian 'y' Enter")
```

### 4. Check Completion

```python
# Session disappears when CLI exits (after 30s sleep)
start_process("tmux list-sessions")
# Returns: "no server running" = completed
```

---

## Why This Pattern?

### Desktop Commander Limitations
- Spawns subprocesses with `</dev/null` (closed stdin)
- No TTY - CLI hangs waiting for terminal
- Cannot send input mid-process
- Timeout eventually kills stuck process

### AppleScript + iTerm2 + tmux Benefits
- **Real PTY** - Proper terminal environment
- **Background** - `miniaturized to true` prevents focus stealing
- **Monitoring** - `tmux capture-pane` reads output
- **Interaction** - `tmux send-keys` handles prompts
- **Persistence** - Session survives if Desktop Claude disconnects

---

## Logs

| CLI | Log Location |
|:----|:-------------|
| claude_cli.sh | `~/Documents/AI/ai_root/ai_general/logs/claude_cli/` |
| codex_cli.sh | `~/Documents/AI/ai_root/ai_general/logs/codex_cli/` |
| cline_cli.py | `~/Documents/AI/ai_root/ai_cline/logs/` |

---

## Quick Reference

```bash
# Start local LLM server (required for Cline)
ai_general/scripts/local_llm/llm_serve.sh code

# Launch Claude CLI (from AppleScript)
~/bin/ai/cli/claude_cli.sh -t -s SESSION -p -a "prompt"

# Launch Codex CLI (from AppleScript)
~/bin/ai/cli/codex_cli.sh -t -s SESSION -p -a "prompt"

# Launch Cline CLI (local LLM)
~/bin/ai/cli/cline_cli.py --queue "task"

# Monitor (Claude/Codex tmux sessions)
tmux capture-pane -t SESSION -p -S -500

# Input
tmux send-keys -t SESSION "response" Enter

# Kill
tmux kill-session -t SESSION

# Check running
tmux list-sessions
```

---

## Common Issues

### "no server running"
- tmux session ended (CLI completed or crashed)
- Check log file for output

### Empty tmux capture
- CLI still initializing
- Wait and retry capture

### CLI hangs
- Waiting for input (permission prompt)
- Use `tmux send-keys` to respond
- Or ensure `-a` flag was used

### Focus stealing
- Forgot `set miniaturized to true`
- iTerm window took focus
