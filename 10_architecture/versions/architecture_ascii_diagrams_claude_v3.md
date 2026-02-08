# Architecture ASCII Diagrams v3 — Claude

## Diagram 1: Primary Coordination Flow

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
