# When to Use the Researcher Agent

## Trigger Patterns

Use `cli-agent:launch_researcher` when the user asks about:
- Past conversations, discussions, or decisions with AI assistants
- How something evolved or changed over time in our work together
- "What did we discuss about X" or "when did we first talk about Y"
- Historical context for current projects or patterns
- Anything requiring search across the conversation archive

## Examples

**Use researcher:**
- "What's the history of the memory system architecture?"
- "When did we first discuss CLI coordination?"
- "How has our approach to context management evolved?"
- "Find conversations about prompt injection"

**Don't use researcher (use knowledge-search MCP instead):**
- Quick lookups in already-indexed/condensed content
- Questions the glossary or topic index can answer directly

## Launch Pattern

```python
cli-agent:launch_researcher(
    prompt="<user's research question>"
)
```

The researcher is a Claude CLI agent that coordinates parallel searches across 18 Gemini shard checkpoints (~52MB corpus, Jan 2025 - Dec 2025). Results are written to `ai_memories/librarian/results/`.

## Response to User

After launching, tell the user:
- Researcher agent launched
- It will search the corpus and write results to the results directory
- This runs async - you can check status with `cli-agent:get_status` or results will appear in `ai_memories/librarian/results/`
