# CLI MCP Configuration - Condensed

**Setup script:** `/Users/shawnhillis/bin/setup_cli_mcp_servers.sh`

## Configured Servers

### Codex CLI (`~/.codex/config.toml`)
```toml
[mcp_servers.desktop-commander]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-desktop-commander"]

[mcp_servers.browsermcp]
command = "npx"
args = ["@browsermcp/mcp"]
```

### Claude CLI (managed via CLI)
```bash
claude mcp add desktop-commander --transport stdio --scope user -- npx -y @modelcontextprotocol/server-desktop-commander
claude mcp add browsermcp --transport stdio --scope user -- npx @browsermcp/mcp
```

## Verification
```bash
codex mcp list
claude mcp list
```

## Tool Availability
- `desktop_commander:*` - Filesystem, processes, search
- `browsermcp:*` - Chrome automation

**Desktop Claude additionally has:** PDF tools, Codex MCP

## Troubleshooting
1. Restart CLI instances to load new MCP
2. Check config files (paths above)
3. Run `mcp list` to verify
4. First npx calls download packages (slow initially)

## Re-run Setup
```bash
/Users/shawnhillis/bin/setup_cli_mcp_servers.sh  # Idempotent
```
