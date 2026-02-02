# Gemini Librarian Operations Playbook

## Metadata
- **Version**: 1.0
- **Created**: 2026-01-31
- **Author**: Claude (Desktop)
- **Status**: Active
- **Prerequisite**: ai_memories/librarian/README.md for architecture overview

## Summary

Operational procedures for the Gemini Librarian system — a semantic search layer over PianoMan's AI conversation archive using Gemini's 1M context window. This playbook covers shard initialization, checkpoint management, query dispatch, and troubleshooting.

## When to Use

- Setting up librarian shards for the first time
- Adding new conversation history to the corpus
- Debugging failed shard queries
- Recovering from corrupted checkpoints

## Prerequisites

- Gemini CLI installed (`gemini` binary in PATH)
- Desktop Commander MCP available
- Corpus files built in `ai_memories/50_shards/`
- Shard manifest at `ai_memories/librarian/shard_manifest.yml`

---

## Procedure 1: Initialize a Single Shard

Creates a Gemini checkpoint with corpus loaded for later resume.

### Steps

1. **Start tmux session with Gemini**
   ```bash
   cd /Users/shawnhillis/Documents/AI/ai_root
   tmux new-session -d -s shard-NN-init \
     'gemini --yolo --prompt-interactive "Read the ENTIRE file ai_memories/50_shards/shard-NN_YYYYMMDD-YYYYMMDD.yml into your context. Report byte count when done."'
   ```

2. **Wait for file read to complete**
   - ~15-20 seconds per MB of corpus
   - 3.5MB file ≈ 60-90 seconds
   ```bash
   sleep 90
   ```

3. **Verify file was read**
   ```bash
   tmux capture-pane -t shard-NN-init -p | tail -30
   ```
   Look for: "Total Byte Count: X,XXX,XXX bytes" and idle prompt.

4. **Save checkpoint**
   ```bash
   tmux send-keys -t shard-NN-init '/chat save shard-NN' Enter
   sleep 3
   tmux send-keys -t shard-NN-init Enter  # Confirm if overwrite prompted
   sleep 5
   ```

5. **Verify checkpoint saved**
   ```bash
   tmux capture-pane -t shard-NN-init -p | grep -i checkpoint
   ```
   Expected: "Conversation checkpoint saved with tag: shard-NN"

6. **Kill init session**
   ```bash
   tmux kill-session -t shard-NN-init
   ```

### Critical Notes

- **Double Enter**: tmux send-keys sometimes needs a second Enter to submit
- **Overwrite prompt**: If checkpoint exists, Gemini asks to confirm overwrite
- **Timing matters**: Save too early = incomplete corpus; save too late = wasted time

---

## Procedure 2: Verify All Checkpoints

Confirms all shards have valid checkpoints saved.

### Steps

1. **Start a temporary Gemini session**
   ```bash
   tmux new-session -d -s checkpoint-verify \
     'gemini --yolo --prompt-interactive "standby"'
   sleep 5
   ```

2. **List saved checkpoints**
   ```bash
   tmux send-keys -t checkpoint-verify '/chat list' Enter
   sleep 3
   tmux capture-pane -t checkpoint-verify -p | grep shard-
   ```

3. **Compare against manifest**
   Expected checkpoints: shard-01 through shard-16 (or current count)

4. **Re-initialize any missing shards** using Procedure 1

5. **Cleanup**
   ```bash
   tmux kill-session -t checkpoint-verify
   ```

---

## Procedure 3: Query a Shard (Manual Test)

Tests that a shard checkpoint restores and responds to queries.

### Steps

1. **Start session and restore checkpoint**
   ```bash
   tmux new-session -d -s shard-test \
     'gemini --yolo --prompt-interactive "standby"'
   sleep 3
   tmux send-keys -t shard-test '/chat resume shard-NN' Enter
   sleep 15  # Wait for checkpoint restore
   ```

2. **Verify restore completed**
   ```bash
   tmux capture-pane -t shard-test -p | tail -20
   ```
   Look for: restored context size (should be 200-400MB)

3. **Send test query**
   ```bash
   tmux send-keys -t shard-test 'What topics are covered in this corpus?' Enter
   sleep 45  # Wait for response
   ```

4. **Capture response**
   ```bash
   tmux capture-pane -t shard-test -p -S -100
   ```

5. **Cleanup**
   ```bash
   tmux kill-session -t shard-test
   ```

---

## Procedure 4: LO Query Dispatch (Programmatic)

How the Librarian Orchestrator dispatches queries to shards.

### Pattern

```bash
# For each shard to query (can parallelize):

# 1. Start session
tmux new-session -d -s shard-NN-query -c /Users/shawnhillis/Documents/AI/ai_root \
  'gemini --yolo --prompt-interactive "standby"'

# 2. Restore checkpoint
sleep 2
tmux send-keys -t shard-NN-query '/chat resume shard-NN' Enter

# 3. Wait for restore
sleep 15

# 4. Send query
tmux send-keys -t shard-NN-query "YOUR QUERY TEXT HERE" Enter

# 5. Wait for response
sleep 45

# 6. Capture output
tmux capture-pane -t shard-NN-query -p -S -200 > /tmp/shard-NN-output.txt

# 7. Kill session
tmux kill-session -t shard-NN-query
```

### Parallelization

Launch steps 1-4 for all target shards, then wait, then capture all outputs.

---

## Procedure 5: Rebuild Corpus After New History

When new conversations are added to 40_histories/.

### Steps

1. **Run corpus builder**
   ```bash
   cd /Users/shawnhillis/Documents/AI/ai_root/ai_memories/librarian
   python3 build_corpus.py
   ```

2. **Check which shards changed**
   - Usually only the last shard grows
   - Rarely, shard boundaries shift requiring multiple re-inits

3. **Re-initialize affected shards** using Procedure 1

4. **Update manifest status** if needed
   - Set re-initialized shards to `status: active`

---

## Troubleshooting

### "Invalid session identifier" when using --resume

**Cause**: `--resume` uses session numbers/UUIDs, not checkpoint tags.

**Solution**: Use `/chat resume <tag>` from inside a running Gemini session.

### Checkpoint save shows in input but doesn't execute

**Cause**: tmux send-keys Enter didn't submit.

**Solution**: Send Enter again:
```bash
tmux send-keys -t SESSION_NAME Enter
```

### "Tool execution denied by policy" during file read

**Cause**: Missing `--yolo` flag or MCP server not configured.

**Solution**: Ensure `--yolo` is passed and `.gemini/settings.json` has correct MCP config.

### Shard response times out

**Cause**: Complex query + large corpus = slow response.

**Solution**: 
- Increase wait time (up to 90s for complex queries)
- Or query fewer shards in parallel to reduce load

### Context shows 0% after restore

**Cause**: Checkpoint didn't actually save the corpus.

**Solution**: Re-initialize the shard from scratch (Procedure 1).

---

## Key Files

| File | Purpose |
|------|---------|
| `ai_root/GEMINI.md` | Shard instructions (auto-loaded) |
| `ai_root/.gemini/settings.json` | Gemini config (compression disabled) |
| `ai_memories/librarian/shard_manifest.yml` | Shard inventory |
| `ai_memories/librarian/lo_prompt.md` | LO instructions |
| `ai_memories/librarian/build_corpus.py` | Corpus builder |
| `ai_memories/librarian/init_all_shards.sh` | Bulk initialization |
| `ai_memories/50_shards/shard-NN_*.yml` | Corpus files |

---

## Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-01-31 | Initial version based on shard-01 initialization learnings |
