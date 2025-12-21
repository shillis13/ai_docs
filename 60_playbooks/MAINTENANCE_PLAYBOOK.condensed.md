# AI Orchestration Maintenance - Condensed

## Daily Checks
```bash
ls ~/Documents/AI/ai_root/ai_comms/claude_cli/notifications/pending/
ls ~/.claude/coordination/tasks/{to_execute,in_progress}/
ls ~/Documents/AI/ai_root/ai_claude/work/.paused
ls ~/Documents/AI/ai_root/ai_claude/CONVERSATION_ACTIVE
```
**WIP Limits:** Desktop 3-5, Orchestrated 10-15, Per-worker 1-2

## Weekly Checks
```bash
find ~/.claude/coordination/tasks/in_progress -type f -mtime +7
grep -i error ~/Documents/AI/ai_root/ai_comms/claude_cli/logs/*.log
grep -i error ~/Documents/AI/ai_root/ai_general/logs/claude/*.log
```
Check: docs currency, ai_general/docs/drafts/ artifacts (>2 weeks stale)

## Key Paths
| System | Location |
|--------|----------|
| Task Coordination | `~/.claude/coordination/` → `ai_comms/claude_cli/` |
| Protocol Spec | `ai_general/specs_and_protocols/protocol_taskCoordination_v5.0.yml` |
| Pulse Script | `~/bin/ai/pulse_orchestrator.sh` |
| General Task List | `ai_claude/work/General_Task_List_v02.md` |
| Notes to Self | `ai_claude/notes_to_myself.md` |
| Task Activity Log | `ai_claude/work/task_activity.log` |
| System Logs | `ai_general/logs/` |
| TODO System | `ai_general/todos/` |

## Troubleshooting

**Pulse Not Waking:**
1. `crontab -l | grep pulse`
2. Check pause/lock files
3. `ioreg -c IOHIDSystem | awk '/HIDIdleTime/ {print int($NF/1000000000); exit}'`

**Tasks Stuck:** Check in_progress/, logs, blocking deps

**Notifications Piling:** Process old→processed/, increase pulse frequency

## Emergency Reset
```bash
rm ai_claude/CONVERSATION_ACTIVE
rm ai_claude/work/.paused
tmux ls && tmux kill-session -t <session>
# Restart pulse cron if needed: crontab -e
```

**Context Emergency (>90%):**
1. Document state → notes_to_myself.md
2. Update task_activity.log with STOP
3. Summarize before context death
4. Fresh chat, load from files
