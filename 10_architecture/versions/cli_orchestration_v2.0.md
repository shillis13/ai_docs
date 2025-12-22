# CLI Orchestration & Monitoring Architecture v1.0

**Topic:** CLI task orchestration, monitoring, and inter-agent communication  
**Status:** Design specification  
**Created:** 2025-11-09  
**Context:** CLI agents as full peers with orchestration capabilities
**Protocol Reference:** Task Coordination Protocol v7.0 (`ai_general/docs/30_protocols/protocol_taskCoordination_latest.yml`)

---

## Core Insight: CLI as Orchestrating Agent

**Previous View (too narrow):**
```
Desktop Claude (orchestrator)
    ↓
  CLI (executor only)
```

**Actual Reality:**
```
Desktop Claude (strategic coordinator)
    ↓
  CLI Orchestrator (tactical coordinator)
    ↓ ↓ ↓
  CLI_A  CLI_B  CLI_C (specialized executors)
```

**Key Recognition:**
- CLI instances can persist longer than Desktop Claude sessions
- CLI can orchestrate complex multi-step workflows
- CLI needs to ask questions, request permissions, report blockers
- CLI-to-CLI coordination required for distributed work
- Task Coordination Protocol v7.0 formalizes a multi-level orchestration hierarchy (Desktop → CLI orchestrator → child orchestrators → executors) with explicit subtask naming `{parent}_t{N}.{M}` and handoff paths

## Orchestration Hierarchy (Protocol v7.0)

```
Desktop Claude (strategic)
  └─ CLI Orchestrator (tactical)
       ├─ Child orchestrators (optional)
       │    └─ Workers (specialized executors)
       └─ Workers (direct executors)
```

- Naming: Parents create subtasks inside their own task dir using `{parent}_t{N}` and `{parent}_t{N}.{M}` for deeper nesting.
- Claiming: Each worker claims via `claimed_{timestamp}_{pid}_` prefix and renames `{task}.md` to `{task}.$$.md` to mark ownership.
- Visibility: Parents keep `child@` symlinks to completed child task dirs so monitoring and audit tools can traverse the full tree from any parent node.

---

## Communication Patterns

### Response File Types (Protocol v7.0)
- Use `.response.md` for milestone or intermediate updates inside task directories
- Use `.completion.md` for final deliverables in `coordination/completed/{task_id}/`

### Symlink Pattern for Child Results (Protocol v7.0)
- Parent creates `child_task@` symlink inside parent task dir pointing to `../../completed/{child_task}/`
- Create link after child task completes, before closing the parent; this keeps traversal simple for monitors and postmortems without duplicating artifacts.
- Symlinks are part of the audit trail—do not replace them with copies.

### Pattern 1: Desktop → Single CLI (Simple Task)

**Flow:**
```
1. Desktop → claude_cli/coordination/to_execute/req_1234.md
2. CLI claims, moves to in_progress/
3. CLI executes
4. CLI → coordination/completed/req_1234/completion.md
5. Desktop reads result
```

**When to use:** Simple, well-defined tasks with clear success criteria.

### Pattern 2: Desktop → Single CLI (Complex with Questions)

**Flow:**
```
1. Desktop → coordination/to_execute/req_1234.md
2. CLI claims, starts work
3. CLI encounters blocker
4. CLI → instant_messaging/active/req_1234/
   "Need AWS credentials - security policy blocks access"
5. Desktop → instant_messaging/active/req_1234/
   "Here are temp credentials: [...]"
6. CLI continues with credentials
7. CLI → coordination/completed/req_1234/completion.md
```

**When to use:** Tasks where blockers, questions, or permission requests likely.

### Pattern 3: Desktop → CLI Orchestrator → Multiple CLIs

**Flow:**
```
1. Desktop → claude_cli_orch/coordination/to_execute/req_1234.md
   Task: "Analyze entire codebase for security issues"

2. CLI_Orch breaks down:
   - Frontend analysis → CLI_A
   - Backend analysis → CLI_B  
   - Infrastructure → CLI_C

3. CLI_Orch → claude_cli_a/tasks/incoming/subtask_1234a.md
4. CLI_Orch → claude_cli_b/tasks/incoming/subtask_1234b.md
5. CLI_Orch → claude_cli_c/tasks/incoming/subtask_1234c.md

6. CLI_A hits blocker:
   CLI_A → claude_cli_orch/instant_messaging/active/subtask_1234a/
   "Found encrypted config - need decryption key"

7. CLI_Orch escalates to Desktop:
   CLI_Orch → Desktop instant_messaging:
   "CLI_A needs decryption key for configs"

8. Desktop provides key via instant_messaging

9. CLI_Orch → CLI_A via instant_messaging: 
   "Here's the key: [...]"

10. CLI_A completes, writes response:
    CLI_A → claude_cli_orch/task_responses/incoming/subtask_1234a_result.md

11. CLI_Orch synthesizes all results
12. CLI_Orch → Desktop coordination/completed/req_1234/completion.md
```

