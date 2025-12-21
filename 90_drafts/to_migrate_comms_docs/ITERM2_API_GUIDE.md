# iTerm2 Python API Implementation Guide for CLI Prompt Injection

**Version:** 1.0 (Draft - awaiting test results)  
**Created:** 2025-11-10  
**Purpose:** Enable direct prompt injection into CLI instances using iTerm2 Python API

---

## Overview

This guide documents how to use the iTerm2 Python API to inject prompts directly into CLI terminal sessions, enabling real-time CLI-to-CLI coordination and Desktop-to-CLI communication without filesystem polling delays.

**Use cases:**
- Desktop Claude → CLI instance coordination
- CLI → CLI peer communication
- Emergency interrupts and urgent notifications
- REPL state sharing between CLI instances

---

## Prerequisites

### System Requirements
- iTerm2 version 3.3.0 or later
- Python 3.7+
- iTerm2 Python API enabled

### Enable iTerm2 Python API

1. Open iTerm2 Preferences (Cmd+,)
2. Navigate to "General" → "Magic"
3. Check "Enable Python API"
4. Restart iTerm2

### Install iTerm2 Python Module

```bash
pip3 install iterm2 --break-system-packages
```

---

## Architecture

### Component Overview

```
Desktop Claude
    ↓
Write prompt file
    ↓
claude_cli/prompting/incoming/test.prompt.txt
    ↓
cli_prompt_daemon.py (monitors directory)
    ↓
cli_prompt_injector.py (iTerm2 API)
    ↓
iTerm2 Session (CLI terminal)
    ↓
CLI instance receives prompt
```

---

## Core Components

*(Implementation details to be filled in after req_1102 testing completes)*

### 1. CLI Prompt Injector
**File:** `~/bin/ai/cli_prompt_injector.py`

### 2. CLI Registry Manager  
**File:** `~/bin/ai/cli_registry_manager.py`

### 3. Prompt Daemon
**File:** `~/bin/ai/cli_prompt_daemon.py`

---

## Related Documentation
- iTerm2 Python API: https://iterm2.com/python-api/
- CLI Coordination v4.0: `coordination_system_v4_digest.md`
- AI Comms Architecture: `ARCHITECTURE_v1.1.md`

---

**Status:** Draft - Will be completed after req_1102 testing  
**Next Update:** Full implementation details, code examples, and test results
