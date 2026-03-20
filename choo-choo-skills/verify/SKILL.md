---
name: verify
description: Confirms implementation matches ticket acceptance criteria. Use after execute skill completes a ticket. Checks code against each acceptance criterion, runs tests, and produces a verification report. If gaps found, hands off to close-gaps skill. Part of choo-choo-skills collection.
license: MIT
compatibility: Requires bash, git, tk CLI.
metadata:
  version: "1.0.0"
  author: choo-choo-skills
---

# Verify Skill

Verifies that an implemented ticket matches its acceptance criteria. Run this after the [execute](../execute/) skill completes a ticket.

## When to Use

Invoke this skill when:
- A ticket has been executed (code written, tests passing)
- You want to confirm implementation matches the ticket's intent
- You need a verification report before marking work complete

## Process

```
┌─────────────────────────────────────────────────────────────────┐
│                    VERIFICATION CYCLE                            │
│                                                                  │
│  1. LOAD ticket and acceptance criteria                         │
│         ↓                                                        │
│  2. READ implementation code                                    │
│         ↓                                                        │
│  3. CHECK each acceptance criterion                             │
│         ↓                                                        │
│  4. RUN test suite                                              │
│         ↓                                                        │
│  5. PRODUCE verification report                                 │
│         ↓                                                        │
│  6. HANDOFF: pass → done, fail → close-gaps                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Verification Checks

### For Each Acceptance Criterion

1. **Parse the criterion** - Understand what it requires
2. **Locate relevant code** - Find implementation that should satisfy it
3. **Verify behavior** - Check code does what's expected
4. **Check test coverage** - Ensure there's a test for this criterion

### Additional Checks

- All tests pass (including new tests)
- No regressions in existing tests
- Code follows project conventions
- No obvious security issues or code smells

## Testing Requirements

### When to Use test Skill

**MANDATORY** for any of these:
- CLI applications (interactive or non-interactive)
- TUI applications
- Long-running processes (servers, workers)
- End-to-end tests
- Integration tests involving process lifecycle

**Use the [test](../test/) skill for these - NEVER raw tmux or blocking bash.**

### When Unit Tests Suffice

For pure functions, libraries, and internal logic, standard unit tests are appropriate:
```bash
npm test
go test ./...
pytest
```

### Verification Test Pattern

```bash
# Start with test run for simple lifecycle tests
test run app "./my-app" -- --wait "Ready" --expect "Feature works"

# For complex scenarios, use step-by-step
test start app "./my-app"
test wait app "Ready" 30
test send app Down Enter
test assert app contains "Success"
test stop app
```

## Output

### Verification Report

```
Verification Report: T-003 - Implement State Manager

Acceptance Criteria:
  ✓ State can be loaded from STATE.md
  ✓ State can be saved atomically
  ✗ Invalid YAML returns clear error
    - Found: Panic on malformed YAML
    - Expected: Structured error with line number

Tests:
  ✓ 8/8 tests passing
  - TestLoadState
  - TestSaveState
  - TestConcurrentAccess
  - ...

Code Quality:
  ✓ Follows project conventions
  ⚠ Consider adding more inline comments

Result: GAPS FOUND
```

### STATE.md Update

- `phase` set to `verification`
- Any findings recorded

## Handoff

**On pass (single ticket):**
> "Verification passed. T-003 is complete. Ready for next ticket."

**On pass (epic complete):**
When all tickets in an epic are verified:
> "Verification passed. All tickets in epic complete. Invoke the `document` skill to update README.md before reporting completion."

**On gaps found:**
> "Verification found 1 gap. Invoke the `close-gaps` skill to fix."

## Related Skills

- [execute](../execute/) - Creates the implementation this skill verifies
- [close-gaps](../close-gaps/) - Fixes issues found during verification
- [document](../document/) - Updates README.md after epic completion
- [test](../test/) - Run automated tests for CLI/TUI applications