**When to use:** Complex, multi-faceted work requiring parallel execution.

---

## Non-Response Communication

**Definition:** Questions/issues that aren't about the final deliverable but about execution process.

### Types of Non-Response Communication

| Type | Example | Channel | Urgency |
|------|---------|---------|---------|
| **Permission request** | "Need sudo access" | instant_messaging | High |
| **Blocker** | "API is down" | instant_messaging | High |
| **Critical question** | "Found 3 configs, which?" | instant_messaging | High |
| **Progress update** | "Stage 2/5 complete" | notifications | Medium |
| **Context request** | "Need original requirements" | note_passing | Low |
| **Peer review hold** | "Ready for review at step 3" | tasks + notification | Medium |

### Examples in Practice

**Permission Request:**
```markdown
# instant_messaging/active/req_1234/thread.jsonl
{"from":"cli_12345","ts":"...","text":"Attempting to write to /var/log requires sudo. Approve elevated permissions?","type":"permission_request"}

# Desktop responds
{"from":"desktop","ts":"...","text":"Approved for this session only. Use sudo password: [...]","type":"permission_grant"}
```

**Blocker:**
```markdown
{"from":"cli_12345","ts":"...","text":"ERROR: Database connection refused. postgres service not running.","type":"blocker"}
{"from":"desktop","ts":"...","text":"Starting postgres now. Retry in 30s.","type":"resolution"}
```

**Critical Question:**
```markdown
{"from":"cli_12345","ts":"...","text":"Found 3 configuration files: dev.conf, staging.conf, prod.conf. Task didn't specify which to use. Which should I analyze?","type":"clarification"}
{"from":"desktop","ts":"...","text":"Use prod.conf - that's the critical one.","type":"answer"}
```

**Peer Review Hold:**
```markdown
# CLI reaches step 3, needs review before proceeding
# Writes notification
echo "req_1234 ready for peer review at step 3" > \
  claude/notifications/pending/cli_review_req1234.txt

# Creates review task
cat > claude/tasks/incoming/review_req1234_step3.md << 'EOF'
# Review Task: req_1234 Step 3 Completion

**Context:** Backend analysis workflow
**Current State:** Database schema analyzed, migration scripts generated
**Hold Point:** Step 3 complete, awaiting review before executing migrations

**Review Checklist:**
- [ ] Schema changes look correct?
- [ ] Migration scripts follow standards?
- [ ] Rollback plan adequate?

**After Review:**
- Approve: Write to claude_cli/instant_messaging/active/req_1234/ with "APPROVED"
- Reject: Write with "REJECTED: [reason]"
EOF
```

---

## Monitoring Approaches

### Approach 1: Passive (File-Based)

**Method:** Desktop Claude polls for updates
```bash
# Check for notifications
ls claude/notifications/pending/

# Check for instant messages
ls claude/instant_messaging/active/*/

# Check task progress
ls claude_cli/coordination/in_progress/*/progress.md
```

**Pros:**
- ✅ Simple, no special infrastructure
- ✅ Works across any transport

**Cons:**
- ❌ Polling delay (minutes)
- ❌ Can't see terminal output
- ❌ No real-time awareness

### Approach 2: Active (iTerm2 Python API)

**Method:** Desktop Claude watches CLI terminal in real-time
```python
# iTerm2 Python API can:
# - Open new terminal tabs/windows
# - Send commands to specific sessions
# - Read terminal output
# - Inject text at cursor
# - Monitor for patterns

# Example monitoring
async def monitor_cli_task(session_id, task_id):
    session = app.get_session_by_id(session_id)
    
    # Watch for specific patterns
    patterns = [
        "ERROR:",          # Errors
        "PERMISSION:",     # Permission requests
        "QUESTION:",       # Questions needing answer
        "COMPLETE:",       # Task completion
    ]
    
    # Stream output
    async for line in session.async_get_screen_streamer():
        for pattern in patterns:
            if pattern in line:
                # Trigger appropriate handler
                await handle_cli_event(pattern, line, task_id)
```

