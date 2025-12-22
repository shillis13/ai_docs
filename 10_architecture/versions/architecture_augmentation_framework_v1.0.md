# AI Augmentation Architecture
## A Framework for Multi-AI Orchestration with Persistent Memory

**Version:** 1.0.0-draft  
**Created:** 2025-12-14  
**Status:** Draft  
**Maintainer:** PianoMan + Claude Desktop

---

## Executive Summary

This architecture extends all five components of standard LLM agent design—Perception, Reasoning/Planning, Memory, Tools, and Action—across multiple AI platforms with shared memory and coordinated execution.

**One-sentence pitch:** *Prosthetics and exoskeletons attached to every limb of an LLM agent, enabling capabilities no single AI instance can achieve alone.*

---

## Part 1: The Baseline LLM Agent Architecture

The following represents the consensus architecture for LLM-based agents as documented across academic literature and industry frameworks (LangChain, AutoGen, CrewAI, etc.).

### 1.1 Canonical Components

```
                    ┌─────────────────────────────────────┐
                    │           PERCEPTION                │
                    │  • User input (text, images, files) │
                    │  • System prompt / instructions     │
                    │  • Tool results fed back            │
                    │  • Environment state                │
                    └──────────────┬──────────────────────┘
                                   │
                                   ▼
                    ┌─────────────────────────────────────┐
                    │       REASONING / PLANNING          │
                    │  • LLM "brain" processes input      │
                    │  • Goal decomposition               │
                    │  • Strategy selection               │
                    │  • Chain-of-thought / reflection    │
                    └──────────────┬──────────────────────┘
                                   │
                    ┌──────────────┴──────────────┐
                    │                             │
                    ▼                             ▼
     ┌─────────────────────────┐   ┌─────────────────────────┐
     │         MEMORY          │   │          TOOLS          │
     │  ┌───────────────────┐  │   │  • API calls            │
     │  │ Short-term        │  │   │  • Code execution       │
     │  │ (context window)  │  │   │  • File operations      │
     │  └───────────────────┘  │   │  • Search / retrieval   │
     │  ┌───────────────────┐  │   │  • External services    │
     │  │ Long-term         │  │   │                         │
     │  │ (vector DB, etc.) │  │   │                         │
     │  └───────────────────┘  │   │                         │
     └─────────────────────────┘   └─────────────────────────┘
                    │                             │
                    └──────────────┬──────────────┘
                                   │
                                   ▼
                    ┌─────────────────────────────────────┐
                    │            ACTION                   │
                    │  • Generate responses               │
                    │  • Execute tool calls               │
                    │  • Update state                     │
                    │  • Return to Perception (loop)      │
                    └─────────────────────────────────────┘
```

### 1.2 The Agent Loop

Standard agent operation follows: **Perception → Reasoning → Action → (Memory update) → Perception...**

This loop assumes:
- Single AI instance
- Session-bounded context
- Human-initiated interaction
- Tools as passive utilities

**Our architecture challenges all four assumptions.**

---

## Part 2: Architecture Extensions Overview

We augment every component of the baseline architecture. The following diagram shows external (baseline) vs. internal (our additions) elements.


