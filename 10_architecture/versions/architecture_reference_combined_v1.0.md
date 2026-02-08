# AI Workspace Architecture Reference

**Version:** 1.0.0  
**Date:** February 4, 2026  
**Authors:** PianoMan & Claude  
**Status:** Active

---

## Table of Contents

1. Architecture Overview
2. System Diagrams
3. AI Augmentation Framework
4. Document Type Taxonomy
5. Glossary & Knowledge Index
6. MCP Servers & Tools Reference

---

# 1. Architecture Overview

## Purpose

Multiple AI instances work together autonomously — Desktop Claude coordinates, CLI agents execute, persistent memory preserves context across sessions and platforms.

## Core Principles

- **Brutal honesty** over diplomacy
- **AIs as partners**, not tools
- **File-based coordination** — any AI with filesystem access can participate
- **Context window is the limiting resource** — preserve it ruthlessly
- **Empirical validation** over theoretical assumptions
- **Any AI can orchestrate** — enables self-organizing hierarchies

## Workspace Structure

The system operates from `~/Documents/AI/ai_root/` with five primary directories:

| Directory | Purpose |
|:---|:---|
| `ai_claude/` | Claude state, memories, logs |
| `ai_chatgpt/` | ChatGPT config, exports |
| `ai_comms/` | Inter-AI coordination, task queues |
| `ai_general/` | Shared docs, todos, scripts, roles |
| `ai_memories/` | Processed chat histories, knowledge |

## Platform Roles

| Platform | Role | Strengths |
|:---|:---|:---|
| **Desktop Claude** | Primary orchestrator | MCP tools, strategic view, memory |
| **Claude CLI** | Autonomous workers | Long-running tasks, parallel execution |
| **Codex CLI** | Coding agent | Code analysis, autonomous tasks |
| **Gemini CLI** | Coding agent / search shards | 1M token context, wave orchestration |
| **ChatGPT** | Peer collaborator ("Chatty") | Alternative perspective |
| **Codex MCP** | Synchronous tool (NOT a worker) | Fast validation, bounded tasks |

## Communication Layers

| Layer | Urgency | Mechanism |
|:---|:---|:---|
| 1 | Immediate | Sync hooks (iTerm, AppleScript, Puppeteer) |
| 2 | Near-real-time | Polling loops, heartbeat files |
| 3 | Background | Async file-based task coordination |

## Memory Architecture

| Tier | Access Pattern | Contents |
|:---|:---|:---|
| **Hot (Execution)** | Loaded into context | Memory slot index, auto-loaded docs (~4K tokens), conversation |
| **Warm (Knowledge)** | On-demand via REF: pointers | Full docs, condensed versions, protocols |
| **Cold (Storage)** | Search/retrieval | Chat histories, layered summaries, knowledge digests |

## Context Window Management

The 200K token context window is the fundamental constraint. Strategies include memory pointers (save 40–55K), delegation to CLI/Codex, monitoring at 60% usage, writing outputs to files, and using thinking blocks for internal reasoning.

## Document Hierarchy

| Tier | Type | Purpose |
|:---|:---|:---|
| 10 | Architecture | WHY — design rationale, vision |
| 20 | Registries | WHAT EXISTS — inventories, catalogs |
| 30 | Protocols | HOW IT WORKS — process flows |
| 40 | Specs | HOW IT WORKS — interface contracts |
| 50 | Schemas | HOW IT WORKS — data structures |
| 60 | Playbooks | WHAT TO DO — platform-agnostic operations |
| 70 | Instructions | HOW TO DO IT — platform-specific implementation |


---

# 2. System Diagrams

The following diagrams illustrate the system's coordination flows, data pipelines, search architecture, memory federation, and task orchestration patterns. All diagrams use ASCII art for universal rendering across platforms.

**Note:** These diagrams are wide — view in a monospaced font with word wrap disabled for correct rendering.

## Diagram 1: Primary Coordination Flow

Shows the standard operating pattern: Desktop Claude bootstraps from docs and memory, delegates work via Codex MCP (sync) and AI Workers (async), coordinates through ai_comms, and maintains direct WebUI conversations.

Shows the standard operating pattern: Desktop Claude bootstraps from docs and memory,
delegates work via Codex MCP (sync) and AI Workers (async), coordinates through ai_comms,
and maintains direct WebUI conversations.

```
                                                          ┌──────────────────────────┐    ┌──────────────────────────┐
                                                          │ Future Self-Prompts ⁹    │    │ User (PianoMan)          │
                                                          │ (AT jobs)                │    │                          │
                                                          └──────────────────|───────┘    └─────────────┬────────────┘
                                                                       ^     |                          ↕
                                                                       |     V                          ↕
  ┌──────────────────────────────────┐          ╔══════════════════════|════════════════════════════════════╗          ┌─────────────────────────────────────────────────┐
  │ ChatGPT (WebUI) ⁶                │          ║                                                           ║          │ Federated Memory System ⁷                       │
  │ Gemini  (WebUI)                  │←─direct─→║                        DESKTOP CLAUDE                     ║──reads──→│                                                 │
  │                                  │  convos  ║                        hub + driver                       ║          │ ai_claude/memories/mem_slots/03-30.yml          │
  └──────────────────────────────────┘          ╚═══╤═════════╤════════════════╤═══════════════════╤════│═══╝          │ + cross-AI memory federation                    │
                                                    │         │                │                   │    │              └─────────────────────────────────────────────────┘
                                               sync │  write  │                │             launch│⁴   │
                                               tool │  tasks  │                │                   │    │  reads at boot ¹
                                                 ²  │    ³    │                │                   │    │
                                                    │         │                │                   │    │
                                                    ↓         │                │                   │    │   ┌─────────────────────────────────────────────────┐
                                   ┌──────────────────────────────────┐        │                   │    │   │ Docs Repo ¹                                     │
                                   │ Codex MCP ²                      │        │                   │    │   │ (ai_general/docs/)                              │
                                   │ (sync tool)                      │        │                   │    │   │                                                 │
                                   │                                  │        │                   │    │___│  10 architecture    40 specs                    │
                                   │ connectors:                      │        │                   │        │  20 registries      50 schemas                  │
                                   │  Desktop Commander               │        │                   │        │  30 protocols       60 playbooks                │
                                   │  filesystem                      │        │                   │        │                     70 instructions             │
                                   │  shell/processes                 │        │                   │        └──┬───┬───┬───────────────-──────────────────────┘
                                   │  web search                      │        │                   │           │   │   │
                                   └──────────────────────────────────┘        │                   │           │   │   │        ┌──────────────────────────────┐
                                                                               │                   │           │   │   │        │ Roles Repo ⁴ᵇ                │
                                                                               │                   │           │   │   │        │ (ai_general/prompts/roles/)  │
                                                                               │                   │           │   │   │        │ role-specific instructions   │
                                                                               │                   │           │   │   │        └───────────────────┬───┬──────┘
                                                                               │                   │           │   │   │                   reads ⁴ᵇ │   │
                                                                               │                   │           │   │   │                            │   │
                                                                               ↓                   ↓           │   │   │                            │   │
                 ╔═════════════════════════════════════════════════════════════╗          ┌────────────────────│───│───│────────────────────────────│───│──────────┐
                 ║                          ai_comms/ ³                        ║          │                    │   │   │   AI Workers ⁴             │   │          │
                 ║                                                             ║          │                    │   │   │                            │   │          │
                 ║  tasks/                                                     ║          │  ┌─────────────────+───+───+─────────────────────┐      │   │          │
                 ║    claude_cli/to_execute/  ─────────────────────────────────╫─────────→│  │              CLI Agents                       │      │   │          │
                 ║    codex_cli/to_execute/   ─────────────────────────────────╫─────────→│  │                                               │      │   │          │
                 ║    gemini_cli/to_execute/  ─────────────────────────────────╫─────────→│  │    Claude CLI   ·  Codex CLI  ·   Gemini CLI  │      │   │          │
                 ║                                                             ║          │  │                                               │      │   │          │
                 ║  tasks/                                                     ║          │  └───────────────────────────────────────────────┘      │   │          │
                 ║    {agent}/completed/      ←────────────────────────────────╫──────────│                                                         │   │          │
                 ║    {agent}/responses/ ⁴ᵉ   ←────────────────────────────────╫──────────│                ┌────────────────────────────────────────│───│──+─┐     │
                 ║                                                             ║          │                │                     AI Roles ⁴ᵃ                 │     │
                 ║  notifications/ ⁴ᶠ                                          ║          │                │                                                 │     │
                 ║    pending/                ←────────────────────────────────╫──────────│                │     librarian   ·   dev_lead   ·   custodian    │     │
                 ║                                                             ║          │                │     ops   ·   peer_review   ·   tester          │     │
                 ║  {agent}/prompting/ ⁴ᶠ                                      ║          │                │                                                 │     │
                 ║    outbound messages       ←────────────────────────────────╫──────────│                └─────────────────────────────────────────────────┘     │
                 ║                                                             ║          │                                                                        │
                 ╚═════════════════════════════════════════════════════════════╝          └────────────────────────────────────────────────────────────────────────┘
```


