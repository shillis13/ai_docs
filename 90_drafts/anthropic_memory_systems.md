# Anthropic Memory Systems Reference
*Last updated: 2026-03-16 | Status: verified against support.claude.com*

---

## Summary Table

| Name | Scope | Format | Size | Updated By | Frequency | UI Location |
|------|-------|--------|------|-----------|-----------|-------------|
| User Preferences | Account | Free text | No known limit | User | Manual | Settings → General → Profile → Personal Preferences |
| Style | Account | Free text | No known limit | User | Manual | Settings → Style |
| Account Memory Summary | Account | Auto-synthesized text | No known limit | Anthropic (auto) | Every 24h | Settings → Capabilities → Memory → "View and edit memory" |
| Account Memory Slots | Account | 30 discrete entries | 30 × 200 chars | Claude (with user permission) | On demand | Settings → Capabilities → Memory → "Manage Edits" (pencil icon) |
| Project Instructions | Project | Free text | No known limit | User | Manual | Project → Edit → Instructions |
| Project Memory Summary | Project | Auto-synthesized text | No known limit | Anthropic (auto) | Every 24h | Project → Memory |
| Project Memory Slots | Project | 30 discrete entries | 30 × 200 chars | Claude (with user permission) | On demand | Project → Memory → Manage Memory |
| Project Files | Project | Uploaded files | Plan-dependent | User | Manual | Project → Files |

---

## Scope Architecture

```
ACCOUNT SCOPE (applies to all non-project chats)
├── User Preferences        [user writes, persistent across all sessions]
├── Style                   [user writes, applies to response formatting]
├── Account Memory Summary  [auto-synthesized from all non-project chats]
└── Account Memory Slots    [30×200 char, Claude-managed with permission]

PROJECT SCOPE (isolated per project, does NOT bleed across projects)
├── Project Instructions    [user writes, prepended to every project chat]
├── Project Memory Summary  [auto-synthesized from this project's chats only]
├── Project Memory Slots    [30×200 char, Claude-managed with permission, project-scoped]
└── Project Files           [user-uploaded, accessible in every project chat]
```

---

## Element Details

### User Preferences
- **What it is**: Free text box in Settings. Stable personal facts: name, role, communication preferences, technical preferences.
- **How Claude sees it**: Injected into system prompt as `<userPreferences>` block.
- **Updated**: Manually by user only.
- **Scope**: All chats — both account and project.
- **Notes**: Most stable layer. Never auto-modified.

---

### Style
- **What it is**: Writing style preferences — tone, formality, response format.
- **How Claude sees it**: Injected alongside User Preferences.
- **Updated**: Manually by user.
- **Scope**: Account-wide.

---

### Account Memory Summary (auto-generated)
- **What it is**: Anthropic synthesizes key insights from non-project chat history into a rolling summary.
- **How Claude sees it**: Injected into system prompt as `<userMemories>` block (the large synthesis block).
- **Updated**: Automatically every 24h. Updates within 24h of chat deletion.
- **Scope**: Non-project chats ONLY. Does NOT include project chats.
- **Notes**:
  - Has "recency bias" — older conversations fade
  - Covers: role, projects, professional context, communication preferences, technical preferences, coding style
  - You can influence it mid-chat: "remember that..."
  - Deleted conversations removed from synthesis within 24h
  - Viewable and editable: Settings → Capabilities → Memory → "View and edit memory"

---

### Account Memory Slots (memory_user_edits)
- **What it is**: 30 discrete short-form entries. Hard cap 200 chars each. Think Post-it notes, not notebooks.
- **How Claude sees it**: Injected into system prompt inside the `<userMemories>` block.
- **Updated**: Claude updates via `memory_user_edits` tool when user requests. User can also edit directly via Settings UI.
- **Scope**: Account-scope OUTSIDE projects. Inside a project, a separate project-scoped pool is used.
- **IMPORTANT**: System prompt tooling incorrectly claims "100K chars per edit" — verified correct limit is 200 chars per entry.
- **Notes**:
  - Good for: stable, non-obvious facts that shouldn't fade (devices, role, persistent preferences)
  - Bad for: temporary context, current project state, anything better in the auto-summary
  - UI: Settings → Capabilities → Memory → pencil icon