```
┌─══════════════════════════════════════════════════════════════════════════════┐
║                         AUGMENTED LLM AGENT ARCHITECTURE                       ║
║                                                                                ║
║    ▓▓▓ = External (baseline LLM capability)                                   ║
║    ░░░ = Internal (our extensions)                                            ║
└─══════════════════════════════════════════════════════════════════════════════┘

                              PERCEPTION
    ┌─────────────────────────────────────────────────────────────────┐
    │ ▓▓▓ User input        │ ░░░ Auto-loaded knowledge files        │
    │ ▓▓▓ System prompt     │ ░░░ Glossary term recognition          │
    │ ▓▓▓ Tool results      │ ░░░ Memory slot injection              │
    │ ▓▓▓ File attachments  │ ░░░ Reference pointer resolution       │
    │                       │ ░░░ CLI task reports / Codex outputs   │
    │                       │ ░░░ Cross-AI message receipt           │
    └─────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
                           REASONING / PLANNING
    ┌─────────────────────────────────────────────────────────────────┐
    │ ▓▓▓ LLM reasoning     │ ░░░ Multi-AI distribution              │
    │ ▓▓▓ Goal decomposition│ ░░░ Specialized agent roles            │
    │ ▓▓▓ Chain-of-thought  │ ░░░ Task coordination protocol         │
    │                       │ ░░░ Codex validation loops             │
    │                       │ ░░░ Peer review patterns               │
    │                       │ ░░░ Orchestrator/worker model          │
    └─────────────────────────────────────────────────────────────────┘
                                     │
                    ┌────────────────┴────────────────┐
                    ▼                                 ▼
              MEMORY                                TOOLS
    ┌───────────────────────────┐    ┌───────────────────────────────┐
    │ ▓▓▓ Context window        │    │ ▓▓▓ API calls                 │
    │ ▓▓▓ Basic RAG             │    │ ▓▓▓ Code execution            │
    │                           │    │ ▓▓▓ File read/write           │
    │ ░░░ Federated mem slots   │    │ ▓▓▓ Web search                │
    │ ░░░ Chat history pipeline │    │                               │
    │ ░░░ Layered summaries     │    │ ░░░ Desktop Commander MCP     │
    │ ░░░ Knowledge digests     │    │ ░░░ Codex MCP (sync exec)     │
    │ ░░░ Cross-AI memory       │    │ ░░░ CLI coordination (async)  │
    │ ░░░ Condensed files       │    │ ░░░ send_prompt.sh            │
    │ ░░░ Glossary/index        │    │ ░░░ AT job scheduling         │
    │ ░░░ Drain-and-fill        │    │ ░░░ Browser automation        │
    │                           │    │ ░░░ Notification system       │
    └───────────────────────────┘    └───────────────────────────────┘
                    │                                 │
                    └────────────────┬────────────────┘
                                     ▼
                                ACTION
    ┌─────────────────────────────────────────────────────────────────┐
    │ ▓▓▓ Generate response │ ░░░ Delegate to other AI instances      │
    │ ▓▓▓ Execute tool call │ ░░░ Autonomous operation (overnight)    │
    │ ▓▓▓ Return to user    │ ░░░ Self-scheduling (AT wake)           │
    │                       │ ░░░ Cross-platform coordination         │
    │                       │ ░░░ Response footers (state tracking)   │
    │                       │ ░░░ Task lifecycle management           │
    └─────────────────────────────────────────────────────────────────┘
```

---

## Part 3: Component Deep-Dives

### 3.1 PERCEPTION — How the AI Receives Input

#### External (Baseline)
| Element        | Description                                    |
|----------------|------------------------------------------------|
| User input     | Text, images, files sent by human              |
| System prompt  | Instructions loaded at conversation start      |
| Tool results   | Output from API calls, searches, file reads    |
| Attachments    | Documents, images uploaded to conversation     |

#### Internal (Our Extensions)
| Element                    | Description                                 | Implementation                             |
|----------------------------|---------------------------------------------|--------------------------------------------|
| **Auto-loaded knowledge**  | Core docs loaded without explicit request   | `_knowledge_manifest.yml` AUTO sequence    |
| **Glossary recognition**   | Term→file mapping enables targeted loading  | `glossary_knowledge_index.condensed.yml`   |
| **Memory slot injection**  | Per-AI persistent state in system prompt    | `ai_claude/memories/mem_slots/*.yml`       |
| **Reference pointers**     | `REF:` syntax for on-demand file loading    | `protocol_reference_pointers.yml`          |
| **CLI task reports**       | Async worker results fed back to coordinator| Task `responses/` directory                |
| **Cross-AI messages**      | Other AI platforms can send to Desktop Claude| `send_prompt.sh`, messaging directories   |

**Key insight:** We've expanded what "input" means beyond user-initiated content to include structured knowledge injection and inter-AI communication.


---

### 3.2 REASONING / PLANNING — How the AI Thinks and Decides

#### External (Baseline)
| Element              | Description                                      |
|----------------------|--------------------------------------------------|
| LLM reasoning        | Model's trained ability to process and respond   |
| Goal decomposition   | Breaking complex requests into steps             |
| Chain-of-thought     | Step-by-step reasoning for complex problems      |
| Strategy selection   | Choosing approaches based on context             |

#### Internal (Our Extensions)
| Element                    | Description                                  | Implementation                              |
|----------------------------|----------------------------------------------|---------------------------------------------|
| **Multi-AI distribution**  | Multiple AI platforms participate in reasoning| Desktop Claude + CLI + Codex + ChatGPT     |
| **Specialized agents**     | Named roles with specific competencies       | Librarian, Custodian, Dev-Lead, Ops         |
| **Orchestrator model**     | Desktop Claude orchestrates, workers execute | Context preservation pattern                |
| **Task coordination**      | Structured protocol for work breakdown       | `protocol_taskCoordination.yml`             |
| **Validation loops**       | Codex reviews work for quality/gaps          | Bidirectional: data loss + reduction        |
| **Peer review**            | AI instances review each other's output      | CLI reviewing Codex, vice versa             |

