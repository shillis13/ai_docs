# AI Orchestration Maintenance Playbook

**Version:** 1.0  
**Created:** 2025-11-27  
**Purpose:** Quick reference for maintaining AI coordination infrastructure

---

## Daily Checks (Each Pulse)

### 1. Communication Queues
```bash
# Check for pending notifications
ls ~/Documents/AI/ai_root/ai_comms/claude_cli/notifications/pending/

# Check task queues
ls ~/.claude/coordination/tasks/to_execute/
ls ~/.claude/coordination/tasks/in_progress/
```

**Action:** Process stale items, move old notifications to `processed/`

### 2. WIP Limits
- Desktop Claude: 3-5 tasks max in progress
- Orchestrated total: 10-15 max
- Per worker: 1-2 max

**Action:** If exceeded, focus on completing before starting new

### 3. Pulse System Health
```bash
# Check if paused
ls ~/Documents/AI/ai_root/ai_claude/work/.paused

# Check lock file
ls ~/Documents/AI/ai_root/ai_claude/CONVERSATION_ACTIVE
```

---

## Weekly Checks

### 4. Stale Task Detection
```bash
# Find in_progress items older than 7 days
find ~/.claude/coordination/tasks/in_progress -type f -mtime +7
```

**Action:** Review and either complete, cancel, or escalate

### 5. Log Review
```bash
# Check coordination logs for errors
grep -i error ~/Documents/AI/ai_root/ai_comms/claude_cli/logs/*.log
grep -i error ~/Documents/AI/ai_root/ai_general/logs/claude/*.log
```

**Action:** Investigate patterns, update notes_to_myself.md

### 6. Documentation Currency
- Check this playbook still matches reality
- Update inventory if new systems added
- Verify spec versions are current

### 7. Artifact Review
- Check `ai_general/docs/drafts/` for pending artifacts
- Disposition: move to permanent home, update, or delete
- Artifacts from completed tasks shouldn't linger >2 weeks

---

## Troubleshooting

### Pulse Not Waking Claude
1. Check cron: `crontab -l | grep pulse`
2. Check pause file: `ls ~/.../ai_claude/work/.paused`
3. Check lock file: `ls ~/.../ai_claude/CONVERSATION_ACTIVE`
4. Check idle timeout: `ioreg -c IOHIDSystem | awk '/HIDIdleTime/ {print int($NF/1000000000); exit}'`

### Tasks Not Completing
1. Check worker status in `in_progress/`
2. Look for blocking dependencies
3. Check if worker has errors in logs
4. May need to restart worker or reassign task

### Notifications Piling Up
1. Check `pending/` directory size
2. Process old items → `processed/`
3. Consider increasing pulse frequency
4. May indicate Desktop Claude not running pulses

---

## Key Paths Quick Reference

| System | Location |
|--------|----------|
| Task Coordination | `~/.claude/coordination/` → `ai_comms/claude_cli/` |
| Protocol Spec | `ai_general/specs_and_protocols/protocol_taskCoordination_v5.0.yml` |
| Pulse Script | `~/bin/ai/pulse_orchestrator.sh` |
| General Task List | `ai_claude/work/General_Task_List_v02.md` |
| Task Activity Log | `ai_claude/work/task_activity.log` |
| Notes to Self | `ai_claude/notes_to_myself.md` |
| TODO System | `ai_general/todos/` |
| System Logs | `ai_general/logs/` |

---

## Documentation Index

### Architecture Docs
- `ai_comms/README.md` - Communication architecture overview
- `ai_general/docs/ARCHITECTURE_AI_COMMUNICATION_AND_MONITORING.md` - Full blueprint
- `ai_comms/docs/DESKTOP_CLI_CONTROL_ARCHITECTURE.md` - Bidirectional control

### Protocol Specs
- `protocol_taskCoordination_v5.0.yml` - Task coordination protocol
- `schema_taskFile_v1.0.yml` - Task file format
- `spec_ai_message_sender_v1.1.yml` - Message injection spec

### Quick References
- `ai_comms/QUICK_REFERENCE.md` - Communication cheat sheet
- `ai_comms/docs/cli_controls/QUICK_REFERENCE.md` - CLI control commands

### Status/Reports
- `ai_comms/COORDINATION_DASHBOARD.md` - Status dashboard
- This playbook - Maintenance guide

---

## Emergency Procedures

### Reset Stuck System
1. Clear lock file: `rm ai_claude/CONVERSATION_ACTIVE`
2. Clear pause: `rm ai_claude/work/.paused`
3. Check tmux sessions: `tmux ls`
4. Kill zombie workers: `tmux kill-session -t <session>`
5. Restart pulse cron if needed

### Context Emergency
If Desktop Claude context critical (>90%):
1. Document current state to notes_to_myself.md
2. Update task_activity.log with STOP entries
3. Summarize to file before context death
4. Start fresh chat, load context from files

---

**End of Playbook**
