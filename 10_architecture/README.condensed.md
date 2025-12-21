# Architecture Documentation (Condensed)

**Source:** `../README.md`

## Index

| Document | Purpose |
|----------|---------|
| `architecture_overview.*` | Full system architecture - layers, components, design |
| `architectural_layer_model.*` | Detailed layer definitions |
| `versions/` | Version history of architecture docs |
| `systems/` | Subsystem-specific architecture (chat pipeline) |

## Quick Reference

### Key Paths
```
~/.claude/coordination/          # Task coordination
~/Documents/AI/ai_root/          # AI ecosystem root
~/Documents/AI/ai_root/ai_memories/  # Memory/history storage
~/Documents/AI/ai_root/ai_general/scripts/  # Core scripts
```

### Core Scripts
```bash
ai_isBusy.sh {target} {id}              # Check AI readiness
send_prompt.sh {target} "{message}"     # Queue prompt
send_notification.sh {target} {id} "{msg}"  # Send notification
monitor.sh {type} {id} [--pattern]      # Read AI output
```

### Task Workflow
```
1. Check inbox: ls ~/.claude/coordination/tasks/to_execute/
2. Claim: mv task.md in_progress/claimed_{timestamp}_{pid}_task/
3. Execute task
4. Write response: response_001_cli_{pid}.md
5. Complete: mv in_progress/... completed/
6. Notify: send_notification.sh desktop {id} "Task complete"
```

## Related Documentation

- `../20_registries/` - Inventories and catalogs
- `../30_protocols/` - Process flows (task coordination)
- `../40_specs/` - Interface contracts
- `../50_schemas/` - Data structures
