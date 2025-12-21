# CLI & Codex Coordination Install - Condensed

## Prerequisites
- Codex CLI: `codex --version`
- Claude CLI: `claude --version` (expects `/opt/homebrew/bin/claude`)
- Desktop config: `~/Library/Application Support/Claude/claude_desktop_config.json`

## Directory Setup
```bash
ln -s /Users/shawnhillis/Documents/AI/ai_root/ai_comms/claude_cli ~/.claude/coordination
```

## MCP Configuration
Add to `claude_desktop_config.json`:
```json
{
  "mcpServers": {
    "codex": {
      "command": "/usr/local/bin/codex",
      "args": ["--stdio"],
      "autoLaunch": true,
      "env": {"CLAUDE_COORDINATION_ROOT": "~/.claude/coordination"}
    }
  }
}
```
Restart Claude Desktop after changes.

## Wrapper Script (`~/bin/bash/claude.sh`)
```bash
#!/bin/bash
export PATH="/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin"
export DISABLE_PROMPT_CMD=1
export CLAUDECODE=1
exec "/opt/homebrew/bin/claude" "${@:-Initialize coordination: read ~/.claude/CLAUDE.md coordination section...}"
```

## Test Procedure
```bash
codex --version
~/bin/bash/claude.sh --version
~/bin/bash/claude.sh  # Launch coordination session
ls ~/.claude/coordination/  # Verify: broadcasts/, direct/, responses/
```

## Troubleshooting
- **codex not found:** `brew install codex-cli` or `npm install -g @anthropic-ai/codex-cli`
- **MCP fails:** Check JSON syntax in config
- **File ops fail:** Read before modify (see CLAUDE.md), use shell utilities
- **PID literal `$$`:** Export `CLI_INSTANCE_ID=$$` first