**Key insight:** Reasoning is no longer confined to a single context window. Complex problems can be distributed across multiple AI instances with different roles and capabilities.

---

### 3.3 MEMORY — How the AI Remembers

#### External (Baseline)
| Element           | Description                                    |
|-------------------|------------------------------------------------|
| Context window    | ~200K tokens of immediate working memory       |
| Basic RAG         | Retrieve documents to augment context          |
| Session history   | Messages within current conversation           |

#### Internal (Our Extensions)
| Element                      | Description                                 | Implementation                        |
|------------------------------|---------------------------------------------|---------------------------------------|
| **Federated memory slots**   | Per-AI owned memory with cross-reference    | `ai_claude/memories/mem_slots/`           |
| **Chat history pipeline**    | Export → YAML → Index → Digest              | `ai_memories/chats/` pipeline         |
| **Layered summaries**        | L0 (raw) → L1 (summary) → L2 (meta)         | Progressive abstraction               |
| **Knowledge digests**        | Processed insights from conversations       | `ai_memories/knowledge/`              |
| **Cross-AI memory**          | CLI agents can read Desktop Claude's memory | `ai_general/memories/ai_ecosystem_manifest.yml` |
| **Condensed files**          | 60-80% token reduction for core docs        | `.condensed.yml` pattern              |
| **Glossary/index**           | Term recognition without full doc load      | Bootstrap problem solution            |
| **Drain-and-fill**           | Context reset preserving essential state    | Chat continuity protocol              |
| **Write-immediate**          | Log observations as they occur, never batch | Memory slot discipline                |

**Key insight:** Memory extends far beyond the context window through layered external stores with varying access patterns (AUTO/TOPIC/DEMAND loading).

```
┌─────────────────────────────────────────────────────────────────────┐
│                     MEMORY ARCHITECTURE                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌─────────────┐                                                   │
│   │  EXECUTION  │  ◄── Loaded into context window                   │
│   │   MEMORY    │      • Memory slots (6KB index)                   │
│   │             │      • Auto-loaded knowledge (~4K tokens)         │
│   │  (HOT)      │      • Current conversation                       │
│   └──────┬──────┘                                                   │
│          │                                                          │
│          │ on-demand loading (REF: pointers, glossary triggers)     │
│          ▼                                                          │
│   ┌─────────────┐                                                   │
│   │  KNOWLEDGE  │  ◄── Files on disk, loaded when needed            │
│   │    STORE    │      • Full knowledge docs                        │
│   │             │      • Condensed versions                         │
│   │  (WARM)     │      • Protocol specs                             │
│   └──────┬──────┘                                                   │
│          │                                                          │
│          │ search/retrieval (explicit request or pipeline)          │
│          ▼                                                          │
│   ┌─────────────┐                                                   │
│   │   MEMORY    │  ◄── Processed chat histories                     │
│   │    STORE    │      • Chat memories (YAML)                       │
│   │             │      • Layered summaries                          │
│   │  (COLD)     │      • Knowledge/decision digests                 │
│   └─────────────┘      • Cross-referenced indexes                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```


---

### 3.4 TOOLS — How the AI Acts on the World

#### External (Baseline)
| Element           | Description                   |
|-------------------|-------------------------------|
| API calls         | Web search, external services |
| Code execution    | Running scripts, computations |
| File operations   | Read/write files in sandbox   |
| Retrieval         | Search vector stores, databases |

#### Internal (Our Extensions)
| Element                   | Description                       | Implementation             |
|---------------------------|-----------------------------------|----------------------------|
| **Desktop Commander MCP** | Full filesystem + process control | Read/write/search anywhere |
| **Codex MCP**             | Synchronous AI execution (30-60s) | Immediate task completion  |
| **CLI coordination**      | Asynchronous multi-step work      | Task lifecycle directories |
| **send_prompt.sh**        | Inject prompts into AI sessions   | Cross-AI communication     |
| **AT scheduling**         | Future prompt execution           | Self-wake patterns         |
| **Browser automation**    | Puppeteer-based web interaction   | Chat orchestration         |
| **Notification system**   | Alert human of events             | terminal-notifier, ntfy    |
| **Chrome control MCP**    | Browser tab manipulation          | Page content extraction    |

