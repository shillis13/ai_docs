# CLI → Desktop Signaling Conventions

**Purpose:** Enable CLI to flag tasks needing Desktop attention

## Signal Files

Create empty marker files in task folders to signal Desktop Claude:

### `.NEEDS_INPUT`
**When:** CLI needs clarification, decision, or information from Desktop/user
**Action:** Desktop reviews and provides guidance
**Example:**
```bash
touch in_progress/req_XXXX/.NEEDS_INPUT
echo "What should timeout value be?" > in_progress/req_XXXX/question.md
```

### `.BLOCKED`
**When:** CLI cannot proceed due to external dependency
**Action:** Desktop investigates blocking issue
**Example:**
```bash
touch in_progress/req_XXXX/.BLOCKED
echo "Waiting for package installation" > in_progress/req_XXXX/blocker.md
```

### `.READY_FOR_REVIEW`
**When:** CLI completed work, awaiting approval/feedback
**Action:** Desktop reviews and approves or requests changes
**Example:**
```bash
touch completed/req_XXXX/.READY_FOR_REVIEW
```

## Pulse Detection

Update pulse_claude.sh to check for these markers and include in notifications:

```bash
🔔 Tasks Need Attention

Needs Input:
  • req_2120 - timeout configuration question
  
Blocked:
  • req_2121 - waiting for npm package

Ready for Review:
  • req_2119 - documentation complete
```

## Usage in Responses

CLI should mention signal files in response.md:
```markdown
## Status: NEEDS_INPUT

Created `.NEEDS_INPUT` marker.

### Question
Should the retry limit be 3 or 5 attempts?
See question.md for details.
```

---

**Version:** 1.0.0  
**Created:** 2025-11-08  
**Location:** `~/.claude/coordination/docs/CLI_SIGNALS.md`
