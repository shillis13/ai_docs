# Task File Schema v2.0

## Metadata

- **name:** Task File Schema
- **version:** 2.0.0
- **date:** 2025-12-11
- **status:** active
- **description:** Schema for req_XXXX_slug.md task definition files
- **used_by:** protocol_taskCoordination_latest.yml
- **changes_from_v1:**
  - Added session field for persistent agent assignment
  - Added bundle support fields

## File Naming

**Pattern:** `req_XXXX_slug.md`

| Component | Description |
|-----------|-------------|
| `req` | Literal prefix |
| `XXXX` | 4-digit zero-padded task ID (e.g., 1001, 1002) |
| `slug` | Kebab-case task description (e.g., system-info, yaml-review) |
| `.md` | Extension (Markdown) |

**Examples:**
- `req_1001_system_info.md`
- `req_1019_yaml_md_review.md`
- `req_2100_parallel_installation.md`

## Folder Naming

**Pattern:** `req_XXXX_slug/`

- Matches task filename without extension
- **Claimed pattern:** `claimed_{timestamp}_{pid}_req_XXXX_slug/`
- **Timestamp format:** `YYYYMMDD_HHMMSS`

## Task ID Allocation

- **Source file:** `_NNNN_next_req_id`
- **Location:** Coordination root directory
- **Format:** Plain text file containing next available ID
- **Increment:** After successful task creation

---

## Required Fields

### Title

- **Format:** `# Request #XXXX: Title`
- **Description:** Markdown H1 header with request ID and descriptive title
- **Validation:** Must match req_XXXX pattern in filename
- **Example:** `# Request #1001: System Information Report`

### Type

- **Format:** `**Type:** {value}`
- **Default:** Standard
- **Allowed values:**

| Value | Description |
|-------|-------------|
| Standard | Normal task execution |
| Research | Information gathering, analysis |
| Installation | Software/tool installation |
| Testing | Test execution, validation |
| Emergency | Urgent, bypass normal queue |
| Maintenance | Cleanup, organization, updates |

### Priority

- **Format:** `**Priority:** {value}`
- **Default:** NORMAL
- **Allowed values:**

| Value | Description |
|-------|-------------|
| LOW | Can wait, do when convenient |
| NORMAL | Standard priority (default) |
| HIGH | Should be done soon |
| URGENT | Do next, interrupt lower priority |
| CRITICAL | Drop everything, do now |

### Posted

- **Format:** `**Posted:** {ISO 8601 timestamp}`
- **Description:** When the task was created
- **Example:** `**Posted:** 2025-11-23T15:30:00-06:00`

### Description

- **Format:** `## Task Description\n{markdown content}`
- **Description:** What needs to be done, context, requirements
- **Minimum length:** 1 sentence

### Expected Response

- **Format:** `## Expected Response\n{description}`
- **Description:** What the Worker should return/produce
- **Purpose:** Sets clear completion criteria

---

## Optional Fields

### Commands

- **Format:** `## Task Commands\n```bash\n{commands}\n```- **Description:** Specific bash commands to execute
- **When omitted:** Worker determines appropriate commands

### Discrete

- **Format:** `**Discrete:** true|false`
- **Description:** If true, task should not be subdivided
- **Default:** false (subdivision allowed)

### Target Worker

- **Format:** `**Target Worker:** {worker_type}` or `{worker_type}:{pid}`
- **Description:** Intended Worker type or specific instance
- **Examples:**
  - `**Target Worker:** claude_cli`
  - `**Target Worker:** codex_cli:12345`
- **When omitted:** Any available Worker may claim

### Session (NEW in v2.0)

- **Format:** `**Session:** {session_name}` or `{session_id}`
- **Description:** Persistent session for agent expertise accumulation
- **Purpose:** Specifies which persistent agent session should handle this task. Enables expertise accumulation across multiple task executions.

**Value types:**

| Type | Description |
|------|-------------|
| session_name | Human-readable name from session_registry.yml (e.g., `librarian-001`) |
| session_id | UUID for the session |

**Examples:**
- `**Session:** librarian-001`
- `**Session:** custodian-001`
- `**Session:** reviewer-001`
- `**Session:** 257c13b3-c315-479e-86fc-eed05cb90a0d`

**Registry:** `~/.claude/session_registry.yml`

**When omitted:** Task runs in ephemeral session (fresh context)

**Launch pattern:** `claude --project {project} --session-id {session} "{prompt}"`

**Use cases:**

| Context | When to use |
|---------|-------------|
| Persistent | Role-based agents (Librarian, Custodian, Reviewer) |
| Persistent | Long-running projects requiring context continuity |
| Persistent | Tasks that build on prior work in same domain |
| Ephemeral | One-off tasks |
| Ephemeral | Parallel workers on same task type |
| Ephemeral | Experiments or explorations |
| Ephemeral | When fresh perspective is valuable |

### Entry Criteria

- **Format:** `## Entry Criteria\n{list of conditions}`
- **Description:** Prerequisites that must be met before starting
- **When omitted:** Task can start immediately when claimed

