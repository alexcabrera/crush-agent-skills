# Ticket Templates

Each ticket type has required sections that must be present for the ticket to be considered complete. An agent should be able to execute a task ticket with ONLY that ticket and the current codebase state.

## Ticket Types

| Type | Scope | Duration | Purpose |
|------|-------|----------|---------|
| `epic` | Project-level | 1-4 weeks | Major feature or initiative |
| `story` | User-facing | 1-3 days | Deliverable user value |
| `task` | Technical | Hours to 1 day | Implementation unit |
| `sub-task` | Granular | Hours | Specific implementation detail |
| `bug` | Fix | Varies | Defect resolution |
| `chore` | Maintenance | Varies | Non-feature work |

**Atomic Task Principle**: Each task ticket maps to exactly one commit. If implementation requires multiple logical commits, the task should be split into multiple tickets. This ensures:
- Clear git history with one commit per unit of work
- Easy bisecting to find when issues were introduced
- Each commit is independently reviewable and revertable

---

## Epic Template

Epics group related stories and provide high-level tracking.

### Required Sections

```markdown
# [Epic Title]

## Summary
[1-3 sentence description of the initiative]

## Business Value
[Why this matters. What problem it solves. Expected impact.]

## Scope
[What's included and explicitly what's NOT included]

## Success Metrics
[How we measure completion. Concrete, testable criteria.]

## Stories
[List of story ticket IDs that comprise this epic]
- [ ] story-xxx: [Story title]
- [ ] story-yyy: [Story title]

## Risks & Mitigations
[Known risks and planned mitigations]

## Dependencies
[External dependencies, other epics, infrastructure needs]

## Timeline
[Target dates or relative ordering if applicable]
```

### Example Creation

```bash
./scripts/ticket create "User Authentication System" \
  --type epic \
  --priority 1 \
  -d "Implement complete user authentication with OAuth2, MFA, and session management"
```

---

## Story Template

Stories represent user-facing functionality that delivers value.

### Required Sections

```markdown
# [Story Title]

## User Story
As a [type of user], I want to [action], so that [benefit].

## Acceptance Criteria
- [ ] [Criterion 1 - testable]
- [ ] [Criterion 2 - testable]
- [ ] [Criterion 3 - testable]

## Tasks
[List of task ticket IDs]
- [ ] task-xxx: [Task title]
- [ ] task-yyy: [Task title]

## Out of Scope
[Explicitly what this story does NOT cover]

## Design Notes
[Key design decisions, diagrams, or references]
```

### Example Creation

```bash
./scripts/ticket create "User Login Flow" \
  --type story \
  --parent epic-auth \
  --priority 1 \
  -d "Allow users to log in with email/password"
```

---

## Task Template (Primary Execution Unit)

Tasks are the primary unit of work for agents. A task must be **self-contained** - an agent should be able to complete it with ONLY this ticket and the current codebase.

### Required Sections

```markdown
# [Task Title]

## Task Summary
[1-2 sentences describing what needs to be done]

## Background Context
[Why this task exists. How it fits into the larger feature.
 Reference parent story/epic IDs. Include relevant history.]

## Documentation Requirements
[What documentation must be created BEFORE implementation]
- [ ] API specification for [endpoint/function]
- [ ] Usage examples
- [ ] Error handling documentation
- [ ] Configuration options

**Documentation Location**: [Where docs should be created]
- `docs/api/[feature].md` or
- `README.md` section or
- Inline code comments

## Implementation Details
[Specific technical guidance. Be explicit about:]
- Files/modules to modify or create
- APIs to implement or consume
- Database schema changes
- Configuration changes
- Key algorithms or patterns to use

### Files Affected
[List specific files that will be created or modified]
- `path/to/file.ext` - [what changes]
- `path/to/other.ext` - [what changes]

### Code Patterns
[Reference existing code that demonstrates the pattern to follow]
See `src/existing-module.ts:45-67` for similar implementation.

## Definition of Done
[Concrete, testable criteria. All must be true when complete.]
- [ ] Documentation written and reviewed
- [ ] Tests written (from documentation)
- [ ] Implementation complete
- [ ] All tests pass
- [ ] [Specific functional requirement met]
- [ ] [Specific non-functional requirement met]

## Edge Cases, Gotchas, and Warnings
[Known pitfalls, edge cases to handle, things that could go wrong]
- **Edge Case 1**: [Description and how to handle]
- **Warning**: [Something to be careful about]
- **Known Issue**: [Existing issue that affects this task]

## Tests to be Created
[Specific test cases that must be written AFTER documentation]
- `[ ] Unit test: [test name/description]
- `[ ] Integration test: [test name/description]
- `[ ] Edge case test: [test name/description]
- `[ ] Documentation examples tested

## Dependencies
[Tasks that must complete first, if any]
- task-xxx must complete (blocking) - typically documentation ticket
```

