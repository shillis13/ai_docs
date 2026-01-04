# Local AI Stack Architecture

**Version:** 1.0.0
**Created:** 2026-01-04
**Status:** PLANNING
**Related:** cli_orchestration.latest.md, federated_memory_architecture.latest.md

---

## Overview

Self-hosted AI infrastructure running on MacBook Pro M4 Max (64GB unified memory). Provides local alternatives to cloud AI services with consolidated model storage and unified management.

**Design Principles:**
1. **Consolidated storage** - Single source of truth for all models
2. **No duplication** - Apps point to shared storage, never copy
3. **Format-appropriate** - GGUF for LLMs, Safetensors for diffusion
4. **CLI-native** - Everything scriptable for automation integration

---

## Hardware Constraints

| Resource | Capacity | Implication |
|----------|----------|-------------|
| Unified Memory | 64GB | 70B models quantized, multiple smaller concurrent |
| Storage | ~2TB | Full model library feasible |
| GPU | M4 Max (40-core) | Metal acceleration, all layers on GPU |
| CPU | 14-16 cores | Parallel inference support |

**Memory Budget:**
- 70B Q4_K_M: ~40GB → runs with 24GB headroom
- 32B Q4: ~20GB → can run 2 concurrent
- 7-13B: ~4-12GB → run several simultaneously
- Always reserve ~15GB for system + inference overhead

---

## Directory Structure

```
ai_local/                           # Renamed from ai_models
├── llm/                            # Language models (GGUF format)
│   ├── coding/                     # Code-focused (Qwen-Coder, DeepSeek-Coder)
│   ├── general/                    # General purpose (Llama, Mistral, Qwen)
│   ├── uncensored/                 # Abliterated/unfiltered (Dolphin, DarkIdol)
│   ├── roleplay/                   # Creative/RP tuned
│   └── embedding/                  # Embedding models (or use LLM embed mode)
│
├── diffusion/                      # Image generation (Safetensors)
│   ├── base/                       # SDXL, Pony, Juggernaut, Flux
│   ├── lora/                       # Style/character LoRAs
│   ├── vae/                        # VAE models
│   ├── controlnet/                 # Pose, depth, edge control
│   ├── upscale/                    # ESRGAN, RealESRGAN
│   └── embedding/                  # Textual inversions
│
├── video/                          # Video generation
│   ├── base/                       # WAN 2.x, AnimateDiff
│   ├── lora/                       # Motion LoRAs
│   └── vae/
│
├── audio/                          # Audio models
│   ├── stt/                        # Whisper variants (GGUF)
│   ├── tts/                        # Piper voices, Coqui
│   └── voice/                      # RVC, voice cloning
│
├── workflows/                      # ComfyUI/Fooocus workflows
│
├── content/                        # Non-model assets
│   ├── characters/                 # SillyTavern character cards (PNG)
│   ├── lorebooks/                  # World info JSON files
│   └── presets/                    # Generation settings
│
└── _config/                        # App integration
    ├── comfyui_extra_paths.yaml    # ComfyUI path mapping
    ├── fooocus_paths.yaml          # Fooocus config
    ├── llm_servers/                # llama.cpp launch configs
    │   ├── coding.sh
    │   ├── general.sh
    │   └── uncensored.sh
    └── manifest.yml                # Model inventory with metadata
```

---

## Stack Components

### Layer 1: Model Storage (ai_local/)

Central repository. All tools point here, none copy.

| Format | Used For | Extension |
|--------|----------|-----------|
| GGUF | LLMs (llama.cpp ecosystem) | .gguf |
| Safetensors | Diffusion models, LoRAs | .safetensors |
| CKPT | Legacy diffusion (Draw Things) | .ckpt |
| ONNX | TTS (Piper) | .onnx |

### Layer 2: Inference Engines

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONSOLIDATED MODEL STORAGE                    │
│                       ~/ai_root/ai_local/                        │
└─────────────────────────────────────────────────────────────────┘
         │              │              │              │
         ▼              ▼              ▼              ▼
┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│ llama.cpp   │ │  ComfyUI    │ │ Whisper.cpp │ │   Piper     │
│ server      │ │             │ │             │ │             │
│ (LLM API)   │ │ (Image/Vid) │ │   (STT)     │ │   (TTS)     │
└─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘
      │  │              │              │              │
      │  │              │              └──────┬───────┘
      │  │              │                     │
      ▼  ▼              ▼                     ▼
