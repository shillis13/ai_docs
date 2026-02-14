# Library System Architecture

**Version:** 1.1.0  
**Created:** 2026-02-01  
**Updated:** 2026-02-11
**Maintainer:** PianoMan  
**Status:** Active

## Overview

The Library System provides semantic search across 4+ years of AI conversation history (~50MB, ~4.7M tokens). It combines a multi-stage processing pipeline with a distributed search architecture using Gemini CLI sessions as corpus holders and a cross-model validation layer for synthesis quality.

### Design Philosophy

**Problem:** How to search years of conversation history when no single context window can hold it all? And critically: how to find *emergent patterns* across the corpus — patterns that can't be expressed as search terms because you don't know they exist until you find them?

**Solution:** 
1. Process conversations into searchable chunks
2. Index chunks with topic metadata
3. Distribute corpus across multiple Gemini CLI sessions (shards)
4. Orchestrate queries via Claude CLI coordinator
5. Validate synthesized results via cross-model adversarial review

### Why Shards Over Preprocessing

The roundtable explored many alternatives: interaction signatures, lens summaries, event stores, constraint compilers, specialist agents, stratified sampling, map-reduce, recursive summarization.

**Where every alternative hit a wall:**
- Preprocessing → schema lock-in (can't query what you didn't anticipate)
- Filter-then-reason → fails for emergent patterns (no anchor to filter on)
- Local LLMs → RAM-bound at this corpus size
- JIT extraction → still needs cheap pre-filter, which we don't have

**What shards uniquely provide:**
1. Zero preprocessing lock-in — raw corpus, query anything
2. Full coverage for emergent patterns — no filtering assumption
3. Handles "find interesting stuff" — the hardest query class
4. Zero marginal cost (subscription)
5. Simple architecture — saved chats + synthesis

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
│        │              │Shard 01│Shard 02│  ...   │Shard 06│   │
│        │              │Gemini  │Gemini  │        │Gemini  │   │
│        │              │~800K   │~800K   │        │~800K   │   │
│        │              └────────┴────────┴────────┴────────┘   │
│        │                                │                      │
│        │                                ▼                      │
│        │              ┌──────────────────────────────────┐    │
│        │              │     Validator (ChatGPT CLI)      │    │
│        │              │     - Adversarial I&T            │    │
│        │              │     - Evidence verification      │    │
│        │              │     - Fabrication detection      │    │
│        │              └──────────────────────────────────┘    │
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
- Each shard: ~800K tokens (~3.5MB)
- ~6 shards cover Jan 2025 - present
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

### Semantic Search: Researcher + Shards + Validator

For complex queries requiring synthesis across multiple conversations:

```
cli-agent:launch_researcher
  prompt: "How did the memory architecture evolve over 2025?"
```

**Architecture:**

1. **Researcher (Claude CLI)** - Orchestration & Synthesis
   - Analyzes query to determine shard scope
   - Dispatches parallel queries to shards
   - Collects and synthesizes results
   - Writes comprehensive response

2. **Shards (Gemini CLI)** - Corpus Holders
   - Each shard = saved Gemini session with ~800K tokens pre-loaded
   - Semantic search within loaded corpus
   - Returns relevant excerpts with structured evidence

3. **Validator (ChatGPT CLI)** - Adversarial I&T
   - Reviews synthesizer claims against shard evidence
   - Checks for fabrication, unsupported assertions
   - Forces revisions or produces structured disagreement

### Why This Model Distribution

**Gemini for Shards:**
- 1M token context window (AI Pro subscription)
- Zero marginal cost for context (unlimited at subscription level)
- Checkpoint/resume allows instant corpus access
- LITM mitigated within 1M limit (see Research Basis below)

**Claude for Researcher/Synthesizer:**
- Gemini CLI config is global, not per-session
- Shards need compaction OFF; orchestration wants it ON
- Can't have different compaction settings for different Gemini sessions
- Claude provides independent compaction settings

**ChatGPT for Validator:**
- Cross-model diversity reduces correlated errors
- Put fabrication-prone model in reductive role
- Validator's job (does this claim have support?) has minimal fabrication surface
- Can't fabricate a challenge

## Validator Architecture

### Core Design

The validator implements adversarial Integration & Test (I&T) pressure on synthesized results. This mirrors software engineering practice: the person who writes code shouldn't be the only one who tests it.

**Key insight:** Put each model in the role where its weaknesses cause least damage.
- Synthesizer role: creative, generative → some fabrication tolerance
- Validator role: reductive, judgmental → minimal fabrication surface

### Structured Output Contract

Shards must return structured evidence, not prose:

```yaml
matches:
  - match_id: "shard-03:chunk-147:msg-12"
    quote: "exact text from source"
    context: "surrounding context"
    timestamp: "2025-07-15"
    relevance: "high"
    
coverage:
  method: "full_scan"  # or "sampled"
  chunks_examined: 847
  chunks_with_matches: 23
  
uncertainty:
  gaps: ["period 2025-03-01 to 2025-03-15 sparse"]
  confidence: "high"
```

**Why this matters:**
- Every claim traceable to specific source
- Counting discipline (not "many instances" but "23 instances")
- Stable evidence addressing for validator cross-reference
- Temporal gap detection

### Validator Contract

