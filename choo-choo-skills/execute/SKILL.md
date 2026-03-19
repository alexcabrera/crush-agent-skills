---
name: execute
description: Implements a single ticket by writing code and tests. Use when you have a validated ticket plan and want to implement a specific ticket. Follows TDD: writes tests first, then implementation. Updates ticket status and STATE.md. Part of choo-choo-skills collection.
license: MIT
compatibility: Requires bash, git, tk CLI.
metadata:
  version: "1.0.0"
  author: choo-choo-skills
---

# Execute Skill

Implements a single ticket from a validated plan. This skill writes code, creates tests, and ensures the implementation matches the ticket's acceptance criteria.

## When to Use

Invoke this skill when:
- You have a validated ticket (from [validate](../validate/) skill)
- You want to implement a specific ticket
- Dependencies for the ticket are complete (status: done)

## Process

```
┌─────────────────────────────────────────────────────────────────┐
│                    EXECUTION CYCLE                               │
│                                                                  │
│  1. LOAD ticket from .tickets/                                  │
│         ↓                                                        │
│  2. VERIFY dependencies are complete                            │
│         ↓                                                        │
│  3. UPDATE STATE.md (focus: this ticket)                        │
│         ↓                                                        │
│  4. WRITE TESTS first (no implementation)                       │
│         ↓                                                        │
│  5. IMPLEMENT until tests pass                                  │
│         ↓                                                        │
│  6. RUN full test suite                                         │
│         ↓                                                        │
│  7. UPDATE ticket status to "done"                              │
│         ↓                                                        │
│  8. COMMIT all changes (one commit per ticket)                  │
│         ↓                                                        │
│  9. CLEAR STATE.md focus                                        │
│         ↓                                                        │
│  10. HANDOFF to verify skill                                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Input Requirements

**Ticket must have:**
- Valid ID in `.tickets/`
- Description of what to implement
- Acceptance criteria (for verification)
- Status: `todo` or `doing`
- All dependencies with status: `done`

## Output

### Code Changes
- Implementation files
- Test files
- Any configuration updates

### Git Commit
- **One commit per ticket** - non-negotiable
- Commit after tests pass and ticket is marked done
- Commit message format: `<ticket-id>: <ticket-title>`
- Include implementation files, test files, and ticket status change

### Ticket Update
- Status changed to `done`
- Notes updated with implementation details

### STATE.md Update
- `focus` set during execution, cleared on completion
- Any learnings added

## Example Invocation

```bash
# Via Crush
> Use the execute skill to implement T-003

# The skill will:
# 1. Read .tickets/T-003-*.md
# 2. Verify T-001 and T-002 (dependencies) are done
# 3. Set STATE.md focus to T-003
# 4. Write tests for T-003's acceptance criteria
# 5. Implement until tests pass
# 6. Update ticket status to done
# 7. Commit with message: "T-003: <ticket-title>"
# 8. Clear focus
```

## Handoff

**On completion:**
> "Ticket T-003 complete. Invoke the `verify` skill to confirm implementation matches acceptance criteria."

**On blocked (dependencies not done):**
> "Cannot execute T-003: dependencies T-001, T-002 not complete. Execute those first."

**On failure:**
> "Execution failed: [reason]. See STATE.md for details."

## Related Skills

- [validate](../validate/) - Ensures ticket is ready for execution
- [verify](../verify/) - Confirms implementation matches intent
- [close-gaps](../close-gaps/) - Fixes issues found during verification
- [test](../test/) - Run automated tests for CLI/TUI applications
