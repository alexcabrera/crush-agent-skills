# Planning Procedures

This document defines the methodology for creating robust, self-critical ticket plans.

## Planning Philosophy

1. **Documentation as Specification**: Documentation defines the contract; tests implement the contract; code fulfills the contract
2. **Self-Sufficiency**: Each task ticket must be completable by an agent with ONLY that ticket + the current codebase
3. **Explicit Over Implicit**: State assumptions, dependencies, and risks explicitly
4. **Pressure-Test Decisions**: Multiple validation gates before finalizing
5. **Traceability**: Every decision should have a rationale that can be reviewed

---

## Phase 0: Input Source

Before starting planning, determine where your requirements come from:

### 0.1 From Prompt-Driven Development (Recommended)

If you have PDD output from the `prompt-driven-development` skill:

```bash
# Check for PDD output
ls specs/{task-name}/
# Expected: design.md, plan.md, requirements.md, research/
```

**When PDD output exists:**
1. Read `specs/{task-name}/design.md` — this is your source of truth for requirements
2. Read `specs/{task-name}/plan.md` — this provides high-level implementation steps
3. **Skip Phase 1 (Discovery)** — PDD already did this
4. **Skip Phase 3 (Documentation Planning)** — design.md is your documentation
5. Proceed directly to **Phase 2 (Decomposition)** using design.md as input

**How to decompose from PDD output:**
- Each "Component" in design.md → Story or Task ticket
- Each "Step" in plan.md → Task ticket
- Acceptance Criteria in design.md → Test requirements in tickets
- Architecture diagram → Dependency structure

### 0.2 Starting Fresh (No PDD)

If no PDD output exists:
- Begin with **Phase 1 (Discovery)**
- Complete all phases in order
- Consider using `prompt-driven-development` skill first for complex features

---

## The Creation Order

Work must be created in this order for each feature:

```
┌─────────────────────────────────────────────────────────────────┐
│                     CREATION ORDER                               │
│                                                                  │
│  1. DOCUMENTATION (acts as specification)                       │
│         ↓                                                        │
│  2. TESTS (implement the documentation exactly)                  │
│         ↓                                                        │
│  3. CODE (makes the tests pass)                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Why this order matters:**
- Documentation forces you to think through the problem completely
- Tests derived from documentation validate your understanding
- Code is the last step, not the first
- Each layer validates the layer above it

---

## Phase 1: Discovery

*Skip this phase if you have PDD output — PDD already performed discovery.*

### 1.1 Codebase Survey

Before planning, understand the current state:

```bash
# Understand project structure
find . -type f -name "*.py" -o -name "*.ts" -o -name "*.go" | head -50

# Identify existing patterns
grep -r "pattern-to-match" --include="*.ext" .

# Check existing tests
find . -path "*/test*" -name "*.ext" | head -20

# Review recent changes
git log --oneline -20
```

### 1.2 Stakeholder Analysis

Document:
- Who requested this work?
- What is the success criteria from their perspective?
- What are the constraints (resources, technical, scope)?

### 1.3 Boundary Definition

Explicitly define:
- **In Scope**: What this plan covers
- **Out of Scope**: What is explicitly excluded
- **Dependencies**: External systems, APIs, libraries required
- **Assumptions**: Conditions assumed to be true

---

## Phase 2: Decomposition

### 2.0 If Coming from PDD

When you have `specs/{task-name}/design.md`:

1. **Extract components from design.md:**
   - Each major component becomes a Story or Task
   - Component interfaces become acceptance criteria
   - Component dependencies become ticket dependencies

2. **Extract steps from plan.md:**
   - Each step becomes one or more Tasks
   - Step objectives become ticket descriptions
   - Step test requirements become ticket test sections

3. **Map acceptance criteria:**
   - Each Given-When-Then in design.md → Test case in ticket
   - These are non-negotiable requirements

**Example mapping:**

```
design.md:
  ## Components and Interfaces
  ### RateLimiter                    →  task-rate-limiter-core
  ### RateLimitMiddleware            →  task-rate-limit-middleware
  
  ## Acceptance Criteria
  ### AC1: Authenticated User Limit  →  Included in test section of tasks
  ### AC2: Anonymous User Limit       →  Included in test section of tasks

