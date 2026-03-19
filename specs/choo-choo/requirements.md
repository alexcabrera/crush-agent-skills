# Requirements: choo-choo

> "I choo-choo-choose you" - Ralph Wiggum's Valentine card to Lisa
> 
> Ralph (loop) + Crush (heart logo) with structure

## Rough Idea

Create a self-perpetuating development flywheel by:
1. Updating the existing `prompt-driven-development` and `ticket-planning` skills to work harmoniously together
2. Creating new skills needed to close the loop: execution, verification, debugging, gap closure

The goal is a system that requires little to no human intervention once the planning phase is over.

---

## Clarification Q&A

### Q1: Scope Prioritization
**A:** Option C - Middle ground. GSD is too prescriptive. Want cleanest implementation mixing in "Ralph Loop" concepts. Skills will be used by Crush coding agent.

---

### Q2: State Management Approach
**A:** Minimal STATE.md + leverage existing tk structure

**Decision:** Use a single `.tickets/STATE.md` for flywheel state, with the following model:

```yaml
---
phase: planning | execution | verification | gap-closure
epic: <epic-ticket-id>
focus: <current-ticket-id>
started: <ISO timestamp>
learnings:
  - "Pattern X works for Y"
decisions:
  - "[YYYY-MM-DD] Decision description"
---
```

**State lives in `.tickets/STATE.md`. Ticket state lives in individual ticket files (existing `tk` behavior).**

Learnings stored via `tk add-note <epic> "pattern discovered..."` on the epic ticket.

**This achieves Ralph-style fresh context: each session reads STATE.md to know where to resume.**

---

### Q3: STATE.md Ownership Model
**A:** Option A - Skills own STATE.md explicitly

Each skill reads STATE.md at start and updates it at end. Clear handoff between skills - no implicit orchestration.

### Q4: Naming
**A:** "flywheel" is a placeholder. Better name TBD after full picture emerges.

### Q5: Skill Taxonomy
**A:** 6 skills with clear separation of concerns

| Skill | Input | Output | Phase |
|-------|-------|--------|-------|
| `design` | Rough idea | requirements.md, design.md | design |
| `plan` | design.md | Ticket tree (epic/stories/tasks) | planning |
| `validate` | Ticket tree | Validation report, execution order | validation |
| `execute` | Ticket, STATE.md | Code, tests, passing test run | execution |
| `verify` | Ticket, implementation | Verification report | verification |
| `close-gaps` | Verification failures | Fixed code, updated tests | gap-closure |

**Existing skills (rename/reorganize):**
- `prompt-driven-development` → `design`
- `ticket-planning` → `plan`

**New skills needed:**
- `validate`
- `execute`
- `verify`
- `close-gaps`

### Q6: Skill Handoff Mechanism
**A:** Hybrid - STATE.md tracks state, skills include handoff hints, agent can infer if needed

### Q7: Implementation Approach
**Exploring:** Wrapper around Crush's headless mode + TUI using Charm's bubbletea

**Reference: Ralph TUI** (github.com/subsy/ralph-tui)
- AI Agent Loop Orchestrator that wraps Claude Code, Codex, etc.
- Runs agents autonomously: SELECT TASK → BUILD PROMPT → EXECUTE → DETECT COMPLETION → NEXT TASK
- TUI for real-time monitoring with keyboard controls
- Uses prd.json or Beads for task tracking
- Has headless mode for CI/CD

**Crush Headless Capabilities:**
- `crush run "prompt"` - non-interactive execution
- `--quiet` - clean stdout (spinner/logs to stderr)
- `--yolo` - auto-accept all permissions
- `--continue` / `--session <id>` - resume sessions
- stdin piped to prompt, stdout for responses
- Skills auto-loaded from `~/.config/crush/skills/`

**Our Approach (differentiated from Ralph):**
- Use `tk` CLI for ticket tracking (not prd.json)
- Use skill system (design, plan, execute, verify, close-gaps)
- STATE.md for session handoff
- Built with bubbletea (same stack as Crush itself)

---

## choo-choo Architecture

**Deliverables:**
| Location | What |
|----------|------|
| `~/Code/agent-skills/` (here) | Skills: `design`, `plan`, `execute`, `verify`, `close-gaps` |
| `~/Code/choo-choo/` (new) | TUI app: visualization + orchestration layer on top of Crush |

**Phases & UI:**

| Phase | Mode | UI | Crush Role |
|-------|------|----|----|
| Design | Interactive | Two-pane (chat + artifact preview) | Embedded session? |
| Plan | Non-interactive | Progress indicator, then ticket explorer | `crush run` with `plan` skill |
| Execute | Background | Kanban board (TODO → DOING → DONE) | `crush run --yolo` with `execute` skill |
| Verify | Background | Kanban + verification status | Sub-agent per ticket |
| Close-gaps | Background | Kanban + fix status | Sub-agent for failures |

**Crush Integration Points:**
- Read state: Query `~/.crush/crush.db` (SQLite WAL, safe for reads)
- Stream updates: Poll `messages` table for new entries
- Run commands: `crush run --quiet --yolo "prompt"`
- Skills: Auto-loaded from `~/.config/crush/skills/` (symlinked from here)
- Sub-agents: Crush spawns internally, tracked via `parent_session_id`

