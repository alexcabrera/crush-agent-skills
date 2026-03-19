---
name: validate
description: Verifies ticket structure is sound and produces an execution plan. Use after plan skill creates tickets, before execution. Checks for dependency cycles, missing dependencies, ticket completeness, and environment readiness. Outputs validation report and deterministic execution order.
license: MIT
compatibility: Requires bash, git, tk CLI.
metadata:
  version: "1.0.0"
  author: choo-choo-skills
---

# Validate Skill

Verifies that a ticket plan is ready for execution. Run this after the [plan](../plan/) skill creates tickets and before the [execute](../execute/) skill begins implementation.

## When to Use

Invoke this skill when:
- Plan skill has created tickets in `.tickets/`
- You want to verify the plan is executable before committing to implementation
- You need to identify dependency issues or missing information
- You want to see the execution order

## Checks Performed

### 1. Dependency Integrity
- No cycles in dependency graph (`tk dep cycle`)
- All referenced dependencies exist
- No orphaned tickets (all reachable from epic)

### 2. Ticket Completeness
- Every ticket has a description
- Every ticket has acceptance criteria
- Stories and tasks have parent references
- Types are valid (epic, story, task, sub-task)

### 3. Environment Readiness
- Required skills exist in `~/.config/crush/skills/`
- tk CLI is available
- STATE.md exists with valid epic reference

## Output

### Validation Report

```
✓ Dependency integrity
  - No cycles detected
  - 12 dependencies validated

✓ Ticket completeness
  - 8/8 tickets have descriptions
  - 7/8 tickets have acceptance criteria (warning: T-003 missing)

✓ Environment ready
  - All skills present
  - tk CLI available

Execution Order:
  Level 0: T-001, T-002, T-003 (can run in parallel)
  Level 1: T-004, T-005 (depend on Level 0)
  Level 2: T-006 (depends on T-004, T-005)
```

### STATE.md Update

Updates `phase` to `validate` and records any warnings.

## Handoff

**On success:**
> "Plan validated. Ready for execution. Invoke the `execute` skill to begin."

**On failure:**
> "Validation failed. Issues found: [list]. Fix these issues and re-run validation."

## Related Skills

- [plan](../plan/) - Creates the tickets this skill validates
- [execute](../execute/) - Consumes the execution plan this skill produces