┌─────────────┐ ┌─────────────┐      ┌───────────────┐
│ LM Studio   │ │  Fooocus    │      │ Voice Pipeline│
│ (Chat GUI)  │ │ (Easy Gen)  │      │               │
└─────────────┘ └─────────────┘      └───────────────┘
```

### Layer 3: User Interfaces

| Tool | Purpose | Connects To |
|------|---------|-------------|
| LM Studio | Interactive chat | Direct GGUF |
| SillyTavern | RP/storytelling | llama.cpp API |
| ComfyUI | Complex workflows | Safetensors |
| Fooocus | Quick image gen | Safetensors |

### Layer 4: Automation Integration

| Tool | Integration Point | Protocol |
|------|-------------------|----------|
| CLI Coordination | llama.cpp server | OpenAI-compat HTTP |
| Desktop Claude | MCP → llama.cpp | MCP tool calls |
| Codex MCP | Direct queries | API calls |


---

## Component Details

### llama.cpp Server (PRIMARY LLM ENGINE)

**Role:** API backend for all LLM tasks
**Why:** Direct GGUF access, no blob storage, OpenAI-compatible API

```bash
# Installation
brew install llama.cpp

# Usage
llama-server \
  --model /path/to/ai_local/llm/coding/qwen-coder-32b-q4.gguf \
  --port 8080 \
  --ctx-size 32768 \
  --n-gpu-layers 99  # All layers on Metal GPU
```

**API Endpoints:**
- `POST /v1/chat/completions` - Chat (OpenAI-compatible)
- `POST /v1/completions` - Completion
- `POST /v1/embeddings` - Embeddings
- `GET /health` - Server status

**Integration with CLI Coordination:**
```bash
# Wrapper: ~/bin/llm-serve
#!/bin/bash
MODEL_DIR="$HOME/Documents/AI/ai_root/ai_local/llm"
case "$1" in
  code)  MODEL="$MODEL_DIR/coding/qwen-coder-32b-q4.gguf"; PORT=8081 ;;
  chat)  MODEL="$MODEL_DIR/general/llama-3.3-70b-q4.gguf"; PORT=8082 ;;
  raw)   MODEL="$MODEL_DIR/uncensored/dolphin-mixtral.gguf"; PORT=8083 ;;
esac
pkill -f "llama-server.*:$PORT" 2>/dev/null
exec llama-server --model "$MODEL" --port $PORT --ctx-size 32768 --n-gpu-layers 99
```

---

### LM Studio (CHAT GUI)

**Role:** Interactive conversations, model experimentation
**Why:** Best model management GUI, uses files in-place

**Configuration:**
1. Settings → My Models → Add directory: `~/Documents/AI/ai_root/ai_local/llm/`
2. LM Studio scans and indexes all GGUFs
3. model.yml files (already present) provide metadata

**Server Mode:** Built-in OpenAI-compatible server on port 1234

---

### ComfyUI (IMAGE/VIDEO POWER)

**Role:** Complex generation workflows, video, precise control
**Why:** Node-based flexibility, full ControlNet/LoRA support

**Storage Integration via extra_model_paths.yaml:**
```yaml
# ~/ComfyUI/extra_model_paths.yaml
ai_local:
    base_path: /Users/shawnhillis/Documents/AI/ai_root/ai_local/
    checkpoints: diffusion/base/
    loras: diffusion/lora/
    vae: diffusion/vae/
    controlnet: diffusion/controlnet/
    embeddings: diffusion/embedding/
    upscale_models: diffusion/upscale/
```

No symlinks needed - ComfyUI native configuration.

---

### Fooocus (IMAGE EASY MODE)

**Role:** Quick generation, inpainting, "just works" defaults
**Why:** Midjourney-like simplicity for rapid iteration

**Storage Integration via symlinks:**
```bash
ln -s ~/Documents/AI/ai_root/ai_local/diffusion/base ~/Fooocus/models/checkpoints
ln -s ~/Documents/AI/ai_root/ai_local/diffusion/lora ~/Fooocus/models/loras
ln -s ~/Documents/AI/ai_root/ai_local/diffusion/vae ~/Fooocus/models/vae
ln -s ~/Documents/AI/ai_root/ai_local/diffusion/controlnet ~/Fooocus/models/controlnet
```

---

### SillyTavern (RP/STORYTELLING)

**Role:** Character-based roleplay, adventure games, narrative
**Why:** Best UI for persistent characters, world info, chat history

**Backend Connection:**
- Points to llama.cpp server (API: `http://localhost:8083`)
- Uses uncensored models for unrestricted creative content
- Character cards stored in `ai_local/content/characters/`
- Lorebooks stored in `ai_local/content/lorebooks/`