### Exit Criteria

- **Format:** `## Exit Criteria\n{list of conditions}`
- **Description:** Conditions that define successful completion
- **When omitted:** Worker judgment determines completion

### Depends On

- **Format:** `**Depends On:** req_XXXX, req_YYYY`
- **Description:** Task IDs that must complete before this task starts
- **Status:** WALK STAGE - for Task Sets

### Sequence ID

- **Format:** `**Sequence:** {NNN}`
- **Description:** Order within a Task Set (001, 002, etc.)
- **Status:** WALK STAGE - for Task Sets

### Notes

- **Format:** `## Notes\n{additional context}`
- **Description:** Any additional information for the Worker
- **Examples:** Background context, related tasks, known issues to watch for

---

## Template

```markdown
# Request #XXXX: {Title}

**Type:** Standard
**Priority:** NORMAL
**Posted:** {ISO 8601 timestamp}
**Session:** {session_name}  # Optional: omit for ephemeral task

## Task Description

{Describe what needs to be done. Include context, requirements, and any
constraints the Worker should know about.}

## Task Commands

```bash
# Commands to execute (optional - omit section if Worker should decide)
{command_1}
{command_2}
```

## Expected Response

{Describe what the Worker should return. Be specific about format,
content, and any files that should be created.}

## Notes

{Optional: Additional context, related tasks, known issues.}
```

---

## Validation Rules

| Rule | Description | Severity |
|------|-------------|----------|
| filename_matches_title | XXXX in filename must match #XXXX in title | error |
| required_sections_present | Title, Type, Priority, Posted, Description, Expected Response must exist | error |
| valid_type | Type must be one of allowed_values | error |
| valid_priority | Priority must be one of allowed_values | error |
| valid_timestamp | Posted must be valid ISO 8601 format | error |
| description_not_empty | Task Description section must have content | error |
| expected_response_not_empty | Expected Response section must have content | warning |

---

## Examples

### Minimal Task

**Filename:** `req_1001_system_info.md`

```markdown
# Request #1001: System Information Report

**Type:** Standard
**Priority:** NORMAL
**Posted:** 2025-11-23T15:30:00-06:00

## Task Description

Gather basic system information for documentation purposes.

## Task Commands

```bash
uname -a
df -h
```

## Expected Response

System details in markdown format with command outputs.
```

### Full-Featured Task

**Filename:** `req_1019_yaml_md_review.md`

```markdown
# Request #1019: YAML to Markdown Review

**Type:** Research
**Priority:** HIGH
**Posted:** 2025-11-03T10:00:00-06:00
**Target Worker:** claude_cli
**Session:** reviewer-001
**Discrete:** true

## Entry Criteria

- All YAML files in target directory exist
- Previous migration task (req_1018) completed

## Task Description

Review all YAML files that were converted from Markdown as part of
the documentation migration. Verify:
1. YAML syntax is valid
2. All content from original MD preserved
3. Structure follows schema conventions

## Task Commands

```bash
find ~/Documents/AI/ai_general/instructions -name "*.yml" -exec yamllint {} \;
```

## Expected Response

Summary report listing:
- Files reviewed
- Validation results per file
- Any issues found with severity
- Recommended fixes

## Exit Criteria

- All files validated
- No critical errors remaining
- Report written to responses/

## Notes

Related to req_1018 (YAML migration) and req_1020 (documentation update).
If issues found, create follow-up tasks for fixes.
```

---

## References

- **Protocol:** `protocol_taskCoordination_latest.yml`
- **Response schema:** See protocol `file_formats.response_file` section
- **Error schema:** See protocol `file_formats.error_file` section
