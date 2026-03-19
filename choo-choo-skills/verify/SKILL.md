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

**On pass:**
> "Verification passed. T-003 is complete. Ready for next ticket or done."

**On gaps found:**
> "Verification found 1 gap. Invoke the `close-gaps` skill to fix."

## Example Invocation

```bash
# Via Crush
> Use the verify skill to check T-003

# The skill will:
# 1. Read .tickets/T-003-*.md
# 2. Extract acceptance criteria
# 3. Review implementation code
# 4. Run tests
# 5. Produce verification report
# 6. Recommend next action
```

## Related Skills

- [execute](../execute/) - Creates the implementation this skill verifies
- [close-gaps](../close-gaps/) - Fixes issues found during verification
- [test](../test/) - Run automated tests for CLI/TUI applications