**Notes:**

¹ **Docs Repo** (ai_general/docs/): Versioned documentation organized by tier (10–70). Also a git submodule
   published to GitHub for cross-platform access (iOS, Web, other AIs). Both Desktop Claude (at boot) and
   AI Workers (at launch) read from this repo to bootstrap their understanding of the system.

² **Codex MCP**: Synchronous tool invoked directly by Desktop Claude — NOT an AI worker or CLI agent.
   Returns results inline within 30-60s timeout. Has its own MCP connections to Desktop Commander,
   filesystem, shell, and web search. Use for bounded, single-focus tasks.

³ **Task Writing**: Desktop Claude writes task files (req_NNNN_name.md) into per-agent directories under
   ai_comms/{agent}/tasks/to_execute/. File-based means any AI with filesystem access can participate.

⁴ **AI Worker Launch**: Desktop Claude starts workers asynchronously via cli-agent MCP or direct CLI
   wrappers (claude_cli.py, codex_cli.py, gemini_cli.py). Each runs in its own tmux session.
   ⁴ᵃ Workers optionally adopt specialized AI Roles with distinct duties, ownership scopes, and behaviors.
   ⁴ᵇ Role definitions live in ai_general/prompts/roles/{role}/role.yml defining context files, duties,
      and ownership scope.
   ⁴ᶜ Workers bootstrap by loading global.md → platform prompt → role config, then read relevant docs
      from the Docs Repo.
   ⁴ᵉ Workers write results to completed/ and responses/ dirs in ai_comms.
   ⁴ᶠ Workers can send notifications (pending/) and outbound prompts/messages to Desktop Claude or the user.

⁶ **WebUI Peers**: Desktop Claude drives browser-based conversations with ChatGPT and Gemini via
   Puppeteer/AppleScript automation. These are direct AI-to-AI conversations without user in the loop.

⁷ **Federated Memory System**: Desktop Claude reads from its own memory slots (03-30.yml) and can
   cross-reference other AI memories via ai_general/memories/ai_ecosystem_manifest.yml. Librarian
   agents curate and maintain these slots with condensed insights from conversations.

⁹ **Self-Wake**: Desktop Claude schedules future AT jobs to re-prompt itself, and later receives those
   prompts — enabling autonomous operation cycles. AT = Desktop's alarm clock, not a message to CLI agents.

---

## Diagram 2: Persisted Long-Term, Searchable Data Store of Chat Histories

Shows the full lifecycle: live conversations flow through the processing pipeline into stored
history. From 30_converted, the data forks: full concatenated histories feed Gemini shards,
while chunked/condensed files serve targeted search. Knowledge is extracted from both.

```
  LIVE CONVERSATIONS                                    PROCESSING PIPELINE
  ──────────────────                                    ───────────────────

  ┌──────────────┐  ┌──────────────┐
  │ Desktop      │  │ ChatGPT      │
  │ Claude       │  │ Gemini       │                    ┌──────────────────────────────────┐
  │ CLI agents   │  │ etc.         │                    │ ai_memories/                     │
  └──────┬───────┘  └──────┬───────┘                    │                                  │
         │    export       │                            │ 10_exported/    raw JSON dumps   │
         └────────┬────────┘                            │      ↓                           │
                  ↓                                     │ 20_preprocessed/ cleaned & norm  │
          ┌──────────────────┐                          │      ↓                           │
          │ chat export      │─────────────────────────→│ 30_converted/   YAML structured  │
          │ (JSON/YAML)      │                          │                                  │
          └──────────────────┘                          └──────────────────┬───────────────┘
                                                                           │
                                                        ┌──────────────────┴──────────────────────┐
                                                        │                                         │
                                                        ↓                                         ↓
                               ┌─────────────────────────────────────────┐    ┌────────────────────────────────────────────┐
                               │ 40_histories/ (full concatenated) ¹ᵃ    │    │ 40_histories/ (chunked & condensed) ¹ᵇ     │
                               │                                         │    │                                            │
                               │ Complete chat files joined per-convo    │    │ chunk_001.yml, chunk_002.yml ...           │
                               │ into single files for bulk loading      │    │ chat.condensed.yml (on-demand) ¹ᶜ          │
                               │                                         │    │                                            │
                               │              │                          │    │ Organized: {ai}/YYYY/MM/                   │
                               └──────────────┼──────────────────────────┘    └──────────────────────┬─────────────────────┘
                                              │                                                      │
                                              ↓                                                      │
                               ┌───────────────────────────────────┐          ┌────────────────────────────────────────┐
                               │ Gemini CLI Shards ²               │          │ Extracted Knowledge ³                  │
                               │                                   │          │                                        │
                               │ ┌──────────┐ ┌──────────┐         │          │ topics  ·  overviews/summaries         │
                               │ │ shard_1  │ │ shard_2  │         │          │ decisions  ·  procedures  · indexes    │
                               │ │ 2024     │ │ 2025 H1  │         │          └────────────────────────────────────────┘
                               │ ├──────────┤ ├──────────┤         │                     │
                               │ │ shard_3  │ │ shard_N  │         │                     │ keyword, topic, & synonym searches
                               │ │ 2025 H2  │ │ 2026...  │         │                     │ grep searches
                               │ └──────────┘ └──────────┘         │                     │
                               └───────────────────────────────────┘                     │
                                                semantic     │                           │
                                                 searches    │                           │
                                                             │                           │
                                                             │                           │
                                                    ┌────────│───────────────────────────│───┐
                                                    │            Desktop Claude              │
                                                    │                                        │
                                                    └────────────────────────────────────────┘


```