**Pros:**
- ✅ Real-time visibility
- ✅ Can inject commands mid-execution
- ✅ See what CLI is actually doing
- ✅ No polling delay

**Cons:**
- ❌ macOS/iTerm2 specific
- ❌ Requires Python API setup
- ❌ Terminal output parsing (fragile)

### Approach 3: Hybrid (Structured + Terminal)

**Method:** Combine file-based messaging with terminal monitoring

**Structured communication (primary):**
```
CLI → instant_messaging: Questions, blockers, requests
Desktop → instant_messaging: Answers, approvals, commands
```

**Terminal monitoring (supplementary):**
```
Desktop watches CLI terminal for:
- Progress indicators
- Error patterns
- Stuck/frozen states
- Resource usage
```

**Pros:**
- ✅ Reliable structured communication
- ✅ Real-time awareness of issues
- ✅ Terminal context for debugging
- ✅ Falls back gracefully

**Cons:**
- ❌ More complex
- ❌ Requires both systems working

---

## iTerm2 Python API Integration

### Desktop → CLI Command Injection

**Current capability:**
```bash
# Desktop Claude can send commands via AppleScript
osascript -e 'tell application "iTerm"
    tell session id "SESSION_ID"
        write text "ls -la"
    end tell
end tell'
```

**Enhanced with Python API:**
```python
# More sophisticated control
import iterm2

async def inject_command(session_id, command):
    session = await app.async_get_session_by_id(session_id)
    await session.async_send_text(command + "\n")
    
async def inject_with_confirmation(session_id, command):
    session = await app.async_get_session_by_id(session_id)
    await session.async_send_text(command)
    # Wait for prompt to return before continuing
    await session.async_wait_for_pattern("❯")  # or $ or >
```

### Real-Time Monitoring

**Monitor for blockers:**
```python
async def monitor_for_blockers(session_id, task_id):
    session = await app.async_get_session_by_id(session_id)
    
    blocker_patterns = [
        r"ERROR:.*",
        r"PERMISSION DENIED",
        r"sudo password",
        r"Waiting for.*",
    ]
    
    async for line in session.async_get_screen_streamer():
        for pattern in blocker_patterns:
            if re.search(pattern, line):
                # Write to instant_messaging automatically
                await escalate_to_desktop(task_id, line)
```

### When iTerm2 API Makes IM Unnecessary (Desktop ↔ CLI)

**Scenario:** Simple clarification
```
WITHOUT IM:
1. CLI writes: instant_messaging/active/req_1234/thread.jsonl
2. Desktop polls every minute
3. Desktop sees question after delay
4. Desktop writes response
5. CLI polls, sees response
[Total: 1-2 minute delay]

WITH iTerm2 API:
1. CLI prints: "QUESTION: Which config file?"
2. iTerm2 API detects pattern immediately
3. Desktop notified in real-time
4. Desktop injects answer directly into terminal
5. CLI continues immediately
[Total: Seconds]
```

**When IM still needed:**
- CLI-to-CLI communication (no terminal access)
- Persistence (conversation history)
- Complex multi-turn discussions
- Structured data exchange
- Asynchronous questions (CLI asks, waits hours for answer)

---

## CLI-to-CLI Communication

### When CLI Orchestrates Multiple CLIs

**Pattern:**
```
CLI_Orchestrator needs to:
1. Assign work to subordinate CLIs
2. Monitor their progress  
3. Answer their questions
4. Collect their results
5. Synthesize final output
```

**Communication requirements:**
```
CLI_Orch → CLI_A/tasks/incoming/: Task assignment
CLI_A → CLI_Orch/instant_messaging/: Questions during execution
CLI_Orch → CLI_A/instant_messaging/: Answers
CLI_A → CLI_Orch/task_responses/incoming/: Results
```

**Key point:** iTerm2 API doesn't help here - CLIs need structured messaging.

---

## Standing Instructions for Orchestrators

### Orchestrator Initialization Protocol

**When CLI becomes an orchestrator, it must:**

