---
name: test
description: Run automated tests for CLI applications, TUIs, and long-running processes. Use when verifying implementations, testing interactive programs, running servers in isolation, or performing integration testing. Provides session management, input simulation, output capture, and pattern-based assertions via tmux isolation.
license: MIT
compatibility: Requires tmux, bash.
metadata:
  version: "2.0.0"
  author: choo-choo-skills
---

# test Skill

Run automated tests for CLI applications, TUIs, and long-running processes in isolated tmux sessions. This is the primary testing tool in the choo-choo workflow.

## ⚠️ CRITICAL: Always Use This Skill, Never Raw tmux

**NEVER run raw tmux commands or blocking bash commands for testing.**

❌ **FORBIDDEN:**
```bash
sleep 15 && tmux -S .tmp/agent.sock capture-pane -p -t test-myapp
tmux -S "$SOCKET" send-keys -t test-app "hello"
sleep 30  # waiting for something
```

Why these are dangerous:
- **Blocks the agent** - Agent cannot proceed until command completes
- **Requires user cancellation** - User must manually interrupt
- **Wrong socket** - May connect to user's tmux instead of isolated test socket
- **No timeout safety** - Can hang indefinitely

✅ **REQUIRED:**
```bash
test wait myapp "Ready" 30        # Non-blocking wait with timeout
test capture myapp                # Instant capture
test run app "./app" -- --wait "Ready" --expect "Menu"  # Full lifecycle
```

**The test skill commands are non-blocking and have built-in timeouts.**

## When to Use

- **verify skill** - Test that implementations meet acceptance criteria
- **execute skill** - Run integration tests after implementing code
- Testing TUI applications (bubbletea, choo-choo, etc.)
- Running servers or long processes that need isolation
- Detecting hangs via timeout + screen capture
- End-to-end testing of interactive CLIs
- Multi-process coordination testing

## Design Philosophy

This skill provides a **testing interface** that happens to use tmux underneath. The tmux details are implementation - focus on the testing primitives:

```
┌─────────────────────────────────────────────────────────────────┐
│                      TESTING INTERFACE                           │
│                                                                  │
│  SESSIONS          INTERACTION           ASSERTIONS              │
│  ─────────         ───────────           ──────────              │
│  start/stop        send keys             wait(pattern, timeout) │
│  list              send literal          contains(text)          │
│  exists            send file             match(regex)            │
│                    capture                                        │
│                                                                  │
│              ↓ IMPLEMENTED VIA TMUX ↓                            │
│                                                                  │
│  - Isolated socket (.tmp/agent.sock)                            │
│  - Session namespacing (test-*)                                  │
│  - Non-blocking from Crush                                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Core Commands

### Session Management

```bash
# Start a new test session with a command
test start <name> <command...>

# Stop a session (graceful shutdown)
test stop <name>

# List active test sessions
test list

# Check if session exists (exit 0/1)
test exists <name>
```

### Interaction

```bash
# Send keystrokes (supports special keys)
test send <name> <keys...>

# Send literal text (no key interpretation)
test send-literal <name> <text>

# Send multi-line text from file (safe for complex content)
test send-file <name> <file>

# Capture current screen
test capture <name>

# Capture with full scrollback history
test capture-full <name>
```

### Assertions

```bash
# Wait for pattern to appear (returns 124 on timeout)
test wait <name> <pattern> [timeout-seconds]

# Assert screen contains exact text
test contains <name> <text>

# Assert screen matches regex
test match <name> <regex>
```

### One-Shot Commands (PREFERRED for simple tests)

```bash
# Full lifecycle in one command - START HERE
test run <name> <command...> -- [options]
  --wait <pattern>      Pattern to wait for after start
  --timeout <seconds>   Timeout for wait (default: 30)
  --send <keys>         Keys to send after wait (can repeat)
  --expect <text>       Assert text appears (can repeat)
  --expect-match <regex> Assert regex matches (can repeat)
  --capture-on-fail     Auto-capture screen on any failure
  --keep-alive          Don't stop session after test

