---
name: test-web
description: Run automated tests for web applications using agent-browser. Use when testing websites, web apps, SPAs, or any application with a browser interface. For CLI/TUI applications, use the test-cli skill instead.
license: MIT
compatibility: Requires agent-browser (Vercel), Chrome/Chromium.
metadata:
  version: "1.0.0"
  author: agent-skills
---

# test-web Skill

Run automated tests for web applications using [agent-browser](https://github.com/vercel-labs/agent-browser) - a headless browser automation CLI built specifically for AI agents.

## When to Use

- Testing web applications (React, Vue, Svelte, etc.)
- Testing SPAs (Single Page Applications)
- Testing static websites
- End-to-end testing of web flows
- Form filling and submission testing
- Screenshot capture and visual verification
- Any application that runs in a browser

**For CLI/TUI applications, use the [test-cli](../test-cli/) skill instead.**

---

## Prerequisites

This skill requires **agent-browser** - Vercel's AI-optimized browser automation tool.

### Installation

```bash
# Via cargo (recommended)
cargo install agent-browser

# Or download prebuilt binary from:
# https://github.com/vercel-labs/agent-browser/releases
```

---

## Quick Reference

```bash
# Basic flow
agent-browser open "https://example.com"
agent-browser snapshot              # Returns accessibility tree with refs (@e1, @e2)
agent-browser click @e1              # Click element by ref
agent-browser fill @e2 "text"        # Fill form field
agent-browser snapshot               # Get new refs after page changes
```

---

## Core Pattern: Snapshot-Ref-Act

agent-browser uses a **ref-based interaction pattern** optimized for AI:

1. **Snapshot** - Get accessibility tree with element refs
2. **Identify** - Find target element by ref (`@e1`, `@e2`, etc.)
3. **Act** - Click, fill, or interact using the ref
4. **Re-snapshot** - After page changes, get new refs

This is deterministic (no CSS selectors needed) and AI-friendly (accessibility tree is easy to parse).

---

## Common Commands

### Navigation

```bash
agent-browser open "https://example.com"
agent-browser back
agent-browser forward
agent-browser reload
```

### Element Interaction

```bash
agent-browser click @e1           # Click element
agent-browser fill @e3 "text"     # Fill input
agent-browser type "hello"        # Type text
agent-browser select @e5 "option" # Select dropdown
agent-browser check @e6           # Check checkbox
```

### Information Retrieval

```bash
agent-browser snapshot            # Accessibility tree with refs
agent-browser snapshot -i         # Interactive elements only
agent-browser snapshot -c         # Compact format
agent-browser screenshot out.png  # Take screenshot
agent-browser text @e1            # Get element text
agent-browser url                 # Get current URL
agent-browser title               # Get page title
```

### Waiting

```bash
agent-browser wait "selector"     # Wait for element
agent-browser wait-url "pattern"  # Wait for URL pattern
agent-browser wait-load           # Wait for page load
```

### Batch Execution

For multiple commands without startup overhead:

```bash
echo '[["open", "https://example.com"], ["snapshot", "-i"], ["click", "@e1"], ["screenshot", "result.png"]]' | agent-browser batch --json
```

---

## Testing Patterns

### Basic Page Load Test

```bash
# Open page and verify content
agent-browser open "http://localhost:3000"
agent-browser wait-load
snapshot=$(agent-browser snapshot -i)

if echo "$snapshot" | grep -q "Welcome"; then
    echo "PASS: Page loaded successfully"
else
    echo "FAIL: Expected 'Welcome' not found"
fi
```

### Form Submission Test

```bash
agent-browser open "http://localhost:3000/login"

# Get snapshot and find form elements
snapshot=$(agent-browser snapshot -i)

# Assuming @e1 is email, @e2 is password, @e3 is submit
agent-browser fill @e1 "test@example.com"
agent-browser fill @e2 "password123"
agent-browser click @e3

# Wait for redirect or success message
agent-browser wait-url "dashboard"
echo "PASS: Login successful"
```

### Navigation Flow Test

```bash
agent-browser open "http://localhost:3000"

# Navigate through app
agent-browser click @nav_products
agent-browser wait-load

# Verify on products page
if agent-browser url | grep -q "products"; then
    echo "PASS: Navigated to products"
fi

# Take screenshot for visual verification
agent-browser screenshot test-results/products-page.png
```

### Visual Regression

```bash
# Capture baseline
agent-browser open "http://localhost:3000"
agent-browser screenshot baseline.png

# After changes, compare
agent-browser open "http://localhost:3000"
agent-browser screenshot current.png

# Use diff tool or agent-browser's diff command
agent-browser diff baseline.png current.png
```

---

## Integration with Virtuous Cycle

When the `execute` skill detects a web application, it will:

1. Start the web server (if needed)
2. Use this skill (test-web) for testing
3. Capture results and verify against acceptance criteria

### App Type Detection

The `execute` skill detects web applications by checking for:

- `package.json` with web frameworks (next, vite, react, vue, svelte, astro)
- `index.html` or `app.html` files
- `go.mod` with `net/http`
- Explicit `type: web` marker in `specs/*/design.md`

---

## Error Handling

```bash
# Check if agent-browser is installed
if ! command -v agent-browser &>/dev/null; then
    echo "FAIL: agent-browser not installed"
    echo "Run: cargo install agent-browser"
    exit 1
fi

# Handle navigation failures
if ! agent-browser open "http://localhost:3000" 2>/dev/null; then
    echo "FAIL: Could not connect to server"
    echo "Is the dev server running?"
    exit 1
fi
```

---

## Tips for AI Agents

1. **Always snapshot before acting** - Get current refs before interaction
2. **Re-snapshot after changes** - Page updates invalidate old refs
3. **Use `-i` flag** - Interactive-only snapshots are more concise
4. **Use batch for speed** - Multiple commands in one call
5. **Check URL for verification** - Often easier than parsing content
6. **Screenshots for debugging** - Capture on failure for diagnosis

---

## Related Skills

- [elaborate](../elaborate/) - Design phase for web applications
- [execute](../execute/) - Autonomous cycle that routes to this skill
- [ui-design](../ui-design/) - UI/UX patterns for web applications
