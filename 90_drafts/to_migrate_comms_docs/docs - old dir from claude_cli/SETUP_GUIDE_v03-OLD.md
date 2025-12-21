# Setup Guide - Protocol v3.0

# Note from PianoMan 2025-11-19
This doc is deprecated but need to make sure there are more current instructions somewhere

## Quick Start

### Step 1: Initialize CLI Instance

**First time a CLI participates:**
```bash
# Create liveness marker
(OLD) touch ~/.claude/.cli_instance_$$
(NEW) touch ~/.claude/instances/claude_cli.instance.$$

# Remember for session
export CLI_INSTANCE_ID=$$

# Auto-cleanup on exit
(OLD) trap "rm -f ~/.claude/.cli_instance_$$" EXIT
(NEW) trap "rm -f ~/.claude/instances/claude_cli.instance.$$" EXIT
```

**Add to CLI's CLAUDE.md:**
```markdown
## Coordination Shortcuts
- "check inbox" → Read BROADCAST.md + DESKTOP_TO_CLI_$CLI_INSTANCE_ID.md # 2025-11-19 NEEDS UPDATING
- "report done" → Write to CLI_$CLI_INSTANCE_ID_TO_DESKTOP.md # 2025-11-19 NEEDS UPDATING

On startup: `touch ~/.claude/.cli_instance_$$` and `export CLI_INSTANCE_ID=$$` # 2025-11-19 NEEDS UPDATING
```

### Step 2: Test Broadcast Flow

**You to Desktop:**
> "Have any CLI check disk space"

**Desktop writes to BROADCAST.md**

**You to CLI:**
> "check inbox"

**CLI responds:**
> "Found Request #1: Check disk space. Should I handle this?"

**You:** "Yes"

**CLI claims, executes, reports back**

---

## File Structure (OLD and REDUNDANT)

```
~/.claude/
├── .cli_instance_12345          # This CLI is alive
├── .cli_instance_12346          # Another CLI alive
├── coordination/
│   ├── BROADCAST.md             # Desktop → Any CLI
│   ├── DESKTOP_TO_CLI_12345.md # Desktop → Specific CLI
│   ├── CLI_12345_TO_DESKTOP.md # CLI responses
│   └── CLI_12346_TO_DESKTOP.md
```

---

## Usage Patterns

### Broadcast (Default)

**User → Desktop:**
"Have CLI install X"

**Desktop → BROADCAST.md:**
Writes request with unique ID

**User → Any CLI:**
"check inbox"

**CLI → User:**
"Found request. Handle it?" → User approves → CLI executes

### Direct Assignment

**User → Desktop:**
"Have dev-cli (the one that did X) install Y"

**Desktop:**
- Finds CLI by name/description
- Checks if alive                       #  could use -r if not running, would require PID to session ID
- Writes to DESKTOP_TO_CLI_XXXXX.md     # OLD

**User → That specific CLI:**
"check inbox"

**CLI:** Sees personal task, executes

### Naming CLIs (DEPRECATE - NO CURRENT USE)

**User → Desktop:**
"Call this one dev-cli" (while working with a CLI)

**Desktop:** Associates name with current CLI's PID

**Later:**
"Have dev-cli do Z" → Desktop knows which one

---

## CLI Startup Template

Add to each CLI's session initialization:

```bash
# Coordination setup (OLD AND REDUNDANT)
if [ ! -f ~/.claude/.cli_instance_$$ ]; then
    touch ~/.claude/.cli_instance_$$
    trap "rm -f ~/.claude/.cli_instance_$$" EXIT
fi
export CLI_INSTANCE_ID=$$
```

---

## Desktop Capabilities (OLD)

**Track CLIs:**
- Monitor which are alive (PID files)
- Remember names assigned to CLIs
- Recall what each CLI has done

**Route work:**
- Broadcast to any available CLI
- Target specific CLI by name/PID/description
- Check if target CLI still alive before assigning

**Monitor responses:**
- Read all CLI_*_TO_DESKTOP.md files
- Match responses by request ID
- Report back to user

---

## Benefits Over v2.0

**v2.0:** Required claiming sequential numbers, potential race conditions

**v3.0:**
- ✅ PID-based = automatic, unique IDs
- ✅ Broadcast-first = user controls who does what
- ✅ Named references = easier than remembering PIDs
- ✅ Liveness tracking = know which CLIs available
- ✅ Flexible routing = broadcast OR target

---

**Version:** 3.0  
**Date:** 2025-10-25
