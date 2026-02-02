# Library System Architecture

**Version:** 1.0.0  
**Created:** 2026-02-01  
**Maintainer:** PianoMan  
**Status:** Active

## Overview

The Library System provides semantic search across 4+ years of AI conversation history (~50MB, ~14M tokens). It combines a multi-stage processing pipeline with a distributed search architecture using Gemini CLI sessions as corpus holders.

### Design Philosophy

**Problem:** How to search years of conversation history when no single context window can hold it all?

**Solution:** 
1. Process conversations into searchable chunks
2. Index chunks with topic metadata
3. Distribute corpus across multiple Gemini CLI sessions (shards)
4. Orchestrate queries via Claude CLI coordinator

## System Components

```
┌─────────────────────────────────────────────────────────────────┐
│                      INGESTION PIPELINE                         │
│  Raw Export → Normalize → Chunk → Condense → Index → Archive   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      SEARCH ARCHITECTURE                        │
│                                                                 │
│  ┌─────────────┐         ┌──────────────────────────────────┐  │
│  │ knowledge-  │         │     Researcher (Claude CLI)      │  │
│  │ search MCP  │         │     - Shard orchestration        │  │
│  │ - Direct    │         │     - Query analysis             │  │
│  │   topic     │         │     - Result synthesis           │  │
│  │   search    │         └──────────────────────────────────┘  │
│  └─────────────┘                        │                      │
│        │                                ▼                      │
│        │              ┌────────┬────────┬────────┬────────┐   │
│        │              │Shard 01│Shard 02│  ...   │Shard 16│   │
│        │              │Gemini  │Gemini  │        │Gemini  │   │
│        │              │~900K   │~900K   │        │~900K   │   │
│        │              └────────┴────────┴────────┴────────┘   │
│        ▼                                                       │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                   Indexed Corpus                         │  │
│  │  40_histories/  - Chunked conversation files            │  │
│  │  50_shards/     - Shard corpus files (~3.5MB each)      │  │
│  │  topic_index/   - Term → conversation mappings          │  │
│  └─────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## Ingestion Pipeline

### Stage 1: Export

Raw conversation exports from various platforms:
- Claude (claude.ai, Desktop, API)
- ChatGPT
- Gemini

Location: `ai_memories/10_raw_exports/`

### Stage 2: Normalize

Convert platform-specific formats to unified YAML structure:

```yaml
metadata:
  chat_id: "abc123"
  platform: "claude"
  title: "conversation_title"
  created: "2025-10-15T14:30:00Z"
  message_count: 150

messages:
  - role: human
    content: "..."
    timestamp: "..."
  - role: assistant
    content: "..."
    timestamp: "..."
```

Location: `ai_memories/20_normalized/`

### Stage 3: Chunk

Split large conversations into ~4000 token chunks for:
- Manageable processing units
- Parallel condensation
- Granular search results

Chunking rules:
- Preserve message boundaries (never split mid-message)
- Target 4000 tokens per chunk
- Maintain conversation flow context

Location: `ai_memories/30_chunked/`

### Stage 4: Condense

Reduce token count by 70-90% while preserving:
- Decisions and their rationale
- Procedures and workflows discovered
- Technical discoveries and insights
- Outcomes and conclusions

Remove:
- Conversational filler ("Great question!", "Let me think...")
- Failed attempts (summarize to one-liner)
- Redundant explanations
- Process narration

Output format:
```yaml
metadata:
  source_chunk: "chat.001.chunk.yml"
  original_tokens: 4000
  condensed_tokens: 800
  reduction: 80%

summary: |
  One-paragraph overview

decisions:
  - decision: "Use YAML as source of truth"
    rationale: "Token efficiency, machine-readable"
    
discoveries:
  - finding: "Gemini context compaction breaks shard integrity"
    implications: "Must disable for corpus-holding sessions"

procedures:
  - name: "Shard initialization"
    steps: [...]
```

Location: `ai_memories/40_histories/{platform}/{YYYY}/{MM}/`

### Stage 5: Index

Build topic index mapping terms to conversations:

```yaml
topics:
  "context_compaction":
    - chat_id: "abc123"
      chunks: [1, 3]
      relevance: "high"
  "librarian_orchestrator":
    - chat_id: "def456"
      chunks: [2]
      relevance: "medium"
```

The index enables fast lookup without loading full corpus.

Location: `ai_memories/librarian/topic_index/`

### Stage 6: Archive to Shards

Combine condensed histories into shard files:
- Each shard: ~900K tokens (~3.5MB)
- 16 shards cover Jan 2025 - present
- Chronological organization

Location: `ai_memories/50_shards/`

## Search Architecture

### Direct Search: knowledge-search MCP

For quick topic lookups without shard orchestration:

```
knowledge-search:search
  query: "context compaction decision"
  mode: "search"  # or "answer" for synthesis
```

**Capabilities:**
- Topic index lookup (Layer 1)
- Synonym expansion
- Grep fallback for terms not in index
- Returns conversation references with context

**Best for:**
- Quick fact lookups
- Finding specific conversations
- When caller will do their own synthesis

### Semantic Search: Researcher + Shards

For complex queries requiring synthesis across multiple conversations:

```
cli-agent:launch_researcher
  prompt: "How did the memory architecture evolve over 2025?"
