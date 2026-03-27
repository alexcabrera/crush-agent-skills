---
name: plan
description: Long-horizon task planning using markdown tickets with dependency tracking. Use for breaking down complex work into structured tickets (epics, stories, tasks). Invoke when planning features spanning multiple context windows, creating project roadmaps, or when work needs to survive context limits. Use agent's builtin todo for immediate tasks; use this skill for work that persists across context resets.
license: MIT
compatibility: "Requires bash, git. Optional: rg (ripgrep) for faster searches."
metadata:
  version: "1.0.0"
  author: agent-skills
---

# Plan Skill

This skill provides structured long-horizon task management using the `tk` ticket system. Tickets are markdown files that persist in the repository, surviving across context windows.

Part of the [agent-skills](../) collection.

## When to Use This Skill

| Scenario | Use |
|----------|-----|
| Immediate tasks (current context) | Agent's builtin `todo` tool |
| Multi-context work | This skill (`tk` tickets) |
| Breaking down a large feature | This skill |
| Creating project roadmaps | This skill |
| Work that needs to survive context limits | This skill |
| Tracking dependencies between tasks | This skill |

### Invocation Triggers

Activate this skill when:
- User asks to "plan" or "break down" a feature or project
- Work will clearly span multiple context windows
- User mentions "roadmap", "epic", "milestone", or similar
- User wants to track dependencies between tasks
- User asks about "tickets" or "issues" in context of planning
- Starting a new major feature or project phase

## Quick Reference

```bash
# Run the bundled ticket script
./scripts/ticket <command> [args]

# For convenience, create an alias
alias tk='./scripts/ticket'
```

### Essential Commands

| Command | Purpose |
|---------|---------|
| `create` | Create a new ticket |
| `show <id>` | Display ticket with relationships |
| `start/close <id>` | Update status |
| `dep <id> <dep-id>` | Add dependency |
| `ready` | List unblocked tickets |
| `blocked` | List blocked tickets |
| `dep tree <id>` | Show dependency tree |

### Ticket Ownership

Each ticket has an `owner` field indicating who is responsible:

```yaml
owner: agent           # Default - any AI agent can pick this up
owner: agent:crush     # Specific agent (e.g., Crush)
owner: human           # Current human operator
```

Filter tickets by owner:
```bash
./scripts/ticket ready -o agent      # Agent-owned tickets
./scripts/ticket ready -o human      # Human-owned tickets
```

---

## Relationship with Design Skill

This skill works in concert with the **design** skill:

```
┌─────────────────────────────────────────────────────────────────┐
│                    SKILL WORKFLOW                                │
│                                                                  │
│  DESIGN SKILL                                                   │
│  (Exploration & Design)                                         │
│         ↓                                                        │
│  specs/{task-name}/design.md   ─────────┐                       │
│  specs/{task-name}/plan.md     ─────────┼──→ PLAN SKILL         │
│                                          │    (Decomposition)    │
│                                          │         ↓             │
│                                          │    .tickets/          │
│                                          │    ├── epic-*.md      │
│                                          │    ├── story-*.md     │
│                                          │    └── task-*.md      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**When to use which skill:**
- **design**: Rough idea → Clarified requirements → Design → Plan
- **plan**: Design → Decomposed tickets → Execution tracking

If you have a design document (from design skill or otherwise), proceed to Planning Workflow.
If you're starting from scratch with a vague idea, use design skill first.

---

## The Creation Order

For each feature, work must proceed in this order:

```
1. DOCUMENTATION (acts as specification)
       ↓
2. TESTS (implement the documentation exactly)
       ↓
3. CODE (makes the tests pass)
```

**Why this order matters:**
- Documentation forces complete thinking before coding
- Tests validate understanding of the spec
- Code is the last step, not the first

---

## Execution Cycle

When executing a plan, follow this strict cycle for EACH ticket:

```
┌─────────────────────────────────────────────────────────────────┐
│                     EXECUTION CYCLE                              │
│                                                                  │
│  1. SELECT next ready ticket                                    │
│         ↓                                                        │
│  2. BREAK DOWN into atomic work units (todo tool)               │
│         ↓                                                        │
│  3. WRITE ALL TESTS FIRST (no implementation code)              │
│         ↓                                                        │
│  4. RE-ANALYZE breakdown, adjust if needed                      │
│         ↓                                                        │
│  5. IMPLEMENT one atomic unit at a time until tests pass        │
│         ↓                                                        │
│  6. VERIFY all Definition of Done criteria met                  │
│         ↓                                                        │
│  7. COMMIT (one commit per ticket - non-negotiable)             │
│         ↓                                                        │
│  8. CLOSE ticket                                                │
│         ↓                                                        │
│  9. REPEAT from step 1                                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

