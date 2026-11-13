---
name: browser-tools
# kept OUT of the model-facing catalog 2026-11-13 by pan at autumn's call (Key 372 closed). Reach it by
# path or /skill:browser-tools. What is measured on THIS box, so the next reader does not re-derive it: the
# deps were never installed until 2026-11-13 (now 210 packages, 125 MB, PUPPETEER_SKIP_DOWNLOAD=1); nothing
# listens on :9222; and Chrome 154 refuses --remote-debugging-port on the DEFAULT profile **in silence** — the
# log was empty and curl got rc=7, while a separate --user-data-dir answered CDP (Protocol 1.3). So this is a
# rendering door into a Chrome I launch, not a door into autumn's logged-in session. browser-start.js cannot run on
# Linux at all: line 48 rsyncs from the macOS profile path and line 55 execs /Applications/Google Chrome.app/…,
# and execSync throws on rsync's non-zero exit, so the script dies before Chrome starts. Read that rsync before
# worrying about it: her profile is the SOURCE and ~/.cache/browser-tools is the destination, so --delete prunes
# the copy and never her profile; the only rm -f in the file (line 33) targets the copy's Singleton* lock files.
# The working shape on this box is my own `google-chrome-stable --remote-debugging-port=<port> --user-data-dir=<dir>`
# plus the six scripts, which only need connect({browserURL}). Whether to add a Linux branch at all is autumn's
# call — it would copy her cookies and logins into ~/.cache/browser-tools, and she has not agreed to that.
# The map that routes me here is ~/.pi/agent/skills/web-dev/SKILL.md.
disable-model-invocation: true
description: Interactive browser automation via Chrome DevTools Protocol. Use when you need to interact with web pages, test frontends, or when user interaction with a visible browser is required.
---

# Browser Tools

Chrome DevTools Protocol tools for agent-assisted web automation. These tools connect to Chrome running on `:9222` with remote debugging enabled.

## Setup

Run once before first use:

```bash
cd {baseDir}    # upstream writes `{baseDir}/browser-tools`, which is wrong once this dir IS the skill dir
npm install     # PUPPETEER_SKIP_DOWNLOAD=1 keeps it to 210 packages and no Chrome download
```

## Start Chrome

```bash
{baseDir}/browser-start.js              # Fresh profile
{baseDir}/browser-start.js --profile    # Copy user's profile (cookies, logins)
```

Launch Chrome with remote debugging on `:9222`. Use `--profile` to preserve user's authentication state.

On Chrome 136+ the flag is refused on the default profile, silently, so `--user-data-dir` is not optional here.
Measured 2026-11-13 on Chrome 154.0.8037.57: default profile → nothing listened, and the log was empty;
`--user-data-dir=/tmp/…` → `Chrome/154.0.8037.57`, `Protocol-Version: 1.3`, a `webSocketDebuggerUrl`.
`browser-start.js` itself is macOS-only (line 48 profile path, line 55 binary path): on Linux it fails before
launching anything. Start Chrome by hand, then use the six scripts against it.

## Navigate

```bash
{baseDir}/browser-nav.js https://example.com
{baseDir}/browser-nav.js https://example.com --new
```

Navigate to URLs. Use `--new` flag to open in a new tab instead of reusing current tab.

## Evaluate JavaScript

```bash
{baseDir}/browser-eval.js 'document.title'
{baseDir}/browser-eval.js 'document.querySelectorAll("a").length'
```

Execute JavaScript in the active tab. Code runs in async context. Use this to extract data, inspect page state, or perform DOM operations programmatically.

## Screenshot

```bash
{baseDir}/browser-screenshot.js
```

Capture current viewport and return temporary file path. Use this to visually inspect page state or verify UI changes.

## Pick Elements

```bash
{baseDir}/browser-pick.js "Click the submit button"
```

**IMPORTANT**: Use this tool when the user wants to select specific DOM elements on the page. This launches an interactive picker that lets the user click elements to select them. The user can select multiple elements (Cmd/Ctrl+Click) and press Enter when done. The tool returns CSS selectors for the selected elements.

Common use cases:
- User says "I want to click that button" → Use this tool to let them select it
- User says "extract data from these items" → Use this tool to let them select the elements
- When you need specific selectors but the page structure is complex or ambiguous

## Cookies

```bash
{baseDir}/browser-cookies.js
```

Display all cookies for the current tab including domain, path, httpOnly, and secure flags. Use this to debug authentication issues or inspect session state.

## Extract Page Content

```bash
{baseDir}/browser-content.js https://example.com
```

Navigate to a URL and extract readable content as markdown. Uses Mozilla Readability for article extraction and Turndown for HTML-to-markdown conversion. Works on pages with JavaScript content (waits for page to load).

## When to Use

- Testing frontend code in a real browser
- Interacting with pages that require JavaScript
- When user needs to visually see or interact with a page
- Debugging authentication or session issues
- Scraping dynamic content that requires JS execution

---

## Efficiency Guide

### DOM Inspection Over Screenshots

**Don't** take screenshots to see page state. **Do** parse the DOM directly:

```javascript
// Get page structure
document.body.innerHTML.slice(0, 5000)

// Find interactive elements
Array.from(document.querySelectorAll('button, input, [role="button"]')).map(e => ({
  id: e.id,
  text: e.textContent.trim(),
  class: e.className
}))
```

### Complex Scripts in Single Calls

Wrap everything in an IIFE to run multi-statement code:

```javascript
(function() {
  // Multiple operations
  const data = document.querySelector('#target').textContent;
  const buttons = document.querySelectorAll('button');
  
  // Interactions
  buttons[0].click();
  
  // Return results
  return JSON.stringify({ data, buttonCount: buttons.length });
})()
```

### Batch Interactions

**Don't** make separate calls for each click. **Do** batch them:

```javascript
(function() {
  const actions = ["btn1", "btn2", "btn3"];
  actions.forEach(id => document.getElementById(id).click());
  return "Done";
})()
```

### Typing/Input Sequences

```javascript
(function() {
  const text = "HELLO";
  for (const char of text) {
    document.getElementById("key-" + char).click();
  }
  document.getElementById("submit").click();
  return "Submitted: " + text;
})()
```

### Reading App/Game State

Extract structured state in one call:

```javascript
(function() {
  const state = {
    score: document.querySelector('.score')?.textContent,
    status: document.querySelector('.status')?.className,
    items: Array.from(document.querySelectorAll('.item')).map(el => ({
      text: el.textContent,
      active: el.classList.contains('active')
    }))
  };
  return JSON.stringify(state, null, 2);
})()
```

### Waiting for Updates

If DOM updates after actions, add a small delay with bash:

```bash
sleep 0.5 && {baseDir}/browser-eval.js '...'
```

### Investigate Before Interacting

Always start by understanding the page structure:

```javascript
(function() {
  return {
    title: document.title,
    forms: document.forms.length,
    buttons: document.querySelectorAll('button').length,
    inputs: document.querySelectorAll('input').length,
    mainContent: document.body.innerHTML.slice(0, 3000)
  };
})()
```

Then target specific elements based on what you find.