**Key insight:** Tools aren't just for accessing external data—they enable AI-to-AI communication and autonomous scheduling.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        TOOL CATEGORIES                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  FILESYSTEM          EXECUTION           COMMUNICATION              │
│  ┌──────────┐       ┌──────────┐        ┌──────────────┐            │
│  │ Desktop  │       │ Codex    │        │ send_prompt  │            │
│  │ Commander│       │ MCP      │        │ .sh          │            │
│  │          │       │ (sync)   │        │              │            │
│  │ • read   │       │          │        │ • inject     │            │
│  │ • write  │       ├──────────┤        │   prompts    │            │
│  │ • search │       │ CLI      │        │ • cross-AI   │            │
│  │ • process│       │ Coord    │        │   messaging  │            │
│  └──────────┘       │ (async)  │        └──────────────┘            │
│                     └──────────┘                                    │
│                                                                     │
│  SCHEDULING          BROWSER             NOTIFICATION               │
│  ┌──────────┐       ┌──────────┐        ┌──────────────┐            │
│  │ AT jobs  │       │ Chrome   │        │ terminal-    │            │
│  │          │       │ Control  │        │ notifier     │            │
│  │ • future │       │ Puppeteer│        │              │            │
│  │   prompts│       │          │        │ • alert      │            │
│  │ • self-  │       │ • tabs   │        │   human      │            │
│  │   wake   │       │ • content│        │ • status     │            │
│  └──────────┘       └──────────┘        └──────────────┘            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

### 3.5 ACTION — What the AI Does

#### External (Baseline)
| Element               | Description                       |
|-----------------------|-----------------------------------|
| Generate response     | Text output to user               |
| Execute tool calls    | Invoke tools and process results  |
| Update conversation   | Add to message history            |

#### Internal (Our Extensions)
| Element                   | Description                       | Implementation                                |
|---------------------------|-----------------------------------|-----------------------------------------------|
| **Delegate to AI**        | Assign work to other AI instances | CLI tasks, Codex calls                        |
| **Autonomous operation**  | Work without human in loop        | Overnight automation                          |
| **Self-scheduling**       | Set future prompts for self       | AT wake patterns                              |
| **Cross-platform coord**  | Orchestrate across AI platforms   | ChatGPT, Gemini integration                   |
| **Response footers**      | Track state across messages       | `context:X \| tokens:Y`                       |
| **Task lifecycle**        | Move tasks through states         | staged → to_execute → in_progress → completed |
| **Parallel execution**    | Multiple workers simultaneously   | Multi-CLI orchestration                       |

**Key insight:** Action extends beyond responding to the user. The AI can spawn other AI work, schedule its own future actions, and operate autonomously for extended periods.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ACTION PATTERNS                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  SYNCHRONOUS                    ASYNCHRONOUS                        │
│  ┌────────────────────┐        ┌────────────────────┐               │
│  │ • Respond to user  │        │ • Delegate to CLI  │               │
│  │ • Codex MCP call   │        │ • Spawn parallel   │               │
│  │ • Tool execution   │        │   workers          │               │
│  │ • File write       │        │ • Schedule AT job  │               │
│  └────────────────────┘        │ • Set self-wake    │               │
│                                └────────────────────┘               │
│                                                                     │
│  AUTONOMOUS                     COORDINATED                         │
│  ┌────────────────────┐        ┌────────────────────┐               │
│  │ • Overnight work   │        │ • Multi-AI chat    │               │
│  │ • Pulse-driven     │        │ • Peer review      │               │
│  │   execution        │        │ • Task handoff     │               │
│  │ • Self-directed    │        │ • Synthesis from   │               │
│  │   task selection   │        │   multiple workers │               │
│  └────────────────────┘        └────────────────────┘               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```


---

## Part 4: What Makes This Architecture Unique

### 4.1 Breaking the Single-Agent Assumption

Standard LLM agent frameworks assume one AI instance per task. We distribute across:

| Platform           | Role                     | Context Persistence               | Strengths                     |
|--------------------|--------------------------|-----------------------------------|-------------------------------|
| **Desktop Claude** | Orchestrator             | Session-bounded + memory slots    | MCP tools, strategic view     |
| **Claude CLI**     | Autonomous workers       | Named sessions persist            | Long-running tasks, parallel execution |
| **Codex MCP**      | Synchronous executor     | None (stateless)                  | Fast validation, code tasks   |
| **ChatGPT**        | Peer AI                  | Project shared memory             | Alternative perspective       |
| **Gemini**         | Peer AI                  | Limited                           | Alternative perspective       |

### 4.2 The Orchestrator Model

```
                         ┌─────────────────┐
                         │ DESKTOP CLAUDE  │
                         │  (Orchestrator) │
                         │                 │
                         │ • Holds context │
                         │ • Makes strategy│
                         │ • Delegates work│
                         └────────┬────────┘
                                  │
           ┌──────────────────────┼──────────────────────┐
           │                      │                      │
           ▼                      ▼                      ▼
    ┌─────────────┐       ┌─────────────┐       ┌─────────────┐
    │   CLI #1    │       │   CLI #2    │       │  Codex MCP  │
    │ (librarian) │       │ (custodian) │       │  (sync)     │
    │             │       │             │       │             │
    │ • Knowledge │       │ • Cleanup   │       │ • Quick     │
    │   tasks     │       │   tasks     │       │   validation│
    └─────────────┘       └─────────────┘       └─────────────┘