plan.md:
  ## Step 1: Token Bucket Algorithm  →  task-token-bucket
  ## Step 2: Redis Integration        →  task-redis-integration
  ## Step 3: Middleware Layer         →  task-rate-limit-middleware
```

### 2.1 Hierarchical Breakdown

Start with epics, decompose to stories, then tasks:

```
Epic: User Authentication
├── Story: User Registration
│   ├── Task: Create user model
│   ├── Task: Implement password hashing
│   └── Task: Build registration endpoint
├── Story: User Login
│   ├── Task: Implement JWT generation
│   ├── Task: Create login endpoint
│   └── Task: Add session management
└── Story: Password Reset
    ├── Task: Generate reset tokens
    └── Task: Create reset endpoint
```

### 2.2 Task Granularity Rules

A task is the right size when:
- Has clear inputs and outputs
- Can be tested independently
- A single agent can own it end-to-end
- Cannot be meaningfully subdivided further

If a task is larger, split it. If smaller, consider merging with related work.

### 2.3 Dependency Identification

For each task, identify:
1. **Hard Dependencies**: Must complete before this task can start
2. **Soft Dependencies**: Should complete before, but not blocking
3. **Reverse Dependencies**: What depends on this task

---

## Phase 3: Documentation Planning

*Skip this phase if you have PDD output — design.md IS your documentation.*

**For PDD-based plans:**
- Reference `specs/{task-name}/design.md` in ticket descriptions
- Do NOT duplicate design.md content in tickets
- Tickets should reference specific sections: "See design.md § Components > RateLimiter"

**For fresh plans:** Continue with the documentation phase below.

---

Documentation is the first artifact created. It serves as the specification that tests and code must follow.

### 3.1 Documentation Required for Each Feature

Every feature must have documentation that answers:

**What is it?**
- Clear description of the feature
- User-facing explanation

**How does it work?**
- Technical implementation overview
- API contracts (endpoints, parameters, responses)
- Data models and schemas
- Algorithm descriptions (if complex)

**How do I use it?**
- Usage examples
- Configuration options
- Integration points

**What could go wrong?**
- Error scenarios
- Edge cases
- Limitations

### 3.2 Documentation Types by Ticket Type

| Ticket Type | Documentation Required |
|-------------|------------------------|
| Epic | Architecture doc, high-level design |
| Story | Feature spec, user documentation |
| Task | API docs, code comments, inline docs |
| Bug | Root cause analysis, fix documentation |

### 3.3 Hypercritical Documentation Review

Documentation must pass this review before any tests or code are written:

**Clarity Review:**
- [ ] Could someone unfamiliar with the project understand this?
- [ ] Are all terms defined or commonly understood?
- [ ] Is the language unambiguous?
- [ ] Are there examples for every claim?

**Completeness Review:**
- [ ] Are all inputs and outputs specified?
- [ ] Are all error cases documented?
- [ ] Are all edge cases covered?
- [ ] Are all configuration options listed?

**Consistency Review:**
- [ ] Does this contradict any existing documentation?
- [ ] Are naming conventions consistent?
- [ ] Are data types consistent across the doc?

**Implementability Review:**
- [ ] Can tests be written directly from this documentation?
- [ ] Are all dependencies mentioned?
- [ ] Are all assumptions stated explicitly?

**Self-Critique Questions:**
- What assumptions am I making that aren't written down?
- What would confuse me if I read this 6 months from now?
- What edge cases am I ignoring because they're inconvenient?
- What would break if the system is under load?
- What would a malicious user try to exploit?

### 3.4 Documentation Tickets

Create documentation tickets as dependencies for implementation tickets:

```bash
# Documentation comes first
./scripts/ticket create "Document auth API spec" \
  --type task \
  --priority 1 \
  -d "Create complete API specification for authentication endpoints"