1. **Establish identity:**
```bash
# Create orchestrator mailbox if doesn't exist
mkdir -p ~/Documents/AI/ai_root/ai_comms/claude_cli_orch_{PID}/
ln -s ~/Documents/AI/ai_root/ai_comms/claude_cli_orch_{PID} \
      ~/.claude/coordination_orch_{PID}
```

2. **Register subordinates:**
```bash
# Create registry of subordinate CLIs
cat > claude_cli_orch_{PID}/registry/subordinates.json << EOF
{
  "orchestrator": "cli_orch_{PID}",
  "subordinates": [
    {"id": "cli_a_{PID}", "role": "frontend_analysis", "status": "idle"},
    {"id": "cli_b_{PID}", "role": "backend_analysis", "status": "idle"},
    {"id": "cli_c_{PID}", "role": "infrastructure_analysis", "status": "idle"}
  ],
  "created": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

3. **Establish communication paths:**
```bash
# For each subordinate, create bidirectional channels
for sub in cli_a cli_b cli_c; do
  # Orchestrator → Subordinate
  mkdir -p claude_cli_${sub}/tasks/incoming/
  mkdir -p claude_cli_${sub}/instant_messaging/active/
  
  # Subordinate → Orchestrator  
  mkdir -p claude_cli_orch_{PID}/task_responses/incoming/
  mkdir -p claude_cli_orch_{PID}/instant_messaging/active/
done
```

4. **Create escalation path to Desktop:**
```bash
# Orchestrator can escalate to Desktop Claude
mkdir -p ~/Documents/AI/ai_root/ai_comms/claude/instant_messaging/active/orch_{PID}/
```

### Orchestrator Operating Instructions

**Standard workflow:**
```markdown
1. Receive complex task from Desktop
2. Break into subtasks
3. For each subtask:
   a. Assign to appropriate subordinate CLI
   b. Monitor subordinate's instant_messaging for questions
   c. Answer questions or escalate to Desktop
   d. Collect responses from task_responses/
4. Synthesize results
5. Report to Desktop via coordination/completed/
```

**Handling subordinate questions:**
```python
# Pseudo-code for orchestrator behavior
def monitor_subordinates():
    for subordinate in subordinates:
        # Check for questions
        questions = read_instant_messaging(subordinate, my_id)
        
        for question in questions:
            if can_answer_myself(question):
                # Answer directly
                write_instant_messaging(subordinate, answer)
            else:
                # Escalate to Desktop
                escalate_to_desktop(question, subordinate_context)
                # Wait for Desktop's answer
                desktop_answer = wait_for_desktop_response()
                # Forward to subordinate
                write_instant_messaging(subordinate, desktop_answer)
```

### Example: Complex Orchestration

**Task:** "Audit entire codebase for security vulnerabilities"

**Orchestration flow:**
```
Desktop → CLI_Orch: req_1234_security_audit.md

CLI_Orch breaks down:
1. Static analysis (CLI_A)
2. Dependency check (CLI_B)
3. Secret scanning (CLI_C)
4. Infrastructure audit (CLI_D)

CLI_Orch → CLI_A/tasks/: Run static analysis (Semgrep, Bandit)
CLI_Orch → CLI_B/tasks/: Check dependencies (npm audit, pip-audit)
CLI_Orch → CLI_C/tasks/: Scan for secrets (gitleaks, trufflehog)
CLI_Orch → CLI_D/tasks/: Audit infra configs (tfsec, checkov)

[All CLIs start work]

CLI_B → CLI_Orch/IM: "Found 47 vulnerable dependencies. Should I generate fix PRs?"
CLI_Orch → Desktop/IM: "CLI_B found 47 vulns. Auto-generate fixes?"
Desktop → CLI_Orch/IM: "Yes, but only for HIGH/CRITICAL"
CLI_Orch → CLI_B/IM: "Generate fixes for HIGH/CRITICAL only"

CLI_C → CLI_Orch/IM: "Found AWS keys in old commit. Should I rotate?"
CLI_Orch → Desktop/IM: "URGENT: CLI_C found AWS keys in git history"
Desktop → CLI_Orch/IM: "Rotating keys now. Tell CLI_C to document locations"
CLI_Orch → CLI_C/IM: "Keys being rotated. Document all locations found"

[All CLIs complete]

CLI_Orch collects all results
CLI_Orch synthesizes:
- Executive summary
- Findings by severity
- Remediation steps
- Fix PRs generated
- Manual actions needed