See [Execution Procedures](references/execution-procedures.md) for full details.

---

## Planning Workflow

### 0. Input Source

**If coming from design skill:**
- Read `specs/{task-name}/design.md` — this is your source of truth
- Read `specs/{task-name}/plan.md` — this provides high-level steps
- Skip to Phase 2 (Decomposition) using these as inputs

**If starting fresh (no design output):**
- Begin with Phase 1 (Discovery)

### 1. Discovery Phase

Before creating any tickets:
1. Search the codebase to understand current state
2. Identify all components that will be affected
3. Note existing patterns and conventions
4. List potential edge cases and risks

### 2. Decomposition Phase

Break work into hierarchical tickets:
- **Epic**: Large feature or initiative (major scope)
- **Story**: User-facing functionality (medium scope)
- **Task**: Technical implementation unit (small scope)
- **Sub-task**: Granular implementation detail (atomic scope)

**Atomic Ticket Principle**: Each task ticket should be completable in a single execution cycle. A task = one commit. If a ticket needs multiple commits, split it.

See [Ticket Templates](references/ticket-templates.md) for required sections per type.

### 3. Documentation Phase

**Documentation is created BEFORE tests and code.** It acts as the specification.

For each feature:
1. Create documentation ticket first
2. Documentation must pass hypercritical review
3. Implementation tickets depend on documentation tickets

See [Planning Procedures](references/planning-procedures.md) for documentation requirements.

### 4. Dependency Mapping

Use `dep` to establish relationships:
```
./scripts/ticket dep <dependent-id> <dependency-id>
```

Meaning: "dependent-id depends on dependency-id" (dependency must complete first)

### 5. Validation Phase

Self-reflective checks before finalizing:

**Completeness Check:**
- [ ] Every component identified in discovery has a ticket
- [ ] All affected files/areas are covered
- [ ] Testing strategy is defined per ticket
- [ ] Documentation tickets created for all features
- [ ] CLI/TUI/process tickets include test skill requirements

**Dependency Check:**
- [ ] `./scripts/ticket dep cycle` returns no cycles
- [ ] Dependency order matches logical implementation order
- [ ] No orphaned tickets (unreachable from epics)
- [ ] Implementation depends on documentation

**Clarity Check:**
- [ ] Each task ticket can be completed by an agent with ONLY that ticket + codebase
- [ ] All required sections present per ticket type
- [ ] Acceptance criteria are testable

**Risk Check:**
- [ ] High-risk items identified and prioritized
- [ ] Rollback strategies noted where appropriate
- [ ] External dependencies documented

### 6. Execution Handoff

When ready to execute:
1. Run `./scripts/ticket ready` to see unblocked tickets
2. Follow [Execution Procedures](references/execution-procedures.md)
3. Load one ticket at a time into agent's builtin todo

---

## Testing Requirements

For any ticket involving **CLI applications, TUIs, or long-running processes**:

**MANDATORY:** Use the [test-cli](../test-cli/) skill for all interactive testing.

When planning tickets, ensure:
- CLI/TUI tickets explicitly require the test skill in acceptance criteria
- Integration test tickets reference test skill commands
- No raw `tmux` or blocking `sleep` commands in test procedures

**Why this matters:**
- Raw tmux commands block the agent and require user cancellation
- The test skill provides non-blocking commands with built-in timeouts
- See test skill's "CRITICAL" section for forbidden/required patterns

## Ticket Interlinking Strategy

### Dependency Links (`dep`)
For sequencing: "B cannot start until A completes"
```
./scripts/ticket dep story-auth task-user-model
```

### Soft Links (`link`)
For related work without blocking: "These are related but independent"
```
./scripts/ticket link task-logging task-metrics
```

### Parent Links (`--parent`)
For hierarchy: "This task belongs to this story"
```
./scripts/ticket create "Add password reset" --parent story-auth
```

## File References

- [Ticket Templates](references/ticket-templates.md) - Required sections per ticket type
- [Planning Procedures](references/planning-procedures.md) - Detailed planning methodology
- [Execution Procedures](references/execution-procedures.md) - How to execute a ticket plan

## Related Skills

### Virtuous Cycle
- [elaborate](../elaborate/) - Requirements clarification (start here, produces design.md)
- [execute](../execute/) - Runs the virtuous cycle after plan approval
- [implement](../implement/) - Implements a single ticket
- [verify](../verify/) - Confirms implementation matches acceptance criteria
- [close-gaps](../close-gaps/) - Fixes issues found during verification

### Testing
- [test-cli](../test-cli/) - CLI/TUI testing
- [test-web](../test-web/) - Web application testing

### Documentation
- [document](../document/) - Generates README.md after epic completion