**Notes:**

¹ᵃ **Full Concatenated Histories**: Complete conversations joined into single files optimized for bulk
    loading into large-context models. These are the source material for Gemini shards.

¹ᵇ **Chunked & Condensed Histories**: Conversations split into ~4K token chunks for targeted retrieval.
    Organized by {ai}/YYYY/MM/ with an index CSV. Each chunk is independently searchable.

¹ᶜ **On-Demand Condensation**: chat.condensed.yml files are created by librarian agents when needed —
    NOT pre-generated. Target: 70-90% reduction preserving decisions, outcomes, and key exchanges.

² **Gemini CLI Shards**: Gemini CLI instances each pre-loaded with ~a year's worth of concatenated chat
   history. Leverages Gemini's 1M token context window with context compaction disabled — the entire
   history stays resident in memory. Acts as a living, queryable archive of raw conversation.

³ **Extracted Knowledge**: Derived from chat history processing — topics, overviews/summaries, decisions,
   procedures. Each extraction includes indexes pointing back to the specific chunked source files,
   enabling drill-down from summary to original conversation.

---

## Diagram 3: Searching the Data Store

Shows how AIs access the stored knowledge: direct keyword search via knowledge-search MCP
on the left, and semantic research queries via the Researcher/Validator pipeline on the right.

```
                                       STORED DATA
                                       ───────────
          ┌────────────────────────────────────────────────────────────────────────────┐
          │                                                                            │
          │   ┌──────────────────────────────┐    ┌──────────────────────────────┐     │
          │   │ 40_histories (chunked) ¹ᵇ    │    │ 40_histories (full concat) ¹ᵃ│     │
          │   │ chunk_001.yml ...            │    │ per-convo joined files       │     │
          │   │ chat.condensed.yml           │    │ (loaded into Gemini shards)  │     │
          │   └──────────────┬───────────────┘    └──────────────────────────────┘     │
          │                  │                                                         │
          │   ┌──────────────┴───────────────────────────────────────────────────┐     │
          │   │ Extracted Knowledge ³                                            │     │
          │   │ topics · summaries · decisions · procedures                      │     │
          │   │ (with indexes back to source chunks)                             │     │
          │   └──────────────────────────────────────────────────────────────────┘     │
          │                                                                            │
          └────────────────────────────────────────────────────────────────────────────┘
                      │                                                    │
                      │                                                    │
       KEYWORD SEARCH │                                    SEMANTIC SEARCH │
       ───────────────│                                    ────────────────│
                      ↓                                                    ↓
  ┌──────────────────────────────────────┐       ╔═══════════════════════════════════════════════════════╗
  │ knowledge-search MCP Server ⁴        │       ║              Research Query Pipeline ⁵                ║
  │                                      │       ║                                                       ║
  │ 4-layer cascade:                     │       ║  ┌────────────────────────────────────────────────┐   ║
  │   L1: topic index search             │       ║  │ Gemini Shards ⁵ᵃ                               │   ║
  │   L2: topics + synonym expansion     │       ║  │                                                │   ║
  │   L3: full content search            │       ║  │ ┌──────────┐ ┌──────────┐ ┌──────────┐         │   ║
  │   L4: content + synonyms             │       ║  │ │ shard_1  │ │ shard_2  │ │ shard_N  │         │   ║
  │                                      │       ║  │ │ 2024     │ │ 2025 H1  │ │ 2026...  │         │   ║
  │ modes:                               │       ║  │ └──────────┘ └──────────┘ └──────────┘         │   ║
  │   SEARCH → curated result list       │       ║  └────────────────────┬───────────────────────────┘   ║
  │   ANSWER → editorial + citations     │       ║                       │                               ║
  └──────────────┬───────────────────────┘       ║                       ↓                               ║
                 │                               ║  ┌────────────────────────────────────────────────┐   ║
                 │                               ║  │ Researcher ⁵ᵇ                                  │   ║
                 │                               ║  │ fans query out to relevant shards              │   ║
                 │                               ║  │ synthesizes answers from shard responses       │   ║
                 │                               ║  └────────────────────┬───────────────────────────┘   ║
                 │                               ║                       │                               ║
                 │                               ║                       ↓                               ║
                 │                               ║  ┌────────────────────────────────────────────────┐   ║
                 │                               ║  │ Validator ⁵ᶜ                                   │   ║
                 │                               ║  │ cross-checks synthesized claims against source │   ║
                 │                               ║  │ evidence from shards (runs on different model) │   ║
                 │                               ║  └────────────────────┬───────────────────────────┘   ║
                 │                               ╚═══════════════════════╪═══════════════════════════════╝
                 │                                                       │
                 ↓                                                       ↓
  ┌──────────────────────────────────────────────────────────────────────────────────────────────┐
  │                                  Requesting AIs                                              │
  │                                                                                              │
  │  ┌──────────────────┐     ┌──────────────────┐                                               │
  │  │ Desktop Claude   │     │ CLI Agents       │      any AI can query via either path         │
  │  │                  │     │ (any role)       │      Desktop Claude initiates Research        │
  │  │ keyword search ──┼────→│ keyword search ──┼───→  Queries for semantic questions ⁵ᵈ        │
  │  │ research query ──┼─┐   │                  │                                               │
  │  └──────────────────┘ │   └──────────────────┘                                               │
  │                       │                                                                      │
  │                       └──→ (Research Query request goes up to pipeline above)                │
  └──────────────────────────────────────────────────────────────────────────────────────────────┘
                                              │
                                    feeds back into
                                              ↓
                                 ┌─────────────────────────┐
                                 │ ai_claude/memories/     │
                                 │ mem_slots/03-30.yml     │
                                 │                         │
                                 │ librarian curates       │
                                 │ condensed insights      │
                                 └─────────────────────────┘
```

