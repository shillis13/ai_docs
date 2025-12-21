# Codex Delegation Reminder (All AIs)

**Applies to:** ALL AI instances (Claude Desktop, Claude CLI, ChatGPT, Gemini, Grok)

**Core Principle:** Delegate heavy analysis to Codex—it has unlimited per-task context.

## For Desktop/Web AI Instances

Before doing analysis/review/exploration work yourself:

```
codex("Your task description here")
```

## For CLI Worker Instances

Before claiming tasks requiring **code analysis** or **bulk file ops**:

```bash
# Copy task to Codex queue
cp task.md ~/Documents/AI/ai_root/ai_comms/codex_cli/to_execute/
```

Or use Codex MCP within your execution workflow.

## The Math

- **Your context:** Finite, degrades with heavy analysis work
- **Codex context:** Unlimited per task, always fresh
- **Smart delegation:** Preserves your context for strategic work

---

**Location:** `~/Documents/AI/ai_root/ai_general/`
**Full guidelines:** `ai_comms/claude_cli/broadcasts/CODEX_REMINDER_PROTOCOL.md`