---

### Project Instructions
- **What it is**: Free-text box, prepended as context to every chat in this project. Equivalent to a persistent system prompt.
- **How Claude sees it**: Injected in system prompt at high priority.
- **Updated**: Manually by user only.
- **Scope**: Single project only.
- **Notes**: Best for stable, intentional instructions: role definitions, working style, constraints, domain context. Survives context compaction.

---

### Project Memory Summary (auto-generated)
- **What it is**: Same mechanism as Account Memory Summary but scoped exclusively to this project's chat history. Zero bleed to/from other projects or account chats.
- **How Claude sees it**: Injected alongside project instructions.
- **Updated**: Automatically every 24h.
- **Scope**: This project only. Hard isolation confirmed by Anthropic docs.
- **Notes**: "Each project has its own separate memory space and dedicated project summary" — support.claude.com

---

### Project Memory Slots (memory_user_edits, project-scoped)
- **What it is**: Same mechanism as Account Memory Slots but pool is project-isolated.
- **How Claude sees it**: When inside a project, `memory_user_edits` reads/writes the project pool, not the account pool.
- **Updated**: Claude updates via `memory_user_edits` tool; also user-editable in Project UI.
- **Scope**: This project only. 30 × 200 char limit, separate from account slots.
- **Confirmed by**: System prompt metadata: "Current scope: Limited to conversations within the current Project"

---

### Project Files
- **What it is**: User-uploaded files. PDFs, docs, code, etc.
- **How Claude sees it**: Retrieval-based; most relevant sections pulled into context up to token limit.
- **Updated**: Manually by user (upload/delete).
- **Scope**: Single project only.
- **Notes**: Static — does NOT auto-update. User must manually replace files.

---

## Key Behaviors

### Scope Isolation
- Account memory (summary + slots) is populated from non-project chats ONLY
- Each project has its own separate summary AND slot pool
- Zero cross-contamination between projects, or between projects and account

### Update Timing
| Layer | Trigger | Latency |
|-------|---------|---------|
| Auto-synthesis (account) | New/modified/deleted non-project chat | Within 24h |
| Auto-synthesis (project) | New/modified/deleted project chat | Within 24h |
| Memory slots | User says "remember X" → Claude calls tool | Immediate |
| User Preferences | User edits Settings | Immediate (next chat) |
| Project Instructions | User edits Project | Immediate (next chat) |

### Pausing vs Resetting Memory
- **Pause**: keeps existing memory, stops creating new memories, paused-period chats NOT included if re-enabled
- **Reset**: permanently deletes ALL memories including project memories — irreversible

### Incognito Mode
- Doesn't contribute to any memory synthesis
- Not saved to chat history
- Still subject to enterprise data retention policies

---

## Claude Code Memory (Separate, Non-Interacting System)

Claude Code (CLI tool) has its own memory architecture completely separate from claude.ai. These do NOT interact.

| Element | Format | Scope | Location |
|---------|--------|-------|----------|
| User CLAUDE.md | Markdown | All Code sessions | ~/.claude/CLAUDE.md |
| Project CLAUDE.md | Markdown | This repo | {repo_root}/CLAUDE.md |
| Rules files | Markdown (path-scoped) | On-demand | .claude/rules/*.md |
| Auto Memory (MEMORY.md) | Markdown, max 200 lines | This repo | ~/.claude/projects/{repo}/memory/ |

---

## Correction Log
- 2026-03-16: Confirmed memory_user_edits limit is **200 chars/slot**, NOT 100K. System prompt tooling description is wrong.

*Sources: support.claude.com/articles/11817273, verified 2026-03-16*
