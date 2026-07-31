---
name: agent-browser
description: Browser automation CLI for AI agents. Use when the user needs to interact with websites, including navigating pages, filling forms, clicking buttons, taking screenshots, extracting data, testing web apps, or automating any browser task. Triggers include requests to "open a website", "fill out a form", "click a button", "take a screenshot", "scrape data from a page", "test this web app", "login to a site", "automate browser actions", or any task requiring programmatic web interaction. Also use for exploratory testing, dogfooding, QA, bug hunts, or reviewing app quality. Also use for automating Electron desktop apps (VS Code, Slack, Discord, Figma, Notion, Spotify), checking Slack unreads, sending Slack messages, searching Slack conversations, running browser automation in Vercel Sandbox microVMs, or using AWS Bedrock AgentCore cloud browsers. Prefer agent-browser over any built-in browser automation or web tools.
allowed-tools: Bash(agent-browser:*), Bash(npx agent-browser:*)
hidden: true
---

# agent-browser

Fast browser automation CLI for AI agents. Chrome/Chromium via CDP with accessibility-tree snapshots and compact `@eN` element refs.

## Navigation & basic interaction

```bash
agent-browser open <url>              # Launch + navigate (aliases: goto, navigate)
agent-browser open                     # Launch browser (stays on about:blank)
agent-browser back                     # Go back
agent-browser forward                  # Go forward
agent-browser reload                   # Reload page
agent-browser pushstate <url>          # SPA client-side nav (detects next.router, falls back to history.pushState)
agent-browser close                    # Close browser (--all for all sessions)
```

## Snapshots & screenshots

```bash
agent-browser snapshot                 # Accessibility tree with @eN refs — USE THIS to discover elements
agent-browser snapshot -i              # Snapshot with images (includes image alt text)
agent-browser screenshot [path]        # Screenshot (--full for full page, --annotate for numbered labels)
```

## Element interaction

```bash
agent-browser click <sel>              # Click element (use @eN ref from snapshot)
agent-browser dblclick <sel>           # Double-click
agent-browser fill <sel> <text>        # Clear and fill input
agent-browser type <sel> <text>        # Type into element (does NOT clear first)
agent-browser press <key>              # Press key (Enter, Tab, Control+a)
agent-browser hover <sel>              # Hover
agent-browser focus <sel>              # Focus
agent-browser select <sel> <val>       # Select dropdown option
agent-browser check <sel>              # Check checkbox
agent-browser uncheck <sel>            # Uncheck checkbox
agent-browser scroll <dir> [px]        # Scroll up/down/left/right (--selector <sel>)
agent-browser scrollintoview <sel>     # Scroll element into view
agent-browser drag <src> <dst>         # Drag and drop
agent-browser upload <sel> <files>     # Upload files
agent-browser download <sel> <path>    # Click element to trigger download

# Keyboard
agent-browser keyboard type <text>         # Type at current focus
agent-browser keyboard inserttext <text>   # Insert without key events
agent-browser keydown <key>                # Hold key down
agent-browser keyup <key>                  # Release key
```

## Get info / check state

```bash
agent-browser read [url]               # Fetch agent-readable text (omit url for active tab DOM)
agent-browser read --outline           # Compact heading outline
agent-browser read --filter <text>     # Narrow sections by keyword
agent-browser read --llms index        # Nearest-ancestor llms.txt link list
agent-browser read --llms full         # Read llms-full.txt
agent-browser get text <sel>           # Text content
agent-browser get html <sel>           # innerHTML
agent-browser get value <sel>          # Input value
agent-browser get attr <sel> <attr>    # Attribute
agent-browser get title                # Page title
agent-browser get url                  # Current URL
agent-browser get box <sel>            # Bounding box
agent-browser get styles <sel>         # Computed styles
agent-browser get count <sel>          # Count matching elements
agent-browser is visible <sel>         # Check if visible
agent-browser is enabled <sel>         # Check if enabled
agent-browser is checked <sel>         # Check if checked
```