### Example Creation

```bash
./scripts/ticket create "Implement password hashing" \
  --type task \
  --parent story-login \
  -d "Add bcrypt password hashing to user registration and login" \
  --design "Use bcrypt with cost factor 12. See src/utils/crypto.ts for pattern." \
  --acceptance "- Passwords hashed before storage\n- Login validates against hash\n- Cost factor configurable"
```

### Completeness Checklist

Before considering a task ticket complete, verify:
- [ ] An agent unfamiliar with the project could complete this task
- [ ] Documentation requirements are specified
- [ ] All file paths are accurate (verify they exist or are clearly new)
- [ ] Referenced code patterns actually exist and demonstrate the approach
- [ ] Acceptance criteria are testable without ambiguity
- [ ] Edge cases cover the likely failure modes
- [ ] Test requirements are specific enough to implement
- [ ] Definition of Done includes documentation, tests, and code

---

## Sub-task Template

Sub-tasks are optional for granular breakdown within a task.

### Required Sections

```markdown
# [Sub-task Title]

## Parent Task
[Link to parent task ID]

## Objective
[Single sentence describing the specific outcome]

## Steps
[Ordered list of steps to complete]
1. [Step 1]
2. [Step 2]

## Expected Output
[What the completed sub-task produces]
```

---

## Bug Template

Bugs track defects that need fixing.

### Required Sections

```markdown
# [Bug Title]

## Description
[Clear description of the bug]

## Steps to Reproduce
1. [Step 1]
2. [Step 2]
3. [Step 3]

## Expected Behavior
[What should happen]

## Actual Behavior
[What actually happens]

## Environment
- OS: [operating system]
- Version: [affected version/commit]
- Browser/Runtime: [if applicable]

## Root Cause Analysis
[Why this happens - fill in during investigation]

## Fix Approach
[How to fix - fill in during investigation]

## Regression Test
[What test prevents this from recurring]
```

### Example Creation

```bash
./scripts/ticket create "Login fails with special characters in password" \
  --type bug \
  --priority 0 \
  -d "Users with special characters (!@#$%) in passwords cannot log in"
```

---

## Chore Template

Chores track maintenance and non-feature work.

### Required Sections

```markdown
# [Chore Title]

## Objective
[What this chore accomplishes]

## Motivation
[Why this is needed now]

## Tasks
- [ ] [Task 1]
- [ ] [Task 2]

## Impact
[What's affected. Any breaking changes or migrations needed.]

## Verification
[How to verify this chore is complete]
```

---

## Documentation Ticket Template

Documentation tickets should be created as dependencies for implementation tickets. They define the specification that tests and code must follow.

### Required Sections

```markdown
# [Documentation Title]

## Scope
[What is being documented - feature, API, process, etc.]

## Audience
[Who will read this documentation]

## Required Sections
[What the documentation must include]
- [ ] Overview/Purpose
- [ ] Technical specification
- [ ] Usage examples
- [ ] Error handling
- [ ] Edge cases
- [ ] Configuration options (if applicable)

## Acceptance Criteria
[How we know the documentation is complete and accurate]
- [ ] All required sections present
- [ ] Examples are tested and working
- [ ] No ambiguity in specifications
- [ ] Reviewer has approved

## Location
[Where the documentation should be created]
- `docs/[filename].md`
- Or inline code comments
- Or README section

## Related Implementation Tickets
[Tickets that depend on this documentation]
- task-xxx
- task-yyy
```

### Example Creation

```bash
./scripts/ticket create "Document auth API specification" \
  --type task \
  --priority 0 \
  -d "Complete API specification for authentication endpoints including all request/response formats, error codes, and usage examples"
```

---

## Rollup Tickets

Rollup tickets track the completion of groups of work.

### Integration Test Ticket

Created at each level (epic, story) to verify integrated functionality.

```markdown
# Integration: [Feature Name]

## Scope
[What functionality is being integration tested]

## Prerequisites
[Tickets that must be complete first]
- [ ] task-xxx
- [ ] task-yyy

## Test Scenarios
- [ ] [Scenario 1: steps and expected result]
- [ ] [Scenario 2: steps and expected result]

## Environment Setup
[Any special setup needed for integration testing]
```

### Completion Checklist Ticket

Created to verify all work in an epic/story is truly done.

```markdown
# Checklist: [Epic/Story Name]

## Tickets
- [ ] task-xxx: [status] - [notes]
- [ ] task-yyy: [status] - [notes]

## Verification Steps
- [ ] All acceptance criteria met
- [ ] All tests passing
- [ ] Documentation updated
- [ ] Code review complete

## Sign-off
- [ ] Functional verification complete
- [ ] Performance acceptable
- [ ] No regressions introduced
```
