# Reference Pointer Loading Instructions

**Version:** 1.0.0
**Created:** 2025-12-05
**Maintainer:** PianoMan
**Target:** Any
**Status:** active

## Related Documents

- Protocol: `../../30_protocols/protocol_reference_pointers_latest.yml`
- Schema: `../../50_schemas/schema_pointer_syntax_latest.yml`

## Recognition

### Pattern

Memory slots matching format: `<TYPE>:<PATH_OR_QUERY> [| <KEY>:<VALUE>]*`

### Types to Recognize

- STARTER, PROJECT, PREFS, CONTEXT, TOOL, REF (PATH-based)
- QUERY (Search-based)
- RECENT (Category-based)

**Action:** Parse each pointer, categorize by type and load behavior

## Load Sequence

### 1. Conversation Start

- **Condition:** PRIORITY:1 AND LOAD:AUTO
- **Action:** Load via Desktop Commander immediately
- **Silent:** true
- **Max Files:** 5
- **Rationale:** Don't announce loading - just have the context ready

### 2. Topic Triggered

- **Condition:** PRIORITY:2 AND LOAD:TOPIC AND topic_matches
- **Action:** Load when conversation enters relevant domain
- **Announce:** optional, brief if at all
- **Matching:** Use tags field in destination file for topic matching

### 3. On Demand

- **Condition:** PRIORITY:3 OR LOAD:DEMAND
- **Action:** Load only when explicitly needed
- **Announce:** yes - explain why loading
- **Example:** "Let me check the coordination protocol spec..."

## Execution Methods

### PATH Pointers

- **Tool:** Desktop Commander read_file
- **Path Base:** ~/Documents/AI/ai_root/
- **Example:** `view ai_memories/60_knowledge/about_user/identity.yml`

### QUERY Pointers

- **Tool:** memory_search (when available) or Desktop Commander start_search
- **Fallback:** ripgrep-based search of ai_memories/
- **Example:** `start_search path=ai_memories pattern="CLI timeout" searchType=content`

### RECENT Pointers

- **Tool:** Desktop Commander list_directory + read_file
- **Method:** List category directory, sort by date, read top N
- **Categories:**
  - session_summaries: ai_claude/work/session_summaries/
  - task_completions: ai_comms/claude_cli/completed/
  - decisions: ai_memories/60_decisions/

## Pointer Chaining

**When:** Destination file contains 'pointers' section

**Behavior:** Treat nested pointers same as memory slot pointers. Follow based on rel type and load hint.

**Depth Limit:** 3

**Circular Prevention:** Track loaded paths in working memory during session. Skip already-loaded files. Warn if circular reference detected.

## Context Conservation

**Principle:** Load what's needed, not everything available

**Max Auto Load:** 5 files at conversation start

**Prefer Summaries:** Use header.summary when full content not needed. Load full content only when detail required.

**Token Awareness:** Each loaded file consumes context. Prefer small, focused destination files. Large files should use PRIORITY:3 / LOAD:DEMAND.

## Error Handling

### File Not Found

- **Action:** Note in thinking, continue without
- **Announce:** false

### Parse Error

- **Action:** Log issue, skip file, suggest user review
- **Announce:** briefly if relevant

### Search No Results

- **Action:** Note no matches found, continue
- **Announce:** only if user explicitly asked for information

## Integration with Existing Behavior

### Native Memory Content

- **Status:** Coexists during transition
- **Priority:** Pointers take precedence over raw content
- **Migration:** Gradually replace content with pointers

### Project Files

- **Status:** Separate system - still loads normally
- **Relationship:** Pointers can reference project file locations

### User Preferences

- **Status:** Separate system - handled by Anthropic
- **Relationship:** Pointers supplement with deeper context

## Practical Examples

### Conversation Start Flow

1. Receive memory slots including:
   - `STARTER:ai_memories/60_knowledge/about_user/identity.yml | PRIORITY:1 | LOAD:AUTO`
   - `PROJECT:ai_claude/work/current_focus.yml | PRIORITY:1 | LOAD:AUTO`
   - `REF:ai_general/docs/40_specs/cli_coordination.yml | PRIORITY:3 | LOAD:DEMAND`

2. Immediately load identity.yml and current_focus.yml (silent)

3. Now know: User is PianoMan, 25+ year dev, working on memory system restructure

4. REF pointer noted but not loaded until CLI coordination discussed

### Topic Triggered Flow

1. User says: "Let's work on the chat orchestrator"

2. Detect topic matches PROJECT pointer tags

3. Load: `PROJECT:ai_claude/work/orchestrator_state.yml | LOAD:TOPIC`

4. Now have current orchestrator status, recent issues, next steps

### On Demand Flow

1. User asks: "What's the exact format for CLI task files?"

2. Recognize need for spec detail

3. Announce: "Let me check the coordination protocol spec..."

4. Load: `REF:ai_general/docs/40_specs/cli_coordination.yml`

5. Provide specific answer from loaded content

## Do Not

- Load all pointers at once
- Announce every file load
- Follow pointer chains deeper than 3 levels
- Re-load already loaded files in same session
- Guess at file contents if load fails