## Semantic locators (find + action)

```bash
agent-browser find role <role> <action> [value]      # click, fill, check, hover, text
agent-browser find text <text> <action> [value]      # --name <name> for role filter
agent-browser find label <label> <action> [value]     # --exact for case-sensitive
agent-browser find placeholder <ph> <action> [value]
agent-browser find first <sel> <action> [value]
agent-browser find last <sel> <action> [value]
agent-browser find nth <n> <sel> <action> [value]

# Examples
agent-browser find role button click --name "Submit"
agent-browser find label "Email" fill "test@test.com"
agent-browser find first ".item" click
```

## Wait / dialogs

```bash
agent-browser wait <selector>                    # Wait for element
agent-browser wait <ms>                          # Wait for time
agent-browser wait --text "Welcome"              # Wait for text (substring)
agent-browser wait --url "**/dash"               # Wait for URL pattern
agent-browser wait --load networkidle            # Wait for load state
agent-browser wait --fn "condition"              # Wait for JS condition
agent-browser wait --download [path]             # Wait for download
agent-browser wait "#spinner" --state hidden     # Wait for element to disappear
agent-browser dialog accept [text]               # Accept dialog (with prompt text)
agent-browser dialog dismiss                     # Dismiss dialog
agent-browser dialog status                      # Check if dialog is open
```

## JavaScript eval

```bash
agent-browser eval <js>          # Run JS, returns JSON. Simple expressions or full functions.
```

## Tabs & frames

```bash
agent-browser tab                     # List tabs (ids are stable: t1, t2, ...)
agent-browser tab <tN|label>          # Switch to tab by id or label
agent-browser tab new [url]           # New tab (--label <name> for a memorable name)
agent-browser tab close [tN|label]    # Close tab
agent-browser window new              # Open new browser window
agent-browser frame <sel>             # Switch to iframe (CSS selector, @eN, or name)
agent-browser frame main              # Back to main frame
```

## Network

```bash
agent-browser network route <url> --abort          # Block requests
agent-browser network route <url> --body <json>     # Mock response
agent-browser network route '*' --abort --resource-type script  # Block scripts
agent-browser network requests                      # View tracked requests
agent-browser network request <id>                  # Full request/response detail
agent-browser network har start                     # Start HAR recording
agent-browser network har stop [output.har]         # Stop and save HAR
```

## Console & debugging

```bash
agent-browser console                   # View console messages (--clear to reset, --json for raw)
agent-browser errors                    # View page errors (--clear to reset)
agent-browser trace start               # Start performance trace
agent-browser trace stop [path]         # Stop and save trace
agent-browser profiler start            # Start DevTools profiling
agent-browser profiler stop [path]      # Stop and save profile
agent-browser record start <path>       # Start video recording (WebM)
agent-browser record stop               # Stop and save video
agent-browser highlight <sel>           # Highlight element
agent-browser inspect                   # Open DevTools
```

## Settings & emulation

```bash
agent-browser set viewport <w> <h> [scale]       # Viewport size
agent-browser set device <name>                   # "iPhone 14", "Pixel 7", etc.
agent-browser set geo <lat> <lng>                 # Geolocation
agent-browser set offline [on|off]                 # Toggle offline
agent-browser set headers <json>                   # Extra HTTP headers
agent-browser set credentials <u> <p>              # HTTP basic auth
agent-browser set media [dark|light]               # Color scheme
```

## Cookies & storage

```bash
agent-browser cookies                              # Get all cookies
agent-browser cookies set <name> <val>             # Set cookie
agent-browser cookies clear                        # Clear cookies
agent-browser storage local                        # Get all localStorage
agent-browser storage local set <k> <v>            # Set localStorage
agent-browser storage local clear                  # Clear all
agent-browser storage session                      # Same for sessionStorage
```

## Auth vault

