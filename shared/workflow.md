# Agent Skills Workflow

This document describes the overall workflow for building software using these agent skills.

## Two Modes of Operation

### Interactive Mode

Human in the loop, answering questions, making decisions.

```
elaborate → plan → [USER APPROVES]
```

### Autonomous Mode

Hands-off execution of the entire plan.

```
execute (virtuous cycle until done)
```

---

## Skill Overview

| Skill | Mode | Purpose |
|-------|------|---------|
| `elaborate` | Interactive | Clarify requirements, design architecture |
| `plan` | Interactive | Decompose design into ticket tree |
| `execute` | Autonomous | Run virtuous cycle until all tickets done |
| `implement` | Either | Implement a single ticket (TDD) |
| `verify` | Either | Confirm implementation vs acceptance criteria |
| `close-gaps` | Either | Fix discrepancies found by verify |
| `document` | Either | Generate README.md |
| `test-cli` | Either | Test CLI/TUI applications via tmux |
| `test-web` | Either | Test web applications via agent-browser |
| `ui-design` | Either | Design UI/UX interaction patterns |

---

## The Virtuous Cycle (execute)

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  1. PICK next ready ticket (respects dependencies)     │
│  2. RESEARCH if needed (new tech, unfamiliar patterns) │
│  3. IMPLEMENT ticket (TDD: tests first)                │
│  4. TEST (test-cli or test-web based on app type)      │
│  5. VERIFY against acceptance criteria                  │
│  6. FIX gaps → loop to VERIFY until clean              │
│  7. REFACTOR if code quality issues detected           │
│  8. INTEGRATE run full suite, check regressions        │
│  9. REPORT progress to STATUS.md                       │
│ 10. LOOP back to step 1 until all tickets done         │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### App Type Detection

`execute` automatically detects whether to use `test-cli` or `test-web`:

**Web indicators:**
- `package.json` with `next`, `vite`, `react`, `vue`, `svelte`, `astro`
- `index.html` or `app.html` present
- `go.mod` with `net/http`
- `design.md` with `type: web` or `type: api`

**CLI indicators:**
- `cmd/` directory with `main.go`
- `Cargo.toml` with `[[bin]]`
- Python scripts with `if __name__ == "__main__"`
- No web indicators

### Escalation Conditions

The autonomous cycle pauses for human input when:

- Blocked on external dependency (API key, service access)
- Unclear requirement with no rational default
- Security-sensitive decision required
- Scope creep detected (new tickets exceed threshold)

---

## Script Interfaces

### `scripts/next`

Determines the next skill to invoke.

```bash
./scripts/next

# Output (line-delimited):
# Line 1: skill-name | "done" | "blocked"
# Line 2: context/argument
# Line 3: human-readable reason

# Exit codes:
# 0 - Has next step
# 1 - Workflow complete
# 2 - Blocked (requires human input)
```

### `scripts/bootstrap`

Installs or checks skill dependencies.

```bash
./scripts/bootstrap --check    # Check only, don't install
./scripts/bootstrap --install  # Install missing dependencies

# Output (line-delimited):
# Line 1: "ready" | "missing"
# Line 2+: Missing dependencies (if any)
# Line N: Install commands

# Exit codes:
# 0 - All dependencies satisfied
# 1 - Missing dependencies (with --check)
# 2 - Installation failed (with --install)
```

---

## State Files

### STATE.md (project-level)

Tracks workflow state. Lives in `.tickets/STATE.md`.

```yaml
---
phase: elaborate | plan | execution | verification | gap-closure | done
epic: <ticket-id>
focus: <ticket-id>
started: <ISO-8601>
app_type: cli | web
learnings:
  - <string>
decisions:
  - <string>
---
```

### STATUS.md (autonomous execution)

Human-readable progress report. Lives in project root.

```markdown
# Execution Status

**Started:** 2026-03-25T10:00:00Z
**Phase:** execution
**App Type:** web

## Tickets

| ID | Status | Verified |
|----|--------|----------|
| T-001 | done | ✅ |
| T-002 | in_progress | - |

## Progress

### T-001: Setup project structure
- Implemented: 2026-03-25T10:15:00Z
- Verified: 2026-03-25T10:20:00Z

## Issues Found
- None

## Blockers
- None
```

---

## Getting Started

1. **Bootstrap:** Run `./<skill>/scripts/bootstrap --install` for skills you'll use
2. **Interactive:** Start with `elaborate` skill to define what you're building
3. **Plan:** Use `plan` skill to break down into tickets
4. **Approve:** Review the plan, approve when ready
5. **Execute:** Run `execute` skill to implement autonomously
6. **Review:** Check STATUS.md for progress, intervene if blocked
