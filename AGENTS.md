# Agent Skills Directory

This directory contains agent skills for structured development workflows. Each skill is a self-contained directory with `SKILL.md`, optional `scripts/`, and `references/`.

## Installation

Symlink this repository to your agent's skills directory:

```bash
# For Crush
ln -s ~/Code/agent-skills ~/.config/crush/skills/agent-skills

# For Claude Code
ln -s ~/Code/agent-skills ~/.claude/skills/agent-skills
```

---

## Skills Overview

| Skill | Mode | Purpose |
|-------|------|---------|
| `elaborate` | Interactive | Transform rough ideas into requirements + architecture |
| `plan` | Interactive | Decompose design into ticket tree |
| `execute` | Autonomous | Run virtuous cycle until all tickets complete |
| `implement` | Either | Implement a single ticket (TDD) |
| `verify` | Either | Confirm implementation matches acceptance criteria |
| `close-gaps` | Either | Fix discrepancies found by verify |
| `document` | Either | Generate comprehensive README.md |
| `test-cli` | Either | Test CLI/TUI applications via tmux |
| `test-web` | Either | Test web applications via agent-browser |
| `ui-design` | Either | Design UI/UX interaction patterns |

---

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

## Workflow

```
┌──────────────────────────────────────────────────────────────┐
│                    INTERACTIVE MODE                           │
│                                                               │
│  1. elaborate  - Clarify requirements, design architecture  │
│  2. plan       - Break into tickets with dependencies        │
│  3. [approve]  - User reviews and approves the plan          │
│                                                               │
└──────────────────────────────────────────────────────────────┘
                         ↓
┌──────────────────────────────────────────────────────────────┐
│                    AUTONOMOUS MODE (execute)                  │
│                                                               │
│  while tickets remain:                                        │
│    pick ticket → implement → test → verify → fix if needed   │
│  done:                                                        │
│    generate documentation                                     │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## Script Interfaces

Each skill has two standard scripts:

### `scripts/next`

Determines the next skill in the workflow:

```bash
./<skill>/scripts/next

# Output:
# Line 1: skill-name | "done" | "blocked"
# Line 2: context/argument
# Line 3: human-readable reason
```

### `scripts/bootstrap`

Installs or checks dependencies:

```bash
./<skill>/scripts/bootstrap --check    # Check only
./<skill>/scripts/bootstrap --install  # Install missing
```

---

## Testing Split

The execute skill automatically routes to the correct testing skill:

| App Type | Skill | Tool |
|----------|-------|------|
| CLI/TUI | `test-cli` | tmux |
| Web | `test-web` | agent-browser |

Detection is based on project structure (package.json, index.html, etc.).

---

## Shared Resources

Cross-skill documentation lives in `shared/`:

- `STATE-structure.md` - State file format
- `workflow.md` - Overall workflow documentation
- `bootstrap-template` - Template for bootstrap scripts

---

## Agent Skills Specification

This repository follows the [Agent Skills specification](https://agentskills.io):

- Each skill is a directory with `SKILL.md` (YAML frontmatter + markdown)
- `name` must match directory name (1-64 chars, lowercase, hyphens)
- `description` must describe what and when (1-1024 chars)
- Optional: `scripts/`, `references/`, `assets/`
- Keep `SKILL.md` under 500 lines when possible
