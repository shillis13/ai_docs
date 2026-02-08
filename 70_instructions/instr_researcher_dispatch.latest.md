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

The researcher creates a query task directory at:
`ai_comms/claude_cli/tasks/in_progress/query{NNNN}_{slug}/`

All shard results and synthesis go to this directory.

## Pipeline Flow

1. **Researcher** dispatches shards, collects evidence, writes synthesis → status file marks `ready_for_validation: true`
2. **Pipeline script** detects completion, launches **validator** (cross-model — must differ from synthesizer) against the same task directory
3. **Validator** checks every claim against shard evidence, writes validation file
4. **Bounce-back** (max 3 rounds): if validator finds issues, synthesis goes back to researcher for revision
5. **Agreement or deadlock**: final output includes both synthesis and validation verdicts
6. **Desktop Claude** reads the validated output from the task directory

Desktop Claude does NOT read the synthesis until validation is complete.
Do NOT read from `ai_memories/librarian/results/` — that path is deprecated for query workflows.

## Response to User

After launching, tell the user:
- Researcher agent launched with query task ID
- Results will go through validation before delivery
- Estimated time: 15-30 minutes depending on scope
- You can check status anytime or wait for completion notification