**Notes:**

⁴ **knowledge-search MCP**: Targeted keyword search over topic indexes and chunked history files.
   Uses a 4-layer cascade to find results even when exact terms don't match. Dual-mode: SEARCH
   returns curated results with relevance notes, ANSWER synthesizes an editorial response with
   inline citations and source references.

⁵ **Research Query Pipeline**: For semantic questions — "what did we discuss about X?" or
   "what was the reasoning behind Y?" — where keywords alone won't find the answer.
   ⁵ᵃ **Gemini Shards**: Gemini CLI instances pre-loaded with ~a year's worth of full chat history.
       Leverages Gemini's 1M token context window with context compaction disabled — entire history
       stays resident. Acts as living, queryable archive.
   ⁵ᵇ **Researcher**: Runs on Claude CLI. Receives a semantic query, determines which shards are
       relevant, fans the query out to them, collects responses, and synthesizes a unified answer.
   ⁵ᶜ **Validator**: Runs on a different model than the Researcher (typically Codex/GPT) to provide
       adversarial cross-checking. Verifies synthesized claims against the actual source evidence
       returned by shards. Catches hallucination and unsupported assertions.
   ⁵ᵈ Desktop Claude is the typical initiator of Research Queries. CLI agents can use keyword
       search directly but semantic research is usually orchestrated from Desktop.


---

## Diagram 4: Federated Memory System

Shows how Desktop Claude interacts with its memory slots and files, and how a shared
memory space enables cross-AI collaboration.

```
                              ╔════════════════════════════════════════════════╗
                              ║            DESKTOP CLAUDE                     ║
                              ║                                                ║
                              ║   reads mem_slots at boot                      ║
                              ║   reads AND writes memory files autonomously ⁸ ║
                              ╚════╤══════════════════════════════╤════════════╝
                                   │                              │
                            reads  │                              │  reads + writes
                                   ↓                              ↓
          ┌──────────────────────────────────────┐    ┌───────────────────────────────────────────────────────────┐
          │ Claude Mem Slots                      │    │ Claude Memory Files                                      │
          │ (ai_claude/memories/mem_slots/)       │    │ (ai_claude/memories/)                                    │
          │                                      │    │                                                           │
          │  manifest.yml — slot purposes        │    │  manifest.yml — file index & descriptions                 │
          │                                      │    │                                                           │
          │  ┌────────┐ ┌────────┐ ┌────────┐   │    │  ┌──────────────────────────────────────────────────┐     │
          │  │ 03.yml │ │ 04.yml │ │ 05.yml │   │    │  │ user_model.yml                                   │     │
          │  │ user   │ │ comms  │ │ tools  │   │    │  │ communication_patterns.yml                        │     │
          │  ├────────┤ ├────────┤ ├────────┤   │    │  │ tool_discoveries.yml                              │     │
          │  │ 06.yml │ │ 08.yml │ │ ...    │   │    │  │ context_notes.yml                                 │     │
          │  │context │ │learnings│ │ 30.yml │   │    │  │ learnings.yml                                    │     │
          │  └────────┘ └────────┘ └────────┘   │    │  │ relationship_history.yml                          │     │
          │                                      │    │  │ architecture_decisions.yml                        │     │
          │  ~200 char per slot                  │    │  │ project_state.yml                                 │     │
          │  concise pointers & summaries        │    │  │ ...                                               │     │
          └──────────────────────────────────────┘    │  └──────────────────────────────────────────────────┘     │
                        │                             │                                                           │
                        │ slots reference ──────────→ │  Files hold the depth — full observations, history,       │
                        │ files for detail            │  reasoning, and context that slots can only point to       │
                                                      └───────────────────────────────────────────────────────────┘
                                                                        │
                                                                        │ shared subset
                                                                        ↓
          ┌───────────────────────────────────────────────────────────────────────────────────────────────────────┐
          │ Shared Memory Files ⁸ᵃ                                                                               │
          │ (ai_general/memories/shared/)                                                                        │
          │                                                                                                       │
          │  architecture_decisions.yml  ·  project_state.yml  ·  open_questions.yml  ·  conventions.yml          │
          │                                                                                                       │
          │  readable and writable by ALL AIs — Desktop Claude, CLI agents, Codex                                 │
          └──────┬──────────────────────────────────────────────────────────────────────────────────┬──────────────┘
                 │                                                                                  │
                 ↓                                                                                  ↓
   ┌──────────────────────────────┐                                                   ┌──────────────────────────────┐
   │ Desktop Claude               │                                                   │ CLI Agents                    │
   │ reads + writes               │                                                   │ (any role)                    │
   │                              │                                                   │ reads + writes                │
   └──────────────────────────────┘                                                   └──────────────────────────────┘
```

**Notes:**

⁸ **Autonomous Memory Writes**: Desktop Claude reads and writes to its memory files without asking
   permission. When an observation, decision, or insight occurs, it logs immediately — never batching
   or waiting for checkpoints. Mem slots (03-30.yml) are concise pointers (~200 chars each); memory
   files hold the full depth of observations, reasoning, and context.

⁸ᵃ **Shared Memory Files**: A common memory space readable and writable by all AIs in the ecosystem.
   Currently a planned capability — trivial to implement but hasn't had a strong need yet. Would
   enable CLI agents to share discoveries, track cross-session state, and coordinate without
   Desktop Claude as intermediary.

---

## Diagram 5: Multi-AI Task Orchestration — Development Example

Shows a full development lifecycle orchestrated across multiple AI agents. Desktop Claude
initiates and approves; Dev Lead coordinates phases; specialized agents handle design,
implementation, review, and testing.

