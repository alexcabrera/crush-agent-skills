---
name: choo-choo
description: |
  [META SKILL - Not auto-discovered by Crush. Reference manually.]
  
  A complete development workflow from idea to implementation. Orchestrates design → plan → validate → execute → verify → close-gaps. Use when starting a new feature or project, or when you want structured development with persistence across context resets. Individual skills can also be used independently.
license: MIT
compatibility: "Requires bash, git. Optional: rg (ripgrep) for faster searches."
metadata:
  version: "1.0.0"
  author: choo-choo-skills
---

# choo-choo

> "I choo-choo-choose you" - Ralph Wiggum

A complete development workflow that takes you from rough idea to verified implementation. Skills persist state via `STATE.md` and tickets, enabling work to survive context resets.

## The Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                    CHOO-CHOO WORKFLOW                            │
│                                                                  │
│  1. DESIGN (interactive)                                        │
│     Rough idea → Requirements → Architecture                    │
│            ↓                                                     │
│  2. PLAN                                                        │
│     Design → Ticket tree with hierarchy + dependencies          │
│            ↓                                                     │
│  3. VALIDATE                                                    │
│     Tickets → Verified plan with execution order                │
│            ↓                                                     │
│  4. EXECUTE                                                     │
│     Ticket → Code + tests (per ticket)                          │
│            ↓                                                     │
│  5. VERIFY                                                      │
│     Implementation → Verification against acceptance criteria   │
│            ↓                                                     │
│  6. CLOSE-GAPS (if needed)                                      │
│     Gaps → Fixes → Re-verify                                    │
│            ↓                                                     │
│  7. DOCUMENT                                                    │
│     Code → README.md (comprehensive, from scratch)              │
│            ↓                                                     │
│  DONE                                                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Testing Requirements

For any work involving **CLI applications, TUIs, or long-running processes**:

**MANDATORY:** Use the [test](test/) skill for all interactive testing.

❌ **FORBIDDEN:**
```bash
sleep 15 && tmux -S .tmp/agent.sock capture-pane -p -t test-app
```

✅ **REQUIRED:**
```bash
test wait app "Ready" 30
test capture app
```

**Why:** Raw tmux commands block the agent and require user cancellation. The test skill provides non-blocking commands with built-in timeouts. This requirement is enforced across all skills: plan, validate, execute, verify, and close-gaps.

## Skills in This Collection

| Skill | Phase | When to Use | Output |
|-------|-------|-------------|--------|
| [design](design/) | Design | Rough idea, unclear requirements | requirements.md, design.md |
| [ui-design](ui-design/) | Design | CLI/TUI interface design | UI specifications |
| [plan](plan/) | Planning | Have design, need to break into work items | Ticket tree in .tickets/ |
| [validate](validate/) | Validation | Have tickets, ready to execute | Execution order, validation report |
| [execute](execute/) | Execution | Implementing a specific ticket | Code, tests |
| [verify](verify/) | Verification | Ticket implemented, needs checking | Verification report |
| [close-gaps](close-gaps/) | Gap Closure | Verification found issues | Fixed code, updated tests |
| [document](document/) | Documentation | Epic verified, update README | README.md |
| [test](test/) | Testing | Testing CLI/TUI applications | Test results |

## Quick Start

### Starting Fresh (New Feature/Project)

```
1. Describe your idea → design skill activates
2. Answer clarification questions
3. Review design.md, approve
4. plan skill creates tickets
5. validate skill verifies plan
6. execute skill runs (autonomous or guided)
7. verify + close-gaps loop until done
```

### With Existing Design

```
1. Invoke plan skill with your design.md
2. Continue from step 4 above
```

### Single Ticket (Interactive)

```
1. Invoke execute skill with ticket ID
2. Agent implements that ticket only
3. verify skill checks the work
```

## State Management

Work state persists in `.tickets/STATE.md`:

```yaml
---
phase: design | plan | validate | execution | verification | gap-closure | done
epic: E-001
focus: T-003
started: 2024-01-15T10:30:00Z
learnings:
  - "Pattern discovered during execution"
decisions:
  - "[2024-01-15] Decision rationale"
---
```

This enables:
- Crash recovery: Resume from last known state
- Context persistence: Each agent context reads STATE.md to know where to continue
- Knowledge capture: Learnings persist for future reference

## Individual vs. Orchestrated Use

**Individual skills** can be invoked directly in Crush:
```
> Use the design skill to help me plan authentication
> Use the execute skill to implement T-003
```

**Orchestrated** via choo-choo TUI:
- Visual workflow with phase gates
- Kanban for execution tracking
- Parallel ticket execution
- Real-time progress display

Both modes use the same skills - choo-choo TUI adds visualization and coordination.

## File Structure

```
project/
├── .tickets/
│   ├── STATE.md           # Workflow state
│   ├── E-001-epic.md      # Epic ticket
│   ├── S-001-story.md     # Story tickets
│   └── T-001-task.md      # Task tickets
├── specs/
│   ├── requirements.md    # Clarified requirements
│   ├── design.md          # Technical design
│   └── research/          # Research notes
└── (your code)
```

## Related

- [choo-choo TUI](https://github.com/your-org/choo-choo) - Visual orchestration layer
- [agentskills.io](https://agentskills.io) - Skill specification
