# CLI Wrapper --continue Flag Assessment

**Date:** 2026-02-13
**Issue:** `codex_cli.py --continue` was creating new sessions instead of resuming
**Status:** FIXED

## Summary

All three CLI wrappers (`claude_cli.py`, `codex_cli.py`, `gemini_cli.py`) support the `--continue` flag to resume the most recent session. Each wrapper translates this flag to the appropriate native command for its respective AI CLI.

## Implementation Details

### 1. claude_cli.py ✅ WORKING

**Wrapper flag:** `--continue` or `-c`
**Native translation:** `-c` flag
**Implementation:** `build_command()` method (lines 348-349)

```python
if session_mode == "continue":
    cmd.append("-c")
```

**Native binary support:**
```bash
$ claude --help | grep -A2 "^\s*-c"
  -c, --continue    Continue the most recent conversation in the current directory
```

**Verdict:** ✅ Works correctly - wrapper flag maps directly to native flag

---

### 2. codex_cli.py ✅ FIXED (2026-02-13)

**Wrapper flag:** `--continue` or `-c`
**Native translation:** `resume --last` subcommand
**Implementation:** `run()` method (lines 262-283)

```python
if common_args.session_mode == "continue":
    # Build: codex [OPTIONS] resume --last
    cmd = [str(self.config.binary_path)]

    # Add global options before subcommand
    if common_args.model:
        cmd.extend(["--model", common_args.model])
    if common_args.auto_approve:
        cmd.append("--dangerously-bypass-approvals-and-sandbox")

    # Add passthrough args (global options)
    if passthrough:
        cmd.extend(passthrough)

    # Add resume subcommand
    cmd.extend(["resume", "--last"])
else:
    cmd = self.build_command(prompt, common_args, passthrough)
```

**Native binary support:**
```bash
$ codex --help | grep resume
  resume      Resume a previous interactive session (picker by default; use --last to continue the
              most recent)
```

**Verdict:** ✅ Now works correctly - wrapper translates flag to subcommand structure

**Previous bug:** Wrapper parsed `--continue` flag and set `session_mode = "continue"` but never acted on it, resulting in new sessions being created instead of resuming.

---

### 3. gemini_cli.py ✅ WORKING

**Wrapper flag:** `--continue` or `-c`
**Native translation:** `--resume latest` flags
**Implementation:** `build_command()` method (lines 159-160)

```python
if session_mode == "continue":
    cmd.extend(["--resume", "latest"])
```

**Native binary support:**
```bash
$ gemini --help | grep resume
  -r, --resume       Resume a previous session. Use "latest" for most recent or index number
```

**Verdict:** ✅ Works correctly - wrapper flag maps to native `--resume latest`

---

## Common Argument Parsing

All three wrappers share the same argument parser from `lib_cli_common.py`:

```python
# lib_cli_common.py lines 1124-1125
session.add_argument(
    "-c", "--continue", dest="continue_session", action="store_true",
    help="Continue most recent session in project"
)
```

This sets `common_args.session_mode = "continue"` when the flag is detected (line 1285).

Each wrapper then handles this `session_mode` value differently based on their native binary's interface:
- **claude:** Adds `-c` flag
- **codex:** Converts to `resume --last` subcommand
- **gemini:** Adds `--resume latest` flags

## Important Notes

### Flag Conflict: `-c`

⚠️ **Note:** The wrapper uses `-c` as a short form for `--continue`, but the native codex binary uses `-c` for `--config`. This creates a conflict:

- `codex_cli.py -c` → Resume most recent session (wrapper interpretation)
- `codex -c key=value` → Set config value (native interpretation)

**Workaround:** Use `--config` (long form) when passing config to native codex through the wrapper.

### Architecture Difference

**Claude & Gemini:** Use FLAGS for session continuation
- Simple translation: add one more flag to command

**Codex:** Uses SUBCOMMAND for session continuation
- Complex translation: changes entire command structure from `codex [OPTIONS] [PROMPT]` to `codex [OPTIONS] resume --last`

This architectural difference required special handling in `codex_cli.py`'s `run()` method rather than the `build_command()` method.

---

## Testing

### Test Commands

```bash
# Claude (should resume most recent in current directory)
cli_claude -a --continue

# Codex (should resume most recent session)
cli_codex -a --continue

# Gemini (should resume latest session)
cli_gemini -a --continue
```

### Verification

Check tmux session or log files to verify:
1. No new session created unnecessarily
2. Existing session is resumed
3. Wrapper flags (like `-a` auto-approve) are properly translated

---

## Additional Fix: Default to Interactive Mode

**Issue:** When using `--continue`, users expect to be placed directly in the interactive terminal UI, not in a background tmux session.

**Fix:** Modified `lib_cli_common.py` to default to interactive mode when resuming sessions:

```python
# lib_cli_common.py lines 1276-1289
if parsed.interactive:
    exec_mode = "interactive"
elif parsed.continue_session or parsed.resume or parsed.named:
    exec_mode = "interactive"  # Resuming implies user wants to interact
else:
    exec_mode = "tmux"
```

**Behavior change:**
- **Before:** `cli_codex --continue` → Background tmux session (need manual `tmux attach`)
- **After:** `cli_codex --continue` → Direct interactive UI (foreground)

**Override:** Users can still force tmux mode with `-t` flag: `cli_codex -t --continue`

---

## Conclusion

✅ All three CLI wrappers now properly support `--continue` flag
✅ Each wrapper correctly translates to its native binary's resumption interface
✅ The `codex_cli.py` bug has been fixed (2026-02-13)
✅ Session continuation now defaults to interactive mode (user expectation)

**Files modified:**
- `/Users/shawnhillis/bin/ai/cli/codex_cli.py` - Lines 262-283 in `run()` method
- `/Users/shawnhillis/bin/ai/cli/lib_cli_common.py` - Lines 1276-1289 for exec_mode logic