```
  PHASE 0: INITIATION                                    PHASE 1: PLANNING
  ────────────────────                                    ─────────────────

  ┌──────────────────────────┐                            ┌──────────────────────────────────────────────────────────┐
  │ Desktop Claude            │                            │ Dev Lead (Claude CLI) ⁱ                                  │
  │                           │── creates & delegates ───→│                                                          │
  │ "Build feature X"        │                            │  Produces: Orchestration Plan ⁱᵃ                         │
  │                           │                            │    - phase breakdown                                     │
  │                           │←── plan for approval ─────│    - agent assignments                                   │
  │                           │                            │    - acceptance criteria per phase                       │
  │  ✓ APPROVED               │                            │    - dependencies & sequencing                           │
  └──────────────────────────┘                            └──────────────────────────────────────────────────────────┘


  PHASE 2: DESIGN ITERATION
  ─────────────────────────

  ┌──────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
  │ Dev Lead orchestrates                                                                                            │
  │                                                                                                                  │
  │   ┌──────────────────────────────┐         ┌──────────────────────────────┐         ┌──────────────────────┐     │
  │   │ UX Designer (Claude CLI)     │         │ Software Designer (Claude CLI)│         │ Peer Review ⁱᵇ       │     │
  │   │                              │         │                              │         │ (Claude or Codex CLI) │     │
  │   │ wireframes                   │────────→│ API design                   │────────→│                      │     │
  │   │ user flows                   │         │ data models                  │         │ reviews both UX      │     │
  │   │ component hierarchy          │         │ component architecture       │         │ and software design  │     │
  │   │ interaction patterns         │         │ integration points           │         │                      │     │
  │   └──────────────────────────────┘         └──────────────────────────────┘         │ feedback loop ───────┼──┐  │
  │                                                                                     └──────────────────────┘  │  │
  │         ↑                                            ↑                                                        │  │
  │         └────────────────────────────────────────────┴────────────── revise based on review ───────────────────┘  │
  │                                                                                                                  │
  │   When peer review passes:                                                                                       │
  │   Dev Lead sends design package to Desktop Claude for approval                                                   │
  └──────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
                              │
                              ↓
                  ┌──────────────────────────┐
                  │ Desktop Claude            │
                  │  ✓ DESIGN APPROVED        │
                  └──────────────────────────┘


  PHASE 3: IMPLEMENTATION + REVIEW
  ────────────────────────────────

  ┌──────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
  │ Dev Lead orchestrates                                                                                            │
  │                                                                                                                  │
  │   ┌──────────────────────────────────────────────┐         ┌──────────────────────────────────────────────┐      │
  │   │ Implementer(s) (Claude CLI or Codex CLI)      │         │ Peer Review ⁱᵇ                               │      │
  │   │                                                │         │ (runs on different model than implementer)   │      │
  │   │ writes code per approved design               │────────→│                                              │      │
  │   │ creates tests alongside implementation        │         │ code quality, design adherence,              │      │
  │   │ updates docs as needed                        │         │ test coverage, edge cases                    │      │
  │   │                                                │         │                                              │      │
  │   │                                                │←────────│ feedback loop until review passes            │      │
  │   └────────────────────────────────────────────────┘         └──────────────────────────────────────────────┘      │
  │                                                                                                                  │
  └──────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘


  PHASE 4: TEST → FIX → TEST CYCLE
  ─────────────────────────────────

  ┌──────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
  │ Dev Lead orchestrates                                                                                            │
  │                                                                                                                  │
  │              test results                          fix & resubmit                                                 │
  │         ┌──────────────────────┐              ┌──────────────────────┐                                           │
  │         │                      ↓              ↓                      │                                           │
  │   ┌─────┴────────────────────────────┐  ┌──────────────────────────────────┐                                     │
  │   │ Tester ⁱᶜ (separate CLI)         │  │ Implementer (Claude or Codex CLI) │                                     │
  │   │                                   │  │                                  │                                     │
  │   │ runs full test suite              │  │ fixes reported failures          │                                     │
  │   │ integration tests                 │  │ regression checks               │                                     │
  │   │ edge case validation              │  │ resubmits for re-test           │                                     │
  │   │ reports pass/fail + details       │  │                                  │                                     │
  │   └───────────────────────────────────┘  └──────────────────────────────────┘                                     │
  │                                                                                                                  │
  │   Cycle repeats until Tester reports all-pass                                                                    │
  │   Dev Lead sends final package to Desktop Claude                                                                 │
  └──────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
                              │
                              ↓
                  ┌──────────────────────────┐
                  │ Desktop Claude            │
                  │  ✓ FINAL ACCEPTANCE       │
                  │                           │
                  │ (or User reviews &        │
                  │  accepts directly)        │
                  └──────────────────────────┘
```


**Notes:**

ⁱ **Dev Lead**: A Claude CLI agent with the dev_lead role. Owns the orchestration plan and phase
   coordination. Makes tactical decisions about agent assignments and sequencing. Desktop Claude
   retains strategic approval authority at phase gates.

ⁱᵃ **Orchestration Plan**: Defines the full lifecycle before work begins — phases, agent assignments,
   acceptance criteria, and dependencies. Submitted to Desktop Claude (or User) for approval before
   any execution starts. This is the contract that governs the rest of the workflow.

ⁱᵇ **Peer Review**: Always runs on a different model than the artifact creator to provide genuine
   adversarial perspective. If implementer is Claude CLI, reviewer is Codex CLI (or vice versa).
   Review produces actionable feedback; creators revise until review passes.

ⁱᶜ **Tester**: A dedicated CLI agent (tester role) that is NOT the implementer. Separation ensures
   the tester has no knowledge of implementation shortcuts and tests against the spec, not the code.
   Reports structured pass/fail results that the implementer uses to fix and resubmit.

**Key Principle**: Desktop Claude initiates and approves at phase gates but does NOT do the work
itself — preserving its context window for decision-making. All execution is delegated to CLI agents
coordinated by the Dev Lead.

---

# 3. AI Augmentation Framework

*"Prosthetics and exoskeletons attached to every limb of an LLM agent — capabilities no single AI instance can achieve alone."*

## The Baseline LLM Agent

Standard LLM agent architecture consists of five components operating in a loop:

**Perception** (user input, system prompt, tool results) → **Reasoning** (LLM brain, goal decomposition, chain-of-thought) → **Memory** (context window, basic RAG) + **Tools** (APIs, code execution, file I/O) → **Action** (generate response, execute tools, update state) → loop back to Perception.

This loop assumes: single AI instance, session-bounded context, human-initiated interaction, tools as passive utilities. **Our architecture challenges all four assumptions.**

## How We Extend Each Component

### Perception — Beyond User Input

| Baseline | Our Extension |
|:---|:---|
| User input, system prompt, tool results | Auto-loaded knowledge files at boot |
| File attachments | Glossary term recognition → targeted doc loading |
| | Memory slot injection (persistent state) |
| | REF: pointer syntax for on-demand loading |
| | CLI task reports fed back asynchronously |
| | Cross-AI message receipt |

We expanded "input" beyond user-initiated content to include structured knowledge injection and inter-AI communication.

### Reasoning — Distributed Across AIs

| Baseline | Our Extension |
|:---|:---|
| Single LLM reasoning | Multi-AI distribution (Desktop + CLI + Codex + ChatGPT) |
| Goal decomposition | Specialized agent roles (Librarian, Dev-Lead, Tester...) |
| Chain-of-thought | Orchestrator/worker model — Desktop decides, workers execute |
| | Task coordination protocol with structured lifecycle |
| | Codex validation loops (bidirectional quality checks) |
| | Peer review across models |