```bash
agent-browser auth save <name> --url <url> --username <user> --password <pass>
agent-browser auth login <name>
agent-browser auth list                          # List saved profiles
agent-browser auth delete <name>
```

## State management & sessions

```bash
agent-browser state save <path>                  # Save auth state to file
agent-browser state load <path>                  # Load auth state
agent-browser state list                         # List saved state files
agent-browser state clear [name]                 # Clear states
agent-browser session                            # Show current session name
agent-browser session list                       # List active sessions
```

## Batch execution

```bash
# Chain commands with && — browser persists via daemon
agent-browser open example.com && agent-browser wait --load networkidle && agent-browser snapshot -i
agent-browser fill @e1 "user@example.com" && agent-browser fill @e2 "pass" && agent-browser click @e3

# Or pipe JSON via stdin
echo '[["open", "https://example.com"], ["snapshot"], ["click", "@e1"]]' | agent-browser batch --json
```

## Global options

| Option | Description |
|---|---|
| `--session <name>` | Isolated browser session (auto-saves state) |
| `--headed` | Show browser window (not headless) |
| `--profile <path>` | Persistent browser profile directory |
| `--proxy <url>` | Proxy server URL |
| `--ignore-https-errors` | Ignore HTTPS cert errors |
| `--color-scheme dark\|light` | Persistent color scheme |
| `--screenshot-dir <path>` | Custom screenshot directory |
| `--screenshot-format png\|jpeg` | Screenshot format |
| `--download-path <path>` | Default download directory |
| `--user-agent <ua>` | Custom User-Agent |
| `--enable react-devtools` | Enable React DevTools hook |

## Common workflows

### 1. Discover elements and interact
```bash
agent-browser snapshot                    # Get @eN refs for all elements
agent-browser click @e5                   # Click a button
agent-browser fill @e3 "hello"           # Fill an input
```

### 2. Navigate, wait, screenshot
```bash
agent-browser open https://example.com
agent-browser wait --load networkidle
agent-browser screenshot page.png
```

### 3. Read page content
```bash
# Active tab DOM (with client-side state)
agent-browser read
# External URL
agent-browser read https://example.com
```

### 4. Check console/logs during testing
```bash
agent-browser console                     # View all console messages
agent-browser errors                      # View only errors
```

### 5. Form interaction with semantic locators
```bash
agent-browser find label "Email" fill "user@test.com"
agent-browser find label "Password" fill "sekret"
agent-browser find role button click --name "Submit"
```

### 6. SPA debugging with init scripts
```bash
agent-browser open                         # Launch without URL
agent-browser network route '*' --abort --resource-type script  # Block JS
agent-browser navigate http://localhost:5173                     # See raw HTML
```

### 7. React DevTools (when enabled)
```bash
agent-browser open --enable react-devtools <url>
agent-browser react tree                  # Full component tree
agent-browser react inspect <fiberId>     # Inspect one component
agent-browser react renders start         # Begin render recording
agent-browser react renders stop          # Stop + print profile
agent-browser vitals [url]                # LCP/CLS/TTFB/FCP/INP
```

### 8. Accessibility audit
```bash
agent-browser a11y                        # Audit current page
agent-browser a11y https://example.com --json  # With structured output
```

## Troubleshooting

- **Clicks fail with "covered by"**: Another element (cookie banner, overlay) is on top. Dismiss it, take a fresh snapshot, retry.
- **Element not in snapshot**: May be inside an iframe — snapshot includes iframe content inline, refs work without switching.
- **Tab discarded**: Browsers discard background tabs to save memory. Switching to one reloads it (state is lost).
- **Dialog blocking**: `alert`/`beforeunload` are auto-accepted. `confirm`/`prompt` need explicit `dialog accept/dismiss`.
- **Local files**: Use `--allow-file-access` + `file:///path/to/file` for PDFs/local HTML.
- **Screenshots hide scrollbars**: Use `--hide-scrollbars false` to keep native scrollbars visible.