```

**Architecture:**

1. **Researcher (Claude CLI)** - Orchestration
   - Analyzes query to determine shard scope
   - Dispatches parallel queries to shards
   - Collects and synthesizes results
   - Writes comprehensive response

2. **Shards (Gemini CLI)** - Corpus Holders
   - Each shard = saved Gemini session with ~900K tokens pre-loaded
   - Semantic search within loaded corpus
   - Returns relevant excerpts with citations

**Why Gemini for Shards:**
- 1M token context window (AI Pro subscription)
- Zero marginal cost for context (unlimited at subscription level)
- Checkpoint/resume allows instant corpus access

### Why Claude CLI for Researcher (Not Gemini)

**The actual decision driver:** Gemini CLI configuration is global, not per-session.

Shards require **context compaction disabled** to preserve full corpus integrity. If compaction runs, Gemini silently drops older content, corrupting the shard.

The Researcher/Orchestrator role **wants compaction enabled** - orchestration conversations can be long, and we want Gemini's intelligent summarization.

**The constraint:** Can't have different compaction settings for different Gemini sessions.

**The solution:** Use a different CLI platform for orchestration.

Claude CLI became the Researcher platform by elimination:
- Has Desktop Commander MCP for spawning shard processes
- Capable tool use and orchestration
- Compaction settings independent of Gemini

*Note: The documentation often frames this as "Claude excels at orchestration, Gemini excels at corpus hosting" - which is true but is post-hoc rationalization. The actual forcing function was the configuration limitation.*

## Shard Details

### Current Shard Inventory

| Shard | Date Range | Files | ~Tokens |
|-------|------------|-------|---------|
| shard-01 | Jan 01 - Apr 04 | 73 | 891K |
| shard-02 | Apr 05 - May 11 | 45 | 748K |
| shard-03 | May 12 - May 30 | 39 | 864K |
| shard-04 | Jun 01 - Jun 11 | 29 | 824K |
| shard-05 | Jun 12 - Jul 10 | 75 | 873K |
| shard-06 | Jul 11 - Aug 27 | 96 | 836K |
| shard-07 | Aug 28 - Oct 05 | 90 | 827K |
| shard-08 | Oct 06 - Oct 11 | 45 | 803K |
| shard-09 | Oct 12 - Oct 19 | 49 | 798K |
| shard-10 | Oct 20 - Oct 26 | 67 | 878K |
| shard-11 | Oct 27 - Nov 03 | 102 | 821K |
| shard-12 | Nov 04 - Nov 14 | 94 | 893K |
| shard-13 | Nov 15 - Nov 22 | 82 | 854K |
| shard-14 | Nov 23 - Dec 03 | 45 | 827K |
| shard-15 | Dec 04 - Dec 13 | 45 | 826K |
| shard-16 | Dec 14 - Dec 15 | 12 | 154K |

### Shard Operations

**Initialize shard (one-time):**
```bash
gemini_cli.py -a -i \
  --files ai_memories/50_shards/shard-NN_YYYYMMDD-YYYYMMDD.yml \
  --save shard-NN
```

**Query shard:**
```bash
gemini_cli.py -a --sync --resume shard-NN "search query"
```

**Critical setting:** Context compaction must be OFF for shard sessions.

### Shard Maintenance

When new conversations accumulate:
1. Process through pipeline (export → condense)
2. Add to appropriate shard or create new shard
3. Re-initialize affected shard session

## Query Flow

### Simple Query (knowledge-search)

```
User: "Find discussions about task coordination"
  │
  ▼
knowledge-search:search
  │
  ├─► Topic index lookup
  │     └─► Returns matching chat_ids + chunks
  │
  └─► Response: List of conversations with excerpts
```

### Complex Query (Researcher)

```
User: "How did our approach to AI coordination evolve?"
  │
  ▼
cli-agent:launch_researcher
  │
  ├─► Task factory creates req_NNNN_query/
  │     ├─► instructions.md (researcher role)
  │     └─► task file (query + metadata)
  │
  ├─► Claude CLI launches with task
  │
  ├─► Researcher analyzes query
  │     └─► Determines: evolution query → ALL shards
  │
  ├─► Dispatches to shards in parallel
  │     ├─► shard-01: "AI coordination evolution"
  │     ├─► shard-02: "AI coordination evolution"
  │     └─► ... (all 16)
  │
  ├─► Collects results
  │     └─► Reads stdout + result files from each
  │
  ├─► Synthesizes
  │     ├─► Deduplicates
  │     ├─► Builds timeline
  │     └─► Constructs narrative
  │
  └─► Writes response to task directory
```

## Key Paths

| Purpose | Path |
|---------|------|
| Raw exports | `ai_memories/10_raw_exports/` |
| Normalized | `ai_memories/20_normalized/` |
| Chunked | `ai_memories/30_chunked/` |
| Condensed histories | `ai_memories/40_histories/` |
| Shard corpus files | `ai_memories/50_shards/` |
| Topic index | `ai_memories/librarian/topic_index/` |
| Shard manifest | `ai_memories/librarian/shard_manifest.yml` |
| Researcher role | `ai_general/roles/researcher/` |
| knowledge-search MCP | `ai_general/apps/mcps/knowledge-search/` |

## Cost Model

**Gemini AI Pro subscription:**
- ~100 requests/day budget
- Each shard query = 1 request
- Simple query (4 shards) = 4 requests
- Comprehensive query (16 shards) = 16 requests

**Storage:**
- Raw exports: ~200MB
- Condensed: ~50MB (75% reduction)
- Shards: ~56MB (16 × 3.5MB)

## Limitations

1. **Latency:** Shard queries take 30-90 seconds depending on scope
2. **Recent gap:** ~2-4 week lag between conversation and searchability
3. **Single-config Gemini:** Can't mix compaction settings across sessions
4. **Shard boundaries:** Conversations spanning shard boundaries may have fragmented context

## Future Considerations

- **Incremental shard updates:** Add to shards without full re-initialization
- **Real-time indexing:** Reduce lag between conversation and searchability
- **Cross-shard context:** Better handling of conversations at shard boundaries
- **Alternative corpus holders:** Evaluate other 1M+ context platforms as they emerge

## Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-02-01 | Initial architecture documentation |