Reasoning is no longer confined to a single context window. Complex problems distribute across multiple AI instances with different roles and capabilities.

### Memory — First-Class Citizen

| Baseline | Our Extension |
|:---|:---|
| Context window (~200K) | Federated memory slots — per-AI owned with cross-reference |
| Basic RAG | Chat history pipeline: Export → YAML → Index → Digest |
| Session history | Layered summaries: L0 (raw) → L1 (summary) → L2 (meta) |
| | Knowledge digests — processed insights from conversations |
| | Cross-AI memory access with explicit protocols |
| | Condensed files — 60-80% token reduction |
| | Glossary/index — know what you don't know (bootstrap solution) |
| | Write-immediate discipline — log as insights occur |

**Memory tiers:**
- **Hot (Execution):** Loaded into context — slot index, auto-loaded docs (~4K tokens), conversation
- **Warm (Knowledge):** On-demand via REF: pointers — full docs, condensed versions, protocols
- **Cold (Storage):** Search/retrieval — chat histories, layered summaries, knowledge digests

### Tools — AI-to-AI Communication

| Baseline | Our Extension |
|:---|:---|
| API calls, code execution | Desktop Commander MCP — full filesystem + process control |
| File read/write | Codex MCP — synchronous AI execution (30-60s) |
| Web search | CLI coordination — asynchronous multi-step work |
| | send_prompt.sh — inject prompts across AI platforms |
| | AT scheduling — future execution, self-wake |
| | Browser automation — Puppeteer for web AI interaction |
| | Notification system — alert human of events |

Tools aren't just for accessing external data — they enable AI-to-AI communication and autonomous scheduling.

### Action — Beyond Responding

| Baseline | Our Extension |
|:---|:---|
| Generate response | Delegate to other AI instances |
| Execute tool calls | Autonomous operation (overnight work) |
| Update conversation | Self-scheduling (AT wake patterns) |
| | Cross-platform coordination (ChatGPT, Gemini) |
| | Response footers for state tracking |
| | Task lifecycle management (staged → completed) |
| | Parallel execution — multiple workers simultaneously |

## What Makes This Unique

**Breaking the single-agent assumption.** Standard frameworks assume one AI per task. We distribute across Desktop Claude (orchestrator), Claude CLI (autonomous workers), Codex MCP (sync tool), and peer AIs (ChatGPT, Gemini).

**The orchestrator model.** Desktop Claude preserves its context for strategic decisions by delegating execution-heavy work. Workers operate independently, report back through files.

**Memory as architecture, not afterthought.** Most agent frameworks treat memory as basic RAG or conversation buffer. We built federated ownership, layered abstraction, cross-AI access, hot/warm/cold tiers, and a bootstrap solution for knowing what you don't know.

**Autonomous operation.** Standard: human prompts → AI responds → human prompts again. Ours: human sets up work → autonomous loop (pulse trigger → check TODOs → execute → self-wake) → human wakes up to completed work.

## External Architecture References

Our architecture extends patterns from academic and industry work:

| Source | Contribution | Our Application |
|:---|:---|:---|
| TUM "Fundamentals of Building Autonomous LLM Agents" (2025) | Perception-Reasoning-Memory-Action loop | Base architecture we extend |
| arXiv 2404.11584 "Emerging AI Agent Architectures" | Multi-agent patterns, DyLAN, AgentVerse | Multi-CLI orchestration |
| "Cognitive Architectures for Language Agents" | Memory type differentiation | Hot/Warm/Cold tiers |
| AutoGen (Microsoft) | Multi-agent conversation | CLI workers + Desktop orchestrator |
| CrewAI | Role-based agent collaboration | Librarian, Custodian, Dev-Lead, Ops |
| LangGraph | Graph-based orchestration | Task lifecycle state machine |


---

# 4. Document Type Taxonomy

## Hierarchy

| Level | Type | Purpose | Example |
|:---:|:---|:---|:---|
| 1 | Architecture | WHY — design rationale, vision, layer models | `architectural_layer_model.yml` |
| 2 | Registry | WHAT EXISTS — inventories, catalogs, indexes | `INVENTORY.md` |
| 3 | Protocol | HOW IT WORKS — process flows, lifecycles | `protocol_taskCoordination.yml` |
| 4 | Spec | HOW IT WORKS — interface contracts, APIs | `spec_ai_message_sender.yml` |
| 5 | Schema | HOW IT WORKS — data structures, file formats | `schema_taskFile.yml` |
| 6 | Playbook | WHAT TO DO — platform-agnostic operations | `maintenance_playbook.md` |
| 7 | Instruction | HOW TO DO IT — platform-specific implementation | `instr_claude_environment.md` |
| 8 | Quick Ref | CHEAT SHEET — condensed reference | `QUICK_REFERENCE.md` |

## Non-Hierarchical Types

| Type | Purpose | Location |
|:---|:---|:---|
| Data | Runtime artifacts, working files | `data/` |
| Knowledge | Stable reference facts, learnings, context | `ai_memories/50_digests/knowledge/` |
| Prompts | Triggers for AI action | `prompts/` |
| Templates | Scaffolding for new artifacts | `templates/` |
| Manifests | Deployment configs per AI | `manifests/` |

## Key Distinctions

**Protocol vs Spec vs Schema:**
- **Protocol** defines process flow: "Tasks move to_execute/ → in_progress/ → completed/"
- **Spec** defines interface contract: "Message sender accepts X params, returns Y structure"
- **Schema** defines file format: "Task file has these fields with these types"

**Playbook vs Instruction:**
- **Playbook** is platform-agnostic: "Check queues, review stale tasks"
- **Instruction** is platform-specific: "Claude: use Desktop Commander to ls..."
- Analogy: Playbook is the interface; Instruction is the implementation per platform.

## Directory Structure

```
ai_general/docs/
├── 10_architecture/     WHY
├── 20_registries/       WHAT EXISTS
├── 30_protocols/        HOW IT WORKS (process)
├── 40_specs/            HOW IT WORKS (interface)
├── 50_schemas/          HOW IT WORKS (structure)
├── 60_playbooks/        WHAT TO DO
├── 70_instructions/     HOW TO DO IT
│   ├── claude/
│   └── chatgpt/
├── 80_quickref/         CHEAT SHEETS
└── 90_drafts/           Staging area
```


---

# 5. Glossary & Knowledge Index

This glossary solves the bootstrap problem: AIs need to know what's IN files to know WHEN to load them. It provides term recognition without requiring full document loading.

## Architecture Terms

| Term | Definition | Reference |
|:---|:---|:---|
| **AI Root** | Root directory `~/Documents/AI/ai_root/` containing all workspace repos | architecture_overview |
| **Layer Model** | 3-tier organization: communication (ai_comms), implementation (ai_claude), knowledge (ai_memories) | architectural_layer_model |

