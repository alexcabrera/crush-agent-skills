---
name: execute
description: Execute a plan autonomously by running the virtuous cycle until all tickets are complete. Use after plan approval to let the agent work hands-off until done. For interactive mode, use elaborate and plan skills instead. This skill runs the full development cycle: pick ticket → research → implement → test → verify → fix → document.
license: MIT
compatibility: Requires bash, git.
metadata:
  version: "1.0.0"
  author: agent-skills
---

# execute Skill

Execute an approved plan autonomously. This skill runs the **virtuous cycle** until all tickets are complete, allowing you to focus on other work while the agent builds your project.

---

## When to Use

Invoke this skill when:
- The plan has been **approved by the user
- You want **hands-off** execution
- Starting a long-running autonomous session
- You want to work on another project while this runs

**Do NOT invoke when:**
- Requirements are still unclear (use [elaborate](../elaborate/) instead)
- Plan hasn't been created (use [plan](../plan/) instead)
- User wants to be involved (stay interactive)

---

## The Virtuous Cycle

```
┌─────────────────────────────────────────────────────────────────┐
│                    EXECUTION CYCLE                           │
│                                                                  │
│  ┌─────────────────────────────────────────────────────┐        │
│  │  1. PICK next ready ticket (respects deps)  │        │
│  │              ↓                                     │        │
│  │  2. RESEARCH if needed (new tech, unfamiliar)  │        │
│  │              ↓                                     │        │
│  │  3. IMPLEMENT ticket (TDD: tests first)        │        │
│  │              ↓                                     │        │
│  │  4. TEST (unit + CLI or web based on type)    │        │
│  │              ↓                                     │        │
│  │  5. VERIFY against acceptance criteria                 │        │
│  │              ↓                                     │        │
│  │  6. FIX gaps → loop to VERIFY until clean       │        │
│  │              ↓                                     │        │
│  │  7. REFACTOR if code quality issues detected            │        │
│  │              ↓                                     │        │
│  │  8. INTEGRATE run full suite, check regressions      │        │
│  │              ↓                                     │        │
│  │  9. REPORT update STATUS.md                            │        │
│  │              ↓                                     │        │
│  │ 10. REPEAT from step 1 until all tickets done            │        │
│  │              ↓                                     │        │
│  │ 11. DOCUMENT generate README.md                           │        │
│  │              ↓                                     │        │
│  │ DONE                                              │        │
│  │                                                     │        │
│  └─────────────────────────────────────────────────────┘        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## App Type Detection

The execute skill automatically detects whether to use CLI or web testing:

### Detection Logic

```bash
detect_app_type() {
    # Web indicators
    if [[ -f "package.json" ]] && grep -q "next\|vite\|react\|vue\|svelte\|astro" package.json 2>/dev/null; then
        echo "web"
        return
    fi
    if [[ -f "index.html" ]] || [[ -f "app.html" ]]; then
        echo "web"
        return
    fi
    
    # Check design.md for explicit marker
    if find specs -name "design.md" -exec grep -l "type:\s*web\|type:\s*api" {} \; 2>/dev/null | grep -q .; then
        echo "web"
        return
    fi
    
    # Default to CLI
    echo "cli"
}
```

### Testing Routing

| App Type | Test Skill | Tool |
|---------|-----------|------|
| CLI | [test-cli](../test-cli/) | tmux |
| Web | [test-web](../test-web/) | agent-browser |

---

## Escalation Conditions

The cycle **pauses for human input** when:

- **Blocked on external dependency** - Can't proceed without user action
- **Unclear requirement** - No rational default available
- **Security-sensitive decision** - Auth, secrets, permissions
- **Scope creep detected** - New tickets exceed threshold (default: 5 new tickets per cycle)
- **Repeated failures** - Same ticket fails 3+ times

When escalated, STATUS.md is updated with blocker details.

---

## Output Files

### STATE.md (machine state)

```yaml
---
phase: execution
epic: E-001
focus: T-003
started: 2026-03-25T10:00:00Z
app_type: web
learnings:
  - "Pattern discovered: using context.cancel for graceful shutdown"
decisions:
  - "[2026-03-25] Using parallel execution for independent tickets"
---
```

### STATUS.md (human-readable progress)

```markdown
# Execution Status

**Started:** 2026-03-25T10:00:00Z
**Phase:** execution
**App Type:** web

## Tickets

| ID | Status | Verified |
|----|--------|----------|
| T-001 | done | ✅ |
| T-002 | done | ✅ |
| T-003 | in_progress | - |
| T-004 | todo | - |

## Progress

### T-001: Setup project structure
- Implemented: 2026-03-25T10:15:00Z
- Verified: 2026-03-25T10:20:00Z

### T-003: Implement auth (current)
- Started: 2026-03-25T11:00:00Z
- Tests written, implementing...

## Issues Found
- None

## Blockers
- None
```

---

## Process

1. **Bootstrap** - Run `./execute/scripts/bootstrap --check`
2. **Read STATE.md** - Determine current phase and focus
3. **Detect app type** - CLI or Web testing route
4. **Pick ticket** - Find next ready ticket from `.tickets/`
5. **Execute cycle** - Run virtuous cycle for that ticket
6. **Report progress** - Update STATUS.md after each ticket
7. **Check escalation** - Pause if escalation conditions met
8. **Repeat** - Continue until all tickets done
9. **Hand off to document** - Generate README when complete

---

## Related Skills

- [elaborate](../elaborate/) - Requirements clarification (before this)
- [plan](../plan/) - Ticket creation (before this)
- [implement](../implement/) - Single ticket implementation
- [test-cli](../test-cli/) - CLI/TUI testing
- [test-web](../test-web/) - Web testing
- [verify](../verify/) - Acceptance criteria verification
- [close-gaps](../close-gaps/) - Fix discrepancies
- [document](../document/) - Generate README.md
