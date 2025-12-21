# CLI Browser Automation - Condensed

**Key insight:** CLI needs explicit workflow instructions but handles execution autonomously.

## Instruction Levels

**Too Vague (fails):** "Download the chat history"

**Goal-Oriented (might fail):** Names extension, gives URL, but no selectors/timing

**Detailed (recommended):**
```markdown
## Task: Export Chat
Extension: "Claude Chat Exporter" (Chrome toolbar)
Chat: https://claude.ai/chat/abc123
Steps:
1. Navigate to chat URL
2. Click extension icon (top-right)
3. Click "Export JSON", wait 3s
4. Click "Export Markdown", wait 3s
Return: {"json_file": "...", "md_file": "...", "new_chat_url": "..."}
```

## CLI Capabilities
**CAN:** Navigate, click, retry, wait, capture URLs, write structured output
**CANNOT:** Know your extensions/workflows/project structure without instruction

## Output Methods

**Response File (Coordination):**
- Desktop posts → tasks/incoming/
- CLI writes → tasks/responses/
- Desktop reads response

**CLI --print Mode:**
```bash
NEW_URL=$(claude --print "Export chat, return URL")
claude --print "Return JSON: {url, file}" | jq .
```

## Best Practices
- Provide specific URLs and extension names
- Name buttons explicitly
- Give timing hints (wait Xs)
- Specify return format
- Include fallbacks

## Template Pattern
1. Detailed first run
2. Refine based on errors
3. Save as reusable template
