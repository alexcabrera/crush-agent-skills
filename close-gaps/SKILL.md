---
name: close-gaps
description: Fixes discrepancies between implementation and acceptance criteria. Use when verify skill finds gaps. Analyzes the gap, implements fixes, updates tests. Loops back to verify for re-checking. Maximum 3 attempts before escalation.
license: MIT
compatibility: Requires bash, git, tk CLI.
metadata:
  version: "1.0.0"
  
---

# Close-Gaps Skill

Fixes gaps found during verification. This skill analyzes discrepancies between implementation and acceptance criteria, implements fixes, and hands off to [verify](../verify/) for re-checking.

## When to Use

Invoke this skill when:
- The [verify](../verify/) skill found gaps
- Implementation doesn't match acceptance criteria
- Tests are failing after implementation

## Process

```
┌─────────────────────────────────────────────────────────────────┐
│                    GAP CLOSURE CYCLE                             │
│                                                                  │
│  1. RECEIVE verification report with gaps                       │
│         ↓                                                        │
│  2. ANALYZE each gap                                            │
│         ↓                                                        │
│  3. DETERMINE root cause                                        │
│         ↓                                                        │
│  4. IMPLEMENT fix                                               │
│         ↓                                                        │
│  5. UPDATE tests if needed                                      │
│         ↓                                                        │
│  6. RUN tests to verify fix                                     │
│         ↓                                                        │
│  7. HANDOFF to verify skill for re-check                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Retry Limit

Maximum **3 attempts** per ticket. If gaps persist after 3 attempts:

1. Mark ticket as `blocked`
2. Log to STATE.md with details
3. Add learning to epic ticket via `tk add-note`
4. Continue with other tickets (if parallel execution)
5. Report blocked tickets at end

## Input

### Verification Report
```
Gaps Found:
  1. Invalid YAML returns clear error
     - Found: Panic on malformed YAML
     - Expected: Structured error with line number
```

### Context
- Ticket ID from STATE.md `focus`
- Attempt count from STATE.md or ticket notes

## Output

### Code Fix
- Modified implementation files
- Updated error handling

### Test Updates
- New tests for the gap scenario
- Updated existing tests if needed

## Testing During Gap Closure

### Unit Test Fixes
For pure function or logic fixes:
```bash
npm test
go test ./...
pytest
```

### CLI/TUI/Process Fixes (MANDATORY test skill)
**Use the [test-cli](../test-cli/) skill for any fixes involving:**
- CLI applications
- TUI applications
- Long-running processes

**NEVER use raw tmux or blocking bash.**

```bash
# Preferred: verify fix with one-shot command
test run app "./my-app" -- --wait "Ready" --expect "Fixed behavior"

# Step-by-step for debugging
test start app "./my-app"
test wait app "Ready" 30
test send app Down Enter
test assert app contains "Expected output"
test stop app
```

### STATE.md Update
- `phase` set to `gap-closure`
- Attempt count incremented
- Learning captured if applicable

## Example Invocation

```bash
# Via Crush (typically auto-invoked after verify finds gaps)
> Use the close-gaps skill to fix the verification issues for T-003

# The skill will:
# 1. Read verification report
# 2. Analyze: "YAML parser panics instead of returning error"
# 3. Implement: Add recover() and error wrapping
# 4. Update test: TestInvalidYAML returns error, not panic
# 5. Run tests
# 6. Hand off to verify skill
```

## Gap Categories

| Category | Example Fix |
|----------|-------------|
| Missing behavior | Add missing feature code |
| Wrong behavior | Correct the logic |
| Error handling | Add proper error returns |
| Edge case | Add boundary condition handling |
| Test coverage | Add missing test cases |
| Documentation | Update comments/docs |

## Handoff

**After fix:**
> "Gap addressed. Invoking `verify` skill to re-check."

**On max retries:**
> "Maximum attempts (3) reached for T-003. Ticket marked blocked. Gap: [description]. See STATE.md for details."

## Related Skills

### Virtuous Cycle
- [execute](../execute/) - Orchestrates the full cycle including this skill
- [verify](../verify/) - Finds gaps this skill fixes, re-checks after fix
- [implement](../implement/) - Original implementation that may need fixing

### Testing
- [test-cli](../test-cli/) - **MANDATORY** for CLI/TUI/process fixes
- [test-web](../test-web/) - For web application fixes

### Workflow
- [elaborate](../elaborate/) - Requirements clarification (start here)
- [plan](../plan/) - Creates tickets for implementation
- [document](../document/) - Updates README.md after epic completion
