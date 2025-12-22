# AI-to-AI Chat Orchestration Protocol

**Version:** 1.0.0
**Created:** 2025-12-09
**Status:** stub

## Purpose

Define how AI agents conduct conversations with each other, including direct driving of web UIs and orchestrated multi-party chats.

## Overview

This protocol covers two modes of AI-to-AI conversation:
1. **Direct driving** - One AI (typically Desktop Claude) controls another AI's web UI
2. **Orchestrated chat** - A coordination layer manages message flow between participants

### Goals
- Enable collaboration between different AI platforms (Claude, ChatGPT, Gemini, Grok)
- Leverage each model's unique strengths
- Provide optional human oversight without blocking autonomous operation
- Capture conversation artifacts for memory system integration

## Direct Driving Mode

Desktop Claude uses automation tools to interact with web-based AI interfaces.

### Mechanisms

| Tool | Use Case | Capabilities |
|------|----------|--------------|
| AppleScript | macOS native apps, basic browser automation | click, type, read_text, window_management |
| Puppeteer | Complex web UI interaction, form filling, response extraction | full_dom_access, screenshot, wait_for_element, evaluate_js |
| Browser MCP | When MCP-based browser control is available | Watch for context cost of snapshots |

### Workflow
1. Navigate to target AI's web interface
2. Inject prompt into input field
3. Submit and wait for response generation
4. Extract response text
5. Process response, decide on continuation

### Challenges
- Response detection timing (knowing when AI has finished)
- Rate limiting and session management
- UI changes breaking automation
- Context window management (both AIs accumulate context)

## Orchestrated Chat Mode

A coordination layer (script, human, or AI) manages message flow between two or more AI participants.

### Participants

| Role | Description | Examples |
|------|-------------|----------|
| Initiator | The AI or human that starts the conversation | Desktop Claude, Human, Scheduled task |
| Respondents | AIs that receive and respond to messages | ChatGPT, Gemini, Grok, Claude Web |
| Coordinator | Entity managing message flow | - |

### Coordinator Modes
- **Autonomous:** No human approval, messages flow freely
- **Supervised:** Human reviews each message before forwarding
- **Hybrid:** Human approves first N exchanges, then autonomous

### Message Flow
1. Initiator generates message → `message_queue/outbound/`
2. Coordinator reviews (if supervised mode): approve/edit/redirect/block/inject
3. Deliver to respondent via automation
4. Extract response → `message_queue/inbound/`
5. Coordinator processes response: forward/summarize/branch/terminate
6. Loop until conversation complete

### Coordination Directory
```
ai_comms/chat_orchestration/
├── sessions/        # Active conversation sessions
├── message_queue/   # Pending messages (inbound/outbound)
├── transcripts/     # Completed conversation logs
└── templates/       # Prompt templates for different conversation types
```

## Conversation Types

| Type | Description | Pattern | Use Case |
|------|-------------|---------|----------|
| Peer Review | One AI reviews another's work | Claude generates → ChatGPT critiques → Claude revises | Code review, document editing, architectural validation |
| Research Synthesis | Multiple AIs contribute knowledge | Round-robin or directed questioning | Gathering diverse perspectives, fact-checking |
| Debate | AIs argue different positions | Thesis → Antithesis → Synthesis | Exploring trade-offs, stress-testing decisions |
| Interview | One AI systematically queries another | Structured questions, follow-ups | Knowledge extraction, capability probing |
| Collaborative Creation | AIs build something iteratively | Draft → Feedback → Revision cycles | Document creation, code development, design work |

## Integration

### Memory System
- Export transcript to `10_exported/`
- Process through standard pipeline
- Tag with participants and conversation_type

### Task System
- Trigger: When conversation identifies actionable work
- Output: Task file in coordination system

### TODO System
- Trigger: When conversation surfaces future work ideas
- Output: TODO in `ai_general/todos/pending/`

## Implementation Status

### Direct Driving
| Component | Status | Notes |
|-----------|--------|-------|
| AppleScript ChatGPT | working | Successfully tested Claude → ChatGPT conversations |
| AppleScript Gemini | working | Successfully tested Claude → Gemini conversations |
| Puppeteer | partial | Framework exists, needs refinement for response detection |
| Browser MCP | experimental | Context cost concerns, prefer file-based snapshot analysis |

### Orchestrated Chat
| Component | Status | Notes |
|-----------|--------|-------|
| Basic Framework | conceptual | Directory structure and message flow defined |
| Coordinator Script | not_started | Need to implement message queue processing |
| Human Approval UI | not_started | Could be CLI-based or simple web UI |

## TODOs
- Implement chat_orchestrator.py coordinator script
- Define session file schema
- Build response detection heuristics
- Create transcript export format
- Test multi-party conversations (3+ AIs)

## Related Documents
- `ai_general/docs/10_architecture/ai_communication_architecture_latest.md`
- `ai_general/docs/10_architecture/architecture_overview.md`
- `ai_general/docs/30_protocols/protocol_taskCoordination_latest.yml`
- `ai_general/docs/60_playbooks/cli_agent_operations.yml`
