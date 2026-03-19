# Testing Patterns

Common testing patterns using the test skill for automated verification.

## Pattern 1: Basic TUI Lifecycle

Test that a TUI starts, displays expected content, and exits cleanly.

```bash
#!/bin/bash
set -e

# Start TUI
test start my-app "./my-app"

# Wait for initial render
test wait my-app "Welcome" 5

# Verify initial state
test contains my-app "Press 'q' to quit"

# Exit cleanly
test send my-app q

# Verify process ended (may take a moment)
sleep 1
if test exists my-app; then
    echo "FAIL: TUI did not exit"
    test stop my-app
    exit 1
fi

echo "PASS: TUI lifecycle test"
```

## Pattern 2: Keyboard Navigation

Test keyboard navigation through a TUI menu.

```bash
#!/bin/bash
set -e

test start nav-test "./my-tui"
test wait nav-test "Menu" 5

# Navigate down
test send nav-test Down
test contains nav-test "Option 2"  # Assuming this highlights

# Navigate up
test send nav-test Up
test contains nav-test "Option 1"

# Select with Enter
test send nav-test Enter
test wait nav-test "Selected" 3

test stop nav-test
echo "PASS: Navigation test"
```

## Pattern 3: Server Startup Verification

Test that a server starts and becomes ready.

```bash
#!/bin/bash

test start api "go run ./cmd/api"

if test wait api "Server listening on" 30; then
    echo "PASS: Server started"
    test capture-full api > /tmp/api-startup.log
    test send api C-c
    sleep 2
    test stop api
else
    echo "FAIL: Server did not start in time"
    test capture-full api > /tmp/api-failure.log
    test stop api
    exit 1
fi
```

## Pattern 4: Hang Detection

Detect when a process hangs.

```bash
#!/bin/bash

test start worker "./long-task"

if ! test wait worker "Complete" 60; then
    echo "WARN: Process may be hung"

    # Capture current state for debugging
    echo "=== Screen at timeout ==="
    test capture worker

    echo "=== Full history ==="
    test capture-full worker > /tmp/hang-debug.log

    # Force stop
    test stop worker
    exit 124
fi

echo "PASS: Process completed"
test stop worker
```

## Pattern 5: Parallel Process Testing

Run multiple processes and verify coordination.

```bash
#!/bin/bash

# Start multiple workers
test start worker-1 "./worker --id=1"
test start worker-2 "./worker --id=2"
test start worker-3 "./worker --id=3"

# Wait for all to be ready
for i in 1 2 3; do
    if ! test wait worker-$i "Ready" 10; then
        echo "FAIL: worker-$i did not start"
        test capture worker-$i
        # Clean up all
        for j in 1 2 3; do test stop worker-$j 2>/dev/null; done
        exit 1
    fi
done

echo "All workers ready"

# Send work to all
for i in 1 2 3; do
    test send worker-$i Enter
done

# Wait for completion
for i in 1 2 3; do
    test wait worker-$i "Done" 30
done

# Clean up
for i in 1 2 3; do test stop worker-$i; done

echo "PASS: Parallel test"
```

## Pattern 6: Multi-Line Input (Agent Communication)

Safely send complex instructions to an agent.

```bash
#!/bin/bash
set -e

# Create instruction file with complex content
cat > /tmp/instructions.txt << 'EOF'
Please fix the following issues in src/auth.ts:

1. Line 42: Missing null check on user.token
   - Add: if (!user.token) throw new Error('No token')

2. Line 58: Race condition in async logout
   - Use Promise.all for parallel cleanup

After fixing, run: npm test
EOF

# Send to agent safely (avoids tmux mode triggers)
test send-file agent-1 /tmp/instructions.txt
test send agent-1 Enter

# Wait for acknowledgment
test wait agent-1 "fixing|Fixed|Done" 60
```

## Pattern 7: Error Handling Verification

Verify error states are handled correctly.