CLI_Orch → Desktop coordination/completed/req_1234/
  ├── response.md (formal report)
  ├── findings.json (structured data)
  └── attachments/
      ├── static_analysis_report.html
      ├── dependency_vulnerabilities.csv
      ├── secret_locations.txt
      └── infrastructure_issues.md
```

---

## Communication Channel Decision Tree

```
Question to answer or decision to make?
│
├─ Can CLI answer itself? (has info/authority)
│  └─> YES: Proceed
│  └─> NO: ↓
│
├─ Simple clarification? (takes <30s to answer)
│  ├─> YES + iTerm2 monitoring available
│  │   └─> Print to terminal, Desktop answers via API
│  └─> YES + iTerm2 not available
│      └─> instant_messaging
│
├─ Complex discussion? (multi-turn, nuanced)
│  └─> instant_messaging (preserves context)
│
├─ Permission/approval needed?
│  └─> instant_messaging (creates audit trail)
│
├─ Between two CLIs?
│  └─> instant_messaging (no terminal access)
│
├─ Urgent blocker?
│  ├─> Desktop ↔ CLI + iTerm2: Terminal + IM both
│  └─> CLI ↔ CLI: instant_messaging
│
├─ Progress update?
│  └─> notifications (batched, non-blocking)
│
└─ Rich context sharing?
   └─> note_passing (formatted, with attachments)
```

---

## Implementation Checklist

### For Desktop Claude

**When assigning complex task:**
```markdown
1. [ ] Assess if task needs orchestration
2. [ ] If yes, specify in task: "This requires orchestration - establish comm paths"
3. [ ] Monitor instant_messaging/active/ for questions
4. [ ] If using iTerm2 API, also monitor terminal for blockers
5. [ ] Respond to questions promptly via appropriate channel
6. [ ] Review peer review holds in tasks/incoming/
```

**When monitoring CLI:**
```markdown
1. [ ] Check notifications/pending/ every pulse cycle
2. [ ] Check instant_messaging/active/ if questions expected
3. [ ] If iTerm2 monitoring active, watch for error patterns
4. [ ] Escalate stuck/blocked CLIs if no progress in N minutes
```

### For CLI Instances

**When starting task:**
```markdown
1. [ ] Read task specification completely
2. [ ] Identify potential blockers/questions
3. [ ] If orchestration required, initialize as orchestrator
4. [ ] Establish comm paths to subordinates (if orchestrating)
5. [ ] Begin work, monitoring for blockers
```

**When hitting blocker:**
```markdown
1. [ ] Assess: Can I solve this myself?
2. [ ] If no: Write to instant_messaging/active/{task_id}/
3. [ ] Include: Context, blocker details, what you need
4. [ ] Wait for response (poll every 30s)
5. [ ] Continue when unblocked
```

**When orchestrating:**
```markdown
1. [ ] Initialize orchestrator identity
2. [ ] Register all subordinates
3. [ ] Establish bidirectional comm with each
4. [ ] Create escalation path to Desktop
5. [ ] Monitor subordinate instant_messaging continuously
6. [ ] Answer questions or escalate appropriately
7. [ ] Synthesize results when all complete
```

---

## Future Enhancements

### Automated Escalation

**Pattern detection:**
```python
# If CLI hasn't responded to question in 5 minutes
if time_since_question > 300:
    escalate_to_higher_authority()
```

### Smart Routing

**Question classification:**
```python
question_type = classify_question(question_text)
if question_type == "permission":
    route_to_desktop()  # Only Desktop can grant permissions
elif question_type == "technical":
    route_to_peer_cli()  # Another CLI might know
elif question_type == "clarification":
    route_to_orchestrator()  # Orch has context
```

### Communication Analytics

**Track patterns:**
```python
# Which questions are most common?
# Which blockers delay work most?
# Which comm channels most effective?
# Optimize based on data
```

---

## Related Documentation

- **Three-Layer Architecture:** `ai_communication_architecture_v1.md`
- **AI Comms Structure:** `~/Documents/AI/ai_root/ai_comms/README.md`
- **Task Coordination Protocol v7.0:** `~/Documents/AI/ai_root/ai_general/docs/30_protocols/protocol_taskCoordination_latest.yml`
- **Daemon Architecture:** `daemon_architecture_v1.md`

---

**Document Size:** ~14KB  
**Status:** Design specification, ready for implementation  
**Next Steps:** Build orchestration examples, implement iTerm2 monitoring