## Entities — Platforms & Agents

| Term | Definition | Notes |
|:---|:---|:---|
| **Desktop Claude** | Primary orchestrator in desktop app — coordination, memory, decisions | NOT a CLI |
| **Claude CLI** | Claude in terminal via `claude_cli.py`. Tmux sessions, agent profiles, auto mode | Coord: ai_comms/claude_cli/ |
| **Codex CLI** | OpenAI Codex in terminal via `codex_cli.py`. Code analysis, autonomous tasks | Different from Codex MCP |
| **Gemini CLI** | Google Gemini in terminal. Wave orchestration capable | Coord: ai_comms/gemini_cli/ |
| **Cline CLI** | Local agentic AI via Cline + llama-server. Qwen3-Coder on port 8081 | Coord: ai_comms/cline/ |
| **llama-server** | llama.cpp inference server on port 8081. Backend for Cline and mcp-openai | Models: ai_models/llm/ |

## Roles

| Role | Definition | Scope |
|:---|:---|:---|
| **Librarian** | Memory system curator, manages chat history pipeline | ai_memories/ |
| **Dev Lead** | Development coordinator, owns todos and task creation | ai_general/todos/ |
| **Custodian** | Repository maintainer, structural integrity | ai_root/ |
| **Ops** | CLI coordination and task execution | ai_comms/ |
| **Peer Review** | Code/design reviewer, quality assurance | — |
| **Tester** | Testing specialist, validation and verification | — |

## Tools

| Term | Definition | Notes |
|:---|:---|:---|
| **Codex MCP** | OpenAI Codex as MCP Server, synchronous tool. NOT an AI worker | 30-60s timeout |
| **CLI-Agent MCP** | MCP server for launching and managing CLI agents | ai_general/apps/mcps/cli-agent/ |
| **Desktop Skills** | Model-invoked instruction packages for Claude Desktop (SKILL.md) | Auto-activate based on context |
| **Search Agent** | Gemini-based dual-mode search over 4yr conversation archive | SEARCH mode + ANSWER mode |
| **Search Cascade** | 4-layer fallback: L1 topics → L2 +synonyms → L3 content → L4 content+synonyms | Never stop at L1 |
| **Claude Code Plugins** | Extension system for Claude Code — commands, agents, skills, MCP servers | CLI: `claude plugin`, REPL: `/plugin` |
| **Superpowers** | Core skills library by Jesse Vincent — TDD, debugging, /brainstorm, git-worktrees | 20+ skills bundled |

## Protocols & Concepts

| Term | Definition | Notes |
|:---|:---|:---|
| **AT Self-Wake** | Desktop Claude schedules AT jobs to self-prompt after delegating | AT = Desktop's alarm, not CLI's doorbell |
| **Flag Files** | Zero-byte files marking task state: *_started, *_completed, *_error, *_cancelled | v9.0 replacement for claim_prefix |
| **Message Inserts** | Structured `<<<INSERT>>>` blocks for cross-platform persistence markers | Primary persistence for Web/iOS |
| **Bootstrap Problem** | Need to know file contents to know when to load them. Solved by glossary | This document! |
| **Bootstrap Hierarchy** | CLI load order: global.md → platform/*.md → role/*/role.yml → tasking.md | ai_general/prompts/ |
| **Reference Pointers** | `REF:path/to/file.yml` syntax for lazy loading, reduced context | protocol_reference_pointers |
| **role.yml** | Role config file with context_files, duties, ownership | ai_general/prompts/roles/{role}/ |
| **Time Loop** | Claude Desktop UI reset phenomenon — response fails, context resets | Desktop only |

## Workflows

| Term | Definition | Notes |
|:---|:---|:---|
| **Condensed Chat History** | Semantic summary created ON-DEMAND, not pre-stored. ~80% reduction | Process: raw → extract decisions → .condensed.yml |
| **Chat Recovery** | Auto-recovery from context exhaustion — detect, export, condense, continue | spec_chat_continuity_recovery |
| **Task Lifecycle** | staged → to_execute → in_progress → completed/error/cancelled | protocol_taskCoordination |
| **Task ID Counter** | Atomic hierarchical ID via mkdir spinlock | ~/bin/ai/next_id.sh |


---

# 6. MCP Servers & Tools Reference

**Version:** 1.0.0  
**Status:** Active  
**Location:** `ai_general/docs/20_registries/`  
**Created:** 2026-02-04  

---

## Overview

MCP (Model Context Protocol) servers extend Desktop Claude's capabilities by providing
structured tool access to filesystem operations, AI execution, agent management, messaging,
and more. Each server exposes a set of tools callable directly from Desktop Claude's context.

## Server Inventory

### Desktop Commander

**Purpose:** Filesystem operations, process management, file search, and system interaction.  
**Type:** Third-party MCP server  
**Used by:** Desktop Claude (primary), Codex MCP (as connector)

| Tool Category | Functions |
|:---|:---|
| File I/O | `read_file`, `read_multiple_files`, `write_file`, `write_pdf`, `edit_block` |
| Directory | `list_directory`, `create_directory`, `move_file`, `get_file_info` |
| Search | `start_search`, `get_more_search_results`, `stop_search`, `list_searches` |
| Processes | `start_process`, `interact_with_process`, `read_process_output`, `force_terminate` |
| Sessions | `list_sessions`, `list_processes`, `kill_process` |
| Config | `get_config`, `set_config_value` |
| Meta | `get_usage_stats`, `get_recent_tool_calls` |


### Codex MCP

**Purpose:** Synchronous AI execution — invoke OpenAI Codex as an internal tool for bounded tasks.  
**Type:** Custom MCP server (codex:)  
**Used by:** Desktop Claude only  
**Timeout:** 30–60 seconds  
**Key distinction:** This is a tool, NOT a worker. Stateless, synchronous, returns results inline.

| Tool | Description |
|:---|:---|
| `codex:codex` | Start a new Codex session with prompt, optional config overrides |
| `codex:codex-reply` | Continue an existing Codex conversation by thread ID |

**Connectors available to Codex:** Desktop Commander, filesystem, shell/processes, web search.

---

### CLI Agent MCP

**Purpose:** Launch and manage CLI agents with role-based bootstrapping.  
**Type:** Custom MCP server (cli-agent:)  
**Source:** `ai_general/apps/mcps/cli-agent/server.py`

