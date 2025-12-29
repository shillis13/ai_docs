# Chat Continuity Recovery System

**Version:** 1.0.0
**Created:** 2025-11-24
**Status:** design
**Maintainer:** PianoMan

## Overview

Automated recovery from broken chats with user control over the process. Detects context exhaustion, provides countdown with controls, then executes multi-step recovery to continue work in fresh chat with preserved context.

## Trigger Detection

**Detection:** Log monitoring detects "prompt is too long" in claude.ai-web.log

**Notification Elements:**
- Title: "🚨 Chat Continuity Recovery"
- Message: "Initiating recovery in XX seconds..."
- Countdown timer
- Buttons: Pause, Cancel, Continue
- Links: HTTPS URL and claude:// URL to broken chat

## Recovery Sequence

### Step 1: Export Chat
Export chat from web UI using ClaudeExporter extension or Puppeteer automation. Output: JSON/MD file in known location.

### Step 2: Generate Condensed History
Upload export to AI (Claude or ChatGPT) with extensive condensing prompt. Output: `condensed_chat_{id}.md`

### Step 3: Generate Faux Context Digest
Same AI session generates context digest from:
- Claude's inner monologue / thinking blocks
- Connecting dots across conversation
- Key decisions and their rationale
- Current task state and next steps

Output: `faux_context_digest_{id}.md`

### Step 4: Download Files
Download generated files to: `~/Documents/AI/ai_root/ai_memories/40_digests/chat_mems/`

### Step 5: Placeholder (Future)
Rewind broken chat to ~70% context, replay prompts 70%→100% in fresh chat as participant, generate true Context Digest at new 85%. Status: not_implemented.

### Step 6-9: New Chat Creation
1. Create new project chat (same project as broken chat)
2. Attach recovery files
3. Write continuation prompt referencing attached files
4. Submit prompt

## Technical Components

### Notification UI Options
- Terminal app: Custom Swift/Electron app with buttons
- Web UI: localhost webpage with controls
- AppleScript dialog: Modal dialog with buttons (blocking)
- Menu bar app: Persistent menu bar with dropdown
- **Recommendation:** AppleScript dialog for MVP, web UI for v2

### Export Automation Options
- ClaudeExporter: Browser extension - may need manual trigger
- Puppeteer: Full automation via CDP
- API: If/when available

### AI Processing Options
- ChatGPT/Claude API: Programmatic but costs money
- Web UI automation: Free tier, Puppeteer-based
- CLI Claude: claude CLI with appropriate prompt

## Dependencies

### Existing
- `chatUrl.py`: Get current chat URL
- `detectBrokenChat.sh`: Monitor for errors
- `brokenChatDaemon.sh`: Polling daemon
- `chat_orchestrator`: Puppeteer-based browser automation

### Needed
- Interactive notification UI with countdown and buttons
- Programmatic chat export trigger
- AI processor for condensed history + digest
- Continuation injector for new chat creation

## Open Questions
- Can ClaudeExporter be triggered programmatically?
- Which AI should process the condensed history? (cost vs reliability)
- How to handle the interactive notification on macOS?
- Desktop app vs web UI for new chat creation?
- How to reliably get chat_id from broken chat?
