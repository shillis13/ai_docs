# CLI Worker Integration with Local LLMs

Guide for CLI workers using local llama.cpp models.

---

## Quick Start

```bash
# Start a model server
llm-serve raw              # Uncensored dolphin on port 8083
llm-serve code             # Coding model on port 8081

# Query it
llm-query --profile raw "explain this code"
llm-query -p code "write a function to parse JSON"
```

---

## Available Profiles

| Profile | Port | Model | Use Case |
|---------|------|-------|----------|
| code | 8081 | Neo-Code 20B | Programming, code review, debugging |
| chat | 8082 | Unsloth F16 | General assistant tasks |
| raw | 8083 | Dolphin 7B | Unrestricted/uncensored queries |
| reason | 8084 | DeepSeek-R1 | Complex reasoning, analysis |

---

## When to Use Local vs Cloud

### Use Local LLMs When:
- **Privacy matters** - Data stays on machine
- **Cost optimization** - No API charges
- **Low latency tasks** - ~90 tokens/sec locally
- **Simple tasks** - Code formatting, basic Q&A, summarization
- **Batch processing** - Many small queries
- **Uncensored content** - Creative writing, roleplay scenarios

### Use Cloud AI (Claude, GPT-4) When:
- **Complex reasoning** - Multi-step logic, nuanced understanding
- **Large context** - 100k+ token conversations
- **Latest knowledge** - Current events, recent APIs
- **Tool use** - Web search, file operations, MCP tools
- **High-stakes decisions** - Where accuracy is critical

---

## Example Use Cases

### Code Review with Local Model
```bash
# Start coding model
llm-serve code &

# Review a file
CODE=$(cat myfile.py)
llm-query -p code "Review this Python code for bugs and improvements:

$CODE"
```

### Quick Summarization
```bash
llm-serve chat &
llm-query -p chat "Summarize in 3 bullets: $(cat long_document.txt)"
```

### Batch Processing
```bash
llm-serve raw &
for file in *.md; do
  echo "=== $file ==="
  llm-query -p raw "Extract key points: $(cat "$file")"
done
```

### Reasoning Check (with DeepSeek-R1)
```bash
llm-serve reason &
llm-query -p reason "Step through this logic and identify any flaws:
1. All cats are mammals
2. Some mammals are pets
3. Therefore all cats are pets"
```

---

## Error Handling

The script handles common errors gracefully:

```bash
# Server not running
$ llm-query -p code "hello"
Error: No server running on port 8081 (profile: code)
Start with: llm-serve code

# Unknown profile
$ llm-query -p unknown "hello"
Error: Unknown profile 'unknown'. Use: code, chat, raw, reason
```

---

## Performance Notes

Tested on Apple M4 Max:

| Model | Tokens/sec | Memory | First Token |
|-------|-----------|--------|-------------|
| Dolphin 7B Q4 | ~91 t/s | 4.6 GB | <100ms |
| Neo-Code 20B Q5 | ~TBD | ~15 GB | TBD |

---

## Integration with CLI Tasks

CLI workers can use local LLMs for subtasks:

```bash
# In a CLI worker script
if llm-query -p code "Is this valid Python: $CODE" | grep -q "valid"; then
  echo "Code validated locally"
else
  # Escalate to cloud for detailed analysis
  echo "Needs cloud review"
fi
```

---

## Files

- `~/bin/llm-serve` - Server launcher with profiles
- `~/bin/llm-query` - Port-based query tool  
- `~/bin/ai/llm-query` - Profile-based query tool (recommended)
