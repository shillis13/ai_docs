# Directory Watcher Guide - Condensed

**Status:** Cron-based polling (60s latency) - works fine for task notifications

## Current Implementation (macOS)
```bash
* * * * * ~/bin/ai/send_scheduled_prompt.sh
* * * * * ~/bin/ai/process_notifications_cron.sh
*/5 * * * * ~/bin/ai/watchdog_pulse.sh
```

**Watches:**
- `06_announcements/to_desktop_claude/*.prompt.txt`
- `~/.claude/notifications/pending/*.txt`
- `~/.claude/daemon.pid`

## Installation
```bash
~/bin/ai/install_cron_jobs.sh
crontab -l | grep "AI Coordination"
tail -f ~/.claude/cron.log
```

## Alternatives (Future)

| Approach | Latency | Cross-Platform | Notes |
|----------|---------|----------------|-------|
| Cron | 60s | Yes (concept) | Current, simple |
| fswatch | <1s | WSL only | `brew install fswatch` |
| Python watchdog | <1s | True | Most portable |
| Automator | <1s | macOS only | Not recommended |

## If Real-Time Needed
```bash
# fswatch approach
fswatch -0 "$PROMPT_DIR" | while read -d "" event; do
    [[ "$event" == *.prompt.txt ]] && ~/bin/ai/send_scheduled_prompt.sh
done
```

## Windows Port Strategy
1. Convert bash → Python (pathlib, pyautogui)
2. Task Scheduler instead of cron
3. Map paths: `~/.claude` → `AppData/Roaming/Claude`

**Verdict:** Don't complicate until needed. Cron works.