---

### Whisper.cpp (SPEECH-TO-TEXT)

**Role:** Voice input transcription
**Why:** Same llama.cpp ecosystem, GGUF format, fast Metal inference

```bash
# Installation
brew install whisper-cpp

# Usage
whisper-cpp \
  --model ~/Documents/AI/ai_root/ai_local/audio/stt/ggml-large-v3.bin \
  --file input.wav
```

---

### Piper (TEXT-TO-SPEECH)

**Role:** Voice output synthesis
**Why:** Fast, quality voices, runs locally

```bash
# Installation
pip install piper-tts

# Usage
echo "Hello world" | piper \
  --model ~/Documents/AI/ai_root/ai_local/audio/tts/en_US-amy-medium.onnx \
  --output_file output.wav
```

---

## App Storage Compatibility Matrix

| App | Format | Uses Central Storage | Method |
|-----|--------|---------------------|--------|
| llama.cpp | GGUF | ✅ Yes | `--model /path/` direct |
| LM Studio | GGUF | ✅ Yes | Settings → model directory |
| ComfyUI | Safetensors | ✅ Yes | `extra_model_paths.yaml` |
| Fooocus | Safetensors | ✅ Yes | Symlinks |
| SillyTavern | N/A | ✅ Yes | API to llama.cpp |
| Whisper.cpp | GGUF | ✅ Yes | `--model /path/` direct |
| Piper | ONNX | ✅ Yes | `--model /path/` direct |
| Draw Things | CKPT | ⚠️ Partial | Symlinks (sandboxed) |
| Diffusion Bee | Internal | ❌ No | App sandbox, can't customize |
| Ollama | Blobs | ❌ No | **Deprecated** - copies to internal storage |

---

## Integration with Existing Infrastructure

### CLI Worker Integration

Local LLMs become another worker type in CLI coordination:

```
ai_comms/
├── claude_cli/          # Cloud Claude workers
├── codex_cli/           # Codex MCP workers
└── local_llm/           # NEW: Local LLM workers
    ├── tasks/
    │   ├── to_execute/
    │   ├── in_progress/
    │   └── completed/
    └── logs/
```

**Wrapper Script:** `~/bin/local-llm-worker`
- Monitors task queue
- Routes to appropriate model (code/chat/raw)
- Returns results via file protocol

### MCP Integration

llama.cpp can be wrapped as MCP server:
- Tool: `local_llm:complete`
- Tool: `local_llm:embed`
- Desktop Claude gains local inference capability

### Use Cases by Model Type

| Task Type | Model | Why |
|-----------|-------|-----|
| Code generation | Qwen-Coder 32B | Best open-source coder |
| Code review | Qwen-Coder 32B | Same, strong analysis |
| Chat history processing | Llama 3.3 70B | Long context, good summarization |
| Semantic search | Embedding mode | Fast vector generation |
| Creative writing | Uncensored models | No guardrails |
| RP/Story continuation | Uncensored + RP-tuned | Character consistency |

---

## Implementation Phases

### Phase 1: Foundation (IMMEDIATE)
- [ ] Rename ai_models → ai_local
- [ ] Install llama.cpp with Metal support
- [ ] Create directory structure
- [ ] Move existing models to correct locations

### Phase 2: Core LLM Stack
- [ ] Configure LM Studio model paths
- [ ] Create llama.cpp server wrapper scripts
- [ ] Test OpenAI-compatible API
- [ ] Create MCP wrapper for local LLM

### Phase 3: Image Stack
- [ ] Configure ComfyUI extra_model_paths.yaml
- [ ] Install/configure Fooocus
- [ ] Set up Fooocus symlinks
- [ ] Verify model sharing works

### Phase 4: Voice Stack
- [ ] Install Whisper.cpp
- [ ] Install Piper
- [ ] Download voice models
- [ ] Create voice pipeline scripts

### Phase 5: Integration
- [ ] SillyTavern backend config
- [ ] CLI worker for local LLM
- [ ] MCP server for Desktop Claude
- [ ] Documentation and testing

---

## Related Documents

- `cli_orchestration.latest.md` - CLI coordination system
- `federated_memory_architecture.latest.md` - Memory system (local LLM for processing)
- `ai_communication_architecture.latest.md` - Cross-AI communication

---

## Revision History

| Date | Version | Changes |
|------|---------|---------|
| 2026-01-04 | 1.0.0 | Initial architecture document |
