# Writing Standards

**Version:** 1.0.0
**Created:** 2025-12-09
**Maintainer:** PianoMan
**Status:** active

## Purpose

Standards for written content in documentation, tasks, and other text files. Optimized for readability in plain text and monospace environments.

## Table Formatting

### Principle

Markdown pipe tables render poorly in plain text, terminals, and monospace editors. Use ASCII box-drawing tables for consistent readability across all viewing contexts.

### When to Use ASCII Tables

- Documentation viewed in terminals or text editors
- Task files read by CLI agents
- Any file that may be viewed without markdown rendering
- Log files and reports

### Bad Example: Markdown Pipe Tables

Markdown pipe tables are misaligned in plain text:

```
| Task | Command/Flag | Notes |
| --- | --- | --- |
| One-off prompt | `claude_cli.sh "..."` | Writes log, new session |
| Use agent | `claude_cli.sh -A dev-lead "..."` | Loads agent prompt, sets agent workdir |
| Continue latest | `claude_cli.sh -c "..."` | Uses project-scoped most recent session |
| Resume by ID | `claude_cli.sh -r <id> "..."` | Pull ID from history/info |
| Named session | `claude_cli.sh -n <name> "..."` | Requires registry entry |
| Background/tmux | `claude_cli.sh -t -s myrun "..."` | Attach with `tmux attach -t myrun` |
| Auto-approve | `claude_cli.sh -a ...` | Skips permission prompts-use sparingly |
| Override dir | `claude_cli.sh -w <dir> ...` | Defaults to `~/Documents/AI/ai_root` |
```

### Good Example: ASCII Box Tables

ASCII box tables are aligned in any monospace context:

```
+-----------------+-----------------------------------+-----------------------------------------+
| Task            | Command/Flag                      | Notes                                   |
+-----------------+-----------------------------------+-----------------------------------------+
| One-off prompt  | `claude_cli.sh "..."`             | Writes log, new session                 |
+-----------------+-----------------------------------+-----------------------------------------+
| Use agent       | `claude_cli.sh -A dev-lead "..."` | Loads agent prompt, sets agent workdir  |
+-----------------+-----------------------------------+-----------------------------------------+
| Continue latest | `claude_cli.sh -c "..."`          | Uses project-scoped most recent session |
+-----------------+-----------------------------------+-----------------------------------------+
| Resume by ID    | `claude_cli.sh -r <id> "..."`     | Pull ID from history/info               |
+-----------------+-----------------------------------+-----------------------------------------+
| Named session   | `claude_cli.sh -n <name> "..."`   | Requires registry entry                 |
+-----------------+-----------------------------------+-----------------------------------------+
| Background/tmux | `claude_cli.sh -t -s myrun "..."` | Attach with `tmux attach -t myrun`      |
+-----------------+-----------------------------------+-----------------------------------------+
| Auto-approve    | `claude_cli.sh -a ...`            | Skips permission prompts-use sparingly  |
+-----------------+-----------------------------------+-----------------------------------------+
| Override dir    | `claude_cli.sh -w <dir> ...`      | Defaults to `~/Documents/AI/ai_root`    |
+-----------------+-----------------------------------+-----------------------------------------+
```

### Compact Variant

Lighter ASCII table without row separators:

```
+-----------------+-----------------------------------+-----------------------------------------+
| Task            | Command/Flag                      | Notes                                   |
+-----------------+-----------------------------------+-----------------------------------------+
| One-off prompt  | claude_cli.sh "..."               | Writes log, new session                 |
| Use agent       | claude_cli.sh -A dev-lead "..."   | Loads agent prompt, sets agent workdir  |
| Continue latest | claude_cli.sh -c "..."            | Uses project-scoped most recent session |
| Resume by ID    | claude_cli.sh -r <id> "..."       | Pull ID from history/info               |
+-----------------+-----------------------------------+-----------------------------------------+
```

### Tools

| Tool | Note |
|------|------|
| vim TableMode plugin | `:TableModeToggle` then type with \| separators |
| column command | `column -t -s'\|'` for quick alignment |
| Online generators | tablesgenerator.com, etc. |

## Future Sections

Placeholder for additional writing standards:

- Heading hierarchy
- Code block formatting
- List formatting
- Line length limits
- File naming conventions