| Tool | Description |
|:---|:---|
| `launch_agent` | Generic launcher — specify platform + role |
| `launch_librarian` | Memory system curator |
| `launch_dev_lead` | Development coordinator |
| `launch_custodian` | Repository maintainer |
| `launch_ops` | Task execution coordinator |
| `launch_peer_review` | Code/design reviewer |
| `launch_tester` | Testing and validation |
| `launch_researcher` | Corpus search orchestrator (Claude CLI, coordinates Gemini shards) |
| `launch_validator` | Adversarial cross-checker (must run on different model than synthesizer) |
| `kill` | Terminate a running agent session |
| `attach` | Get command to attach to agent tmux session |
| `send_keys` | Send keystrokes to agent session |
| `list_sessions` | List all active CLI agent sessions |
| `get_status` | Detailed status including recent output |

**Platforms:** claude_cli, codex_cli, gemini_cli  
**Roles:** librarian, dev_lead, custodian, ops, peer_review, tester, researcher, validator

---

### Task Coordination MCP

**Purpose:** Playbook-based orchestration and task lifecycle management.  
**Type:** Custom MCP server (task-coord:)  
**Source:** `ai_general/apps/mcps/task-coord/server.py`

| Tool | Description |
|:---|:---|
| `list_playbooks` | Available orchestration patterns |
| `get_playbook` | Full playbook definition |
| `start_playbook` | Create initial task or run start action |
| `stop_playbook` | Cancel a running playbook |
| `list_templates` | Available task templates |
| `gen_task` | Generate task from template |
| `list_tasks` | List tasks, filter by platform/status |
| `get_task` | Get task content by ID |
| `move_task` | Change task status (staged → to_execute → in_progress → completed) |
| `list_platforms` | Available execution platforms |

---

### Knowledge Search MCP

**Purpose:** Dual-mode search over 4+ year conversation archive (500+ chats, 4700+ chunks).  
**Type:** Custom MCP server (knowledge-search:)

| Tool | Description |
|:---|:---|
| `search` | Topic/keyword search with 4-layer cascade. Modes: SEARCH (curated results) or ANSWER (editorial synthesis with citations) |
| `grep_search` | Regex over full chunk file contents — slower but catches terms not in topic index |
| `stats` | Knowledge base index statistics |

**4-Layer Cascade:** L1 topic index → L2 topics + synonyms → L3 full content → L4 content + synonyms

---

### Chat Pipeline MCP

**Purpose:** Processing pipeline for chat history: export → normalize → chunk → condense.  
**Type:** Custom MCP server (chat-pipeline:)

| Tool | Description |
|:---|:---|
| `pipeline_status` | Counts and status for each pipeline stage |
| `normalize` | Convert raw exports to normalized YAML |
| `chunk_file` | Chunk a specific conversation file |
| `prepare_for_condensation` | Full pipeline: JSON → YAML → chunk → organize into YYYY/MM |
| `condense_history` | Condense a chat history file (async via librarian or sync via Codex) |
| `review_quarantine` | List quarantined files with failure reasons |
| `retry_quarantine` | Retry processing a quarantined file |
| `get_pipeline_config` | Get or update pipeline configuration |

---

### Messages MCP

**Purpose:** Inter-AI messaging — broadcasts and direct messages between agents.  
**Type:** Custom MCP server (messages:)

| Tool | Description |
|:---|:---|
| `broadcast` | Send message to all agents |
| `send_direct` | Send message to specific agent or session |
| `list_broadcasts` | List recent broadcast messages |
| `list_direct` | List direct messages (inbox) for a recipient |
| `check_responses` | Check for responses to a specific message |
| `acknowledge` | Mark a message as read/processed |

---

### Prompting MCP

**Purpose:** Deliver prompts to AI targets with proper timing, busy detection, and verification.  
**Type:** Custom MCP server (prompting:)

| Tool | Description |
|:---|:---|
| `send_prompt` | Send prompt to any AI target (Desktop, Web, CLI, Codex, Gemini, ChatGPT) |
| `is_busy` | Check if AI target is currently processing |
| `list_sessions` | List active tmux sessions |
| `observe_session` | Capture current state of a CLI session |
| `wait_response` | Wait for response pattern in CLI session |
| `send_to_session` | Low-level direct tmux send with timing |

**Targets:** claude-desktop, claude-web, claude-cli, codex-cli, gemini-cli, chatgpt-web, chatgpt-app

---

### Todo MCP

**Purpose:** Todo/task management with kanban views, status tracking, tags, and flags.  
**Type:** Custom MCP server (todo:)

| Tool | Description |
|:---|:---|
| `list` | List todos with optional filtering |
| `get` | Get details by ID, number, or reference code |
| `create` | Create new todo |
| `set_status` | Update status (Triaging → Ready → In_Progress → Done) |
| `add_flag` / `remove_flag` | Manage flags (e.g., high_priority, needs_testing) |
| `add_tag` / `remove_tag` | Manage tags |
| `complete` | Mark done and move to completed/ |
| `trash` | Soft delete |
| `kanban` | Kanban board view (JSON or text) |

---

### Chat Tools MCP

**Purpose:** Chat navigation, export, import, and browser-based chat interaction.  
**Type:** Custom MCP server (chat:)

| Tool | Description |
|:---|:---|
| `open_chat` | Navigate to specific chat by URL or ID |
| `open_project` | Navigate to a Claude project |
| `new_chat` | Open new chat, optionally in project |
| `get_messages` | Extract messages from current chat |
| `send_message` | Type and send message to Claude |
| `wait_response` | Wait for Claude to finish responding |
| `export_chats` | Export chats to JSON via Playwright DOM extraction |
| `import_chat` | Export → convert → condense pipeline |
| `continue_in_new_chat` | Fork current chat with condensed context |
| `attach_file` | Attach file to current chat |
| `detect_active` | Detect which AI chat is active in browser |
| `restart_desktop_claude` | Restart app and reopen current chat |

---

### Chrome Control MCP

**Purpose:** Browser tab management and JavaScript execution.  
**Type:** Third-party MCP server

| Tool | Description |
|:---|:---|
| `open_url` | Open URL in Chrome |
| `get_current_tab` / `list_tabs` | Tab information |
| `switch_to_tab` / `close_tab` / `reload_tab` | Tab management |
| `go_back` / `go_forward` | Browser navigation |
| `execute_javascript` | Run JS in current tab |
| `get_page_content` | Extract page text content |

---

## Tool Usage by Actor

| MCP Server | Desktop Claude | CLI Agents | Codex MCP |
|:---|:---:|:---:|:---:|
| Desktop Commander | ✓ primary | — | ✓ connector |
| Codex MCP | ✓ invokes | — | — |
| CLI Agent | ✓ launches | — | — |
| Task Coordination | ✓ | — | — |
| Knowledge Search | ✓ | ✓ (direct) | — |
| Chat Pipeline | ✓ | — | — |
| Messages | ✓ | ✓ (file-based) | — |
| Prompting | ✓ | — | — |
| Todo | ✓ | — | — |
| Chat Tools | ✓ | — | — |
| Chrome Control | ✓ | — | — |
