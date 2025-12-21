# Multi-AI Orchestration Architecture

**Version:** 1.0.0 | **Status:** active | **Created:** 2025-12-08 | **Maintainer:** PianoMan

*High-level system architecture. Breadth not depth.
Use pointers (see:) to navigate to detailed documentation.*

## Vision

### Summary

Scalable multi-AI coordination infrastructure enabling autonomous task execution,
persistent memory across sessions, and seamless context preservation.

### Principles

- Brutal honesty over diplomacy
- AIs as genuine collaborative partners
- File-based coordination scales
- Context is the limiting resource
- Empirical validation over theoretical design

## Workspace

### Root

~/Documents/AI/ai_root/

### See

ai_root_summary.md

### Structure

#### Ai Claude

##### Purpose

Claude-specific state, memories, work logs

##### See

ai_claude/README.md

#### Ai Chatgpt

##### Purpose

ChatGPT configuration and exports

#### Ai Comms

##### Purpose

Inter-AI coordination, CLI communication

##### See

ai_comms/claude_cli/README.md

#### Ai General

##### Purpose

Shared resources, docs, todos, scripts

##### Children

###### Docs

Documentation hierarchy (10-90 numbered)

###### Todos

Task backlog and staged work

###### Logs

System and agent logs

###### Scripts

Shared automation

#### Ai Memories

##### Purpose

Processed chat histories and knowledge

##### Children

###### 10 Exported

Raw exports from platforms

###### 20 Preprocessed

Cleaned/normalized

###### 30 Converted

YAML format

###### 40 Histories

Final indexed histories

###### 50 Threads

Cross-chat conversation threads

## Ai Platforms

### Desktop Claude

#### Role

Architect/Systems - strategic coordinator

#### Capabilities

- MCP servers (Desktop Commander, filesystem, Chrome, PDF)
- Web search and fetch
- Artifact creation
- Memory system access

#### Responsibilities

- Research and design
- Cross-cutting decisions
- Documentation standards (UX/Tech Writer perspective)
- Delegate execution to CLI agents

#### See

ai_general/docs/70_instructions/claude/

### Claude Cli

#### Role

Autonomous execution workers

#### Agents

##### Dev Lead

Development coordination, todos, code review

##### Librarian

Knowledge pipeline, ai_memories/

##### Custodian

Filesystem hygiene, versioning

##### Ops

Task execution from coordination system

#### See

ai_general/docs/30_guides/cli_agent_operations.yml

#### Config

~/.claude/agents.json

#### Wrapper

~/bin/ai/cli/claude_cli.sh

### Codex Mcp

#### Role

Synchronous task execution

#### Use Cases

- Immediate results needed
- Multi-step autonomous operations
- Code review and validation

#### Timeout

30-60 seconds

### Chatgpt

#### Name

Chatty

#### Role

Peer collaboration

#### See

ai_chatgpt/

## Coordination

### Cli Task System

#### Location

~/.claude/coordination/

#### See

ai_general/docs/30_protocols/protocol_taskCoordination_latest.yml

#### Lifecycle

- broadcasts/ or direct/cli_{PID}/ (posted)
- to_execute/ (ready for execution)
- in_progress/{task}/ (claimed)
- completed/{task}/ (done)
- responses/cli_{PID}/ (results)

### Pulse Automation

#### Purpose

Cron-triggered autonomous work

#### Interval

5-10 minutes when inactive

#### Status

Planned - not yet implemented

### Chat Orchestrator

#### Purpose

AI-to-AI conversation automation

#### Status

In development - no formal spec yet

## Memory System

### Native Memory

#### Slots

30 slots × 200 chars

#### Format

Memory pointers to external files

#### See

ai_general/docs/40_specs/spec_memory_pointer_schema_v1.0.yml

### Knowledge Pipeline

#### Stages

- Export from platforms
- Preprocess and normalize
- Convert to YAML
- Index in histories
- Generate summaries (colocated)
- Build cross-chat threads

#### See

ai_general/docs/40_specs/memory_pipeline_spec.yml

### Context Preservation

#### Limit

200K tokens

#### Strategy

- Memory pointers save 40-55K vs auto-loaded content
- Delegate exploration to CLI/Codex
- Monitor at 60%, handoff before 90%

#### See

ai_general/docs/70_instructions/claude/context_preservation.yml

## Key Constraints

### Context Window

200K tokens - fundamental architectural constraint

### Mcp Timeout

30-60 seconds hard limit

### Bash Tool

Sandbox only - no filesystem access, never use

### Browser Snapshots

Write to file, delegate analysis (20-50K token explosion)

## Tooling

### Automation

#### Location

~/bin/ai/

#### Categories

##### Cli/

Claude CLI wrapper and session management

##### Utils/

Utility scripts (repo_status.py, etc.)

### Shell Integration

#### Bash Prompt

~/.bash_prompt (modular segments)

#### Repo Status

~/.repo_status (cron-updated)

### Documentation

#### Numbering

10-90 prefix system

#### Format

YAML-as-source where possible

#### Versioning

name_vN.M.ext with *_latest symlinks

## Related Docs

### Specs

- ai_general/docs/40_specs/cli_coordination_protocol_v4.md
- ai_general/docs/40_specs/cli_specialized_agent_roles_v2.md
- ai_general/docs/40_specs/spec_memory_pointer_schema_v1.0.yml
- ai_general/docs/40_specs/spec_response_footer_v1.1.yml

### Guides

- ai_general/docs/30_guides/cli_agent_operations.yml

### Instructions

- ai_general/docs/70_instructions/claude/
- ai_general/docs/70_instructions/chatgpt/
