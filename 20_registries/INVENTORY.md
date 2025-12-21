# AI Coordination Systems Inventory

**Version:** 1.0 DRAFT  
**Created:** 2025-11-27  
**Status:** In Progress  
**Location:** `ai_general/todos/completed/todo_0001_ai_orchestration_overview/`

---

## 1. CLI Task Coordination Protocol

**Version:** v5.0 (protocol_taskCoordination_v5.0.yml)  
**Location:** `ai_comms/claude_cli/` (symlinked from `~/.claude/coordination/`)  
**Spec:** `ai_general/specs_and_protocols/protocol_taskCoordination_v5.0.yml`

**Purpose:** Async task delegation from Desktop Claude to CLI Claude instances.

**Present State:**
- Directory-based lifecycle: `to_execute/` → `in_progress/` → `completed/`
- Broadcasts for any CLI, direct assignments for specific instances
- Response collection in `responses/` and `synthesis/`
- Notification system integrated

**Known Issues:**
- [ ] `staged/` directory mentioned in docs but workflow unclear
- [ ] Task spec format not fully documented
- [ ] Worker zombie detection criteria undefined

**Test Coverage:** Manual testing only, no automated regression

---

## 2. Chat Orchestrator (Puppeteer)

**Location:** `~/bin/projects/chat_orchestrators/chat_orchestrator_puppeteer/`  
**Spec:** `spec_ai_message_sender_v1.1.yml`

**Purpose:** Automated relay between AI chat interfaces (Claude web, ChatGPT web).

**Present State:**
- Puppeteer-based browser automation
- JSONL logging for message history
- Manual/auto relay modes with pause/resume
- Message modification capabilities

**Known Issues:**
- [ ] Requires Chrome with `--remote-debugging-port=9222`
- [ ] DOM selectors may break on UI updates
- [ ] Message extraction bugs reported in some scenarios

**Test Coverage:** Manual testing, no automated suite

---

## 3. Desktop Prompt Sender (AppleScript)

**Location:** `~/bin/projects/chat_orchestrators/claude_desktop_prompt_sender_v2.sh`  
**Related:** `~/bin/ai/pulse_orchestrator.sh`

**Purpose:** Inject prompts into Desktop Claude app via AppleScript automation.

**Present State:**
- Clipboard-paste method for reliable text injection
- File-based trigger queue at `ai_claude/work/`
- Integrated with pulse system

**Known Issues:**
- [ ] Cannot reliably READ responses (send-only)
- [ ] Depends on UI element stability
- [ ] No way to detect if Claude is mid-response

**Test Coverage:** Manual testing only

---

## 4. Pulse System (Cron-based Wake-up)

**Location:** `~/bin/ai/pulse_orchestrator.sh`  
**Prompt:** `ai_claude/work/pulse_prompt.md`  
**Logs:** `ai_general/logs/claude/pulse.log`

**Purpose:** Periodic wake-up for autonomous Desktop Claude operation.

**Present State:**
- Cron job every 10 minutes
- Lock file support (`ai_claude/CONVERSATION_ACTIVE`)
- Idle auto-release (30 min threshold)
- Pause file support (`ai_claude/work/.paused`)

**Known Issues:**
- [ ] Cron location not documented in central place
- [ ] No metrics collection (queue depth, etc.)
- [ ] Pausing is one-way until user returns

**Test Coverage:** Manual testing only

---

## 5. Notification System

**Location:** `ai_comms/claude/notifications/`  
**Format:** YAML files with structured metadata

**Purpose:** Async notifications between coordination systems and Desktop Claude.

**Present State:**
- `pending/` and `processed/` directories
- YAML format with id, target, priority, message fields
- Manual processing during pulse checks

**Known Issues:**
- [ ] No automated delivery mechanism
- [ ] Priority levels not clearly defined
- [ ] Old notifications can accumulate

**Test Coverage:** None

---

## 6. Codex CLI Coordination

**Location:** `ai_comms/codex_cli/`  
**Init:** `~/.claude/coordination/init/codex_cli_init.md`

**Purpose:** Task coordination for Codex CLI autonomous execution.

**Present State:**
- Same directory pattern as Claude CLI
- Symlinked into coordination structure
- Used for synchronous task execution

**Known Issues:**
- [ ] Underutilized - most work goes to Claude CLI
- [ ] No clear differentiation criteria from Claude CLI

**Test Coverage:** Minimal

---

## Summary Metrics

| System | Status | Test Coverage | Priority Issues |
|--------|--------|---------------|-----------------|
| CLI Task Coordination | Operational | Manual | 3 |
| Chat Orchestrator | Operational | Manual | 3 |
| Desktop Prompt Sender | Operational | Manual | 3 |
| Pulse System | Operational | Manual | 3 |
| Notification System | Operational | None | 3 |
| Codex CLI | Underutilized | Minimal | 2 |

---

## Documentation Index (26 files identified)

### Architecture & Design
- `ai_comms/README.md` - Three-layer communication architecture
- `ai_comms/docs/DESKTOP_CLI_CONTROL_ARCHITECTURE.md` - Bidirectional control design
- `ai_general/docs/ARCHITECTURE_AI_COMMUNICATION_AND_MONITORING.md` - End-to-end blueprint
- `ai_general/docs/cli_orchestration_v1.md` - CLI orchestration patterns
- `ai_general/docs/architectural_layer_model.yml` - Layer model

### Protocol Specifications
- `ai_general/specs_and_protocols/protocol_taskCoordination_v5.0.yml` - Task protocol v5.0
- `ai_general/specs_and_protocols/schema_taskFile_v1.0.yml` - Task file schema
- `ai_general/specs_and_protocols/spec_ai_message_sender_v1.1.yml` - Message injection
- `ai_general/specs_and_protocols/spec_chat_continuity_recovery_v1.0.yml` - Recovery spec

### Quick References
- `ai_comms/QUICK_REFERENCE.md` - Communication cheat sheet
- `ai_comms/docs/cli_controls/QUICK_REFERENCE.md` - CLI commands
- `ai_comms/COORDINATION_DASHBOARD.md` - Status dashboard

### Installation & Setup
- `ai_general/docs/install_cli_codex_coordination_v1.md` - Codex CLI setup
- `ai_general/docs/cli_mcp_configuration.md` - MCP configuration

---

## Next Steps

1. [ ] Document `staged/` directory workflow
2. [ ] Create Task spec format template
3. [ ] Define worker zombie detection criteria (suggest: 24h no activity)
4. [ ] Build automated test suite for critical paths
5. [x] Create maintenance playbook ✓ DONE
6. [x] Consolidate scattered documentation ✓ INDEXED

---

**Document Status:** v1.0 - Ready for review
**Last Updated:** 2025-11-27
