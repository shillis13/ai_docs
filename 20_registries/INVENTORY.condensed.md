# AI Coordination Systems Inventory (Condensed)
**Source:** INVENTORY.md | v1.0 | 2025-11-27

## Systems Overview

| System | Location | Purpose | Test Coverage |
|--------|----------|---------|---------------|
| CLI Task Coordination v5.0 | `ai_comms/claude_cli/` | Desktop→CLI task delegation | Manual |
| Chat Orchestrator | `~/bin/projects/chat_orchestrators/.../puppeteer/` | Claude↔ChatGPT relay | Manual |
| Desktop Prompt Sender | `~/bin/projects/.../claude_desktop_prompt_sender_v2.sh` | AppleScript prompt injection | Manual |
| Pulse System | `~/bin/ai/pulse_orchestrator.sh` | Cron wake-up (10min) | Manual |
| Notification System | `ai_comms/claude/notifications/` | Async notifications (YAML) | None |
| Codex CLI | `ai_comms/codex_cli/` | Codex task coordination | Minimal |

## Known Issues by System

**CLI Task Coordination:**
- staged/ workflow unclear
- Task spec format not fully documented
- Worker zombie detection criteria undefined

**Chat Orchestrator:**
- Requires Chrome with `--remote-debugging-port=9222`
- DOM selectors may break on UI updates
- Message extraction bugs in some scenarios

**Desktop Prompt Sender:**
- Cannot reliably READ responses (send-only)
- Depends on UI element stability
- No way to detect if Claude is mid-response

**Pulse System:**
- Cron location not documented centrally
- No metrics collection (queue depth, etc.)
- Pausing is one-way until user returns

**Notification System:**
- No automated delivery mechanism
- Priority levels not clearly defined
- Old notifications can accumulate

**Codex CLI:**
- Underutilized - most work goes to Claude CLI
- No clear differentiation criteria from Claude CLI

## Key Specs
- `protocol_taskCoordination_v5.0.yml` - Task lifecycle
- `spec_ai_message_sender_v1.1.yml` - Message injection
- `schema_taskFile_v1.0.yml` - Task file format

## Doc Categories (26 files)
- **Architecture:** ai_comms/README.md, DESKTOP_CLI_CONTROL_ARCHITECTURE.md
- **Protocols:** protocol_taskCoordination_v5.0.yml, spec_ai_message_sender_v1.1.yml
- **Quick Refs:** ai_comms/QUICK_REFERENCE.md, COORDINATION_DASHBOARD.md

## Next Steps
- [ ] Document staged/ workflow
- [ ] Create Task spec format template
- [ ] Define worker zombie detection criteria (suggest: 24h no activity)
- [ ] Build automated test suite for critical paths
- [x] Create maintenance playbook
- [x] Consolidate scattered documentation
