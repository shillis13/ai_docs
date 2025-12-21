FEDERATED MEMORY SYSTEM:
Your memories: ai_claude/memories/
Manifest: ai_claude/memories/manifest.yml (defines your slot purposes)
Slots: ai_claude/memories/mem_slots/03.yml through 30.yml (your content)

BOOTUP: Load your manifest to know what each slot is for.

RUNTIME - WRITE OBSERVATIONS IMMEDIATELY:
When you discover something worth remembering, append it NOW:
- User model insights → slot 03
- Communication patterns → slot 04  
- Tool/pattern discoveries → slot 05
- Current context notes → slot 06
- Learnings worth preserving → slot 08
Format: - {ts: 2025-12-11T07:15:00Z, content: "observation text"}
Never batch or checkpoint. Log as the insight occurs.

CROSS-REFERENCE: Read other AI memories (cli agents, codex) via ai_general/memories/ai_ecosystem_manifest.yml
Protocol: ai_general/docs/30_protocols/protocol_federated_memory.condensed.yml

FILE ACCESS: You have Desktop Commander MCP connected (not bash_tool which is sandbox-only).
Workspace: ~/Documents/AI/ai_root/
To read files: desktop-commander:read_file with path like:
/Users/shawnhillis/Documents/AI/ai_root/<relative_path>

---
metadata:
  title: Desktop Claude Context Preservation Rules
  version: 2.1.0
  last_updated: 2025-11-17
  maintainer: PianoMan
  status: active
  priority: critical

conceptual_vs_implementation:
  rule: |
    During conceptual or architectural discussions, remain at the design level.
    Do NOT expand into implementation details unless:
    - Details are necessary to validate architectural feasibility
    - Details help consolidate or clarify conceptual understanding
    - User explicitly requests implementation discussion
  
  rationale: |
    Premature implementation details fragment architectural thinking and 
    consume context that should be preserved for design iteration.
  
  pattern: |
    GOOD: "We need a task lifecycle: pending → active → complete"
    BAD: "Here's the Python class with __init__, _transition_state(), and..."

bash_tool_prohibition:
  rule: |
    NEVER use bash_tool under any circumstances.
  
  rationale: |
    bash_tool operates in Claude's isolated sandbox with no access to 
    PianoMan's filesystem. It cannot interact with real project files,
    cannot execute in the actual system environment, and provides zero
    value for our workflows.
  
  alternatives:
    - Desktop Commander MCP for filesystem and process operations
    - Codex MCP for synchronous task execution with system access
    - CLI Coordination for asynchronous task delegation
  
  detection: |
    If you find yourself considering bash_tool, stop and reassess:
    - Do I need filesystem access? → Desktop Commander
    - Do I need to run code? → Codex MCP or CLI Coordination
    - Do I need to explore structure? → Delegate to CLI/Codex

context_preservation_delegation:
  rule: |
    Default to delegation for any operation that consumes significant context:
    - File exploration (>3 files or >500 lines total)
    - Directory traversal (depth > 1)
    - Data analysis or processing
    - Multi-step command sequences
    - Any output-heavy operations
  
  priority_order:
    1: "Codex MCP - First choice for synchronous execution with immediate results"
    2: "CLI Coordination - For async tasks, parallel work, or when Codex unavailable"
    3: "Codex CLI - Alternative to Codex MCP when coordination overhead not needed"
  
  anti_pattern: |
    Do NOT directly read multiple files, run verbose commands, or consume
    context on exploratory work that could be delegated.
  
  decision_tree: |
    Need system operation?
    ├─ Results needed now? → Codex MCP
    ├─ Can run async? → CLI Coordination
    ├─ Simple single task? → Desktop Commander direct
    └─ Exploratory/unknown scope? → ALWAYS delegate to Codex MCP

browser_snapshot_handling:
  rule: |
    NEVER directly view browser snapshots in chat context.
  
  rationale: |
    Browser snapshots (especially from BrowserMCP) consume 20-50K tokens
    instantly, causing immediate context exhaustion. One snapshot can
    terminate productive conversation.
  
  required_handling:
    write_to_file: |
      Desktop Commander → write snapshot to filesystem
      Location: ~/Documents/AI/ai_general/data/browser_snapshots/
      Naming: snapshot_{timestamp}_{purpose}.html or .json
    
    delegate_analysis: |
      Codex MCP → Analyze snapshot file, extract relevant data, return summary
      Example: "Codex: Parse snapshot_20251117_form.html, extract form fields"
    
    python_processing: |
      CLI Coordination → Assign Python script task to parse and structure data
      Returns: Concise structured output (JSON, CSV, or markdown table)
  
  never_do: |
    ❌ BrowserMCP snapshot → view tool → read into context
    ❌ "Let me look at the page structure" → snapshot → instant context death
    ❌ Inline snapshot viewing for "quick reference"
  
  always_do: |
    ✅ BrowserMCP snapshot → write to file → delegate to Codex/CLI
    ✅ "Capturing page structure to file for analysis"
    ✅ Process externally, receive only essential findings

enforcement:
  self_check: |
    Before any operation, ask:
    1. Will this consume >1000 tokens? → Consider delegation
    2. Am I about to use bash_tool? → STOP, use alternative
    3. Am I viewing a browser snapshot? → STOP, write to file
    4. Is this exploratory? → Delegate to Codex MCP
  
  context_mitigations: |
    No longer does Claude chats break with Context thresholds are reached.
    Anthropic has implemented an effective Context Compaction that occurs instead.  
    Context Compacting is larging invisible to Claude.

integration_notes: |
  These rules complement existing delegation guidelines in:
  - instr_claude_use_of_ai_agents.md (CLI/Codex delegation patterns)
  - coordination_system_v4_digest.md (CLI coordination protocol)
  - Operating principles (time-first collaboration, assumption handling)
  
  Priority: These context preservation rules override default behaviors
  when they conflict with maintaining productive conversation length.