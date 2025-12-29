# Task Template: Condensation with Codex Review

**Version:** 2.1.0  
**Purpose:** Batch-capable condensation task template with iterative Codex validation

---

## Template

```yaml
---
target_worker: {{TARGET_WORKER}}
type: condensation
priority: {{PRIORITY}}
created: "{{CREATED_TIMESTAMP}}"
---

# Batch Condensation Task

## Parameters
- Token reduction: ≥{{TOKEN_REDUCTION_MIN}}% (target {{TOKEN_REDUCTION_TARGET}}%)
- Max iterations per file: {{MAX_ITERATIONS}}
- Data loss tolerance: ≤{{DATA_LOSS_TOLERANCE}}%
- Reviewer: {{REVIEWER}}

## Files to Condense

{{FILES_LIST}}

## Workflow (per file)

1. Read source, count words: `wc -w "<source>"`
2. Create `.condensed.yml` preserving all semantic meaning
3. Submit to {{REVIEWER}}:
   ```
   {{REVIEWER}} "Review <target> against <source>.
   Check: (1) DATA_LOSS (2) REDUCTION opportunities (3) ACCURACY
   Verdict: ACCEPT | REVISE_FOR_LOSS | REVISE_FOR_REDUCTION | REJECT"
   ```
4. On REVISE: apply feedback, re-submit (max {{MAX_ITERATIONS}}x)
5. On ACCEPT: verify/create symlink
6. Log completion metrics

## Execution Strategy

{{EXECUTION_STRATEGY}}

## Progress Tracking

After each file, append to `progress.yml` in task directory:
```yaml
- file: <source_filename>
  started: <timestamp>
  completed: <timestamp>
  source_words: <n>
  condensed_words: <n>
  reduction_pct: <n>%
  iterations: <n>
  verdict: ACCEPT
```

## Completion

When all files processed, write `completion.md`:
```yaml
completed: "<timestamp>"
files_processed: <n>
total_source_words: <n>
total_condensed_words: <n>
avg_reduction_pct: <n>%
avg_iterations: <n>
all_accepted: true|false
```
```

---

## Variable Reference

| Variable | Description | Default |
|----------|-------------|---------|
| `{{TARGET_WORKER}}` | Agent to claim task | `librarian` |
| `{{PRIORITY}}` | Execution priority | `medium` |
| `{{CREATED_TIMESTAMP}}` | ISO timestamp | (generated) |
| `{{TOKEN_REDUCTION_MIN}}` | Minimum acceptable % | `60` |
| `{{TOKEN_REDUCTION_TARGET}}` | Target % | `70` |
| `{{MAX_ITERATIONS}}` | Max review cycles per file | `3` |
| `{{DATA_LOSS_TOLERANCE}}` | Max acceptable loss % | `5` |
| `{{REVIEWER}}` | Review command | `codex_cli.sh` |
| `{{FILES_LIST}}` | YAML list of source→target pairs | (required) |
| `{{EXECUTION_STRATEGY}}` | How to process batch | (see below) |

### FILES_LIST Format

```yaml
files:
  - source: ai_general/docs/10_architecture/versions/file_v1.0.md
    target: ai_general/docs/10_architecture/versions/file_v1.0.condensed.yml
  - source: ai_general/docs/20_registries/versions/another_v1.0.md
    target: ai_general/docs/20_registries/versions/another_v1.0.condensed.yml
  # ... 1 to N files
```

### EXECUTION_STRATEGY Options

**Sequential (learning mode):**
```yaml
execution:
  mode: sequential
  note: Process files in order. Learning accumulates in session context.
```

**Wave (balanced):**
```yaml
execution:
  mode: wave
  wave_1_count: 5
  wave_1_note: First 5 sequential, write lessons_learned.yml
  wave_2_parallel: 4
  wave_2_note: Remaining files, 4 parallel workers reading lessons first
```

**Parallel (max throughput):**
```yaml
execution:
  mode: parallel
  workers: 4
  note: All files claimable by any worker. Use when pattern established.
```

---

## Task ID Convention

Task ID derived from filename:
- `0023_librarian_batch_condensation.md` → task_id: `0023`
- `0024_librarian_condense_instructions.md` → task_id: `0024`

Worker extracts from `basename` - no need to duplicate in content.

---

## Example: Single File Task

**Filename:** `0025_librarian_condense_instr_development.md`

```yaml
---
target_worker: librarian
type: condensation
priority: medium
created: "2025-12-24T22:00:00Z"
---

# Condense: instr_development_v2.1.md

## Parameters
- Token reduction: ≥60% (target 70%)
- Max iterations per file: 3
- Data loss tolerance: ≤5%
- Reviewer: codex_cli.sh

## Files to Condense

files:
  - source: ai_general/docs/70_instructions/versions/instr_development_v2.1.md
    target: ai_general/docs/70_instructions/versions/instr_development_v2.1.condensed.yml

## Execution Strategy

execution:
  mode: sequential
```

---

## Example: Batch Task (33 files)

**Filename:** `0023_librarian_batch_condensation.md`

```yaml
---
target_worker: librarian
type: condensation
priority: high
created: "2025-12-24T21:15:00Z"
---

# Batch Condensation: Fix Broken Symlinks

## Parameters
- Token reduction: ≥60% (target 70%)
- Max iterations per file: 3
- Data loss tolerance: ≤5%
- Reviewer: codex_cli.sh

## Files to Condense

files:
  # 10_architecture (3 files)
  - source: ai_general/docs/10_architecture/versions/architectural_layer_model_v1.0.md
    target: ai_general/docs/10_architecture/versions/architectural_layer_model_v1.0.condensed.yml
  - source: ai_general/docs/10_architecture/versions/architecture_overview_v1.0.md
    target: ai_general/docs/10_architecture/versions/architecture_overview_v1.0.condensed.yml
  # ... (30 more files)

## Execution Strategy

execution:
  mode: wave
  wave_1_count: 5
  wave_1_note: First 5 files sequential - capture patterns in lessons_learned.yml
  wave_2_parallel: 4
  wave_2_note: Remaining 28 files with 4 parallel workers
```

---

## Related

- `spawn_subtasks.sh` - Optional: split batch into per-file tasks for parallel claiming
- `spec_task_id_counter.latest.md` - Atomic ID generation
- `protocol_taskCoordination.latest.md` - Task lifecycle
