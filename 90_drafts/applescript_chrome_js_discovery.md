# AppleScript Chrome JavaScript Execution Discovery

**Date:** 2025-12-05
**Status:** Verified working
**Implication:** Potential Puppeteer replacement for chat orchestration

## Discovery

Chrome's AppleScript dictionary includes `execute javascript` command that works WITHOUT:
- Chrome DevTools Protocol (CDP)
- `--remote-debugging-port=9222`
- Separate debug Chrome instance

It works directly on the user's normal Chrome session.

## Verified Capabilities

| Operation | Works | Method |
|-----------|-------|--------|
| Read page title | ✅ | `execute javascript "document.title"` |
| Read page URL | ✅ | `execute javascript "window.location.href"` |
| Query DOM | ✅ | `execute javascript "document.querySelectorAll(...)"` |
| Read element content | ✅ | `execute javascript "element.textContent"` |
| Focus elements | ✅ | `execute javascript "element.focus()"` |
| Type text | ✅ | `System Events keystroke` after JS focus |
| Clear input | ✅ | `Cmd+A, Delete` via System Events |
| Click buttons | ✅ | `execute javascript "button.click()"` |
| Read message content | ✅ | Query message containers |

## Basic Pattern

```applescript
-- Read from page
tell application "Google Chrome"
    tell active tab of window 1
        set result to execute javascript "document.title"
    end tell
end tell

-- Type into page (hybrid approach)
tell application "Google Chrome"
    activate
    tell active tab of window 1
        execute javascript "document.querySelector('[contenteditable]').focus()"
    end tell
end tell
delay 0.2
tell application "System Events"
    keystroke "Hello world"
end tell
```

## Advantages Over Puppeteer

1. **No setup required** - Works with user's normal Chrome
2. **Already authenticated** - User's logged-in sessions available
3. **Simpler architecture** - No CDP, no debug port, no separate profile
4. **Native macOS** - Uses standard system APIs

## Disadvantages

1. **macOS only** - Puppeteer is cross-platform
2. **Must be visible** - Can't run headless
3. **Timing fragility** - Keystroke delays need tuning
4. **Accessibility permissions** - System Events requires permission
5. **Harder debugging** - No network/console inspection

## Implications for Chat Orchestrator

Could create AppleScript-based orchestrator that:
- Works with any Chrome tab (no debug instance)
- Uses existing logged-in sessions
- Dramatically simpler setup
- Could be shell script + osascript (no Node.js needed)

## Next Steps

- [ ] Prototype AppleScript-based message sender
- [ ] Test response detection reliability
- [ ] Compare timing/reliability with Puppeteer approach
- [ ] Evaluate hybrid approach (AppleScript for simple, Puppeteer for complex)

## Related

- Chrome Control MCP uses this same mechanism
- Puppeteer implementation: `chat_orchestrator_puppeteer/`
- Original discovery conversation: This chat
