# Architecture Documentation

## Current Version

**[architecture_v1.2.yml](architecture_v1.2.yml)** - Current system design (2025-11-20)

Comprehensive architecture for AI scripts and orchestration system, including:
- Core entry points (ai_isBusy, send_prompt, send_notification, monitor)
- Scheduling system (time-based prompt delivery)
- Coordination system (task assignment and execution)
- Terminal session tracking
- Notification pipeline
- Migration strategy from pulse/ to new systems

## Archive

**[archive/ARCHITECTURE_v1.1.md](archive/ARCHITECTURE_v1.1.md)** - Previous version (superseded 2025-11-20)

Initial unified architecture with three-layer messaging (sync, polling, async) and coordination system foundation.

## Version History

### v1.2.0 (2025-11-20) - Current
- **Status:** Active
- **Changes:**
  - Added send_notification.sh system (req_2123) with 4 channels: desktop, webui, cli, user
  - Added scheduling triple: set_scheduled_prompt.sh, send_scheduled_prompt.sh, scheduled_prompts_daemon.sh (req_2124)
  - Unified monitor.sh with pattern matching, output modes, and line limits (req_1005)
  - Standardized _internal/ structure across all entry points
  - Defined migration strategy from pulse/ to new monitor+notification systems
  - Enhanced coordination system with scheduling/ and notifications/ directories
  - Created comprehensive YAML-based architecture documentation

### v1.1.0 (2025-11-10) - Superseded
- **Status:** Archived
- **Changes:**
  - Initial unified architecture implementation
  - Established flat-plus-internal layout for scripts/
  - Created coordination system (tasks, broadcasts, logs)
  - Implemented ai_isBusy.sh and send_prompt.sh entry points
  - Terminal session tracking via ITERM_SESSION_ID
  - Three-layer communication architecture (synchronous, polling, async)

## Quick Reference

### Entry Points

| Script | Purpose | Status |
|--------|---------|--------|
| `ai_isBusy.sh` | Check if AI can accept prompts | ✅ Implemented |
| `send_prompt.sh` | Send prompts to AI instances | ✅ Implemented |
| `send_notification.sh` | Non-blocking notifications | ✅ Implemented |
| `monitor.sh` | Read content from AI interfaces | 🔄 In Progress |
| `set_scheduled_prompt.sh` | Create/manage schedules | ✅ Implemented |
| `send_scheduled_prompt.sh` | Execute scheduled prompts | ✅ Implemented |
| `scheduled_prompts_daemon.sh` | Background scheduler | ✅ Implemented |

### Key Locations

- **Scripts:** `~/Documents/AI/ai_root/ai_general/scripts/`
- **Coordination:** `~/.claude/coordination/`
- **Logs:** `~/Documents/AI/ai_root/ai_general/scripts/logs/`
- **Scheduling:** `~/.claude/coordination/scheduling/`

## Usage Examples

```bash
# Check if Claude Desktop is ready
ai_isBusy.sh desktop 48B10940

# Send a prompt to Claude CLI
send_prompt.sh claude-cli "Check coordination inbox"

# Send a high-priority notification
send_notification.sh desktop 48B10940 "Task complete" --priority high --sound

# Monitor CLI session for errors
monitor.sh cli $ITERM_SESSION_ID --pattern "ERROR:" --output cleaned

# Schedule a periodic check
set_scheduled_prompt.sh create check_inbox desktop 48B10940 "check inbox" "@hourly" normal

# Start the scheduling daemon
scheduled_prompts_daemon.sh start
```

## Documentation Standards

- **Format:** YAML for structured architecture docs, Markdown for guides
- **Versioning:** Increment minor version for additions, major version for breaking changes
- **Maintenance:** Update architecture doc when adding new components or changing interfaces
- **Archive:** Move superseded versions to archive/ directory

## Related Documentation

- **[TODO_MANAGER_STATUS_2025-11-20.md](../TODO_MANAGER_STATUS_2025-11-20.md)** - Implementation status
- **[coordination_system_v4_digest.md](../coordination_system_v4_digest.md)** - CLI coordination protocol
- **[ai_communication_architecture_v1.md](../ai_communication_architecture_v1.md)** - Three-layer messaging

## For New AI Instances

1. Read [architecture_v1.2.yml](architecture_v1.2.yml) to understand system design
2. Review entry point signatures and examples
3. Understand coordination workflow: `incoming/ → in_progress/ → completed/`
4. Learn essential commands (ai_isBusy, send_prompt, send_notification, monitor)
5. Practice with test commands before automation

## For Developers

### Adding a New Entry Point
1. Create `scripts/{name}.sh` with router pattern
2. Implement `_internal/_{name}_{channel}.sh` for each channel
3. Write integration tests in `scripts/tests/`
4. Update `architecture_v1.2.yml`
5. Add examples to this README

### Adding a New Channel
1. Create `_internal/_ai_isBusy_{channel}.sh`
2. Create `_internal/_monitor_{channel}.sh`
3. Create `_internal/_send_notification_{channel}.sh`
4. Update entry point case statements
5. Write channel-specific tests
6. Update documentation

## Future Roadmap

See the "Future Enhancements" section in architecture_v1.2.yml for planned features:
- Multi-AI coordination (Gemini, Grok integration)
- Web UI for monitoring and control
- Advanced scheduling (dependencies, conditions, retries)
- Comprehensive logging and observability
- Cross-platform support (Linux, Windows)

---

**Last Updated:** 2025-11-20  
**Maintainers:** Claude CLI instances, Codex CLI, Desktop Claude  
**Questions?** Post to `~/.claude/coordination/note_passing/incoming/`
