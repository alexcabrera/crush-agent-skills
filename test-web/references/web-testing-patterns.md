# Web Testing Patterns

This reference documents common patterns for testing web applications with agent-browser.

## The Snapshot-Ref Pattern

The core pattern for AI-driven web testing:

```bash
# 1. Navigate to page
agent-browser open "http://localhost:3000"

# 2. Get accessibility snapshot (returns refs like @e1, @e2)
agent-browser snapshot -i

# 3. Interact using refs
agent-browser click @e3
agent-browser fill @e5 "user@example.com"

# 4. Re-snapshot to get new refs
agent-browser snapshot -i
```

**Why this pattern:**
- Accessibility tree is AI-friendly (structured, not raw HTML)
- Refs are deterministic (point to exact element from snapshot)
- No DOM knowledge required (unlike CSS selectors)

## Batch Mode for Speed

Multiple commands in one call to avoid process startup overhead:

```bash
echo '[["open", "http://localhost:3000"], ["snapshot", "-i"], ["click", "@e3"]]' | agent-browser batch --json
```

## Waiting for Conditions

```bash
# Wait for element to appear
agent-browser wait --selector "#result" --timeout 10

# Wait for text content
agent-browser wait --text "Success" --timeout 5

# Wait for URL pattern
agent-browser wait --url "/dashboard"
```

## Form Testing

```bash
# Fill and submit form
agent-browser fill @email "test@example.com"
agent-browser fill @password "secret123"
agent-browser click @submit

# Verify result
agent-browser snapshot -i | grep -q "Welcome"
```

## Screenshot for Debugging

```bash
# Capture screenshot on failure
agent-browser screenshot failure.png

# Annotated screenshot (shows element refs)
agent-browser screenshot --annotate annotated.png
```

## Authentication Patterns

### Using Persistent Profiles

```bash
# Create profile (preserves cookies, localStorage)
agent-browser profile create --name my-auth

# Use profile
agent-browser open "http://app.com" --profile my-auth

# Save session state
agent-browser state save auth-state.json
```

### Login Flow

```bash
# Navigate to login
agent-browser open "http://app.com/login"
agent-browser snapshot -i

# Fill credentials
agent-browser fill @email "user@example.com"
agent-browser fill @password "password"
agent-browser click @login-btn

# Wait for redirect
agent-browser wait --url "/dashboard" --timeout 10
```

## Multi-Page Testing

```bash
# List tabs
agent-browser tabs

# Switch tabs
agent-browser tab switch 1

# Open new tab
agent-browser tab new "http://app.com/settings"
```

## API Testing with Network Interception

```bash
# Mock API responses
agent-browser route mock "/api/user" --response '{"name": "Test"}'

# Block requests
agent-browser route block "/api/analytics"

# Verify mocked data
agent-browser open "http://app.com/profile"
agent-browser snapshot -i | grep -q "Test"
```

## Error Detection

```bash
# Capture console errors
agent-browser console

# Check for specific errors
agent-browser console | grep -i "error\|failed"

# Check page for error text
agent-browser snapshot -i | grep -i "error\|failed"
```

## Common Selectors

When refs aren't available, use semantic selectors:

```bash
# By ARIA role
agent-browser click --role button --name "Submit"

# By text content
agent-browser click --text "Sign In"

# By placeholder
agent-browser fill --placeholder "Email" "test@example.com"

# By label
agent-browser click --label "Username"

# By data-testid
agent-browser click --testid "submit-btn"
```

## Cleanup Pattern

Always clean up after tests:

```bash
# Close browser
agent-browser close

# Or if using profile, clear it
agent-browser profile delete my-auth
```