---

## Open Questions

### Q9: Interactive Design Phase
**A:** Option D - choo-choo lightweight chat + `crush run`

choo-choo manages the conversation flow, constructs prompts, calls `crush run --continue` for each message, displays responses. Uses Crush for all AI work, but choo-choo controls the design skill invocation and artifact handling.

---

### Q10: Project Initialization
**A:** Option A - Minimal

`choo-choo` in empty directory creates:
```
.tickets/
  STATE.md
  tk (copy of CLI)
specs/
  requirements.md
```

No assumptions about project structure - the design phase figures that out.

---

### Q11: Verify/Close-gaps Loop
**A:** Immediate fix - Per-ticket loop: execute → verify → close-gaps → verify → ... until passing. Maximum retry limit (e.g., 3) before escalating.

---

### Q12: Kanban Interaction
**A:** Tabbed popup (C) - while execution continues in background

| Tab | Content |
|-----|---------|
| Details | Title, description, status, links, parent/dependencies, notes |
| Log | Live `crush run` output tail for this ticket |
| Diff | Files changed, diff view |

---

### Q13: Human Intervention Points
**A:** Phase gates only (B)

| Gate | Action |
|------|--------|
| Design → Plan | Review requirements.md, design.md, approve to continue |
| Plan → Execute | Review ticket tree, approve to start autonomous execution |

Once executing, let it run. Escalate on max retries/blocked tickets.

---

### Q14: Blocked Tickets
**A:** Impossible by design

Blocked tickets are a planning failure, not an execution concern. The `plan` skill must:
- Establish ticket hierarchy (epic → stories → tasks)
- Define explicit dependencies between tickets
- Validate dependency graph is acyclic (`tk dep cycle` check)
- Output execution order that respects dependencies

The `execute` skill follows that order. If execution fails (code errors, tests), that's a gap-closure concern, not a blocked ticket.

---

### Q15: Plan Validation Gate
**A:** Full pre-flight (C) via new `validate` skill

New skill added to taxonomy:
| Skill | Input | Output | Phase |
|-------|-------|--------|-------|
| `design` | Rough idea | requirements.md, design.md | design |
| `plan` | design.md | Ticket tree (epic/stories/tasks) | planning |
| `validate` | Ticket tree | Validation report, execution order | validation |
| `execute` | Ticket, STATE.md | Code, tests, passing test run | execution |
| `verify` | Ticket, implementation | Verification report | verification |
| `close-gaps` | Verification failures | Fixed code, updated tests | gap-closure |

**`validate` checks:**
- Dependency graph integrity (no cycles, all deps exist)
- Ticket completeness (descriptions, acceptance criteria)
- Environment readiness (skills exist, tk CLI available, project structure valid)
- Execution order output (deterministic sequence respecting deps)

**choo-choo UX:** Show validation progress in real-time as Crush runs the validate skill.

---

### Q16: Execution Concurrency
**A:** Parallel with clean fallback

**Metadata from `validate`:**
```yaml
execution_plan:
  parallel: true  # false forces sequential
  levels:
    - level: 0
      tickets: [T-001, T-002, T-003]
    - level: 1
      tickets: [T-004, T-005]
    - level: 2
      tickets: [T-006]
```

**choo-choo implementation:**
- Parallel mode: goroutines per level, `exec.Command` per ticket
- Sequential mode (`--sequential` flag): iterate tickets in order, single process
- Fallback: if parallel fails/hangs, user can restart with `--sequential`

**Failure handling per level:**
- Continue other tickets in same level
- Mark failed ticket, log to STATE.md
- Proceed to next level with remaining tickets

---

## Requirements Summary

**choo-choo**: A TUI orchestration layer that wraps Crush, providing structured design → plan → validate → execute → verify → close-gaps workflow.

### Deliverables

| Location | What |
|----------|------|
| `~/Code/agent-skills/choo-choo-skills/` | 6 skills + meta skill: `design`, `plan`, `validate`, `execute`, `verify`, `close-gaps` |
| `~/Code/choo-choo/` | Go/bubbletea TUI app |

### Skills

| Skill | Input | Output |
|-------|-------|--------|
| `design` | Rough idea | requirements.md, design.md |
| `plan` | design.md | Ticket tree with hierarchy + deps |
| `validate` | Ticket tree | Validation report, execution plan |
| `execute` | Ticket | Code, tests |
| `verify` | Implementation | Verification report |
| `close-gaps` | Failures | Fixed code |

### choo-choo UX Flow

1. **Init**: `choo-choo` in empty dir → creates `.tickets/`, `specs/`
2. **Design**: Interactive chat + artifact preview (two-pane)
3. **Gate**: Review/approve design
4. **Plan**: Non-interactive, shows progress
5. **Gate**: Review ticket tree, validation report
6. **Execute**: Kanban view, parallel execution, tabbed ticket popups
7. **Done**: All tickets complete

### Key Decisions

- STATE.md for session handoff (Ralph-style fresh context)
- Phase gates for human intervention
- Blocked tickets impossible by design (validate ensures valid execution order)
- Parallel execution with `--sequential` fallback
- Full Crush delegation for all AI work

---

## Final Skill Taxonomy