```bash
#!/bin/bash

# Run in directory without required files
cd /tmp/empty-dir
test start error-test "choo-choo"

# Should show error message, not crash
if test wait error-test "Error|error|No project" 5; then
    # Verify it's still running (didn't crash)
    if test exists error-test; then
        echo "PASS: Error handled gracefully"
        test stop error-test
    else
        echo "FAIL: Crashed on error"
        exit 1
    fi
else
    echo "FAIL: No error message shown"
    test capture error-test
    test stop error-test
    exit 1
fi
```

## Pattern 8: Integration with choo-choo Workflow

Test the full choo-choo workflow.

```bash
#!/bin/bash
# test-choo-choo.sh - Integration test for choo-choo

set -e

# Setup test project
TEST_DIR=$(mktemp -d)
cd "$TEST_DIR"

# Build choo-choo
cd /path/to/choo-choo
go build -o "$TEST_DIR/choo-choo" ./cmd/choo-choo
cd "$TEST_DIR"

# Test 1: Init phase
echo "Test: Init phase..."
test start choo-choo "./choo-choo"
test wait choo-choo "choo-choo" 5
test contains choo-choo "init"
test stop choo-choo
echo "PASS: Init phase"

# Test 2: STATE.md created
if [[ -f "STATE.md" ]]; then
    echo "PASS: STATE.md created"
else
    echo "FAIL: STATE.md not created"
    exit 1
fi

# Test 3: .tickets/ created
if [[ -d ".tickets" ]]; then
    echo "PASS: .tickets/ created"
else
    echo "FAIL: .tickets/ not created"
    exit 1
fi

# Cleanup
rm -rf "$TEST_DIR"
echo "All tests passed!"
```

## Pattern 9: Acceptance Criteria Testing

Map acceptance criteria to test assertions.

```bash
#!/bin/bash
# verify-ticket.sh - Test ticket acceptance criteria

set -e
TICKET_ID="${1:-T-001}"

echo "Verifying $TICKET_ID..."

# Read acceptance criteria from ticket
CRITERIA=$(grep -A 20 "## Acceptance Criteria" ".tickets/$TICKET_ID-*.md" | grep -E "^\-")

test start app "./my-app"
test wait app "Ready" 10

PASS=0
FAIL=0

# Test each criterion
while IFS= read -r criterion; do
    [[ -z "$criterion" ]] && continue

    echo "Checking: $criterion"

    # Convert criterion to test pattern
    # This is a simplified example - real implementation would be smarter
    if test contains app "$criterion" 2>/dev/null; then
        echo "  ✓ PASS"
        ((PASS++))
    else
        echo "  ✗ FAIL"
        ((FAIL++))
    fi
done <<< "$CRITERIA"

test stop app

echo ""
echo "Results: $PASS passed, $FAIL failed"
[[ $FAIL -eq 0 ]] || exit 1
```

## Pattern 10: Cleanup on Exit

Ensure sessions are always cleaned up.

```bash
#!/bin/bash

SESSION_NAME="test-session"

cleanup() {
    echo "Cleaning up..."
    test stop "$SESSION_NAME" 2>/dev/null || true
    test kill "$SESSION_NAME" 2>/dev/null || true
}
trap cleanup EXIT

test start "$SESSION_NAME" "./my-app"
test wait "$SESSION_NAME" "Ready" 10

# ... run tests ...

# cleanup() runs automatically on exit
```

## Debugging Tips

1. **Capture before stopping**: Always capture screen before stopping a session
2. **Use `capture-full`**: Includes scrollback for complete history
3. **Check `exists` first**: Verify session exists before sending
4. **Timeout generously**: TUIs and LLMs may be slow
5. **Log everything**: Redirect capture output to files for CI
6. **Use `trap cleanup EXIT`**: Ensure sessions are cleaned up on script failure

## Integration with CI

```yaml
# .github/workflows/test.yml
- name: Run TUI Tests
  run: |
    chmod +x scripts/test
    ./scripts/test start app "./build/my-app"
    ./scripts/test wait app "Ready" 30
    ./scripts/test capture-full app > test-output.log
    ./scripts/test stop app
```