# Assert with auto-capture on failure
test assert <name> contains <text>
test assert <name> matches <regex>
test assert <name> exits-within <seconds>
```

**Example - Full test in one command:**
```bash
test run app "./my-app" -- \
  --wait "Ready" \
  --timeout 30 \
  --send Down \
  --send Enter \
  --expect "Selected" \
  --capture-on-fail
```

## Special Keys

| Key | Syntax |
|-----|--------|
| Enter | `Enter` or `C-m` |
| Escape | `Escape` or `C-[` |
| Tab | `Tab` |
| Backspace | `BSpace` |
| Ctrl+C | `C-c` |
| Ctrl+D | `C-d` |
| Ctrl+Z | `C-z` |
| Arrow Keys | `Up` `Down` `Left` `Right` |
| Page Up/Down | `PageUp` `PageDown` |
| Home/End | `Home` `End` |

## Usage Patterns

### Pattern 1: Test TUI Lifecycle

```bash
# Start TUI
test start my-app "./my-app"

# Wait for initial render
test wait my-app "Welcome" 5

# Verify initial state
test contains my-app "Press 'q' to quit"

# Exit cleanly
test send my-app q

# Verify process ended
sleep 1
test exists my-app || echo "PASS: Exited cleanly"
```

### Pattern 2: Test Server Startup

```bash
test start api "go run ./cmd/api"

if test wait api "Server listening" 30; then
    echo "PASS: Server started"
    test capture-full api > server.log
    test send api C-c
    test stop api
else
    echo "FAIL: Server did not start"
    test capture-full api > failure.log
    test stop api
fi
```

### Pattern 3: Hang Detection

```bash
test start worker "./build-task"

if ! test wait worker "Complete" 60; then
    echo "WARN: Process may be hung"
    test capture worker
    test stop worker
    exit 124
fi

test stop worker
```

### Pattern 4: Multi-Process Testing

```bash
# Start multiple workers
test start worker-1 "./worker --id=1"
test start worker-2 "./worker --id=2"

# Wait for all to be ready
for id in 1 2; do
    test wait worker-$id "Ready" 10 || exit 1
done

# Send work
test send worker-1 Enter
test send worker-2 Enter

# Wait for completion
for id in 1 2; do
    test wait worker-$id "Done" 30
done

# Cleanup
for id in 1 2; do test stop worker-$id; done
```

### Pattern 5: Multi-Line Input (Agent Communication)

```bash
# Create instruction file
cat > /tmp/instructions.txt << 'EOF'
Fix the authentication bug:
1. Add null checks
2. Handle expired tokens
EOF

# Send safely (avoids tmux mode triggers)
test send-file agent-1 /tmp/instructions.txt
test send agent-1 Enter
```

## Integration with choo-choo

### With verify skill

After implementing a ticket, the verify skill uses this test skill to:
1. Start the application in isolation
2. Send inputs based on acceptance criteria
3. Capture and assert on outputs
4. Verify exit behavior

### With execute skill

During execution, this skill enables:
1. Running integration tests without blocking Crush
2. Testing CLI tools end-to-end
3. Starting dev servers for interactive testing

## Isolation Architecture

Sessions use an isolated tmux socket to avoid conflicts with user sessions:

```
Socket: .tmp/agent.sock (configurable via TEST_SOCKET env)
Session prefix: test- (all sessions are namespaced)
Default window size: 120x40
```

This ensures:
- Tests don't interfere with user's tmux sessions
- Multiple Crush sessions can run tests independently
- Clean separation between test environments

## Script Location

All scripts are in `choo-choo-skills/test/scripts/`:
- `test` - Main CLI wrapper

## Best Practices

1. **Always capture before stopping** - Preserve screen state for debugging
2. **Use `send-file` for multi-line text** - Avoids tmux mode triggers
3. **Check `exists` before sending** - Avoid errors on crashed processes
4. **Set generous timeouts** - TUIs and LLMs may be slow
5. **Use `capture-full` for debugging** - Includes complete scrollback

## Related Skills

- [verify](../verify/) - Uses test skill for acceptance criteria verification
- [execute](../execute/) - Uses test skill for integration testing
- [close-gaps](../close-gaps/) - Uses test skill to verify fixes