Validator receives:
- Synthesizer's claims
- All shard evidence (with match_ids)

Validator produces:
```yaml
verdicts:
  - claim: "Memory architecture went through 3 major revisions"
    verdict: grounded  # or unsupported, partial
    evidence: ["shard-02:chunk-89:msg-5", "shard-04:chunk-201:msg-3", ...]
    notes: "Supported by explicit version declarations"
    
  - claim: "The shift to federation was driven by performance concerns"
    verdict: unsupported
    notes: "No evidence for 'performance' motivation found"
    
temporal_check:
  fabricated_gaps: false
  note: "Timeline consistent with evidence"
```

**Critical constraints:**
- Validator only judges, never rewrites
- Every claim must link to specific match_ids
- Fails loud if can't access sources
- No creative license

### Bounce-Back Protocol

When validator finds issues:

1. **Round 1:** Synthesizer produces initial output
2. **Validator:** Flags unsupported/partial claims
3. **Round 2:** Synthesizer must produce evidence OR retract (not just fold)
4. **Validator:** Re-evaluates
5. **Round 3 (max):** Final attempt

**Deadlock handling:**
- Script owns round counter, not agents
- Agreement = all claims either `grounded` or `retracted`
- After 3 bounces without agreement → structured disagreement report
- No voting, no compromise — document the dispute

### Generic Validator Role

This pattern applies beyond corpus search:
- Chat history condensation (did condensed version preserve key decisions?)
- Knowledge extraction (are extracted patterns grounded in source?)
- Cross-AI digest federation (are shared memories accurate?)
- Any transform from source → derived output

## Shard Details

### Research Basis: Lost in the Middle

**Original LITM findings (2023):**
- U-shaped attention: models lose accuracy for info in middle of context
- 30%+ degradation when critical info moves from edges to middle
- Appeared with 10-20 documents

**Recent findings (2024-2025):**
- Llama-3.1-405b degrades after 32K tokens
- GPT-4-0125-preview degrades after 64K tokens
- Effective capacity typically 60-70% of advertised max
- Claude 4 Sonnet: <5% degradation across full 200K range

**Gemini 2.5 Flash (2025):**
- LITM "substantially mitigated or eliminated" for direct Q&A
- Perfect accuracy on needle-in-haystack at all positions within 1M limit
- Attributed to: ALiBi positional encoding, training curriculum changes
- Caveat: "more complex forms of failure remain" (multimodal, multi-needle)

**Decision:** 800K tokens per shard (conservative margin within 1M)
- 4.7M ÷ 800K = ~6 shards
- Simpler orchestration than 16 × 500K

### Critical Shard Properties

1. **Pristine per query:** Saved chat state used for each query — no drift
2. **Zero compression:** Configured to never compact — no loss
3. **No re-loading:** Context persists in saved chats, not re-injected per query
4. **Static historical:** Only newest shard updates; older shards immutable

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
2. Add to current shard until ~800K
3. When current shard full, create new shard
4. Initialize new shard session
5. Older shards remain untouched (static historical slices)

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

### Complex Query (Researcher + Validator)

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
  │     └─► ... (all 6)
  │
  ├─► Shards return structured evidence
  │     └─► match_id, quote, context, timestamp
  │
  ├─► Synthesizer builds narrative from evidence
  │     └─► Every claim links to match_ids
  │
  ├─► Validator reviews synthesis
  │     ├─► Checks each claim against evidence
  │     ├─► Returns verdicts (grounded/unsupported/partial)
  │     └─► Flags temporal inconsistencies
  │
  ├─► Bounce-back if needed (max 3 rounds)
  │
  └─► Final output with grounded claims only
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
| Validator role | `ai_general/roles/validator/` |
| knowledge-search MCP | `ai_general/apps/mcps/knowledge-search/` |

## Cost Model

**Gemini AI Pro subscription:**
- ~100 requests/day budget
- Each shard query = 1 request
- Full query (6 shards) = 6 requests
- With validation: +2-4 requests for bounce-backs

**ChatGPT (Validator):**
- Pay-per-use or subscription
- 1-3 calls per validated query

**Storage:**
- Raw exports: ~200MB
- Condensed: ~50MB (75% reduction)
- Shards: ~21MB (6 × 3.5MB)

## Limitations

1. **Latency:** Full validated query takes 60-180 seconds
2. **Recent gap:** ~2-4 week lag between conversation and searchability
3. **Single-config Gemini:** Can't mix compaction settings across sessions
4. **Shard boundaries:** Conversations spanning shard boundaries may have fragmented context
5. **Pattern-finding uncertainty:** LITM research focused on single-fact retrieval; multi-instance pattern queries may behave differently

## Future Considerations

- **Incremental shard updates:** Add to shards without full re-initialization
- **Real-time indexing:** Reduce lag between conversation and searchability
- **Cross-shard context:** Better handling of conversations at shard boundaries
- **Hook-based enforcement:** Gemini CLI hooks to enforce structured output contract
- **Evidence dedup index:** Thin local layer mapping quote hashes to source locations

## Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-02-01 | Initial architecture documentation |
| 1.1.0 | 2026-02-11 | Added validator architecture, updated shard sizing (800K/6 shards), LITM research basis, structured output contract, bounce-back protocol |