# Implementation depends on documentation
./scripts/ticket create "Implement auth endpoints" \
  --type task \
  --priority 1

# Establish dependency
./scripts/ticket dep task-auth-impl task-auth-docs
```

---

## Phase 4: Dependency Mapping

### 4.1 Create Dependency Graph

```bash
# Add dependencies
./scripts/ticket dep task-jwt task-user-model    # JWT task needs user model
./scripts/ticket dep task-login task-jwt          # Login needs JWT
./scripts/ticket dep task-reset task-email        # Reset needs email service
```

### 4.2 Validate Dependency Structure

```bash
# Check for cycles
./scripts/ticket dep cycle

# View the tree
./scripts/ticket dep tree epic-auth --full
```

### 4.3 Dependency Anti-Patterns to Avoid

- **Cycles**: A -> B -> C -> A (detected by `./scripts/ticket dep cycle`)
- **Diamond Problem**: A depends on B and C, both depend on D (usually fine, but verify)
- **Orphaned Tasks**: Tasks with no path to/from epics
- **Deep Chains**: More than 5 levels of dependencies (reconsider structure)

---

## Phase 5: Self-Reflection Gates

Before finalizing the plan, pass through these validation gates.

### Gate 1: Completeness Check

**Checklist:**
- [ ] Every file/module identified in discovery has at least one task
- [ ] All affected areas covered (frontend, backend, database, tests, docs)
- [ ] Testing strategy defined at each level
- [ ] Documentation updates included if needed

**Self-Critique Questions:**
- What did I miss?
- What would a newcomer to this codebase not know?
- What would break if I stopped halfway through?

### Gate 2: Dependency Integrity

**Checklist:**
- [ ] `./scripts/ticket dep cycle` returns clean
- [ ] All dependencies resolve to existing tickets
- [ ] No orphaned tickets (all connect to an epic)
- [ ] Critical path identified and documented

**Self-Critique Questions:**
- Are there hidden dependencies I haven't captured?
- Could any task be started in parallel that I've marked as sequential?
- What if a dependency fails or is blocked?

### Gate 3: Task Clarity

**For each task ticket, verify:**
- [ ] An agent unfamiliar with the project could complete this
- [ ] All file paths are accurate (verified they exist or clearly new)
- [ ] Code pattern references point to actual working examples
- [ ] Acceptance criteria are unambiguous and testable
- [ ] Edge cases cover likely failure modes

**Self-Critique Questions:**
- What would confuse someone reading this ticket?
- What context am I assuming that isn't written down?
- Could this be misinterpreted? How?

### Gate 4: Risk Assessment

**Checklist:**
- [ ] High-risk items identified and flagged (priority 0-1)
- [ ] Rollback strategies noted for risky changes
- [ ] External dependencies documented with fallback plans
- [ ] Performance implications considered

**Self-Critique Questions:**
- What could go catastrophically wrong?
- What changes are irreversible?
- Where are the unknowns?

### Gate 5: Reality Check

**Checklist:**
- [ ] Scope estimates are reasonable
- [ ] Required capabilities are available
- [ ] Infrastructure/tooling is available
- [ ] No blockers from external teams/deps

**Self-Critique Questions:**
- Am I underestimating scope?
- What have I never done before in this plan?
- What external factors could derail this?

### Gate 6: Documentation Quality

**Checklist:**
- [ ] All features have documentation tickets as dependencies
- [ ] Documentation passes the hypercritical review (see Phase 3.3)
- [ ] Tests can be written directly from documentation
- [ ] No feature starts implementation before documentation is complete

**Self-Critique Questions:**
- Is the documentation specific enough to write tests from?
- What ambiguity exists that could lead to wrong implementation?
- Does documentation match existing system behavior?

---

## Phase 6: Rollup & Tracking Tickets

### 6.1 Create Integration Test Tickets

At each level, create integration verification:

```bash
# Story-level integration
./scripts/ticket create "Integration: Login Flow" \
  --type task \
  --parent story-login \
  -d "Verify complete login flow end-to-end"

