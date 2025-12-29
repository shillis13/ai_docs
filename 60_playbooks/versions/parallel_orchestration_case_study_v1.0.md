# Multi-Level Parallel Orchestration Case Study

**Version:** 1.0.0
**Created:** 2025-12-10
**Maintainer:** PianoMan
**Status:** active
**Category:** case-study

## Summary

Documented execution of parallel document condensation across 7 directories using 7 simultaneous Claude CLI workers, each orchestrating Codex for validation, with Desktop Claude monitoring via AT self-wake pattern.

## Orchestration Layers

### Level 1: Desktop Claude (Strategic Coordinator)

**Responsibilities:**
- Created task specification (req_1063)
- Launched 7 parallel CLI workers
- Set AT self-wake alarms
- Monitored progress on wake
- Verified quality of results

**Did NOT do:** File condensation, validation, direct file manipulation

### Level 2: Claude CLI Workers

**Agent:** librarian (`-A librarian` flag)
**Count:** 7 parallel instances

**Session Names:**
- condense_10_arch
- condense_20_reg
- condense_30_proto
- condense_40_specs
- condense_50_schemas
- condense_60_play
- condense_70_instr

**Responsibilities:**
- Read original files in assigned directory
- Create condensed versions
- Launch Codex for validation
- Iterate on gaps (max 3 rounds)
- Write final condensed files

**Launch Pattern:**
```bash
claude_cli.sh -t -a -s {session_name} -A librarian "{prompt}"
```

### Level 3: Codex (Validator)

**Invoked by:** Claude CLI workers
**Purpose:** Compare original vs condensed, find gaps

```bash
codex_cli.sh -t -s val_{name} -a "Compare ORIGINAL vs CONDENSED. List operational info missing."
```

## Self-Wake Pattern

**Mechanism:** AT job scheduling future prompt

```bash
cat << 'EOF' | at now + 5 minutes
send_prompt.sh claude-desktop "AT wake-up: Check status" --force --submit
EOF
```

**Key Insight:** "AT is my wake-up alarm, not CLI's doorbell"

**On Wake Actions:**
1. Check tmux sessions still alive
2. Count files in condensed/ directories
3. Verify progress vs last check
4. Set another AT if work continues
5. Wrap up and report if complete

## Execution Timeline

| Time | Event |
|------|-------|
| 21:36 | Launched 7 parallel CLI workers |
| 21:38 | Set first AT wake (5 min) |
| 21:43 | Wake #1 - 2 directories complete |
| 21:48 | Wake #2 - 5 directories complete |
| 21:52 | Wake #3 - 6 complete, 10_arch partial |
| 21:57 | Wake #4 - All sessions exited, incomplete |
| 21:58 | Launched cleanup worker |
| 22:02 | Final verification - ALL COMPLETE |

## Results

| Metric | Value |
|--------|-------|
| Directories processed | 7 |
| Files condensed | ~48 |
| Original lines | 12,773 |
| Condensed lines | 3,658 |
| Reduction | 72% |
| Tokens saved | ~71,000 |
| Total time | ~25 minutes |

## Lessons Learned

1. Parallel workers dramatically reduce wall-clock time
2. librarian agent appropriate for file-heavy documentation work
3. AT self-wake provides agency without human loop
4. Some workers may exit early or miss files - cleanup pass needed
5. Codex validation catches real gaps (not just rubber-stamping)

## Prerequisites

- claude_cli.sh wrapper with -t, -a, -A flags
- codex_cli.sh for validation
- send_prompt.sh with --force flag
- AT daemon running
- Defined agent profiles in ~/.claude/agents.json

## Replication Steps

1. Create task spec defining scope per worker
2. Launch N workers with unique session names
3. Set AT wake for monitoring interval
4. On wake: check sessions, count outputs, assess progress
5. Repeat AT until all sessions exit
6. Verify completeness, run cleanup if needed
7. Quality-check sample outputs
