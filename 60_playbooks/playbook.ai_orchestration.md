# AI Orchestration Playbook

**Version:** 1.0.0
**Last Updated:** 2026-01-02
**Maintainer:** PianoMan
**Status:** active

## Purpose

Guide for AI Orchestrators (Desktop Claude, scripts, cron jobs) to launch and coordinate AI workers across platforms. Covers use cases, command patterns, and the orthogonal relationship between Agent roles and platform types.

## Core Concept: Agents Are Platform-Agnostic

Agent roles (librarian, dev-lead, custodian, ops) are **orthogonal** to platform type (Claude, Codex, Gemini). Any agent role can run on any platform:

| Agent Role | Claude | Codex | Gemini |
|------------|--------|-------|--------|
| librarian  | ✅ | ✅ | ✅ |
| dev-lead   | ✅ | ✅ | ✅ |
| custodian  | ✅ | ✅ | ✅ |
| ops        | ✅ | ✅ | ✅ |

**Why this matters:**
- Librarians on different platforms can process in parallel
- Dev-lead on Codex gets different capabilities than on Claude
- Specialized platforms (Codex for code, Gemini for analysis) + specialized roles = powerful combinations

## Platform Selection Guide

| Platform | Best For | Strengths | Limitations |
|----------|----------|-----------|-------------|
| **Claude** | Complex reasoning, multi-turn tasks | Session history, MCP tools, context continuity | Slower, higher cost |
| **Codex** | Code generation, file operations | Fast, auto-approve, good at code | No session history, simpler reasoning |
| **Gemini** | Analysis, research, large context | Large context window, model selection | Limited session management |

## Standard Orchestrator Commands

### Launch Worker with Prompt

```bash
# Claude worker
claude_cli.py -t -s worker_001 -a -A librarian "Process the export queue"

# Codex worker (faster, code-focused)
codex_cli.py -t -s worker_001 -a -A dev-lead "Implement the parser"

# Gemini worker (large context analysis)
gemini_cli.py -t -s worker_001 -a -m gemini-2.5-pro -A librarian "Analyze the dataset"
```

### Launch Worker with Task File

```bash
# Claude with task
claude_cli.py -t -s task_runner -a -T ai_comms/claude_cli/tasks/to_execute/req_1050.md

# Codex with task
codex_cli.py -t -s task_runner -a -T ai_comms/codex_cli/tasks/to_execute/req_2001.md

# Gemini with task
gemini_cli.py -t -s task_runner -a -T ai_comms/gemini_cli/tasks/to_execute/analysis_001.md
```

### Launch Worker Claiming Next Task

```bash
# Claude claims from its queue
claude_cli.py -t -s queue_worker -a --any-task

# Codex claims from its queue
codex_cli.py -t -s queue_worker -a --any-task

# Gemini claims from its queue
gemini_cli.py -t -s queue_worker -a --any-task
```

## Flag Reference

| Flag | Purpose |
|------|---------|
| `-t` | Run in tmux (background, monitorable) |
| `-s NAME` | Tmux session name (for identification) |
| `-a` | Auto-approve (non-interactive) |
| `-A TYPE` | Agent role (librarian, dev-lead, custodian, ops) |
| `-T FILE` | Task file to execute |
| `--any-task` | Claim first available from queue |
| `-w DIR` | Override working directory |
| `--on-conflict MODE` | If session already running: fork, exit, queue |

## Monitoring Workers

```bash
# View current output
tmux capture-pane -t worker_001 -p

# Send message to worker
tmux send-keys -t worker_001 'Continue processing' Enter

# Attach for interactive control
tmux attach -t worker_001

# Kill session when done
tmux kill-session -t worker_001

# List all sessions
tmux list-sessions | grep -E "claude|codex|gemini"
```

## Use Cases

### Parallel Processing (Same Agent, Multiple Platforms)

Scenario: Large batch of files to condense. Launch librarians on multiple platforms:

```bash
# Split work across platforms for speed
claude_cli.py -t -s lib_claude -a -A librarian "Process files 001-100"
codex_cli.py -t -s lib_codex -a -A librarian "Process files 101-200"
gemini_cli.py -t -s lib_gemini -a -A librarian "Process files 201-300"
```

### Pipeline Processing (Different Agents, Sequential)

Scenario: Code review workflow requiring design → implementation → review:

```bash
# Phase 1: Design (dev-lead plans)
claude_cli.py -t -s design_phase -a -A dev-lead -T tasks/design_req.md

# Phase 2: Implement (after design complete)
codex_cli.py -t -s impl_phase -a -A dev-lead -T tasks/impl_req.md

# Phase 3: Review (after implementation)
codex_cli.py -t -s review_phase -a -T tasks/review_req.md
```

### Crowd-Sourced Problem Solving

Scenario: Need multiple perspectives on a problem:

```bash
# Launch same task to multiple platforms
claude_cli.py -t -s perspective_1 -a -T tasks/problem_analysis.md
codex_cli.py -t -s perspective_2 -a -T tasks/problem_analysis.md
gemini_cli.py -t -s perspective_3 -a -T tasks/problem_analysis.md

# Orchestrator synthesizes results
```

### Task Queue Processing

Scenario: Continuous queue processing with dedicated workers:

```bash
# Long-running queue processors
claude_cli.py -t -s claude_queue -a --any-task
codex_cli.py -t -s codex_queue -a --any-task

# Monitor queues
watch -n 30 'ls ai_comms/*/tasks/to_execute/'
```

## Session Conflict Handling

When launching with `-s NAME` and that session exists:

| Mode | Behavior |
|------|----------|
| `fork` (default) | Create new session with suffix (worker_001_fork_1) |
| `exit` | Exit with error, don't launch |
| `queue` | Queue task for later execution |

```bash
# Explicit conflict handling
claude_cli.py -t -s worker_001 -a --on-conflict exit -A librarian "Task"
```

## Best Practices

1. **Always use `-t` for orchestrated work** - Enables monitoring and control
2. **Use descriptive session names** - Makes monitoring easier
3. **Match platform to task type** - Codex for code, Claude for reasoning
4. **Use task files for complex work** - Prompts for simple tasks
5. **Check for existing sessions** before launching with same name
6. **Log orchestrator decisions** - Track which workers were launched for what

## Related Documentation

- `ai_general/docs/30_protocols/protocol_taskCoordination.latest.yml` - Task file format
- `ai_general/docs/10_architecture/cli_orchestration.latest.md` - CLI architecture
- `ai_general/prompts/agent.*.md` - Agent role definitions