# Epic-level integration
./scripts/ticket create "Integration: Auth System" \
  --type task \
  --parent epic-auth \
  -d "Verify all auth flows work together"
```

### 6.2 Create Completion Checklists

```bash
./scripts/ticket create "Checklist: Auth Epic Complete" \
  --type task \
  --parent epic-auth \
  -d "Verify all auth tickets are complete and verified"
```

### 6.3 Wire Integration Dependencies

```bash
# Integration tests depend on their component tasks
./scripts/ticket dep task-integ-login task-jwt task-login-endpoint
./scripts/ticket dep task-integ-auth task-integ-login task-integ-register
```

---

## Phase 7: Plan Review

### 7.1 Generate Plan Overview

```bash
# Show the full structure
./scripts/ticket dep tree epic-main --full

# List all tickets by status
./scripts/ticket ready
./scripts/ticket blocked
```

### 7.2 Document the Plan

Create a plan summary file:

```markdown
# [Feature Name] Plan

## Overview
[1-2 paragraph summary]

## Epics
- epic-xxx: [Epic Name] - [status]
  - story-yyy: [Story Name] - [status]

## Critical Path
[Ordered list of tickets on the critical path]

## Risks
[Top 3 risks and mitigations]

## Success Criteria
[How we know the plan is complete]
```

### 7.3 Final Validation

Before executing:
1. Walk through the plan end-to-end mentally
2. Verify each ticket against the templates
3. Confirm dependency order matches logical implementation order
4. Ensure test coverage is planned at all levels

---

## Anti-Patterns to Avoid

1. **Vague Tasks**: "Improve performance" without specifics
2. **Missing Context**: Tasks that assume unwritten knowledge
3. **Hidden Dependencies**: Tasks that implicitly need other work
4. **Orphaned Work**: Tasks not connected to any story/epic
5. **No Test Strategy**: Tasks without defined testing approach
6. **No Documentation**: Features without specs
7. **Code-First Thinking**: Writing code before documentation and tests
8. **Assumed Success**: No rollback or error handling consideration
9. **Perfect Planning**: Endless planning without execution

---

## Quick Reference: Planning Checklist

Before finalizing any plan:

```
[ ] Input Source: PDD output found OR starting fresh
[ ] Discovery: Codebase surveyed, boundaries defined (skip if PDD)
[ ] Decomposition: Epics → Stories → Tasks, right granularity
[ ] Documentation: Specs created for all features (or referenced from PDD)
[ ] Dependencies: All captured, no cycles, no orphans
[ ] Gate 1 - Completeness: All areas covered
[ ] Gate 2 - Integrity: Dependencies validated
[ ] Gate 3 - Clarity: Tasks are self-sufficient
[ ] Gate 4 - Risk: High-risk items flagged
[ ] Gate 5 - Reality: Scope is well-defined
[ ] Gate 6 - Documentation: Specs are testable
[ ] Rollups: Integration tests and checklists created
[ ] Review: Plan documented and validated
```

---

## PDD → Ticket Mapping Reference

When consuming PDD output, use this mapping:

| PDD Artifact | Maps To | Notes |
|--------------|---------|-------|
| design.md → Overview | Epic description | High-level what and why |
| design.md → Components | Stories/Tasks | One ticket per component |
| design.md → Interfaces | Acceptance criteria | Function signatures, API contracts |
| design.md → Data Models | Task details | Schema definitions |
| design.md → Acceptance Criteria | Test requirements | Given-When-Then → test cases |
| design.md → Error Handling | Task edge cases | Error scenarios to handle |
| plan.md → Step N | Task(s) | May split into multiple tasks |
| plan.md → Tests Required | Ticket test section | What tests must pass |
| plan.md → Integration | Dependencies | What this step depends on |
| requirements.md | Context only | Already incorporated in design |
| research/ | Reference | Link for implementation details |
