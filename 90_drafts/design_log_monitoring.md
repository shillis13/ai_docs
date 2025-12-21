---
title: Claude Desktop Log Monitoring - Design Discussion
version: draft_0.1
created: 2025-12-15
status: DISCUSSION_NEEDED
---

# Problem Statement

Claude Desktop generates logs that contain useful signals:
- `mcp.log` - Every tool call with full JSON (files read, commands run)
- `main.log` - App events, URL reloads (chat switches)
- Session Storage LevelDB - Chat UUIDs, timestamps

We want to monitor these to detect:
1. New chat started (need to run boot validation)
2. Context-injecting files NOT read (missing boot)
3. Context compaction occurred
4. CLI/Agent interactions started
5. (Future) Footer missing from responses

# Architecture Options

## Option A: Monolithic Specialized Script
Single script that does everything - parsing, pattern matching, actions.
- PRO: Simple to write initially
- CON: Hard to maintain, can't reuse pieces, config is hardcoded
- REJECTED

## Option B: Split Pipeline Architecture

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Parser     │───▶│  Evaluator   │───▶│   Actions    │
│              │    │              │    │              │
│ log → struct │    │ patterns +   │    │ - log        │
│              │    │ config file  │    │ - notify     │
│              │    │              │    │ - send_prompt│
└──────────────┘    └──────────────┘    │ - publish    │
                                        └──────────────┘
```

### Component: Parser
**Input:** Raw log files (mcp.log, main.log, LevelDB strings)
**Output:** Structured events (JSON lines)

Event types:
- `{type: "chat_switch", uuid: "...", timestamp: "..."}`
- `{type: "file_read", path: "...", timestamp: "..."}`  
- `{type: "tool_call", tool: "...", args: {...}, timestamp: "..."}`
- `{type: "app_start", timestamp: "..."}`

### Component: Evaluator
**Input:** Structured events from parser
**Config:** YAML file with patterns and actions

### Component: Actions
Uses EXISTING infrastructure:
- `send_prompt.sh claude-desktop "..." --fb_queue`
- `send_notification.sh user "..."`
- Write to a log/state file
- Publish to announcement dir

# Config File Design

```yaml
# monitor_config.yml
monitors:
  boot_validation:
    description: "Ensure Claude reads context-injecting files on new chat"
    trigger:
      event: chat_switch
      within_seconds: 60  # Grace period
    expect:
      any_of:
        - file_read: "*tools_manifest*"
        - file_read: "*directory_structure_reference*"
    if_missing:
      actions:
        - type: send_prompt
          target: claude-desktop
          message: "Please run boot validation - read manifests"
          options: ["--fb_queue"]
        - type: notify
          target: user
          message: "Claude missing boot validation"
        - type: log
          file: boot_validation.log
          
  context_compaction:
    description: "Detect context compaction"
    trigger:
      event: any
      pattern: "context.*compact|compaction"
    actions:
      - type: notify
        target: user
        message: "Context compaction detected"
      - type: log
        file: context_events.log

  cli_interactions:
    description: "Track CLI/Agent usage"
    trigger:
      event: tool_call
      pattern: "claude_cli|codex_cli|tmux"
    actions:
      - type: log
        file: cli_interactions.log
```

# Questions to Resolve

1. **Parser granularity**: Should parser output raw events or pre-aggregate?
   - Raw: More flexible, evaluator has full control
   - Aggregated: Simpler config, but less flexible

2. **State management**: How to track "within N seconds" windows?
   - In-memory during daemon run
   - Persist to state file for restart recovery
   - Both?

3. **Action execution**: Sync or async?
   - Sync simpler but could block
   - Async more robust but complex

4. **Cron vs Daemon**: How often to run?
   - Daemon with N-second polling
   - Cron every minute
   - fswatch/inotify on log files?

5. **Integration with existing scripts**: 
   - Use send_prompt.sh directly or wrap it?
   - How to handle target selection?

6. **Config location**: Where should monitor_config.yml live?
   - `ai_general/configs/monitor_config.yml`?
   - Part of existing config structure?

# Prior Work Reference

- Directory characterization: `~/Library/Application Support/Claude/DIRECTORY_CHARACTERIZATION.md`
- Chat about log exploration: https://claude.ai/chat/20dc39cf-5e87-417b-916c-c245bf72182b
- Existing send_prompt.sh: `ai_general/scripts/prompting/send_prompt.sh`

# Next Steps

1. [ ] Agree on architecture (Option B?)
2. [ ] Define event schema for parser output
3. [ ] Design config file format
4. [ ] Determine which existing scripts to reuse vs extend
5. [ ] Build parser first (can test independently)
6. [ ] Build evaluator with config
7. [ ] Wire up to existing action scripts
