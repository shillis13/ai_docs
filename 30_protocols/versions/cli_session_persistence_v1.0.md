# Claude CLI Session Persistence Architecture

**Discovered:** 2025-12-05  
**Status:** Documented, wrapper script created

---

## How It Works

### Session Storage Model

```
Local (~/.claude/):
├── history.jsonl          # Session index (sessionId, timestamp, project, prompt)
├── session-env/<uuid>/    # Empty - placeholder for env vars
├── file-history/<uuid>/   # File changes made during session
├── debug/<uuid>.txt       # Debug logs
└── projects/              # Per-project settings (path-encoded)

Server-side (Anthropic):
└── Conversation content   # Retrieved via sessionId
```

**Key insight:** Conversation content is NOT stored locally. The `sessionId` is a key to retrieve server-stored context via `claude -r <sessionId>`.

### history.jsonl Format

```json
{
  "display": "the user prompt",
  "pastedContents": {},
  "timestamp": 1764983075775,
  "project": "/path/to/working/directory",
  "sessionId": "257c13b3-c315-479e-86fc-eed05cb90a0d"
}
```

Sessions are **scoped by project directory**. Running `-r` in a directory shows only sessions from that directory.

---

## CLI Commands

```bash
# Continue most recent session in current directory
claude -c "continue prompt"

# Resume specific session by ID
claude -r abc123 "resume prompt"

# Interactive session picker
claude -r
```

---

## Session-Aware Wrapper

**Location:** `~/bin/ai/claude_cli_session.sh`

### Query Commands
```bash
# List sessions for a project
./claude_cli_session.sh list ~/path/to/project

# Get most recent session ID
./claude_cli_session.sh latest ~/path/to/project

# Get info about specific session
./claude_cli_session.sh info <session_id>
```

### Execution Commands
```bash
# Start new session
./claude_cli_session.sh new ~/project "prompt"

# Continue most recent
./claude_cli_session.sh continue ~/project "prompt"

# Resume specific session
./claude_cli_session.sh resume <session_id> "prompt"

# Auto mode (continue if exists, else new)
./claude_cli_session.sh auto ~/project "prompt"

# Execute task file with session directives
./claude_cli_session.sh task /path/to/task.md
```

---

## Task File Format

```yaml
---
session_mode: auto  # auto | new | continue | resume:<ID>
project_dir: ~/Documents/AI/ai_root
# claude_args: --model opus
---
# Prompt content below

Your task description here...
```

**Template:** `~/.claude/coordination/docs/task_template_session_aware.md`

---

## Coordination Protocol Integration

### Pattern: Persistent Worker Sessions

```yaml
# Task can specify session handling
task:
  id: req_1050
  session_mode: continue
  project_dir: ~/bin/projects/chat_orchestrators/
```

CLI worker accumulates context about codebase across multiple tasks. Resume later with full understanding preserved.

### Pattern: Fresh Sessions for Isolated Work

```yaml
task:
  id: req_1051
  session_mode: new
  project_dir: ~/tmp/scratch/
```

Disposable session for one-off work that shouldn't pollute other contexts.

### Pattern: Named Session Streams

```bash
# Debug stream
./claude_cli_session.sh resume debug-session-id "investigate the bug"

# Feature stream  
./claude_cli_session.sh resume feature-session-id "continue implementation"
```

Multiple logical work streams per project, each with accumulated context.

---

## Implications for Multi-AI Orchestration

1. **CLI workers can build expertise** - A CLI repeatedly working on chat_orchestrators/ accumulates understanding

2. **Session continuity is free** - No custom implementation needed, just leverage native `-c`/`-r`

3. **Project-scoped context** - Natural isolation between different codebases

4. **Server-side storage** - No local cleanup needed, Anthropic handles retention

---

## Open Questions

- Session retention duration? (How long does Anthropic keep session data?)
- Session size limits? (Can a session grow indefinitely?)
- Cross-device resume? (Can session started on one machine resume on another?)

---

**Files Created:**
- `~/bin/ai/cli/claude_cli_session.sh` - Session-aware wrapper (643 lines)
- `~/.claude/session_registry.yml` - Named session registry
- `ai_general/docs/30_protocols/cli_session_persistence.md` - This doc
- `ai_general/templates/cli_task_session_aware.md` - Task file template

**Symlinks in ai_comms/claude_cli/:**
- `session_persistence.md` → protocol doc
- `task_template.md` → task template