```

**Why this works:** Desktop Claude preserves context for strategic decisions by delegating execution-heavy work. Workers operate independently, report back through files.

### 4.3 Memory as First-Class Citizen

Most agent frameworks treat memory as an afterthought (basic RAG, conversation buffer). We built an entire memory infrastructure:

1. **Federated ownership** — Each AI owns its memory slots
2. **Layered abstraction** — Raw → Summary → Meta-summary
3. **Cross-AI access** — With explicit protocols
4. **Hot/Warm/Cold tiers** — Different access patterns
5. **Bootstrap solution** — Glossary enables knowing what you don't know

### 4.4 Autonomous Operation

Standard: Human sends prompt → AI responds → Human sends next prompt

Ours: 
```
Human sets up task
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│                    AUTONOMOUS LOOP                          │
│                                                             │
│   ┌──────────┐      ┌──────────┐      ┌──────────┐          │
│   │  Pulse   │ ───► │  Check   │ ───► │ Execute  │          │
│   │ trigger  │      │  TODOs   │      │   work   │          │
│   └──────────┘      └──────────┘      └──────────┘          │
│        ▲                                    │               │
│        │                                    │               │
│        └────────────────────────────────────┘               │
│                    (AT self-wake)                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
       │
       ▼
Human wakes up to completed work
```

---

## Part 5: Component Inventory

### 5.1 Repositories (Git Submodules)

┌─────────────────────────────────────────────────────────────┌
| Repo          | Purpose                     | Primary Owner |
|---------------|-----------------------------|---------------|
| `ai_root`     | Parent container            | System        |
└─────────────────────────────────────────────────────────────┘
| `ai_general`  | Shared resources, knowledge | All AIs       |
|               | protocols                   |               |
└─────────────────────────────────────────────────────────────┘
| `ai_claude`   | Claude-specific config,     | Claude        |
|               | memory, instructions        |               |
└─────────────────────────────────────────────────────────────┘
| `ai_comms`    | Inter-AI communication      | System        |
|               | infrastructure              |               |
└─────────────────────────────────────────────────────────────┘
| `ai_memories` | Processed chat histories    | Pipeline      | 
|               | digests                     |               |
└─────────────────────────────────────────────────────────────┘

### 5.2 Key Protocols

| Protocol                 | Purpose                          | Location                               |
|--------------------------|----------------------------------|----------------------------------------|
| Task Coordination v4     | CLI work assignment lifecycle    | `protocol_taskCoordination.yml`        |
| Reference Pointers       | On-demand file loading           | `protocol_reference_pointers.yml`      |
| Federated Memory         | Cross-AI memory access           | `protocol_federated_memory.yml`        |
| Chat Orchestration       | AI-to-AI communication           | `protocol_ai_chat_orchestration.yml`   |
| Condensed Resolution     | File loading preferences         | `protocol_condensed_resolution.yml`    |

### 5.3 Key Tools

| Tool               | Type           | Purpose                            |
|--------------------|----------------|------------------------------------|
| Desktop Commander  | MCP Server     | Filesystem, processes, search      |
| Codex MCP          | MCP Server     | Synchronous AI execution           |
| chrome-control     | MCP Server     | Browser automation                 |
| claude_cli.sh      | Shell wrapper  | Launch Claude CLI with options     |
| codex_cli.sh       | Shell wrapper  | Launch Codex MCP sessions          |
| send_prompt.sh     | Shell script   | Inject prompts cross-AI            |


---

## Part 6: External Architecture References

Our architecture extends patterns documented in academic and industry literature. Key sources:

### 6.1 Academic Foundations

| Source                                                           | Contribution                                     | Our Application                 |
|------------------------------------------------------------------|--------------------------------------------------|---------------------------------|
| **"Fundamentals of Building Autonomous LLM Agents"** (TUM, 2025) | Perception-Reasoning-Memory-Action loop          | Base architecture we extend     |
| **"Landscape of Emerging AI Agent Architectures"** (arXiv 2404.11584) | Multi-agent patterns, DyLAN, AgentVerse     | Multi-CLI orchestration         |
| **"Cognitive Architectures for Language Agents"**                | Memory type differentiation                      | Our Hot/Warm/Cold tiers         |
| **Lewis et al. (2020)**                                          | RAG - combining parametric + non-parametric memory | Knowledge file loading        |

### 6.2 Industry Frameworks

| Framework              | Pattern                          | Our Analog                              |
|------------------------|----------------------------------|-----------------------------------------|
| **LangChain**          | Chains, memory, tools            | Protocol-based coordination             |
| **AutoGen** (Microsoft)| Multi-agent conversation         | CLI workers + Desktop orchestrator      |
| **CrewAI**             | Role-based agent collaboration   | Librarian, Custodian, Dev-Lead, Ops     |
| **LangGraph**          | Graph-based orchestration        | Task lifecycle state machine            |

### 6.3 Canonical Diagrams

The baseline LLM agent architecture we extend is documented at:
- [Prompt Engineering Guide - LLM Agents](https://www.promptingguide.ai/research/llm-agents)
- [A Visual Guide to LLM Agents](https://newsletter.maartengrootendorst.com/p/a-visual-guide-to-llm-agents) (Maarten Grootendorst)
- [Emerging Architectures for LLM Applications](https://a16z.com/emerging-architectures-for-llm-applications/) (Andreessen Horowitz)

---

## Part 7: What This Document Is For

Per original requirements:

| Audience         | Use                                                    |
|------------------|--------------------------------------------------------|
| **AI workers**   | Understand where they fit, what's required of them     |
| **PianoMan**     | Coalesce everything under one conceptual model         |
| **New readers**  | Starting point for understanding system breadth        |
| **Other users**  | Communicate what this system is and does               |

### How to Use This Document

1. **Onboarding:** Read Parts 1-2 for conceptual framework, Part 3 for component details
2. **Reference:** Use Part 5 inventories to find specific protocols/tools
3. **Context:** Understand any component by locating it in the 5-layer model
4. **Extension:** When adding new capabilities, identify which layer(s) they augment

---

## Appendix A: Summary Table — External vs Internal

| Component       | External (Baseline)                     | Internal (Our Extensions)                                           |
|-----------------|-----------------------------------------|---------------------------------------------------------------------|
| **Perception**  | User input, system prompt, tool results | Auto-loading, glossary, memory slots, cross-AI messages             |
| **Reasoning**   | Single LLM, goal decomposition          | Multi-AI, specialized roles, orchestrator model, validation loops   |
| **Memory**      | Context window, basic RAG               | Federated slots, layered summaries, cross-AI, condensed files       |
| **Tools**       | APIs, code execution, file I/O          | Desktop Commander, Codex, CLI coordination, AT scheduling           |
| **Action**      | Response generation, tool calls         | Delegation, autonomous operation, self-scheduling, parallel execution|

---

## Appendix B: Document Relationships

This document serves as the **architectural spine**. Related documents provide depth:

```
architecture_augmentation_framework.v1.0.md (THIS DOC - conceptual overview)
       │
       ├── 10_architecture/
       │   ├── cli_orchestration_latest.md (CLI coordination details)
       │   ├── daemon_architecture_latest.md (pulse/scheduling systems)
       │   └── federated_memory_architecture.yml (memory system details)
       │
       ├── 30_protocols/
       │   ├── protocol_taskCoordination.yml (task lifecycle)
       │   ├── protocol_reference_pointers.yml (file loading)
       │   └── protocol_federated_memory.yml (cross-AI memory)
       │
       ├── 60_playbooks/
       │   ├── parallel_orchestration_case_study.yml (multi-CLI example)
       │   └── cli_agent_operations.yml (CLI usage guide)
       │
       └── 20_registries/
           ├── glossary_knowledge_index.yml (term definitions)
           └── tools_manifest.yml (tool inventory)
```

---

**Document Status:** Draft v1.0 — Ready for review and iteration

