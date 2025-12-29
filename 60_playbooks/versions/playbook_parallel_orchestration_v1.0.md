# Parallel Worker Orchestration Playbook

**Version:** 1.0.0
**Created:** 2025-12-10
**Maintainer:** PianoMan
**Status:** active
**Category:** playbook

## Summary

How to deploy multiple Claude CLI workers in parallel with AT self-wake monitoring. Desktop Claude orchestrates without doing the work.

## When to Use

- Large task divisible into independent subtasks
- Same operation across multiple directories/files
- Work that benefits from parallelism
- Estimated time >10 minutes for single worker

## Prerequisites

- claude_cli.sh wrapper with -t, -a, -A flags
- send_prompt.sh with --force flag
- AT daemon running (launchctl on macOS)
- Agent profiles in ~/.claude/agents.json

## Steps

### 1. Design Partitioning

Divide work into independent units.

**Example:** 7 directories → 7 workers
**Rule:** Units must not have write conflicts.

### 2. Select Agent

| Agent | Use For |
|-------|---------|
| librarian | Knowledge work, documentation, condensation |
| custodian | File reorganization, structure validation |
| dev-lead | Code review, task coordination |
| ops | Process monitoring, system tasks |

### 3. Launch Workers

```bash
for unit in $UNITS; do
  rm -f /tmp/claude_cli.*.sh  # Prevent temp file collision
  sleep 2  # Stagger launches
  claude_cli.sh -t -a -s worker_$unit -A $AGENT "$PROMPT for $unit"
done
```

**Flags:**
- `-t`: Detached tmux session (survives parent exit)
- `-a`: Auto-approve operations
- `-s NAME`: Named session for monitoring
- `-A AGENT`: Agent profile

### 4. Set Wake Alarm

```bash
cat << 'EOF' | at now + N minutes
send_prompt.sh claude-desktop "AT wake-up: Check workers" --force --submit
EOF
```

**Interval Guidance:**
- Simple tasks: 3-5 minutes
- Complex tasks: 5-10 minutes
- Long running: 10-15 minutes

### 5. On Wake Check

**Commands:**
```bash
# Sessions alive?
tmux list-sessions | grep worker_

# Progress?
ls output_dir/ | wc -l

# Stuck?
tmux capture-pane -t worker_X -p -S -20
```

**Decision:**
- All exited + complete → Verify outputs, wrap up
- All exited + partial → Launch cleanup worker
- Still running → Set another AT, sleep
- Stuck → Investigate, potentially restart

### 6. Cleanup

- Kill any stale tmux sessions
- Move task files to completed/
- Verify all expected outputs exist
- Quality-check sample outputs

## Monitoring Commands

| Action | Command |
|--------|---------|
| List sessions | `tmux list-sessions \| grep $PREFIX` |
| Check output | `tmux capture-pane -t $SESSION -p -S -50` |
| Count progress | `ls $OUTPUT_DIR/*.yml \| wc -l` |
| Kill session | `tmux kill-session -t $SESSION` |

## Common Issues

| Issue | Symptom | Fix |
|-------|---------|-----|
| Temp file collision | mkstemp failed | `rm -f /tmp/claude_cli.*.sh` before each launch |
| Worker exits early | Session gone, work incomplete | Launch targeted cleanup worker |
| Codex validation timeout | Worker stuck | Kill stale val_* sessions |

## Example Deployment

- **Task:** Condense 54 docs across 7 directories
- **Workers:** 7
- **Agent:** librarian
- **Time:** ~25 minutes
- **Result:** 72% reduction, all validated
- **See:** `60_playbooks/parallel_orchestration_case_study.yml`
