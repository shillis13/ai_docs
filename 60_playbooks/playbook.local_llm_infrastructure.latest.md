# Local LLM Infrastructure Playbook

**Version:** 1.0.0  
**Last Updated:** 2026-01-06  
**Status:** Active

## Overview

Unified local LLM infrastructure using MLX (Apple Silicon optimized) as primary backend, with llama.cpp fallback for GGUF-only models. All models expose OpenAI-compatible APIs for seamless integration with Cline, MCP servers, and CLI tools.

## Architecture

```
┌─────────────────────────────────────────────────────┐
│     Consumers (Cline / MCP / Scripts / CLI)         │
└──────────────────────┬──────────────────────────────┘
                       │ OpenAI-compatible API
                       ▼
┌─────────────────────────────────────────────────────┐
│              llm_serve.sh v2                        │
│  ┌──────────────────┐    ┌────────────────────────┐ │
│  │  mlx_lm.server   │    │    llama-server        │ │
│  │  (MLX models)    │    │    (GGUF models)       │ │
│  │  5-8x faster     │    │    fallback only       │ │
│  └──────────────────┘    └────────────────────────┘ │
└─────────────────────────────────────────────────────┘
                       │
      ┌────────────────┼────────────────┐
      ▼                ▼                ▼
┌───────────┐   ┌───────────┐   ┌──────────────┐
│ Qwen3-    │   │ Qwen3-    │   │ DeepSeek-R1  │
│ Coder-30B │   │ Coder-30B │   │ 32B (GGUF)   │
│ MLX-4bit  │   │ MLX-6bit  │   │              │
└───────────┘   └───────────┘   └──────────────┘
      ↑              ↑
      └──────────────┴── Shared with LM Studio
```

## Quick Start

### Start a Server
```bash
# Coding model (MLX, ~12GB VRAM)
ai_general/scripts/local_llm/llm_serve.sh code

# Higher quality 6-bit variant
ai_general/scripts/local_llm/llm_serve.sh code6

# Reasoning model (llama.cpp, GGUF)
ai_general/scripts/local_llm/llm_serve.sh reason --np 4  # 4 parallel slots
```

### Query from CLI
```bash
ai_general/scripts/local_llm/llm_query.sh -p code "Write a Python function to parse JSON"
ai_general/scripts/local_llm/llm_query.sh -p reason "Explain the trade-offs of microservices"
```

### Check Status
```bash
curl http://localhost:8081/health
curl http://localhost:8081/v1/models
```

## Model Profiles

| Profile | Model | Backend | Port | Memory | Speed |
|---------|-------|---------|------|--------|-------|
| `code` | Qwen3-Coder-30B-4bit | MLX | 8081 | ~12GB | ~50 tok/s |
| `code6` | Qwen3-Coder-30B-6bit | MLX | 8081 | ~16GB | ~40 tok/s |
| `chat` | Qwen3-32B | MLX | 8082 | TBD | TBD |
| `reason` | DeepSeek-R1-32B | llama.cpp | 8084 | ~18GB | ~10 tok/s |
| `raw` | Dolphin-7B | llama.cpp | 8083 | ~4GB | ~30 tok/s |

## Performance Benchmarks

Tested on M4 Max 64GB:

| Test | llama.cpp (GGUF) | MLX | Speedup |
|------|------------------|-----|---------|
| Short completion (~200 tok) | 29s | 6s | **4.8x** |
| Long completion (~500 tok) | 51s | 37s | **1.4x** |
| Throughput | ~10 tok/s | ~50 tok/s | **5x** |
| Memory (30B 4-bit) | ~18GB | ~12GB | **33% less** |

## File Locations

### Scripts
```
ai_general/scripts/local_llm/
├── llm_serve.sh        # Unified server launcher (MLX + llama.cpp)
└── llm_query.sh        # CLI query tool
```

### Archived (experimental MCP workarounds)
```
ai_general/scripts/_archive/2026-01-06/
├── dc_proxy_mcp.js       # Filtered broken DC schemas for llama.cpp
├── mcp_truncated.py      # Stripped tool descriptions for Qwen3
├── download_llm_models.sh # Old GGUF download script
└── llm_serve_v1_backup.sh # Original llama.cpp-only launcher
```

### Models
```
ai_models/llm/
├── lmstudio-community/
│   ├── Qwen3-Coder-30B-A3B-Instruct-MLX-4bit/  # Primary coding
│   └── Qwen3-Coder-30B-A3B-Instruct-MLX-6bit/  # Higher quality
└── deepseek-ai/
    └── DeepSeek-R1-Distill-Qwen-32B-Q4_K_M/    # Reasoning (GGUF)
```

### Prompts
```
ai_general/prompts/
└── condensation_prompt_v1.yml  # Chat condensation template
```

## Integration Points

### MCP (Local LLM Server)
The "Local LLM (llama.cpp)" MCP works with both backends:
```javascript
// Same API regardless of backend
await llama_cpp_chat({
  profile: "code",
  messages: [{ role: "user", content: "Hello" }],
  max_tokens: 1024
});
```

### Cline (Agentic Coding)
Configure Cline to use local endpoint:
- URL: `http://localhost:8081/v1`
- Model: (any, ignored)
- No API key needed

### Python Scripts
```python
import requests

response = requests.post(
    "http://localhost:8081/v1/chat/completions",
    json={
        "messages": [{"role": "user", "content": "Hello"}],
        "max_tokens": 1024
    }
)
print(response.json()["choices"][0]["message"]["content"])
```

## Troubleshooting

### Server won't start
```bash
# Check if port in use
lsof -i :8081

# Kill existing server
pkill -f "mlx_lm.server.*8081"
pkill -f "llama-server.*8081"
```

### Model not found
```bash
# Verify model path
ls -la ~/Documents/AI/ai_root/ai_models/llm/lmstudio-community/
```

### Slow responses
- MLX should be 5x faster than llama.cpp
- If using llama.cpp, add `--np 4` for parallel slots
- Check Activity Monitor for memory pressure

## Migration Notes

### From llama.cpp to MLX
1. MLX models are directories, not single .gguf files
2. Same models work in LM Studio and mlx_lm.server
3. No duplicate storage needed
4. API is identical (OpenAI-compatible)

## Related Documentation
- REF:ai_general/prompts/condensation_prompt_v1.yml
- REF:ai_general/docs/70_instructions/claude/know_codex_mcp.latest.condensed.yml
