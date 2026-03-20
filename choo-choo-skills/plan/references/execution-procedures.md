# Execution Procedures

This document defines how to execute a ticket plan, bridging long-horizon ticket planning with near-term todo task management.

## Execution Philosophy

1. **One Ticket at a Time**: Never work on multiple tickets simultaneously
2. **Tests First**: Write all tests before any implementation code
3. **Atomic Work Units**: Break tickets into smallest possible units of work
4. **Complete Before Moving On**: Finish each ticket fully (commit + push) before starting the next
5. **Document Blockers**: If blocked, add a note and move to the next ready ticket

---

## The Execution Cycle

The execution cycle is a strict loop that must be followed without deviation:

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
│  7. CLOSE ticket, COMMIT, PUSH                                  │
│         ↓                                                        │
│  8. REPEAT from step 1                                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Step 1: Select Next Ticket

```bash
# List unblocked tickets ready to work
./scripts/ticket ready

# Filter by owner if applicable
./scripts/ticket ready -o agent
# Or filter for human-owned tickets
./scripts/ticket ready -o human
```

**Prioritization order:**
1. **Priority**: Lower number = higher priority (0 is highest)
2. **Dependency Order**: Tickets blocking others get preference
3. **FIFO**: Within same priority, earlier tickets first

```bash
# See what's blocking other work (prioritize these)
./scripts/ticket blocked
```

Mark the ticket as in progress:
```bash
./scripts/ticket start task-xxx
```

---

## Step 2: Break Down into Atomic Work Units

Use the agent's builtin `todo` tool to decompose the ticket into atomic units of work.

**View the ticket first:**
```bash
./scripts/ticket show task-xxx
```

**Atomic unit criteria:**
- Has a clear, testable outcome
- Cannot be meaningfully subdivided further
- Maps to one or more test cases

**Create todo items following this structure:**

```json
[
  {"content": "Write test: [specific test case 1]", "status": "pending", "active_form": "Writing test: [specific test case 1]"},
  {"content": "Write test: [specific test case 2]", "status": "pending", "active_form": "Writing test: [specific test case 2]"},
  {"content": "Write test: [edge case test]", "status": "pending", "active_form": "Writing test: [edge case test]"},
  {"content": "Implement: [atomic unit 1]", "status": "pending", "active_form": "Implementing: [atomic unit 1]"},
  {"content": "Implement: [atomic unit 2]", "status": "pending", "active_form": "Implementing: [atomic unit 2]"},
  {"content": "Verify all tests pass", "status": "pending", "active_form": "Verifying all tests pass"},
  {"content": "Verify Definition of Done", "status": "pending", "active_form": "Verifying Definition of Done"}
]
```

**CRITICAL**: All test-writing todos must come before all implementation todos.

---

## Step 3: Write All Tests First

**This is Test-Driven Development. No implementation code should be written at this stage.**

### 3.1 Start with the first test

Mark the first test todo as `in_progress`:
```json
{"content": "Write test: [specific test case 1]", "status": "in_progress", ...}
```

### 3.2 Write failing tests

For each test:
1. Create the test file if it doesn't exist
2. Write the test case based on the ticket's acceptance criteria
3. Run the test - it MUST fail (no implementation exists yet)
4. Mark the test todo as completed
5. Move to the next test

### 3.3 Test Coverage Requirements

Tests must cover:
- **Happy path**: Normal expected behavior
- **Edge cases**: Boundary conditions, empty inputs, max values
- **Error cases**: Invalid inputs, permission failures, network errors
- **Integration points**: If the ticket involves external systems

### 3.4 All tests must be written

Do not proceed to Step 4 until ALL test todos are marked complete.

```bash
# Run all tests - they should all fail (expected)
# This confirms your tests are correctly checking the behavior
```

---

## Step 4: Re-Analyze the Breakdown

Before writing any implementation, critically review your breakdown:

**Self-Critique Questions:**
1. Are the atomic units truly atomic? (Can any be split further?)
2. Does each implementation unit map to specific tests?
3. Are there missing test cases you didn't think of initially?
4. Is the order of implementation logical?
5. Are there dependencies between implementation units you missed?

**Adjust the todo list if needed:**
- Add missing tests → go back to Step 3
- Split too-large implementation units
- Reorder implementation units if dependencies require it
- Remove redundant items

**Do not proceed until satisfied with the breakdown.**

---

## Step 5: Implement One Atomic Unit at a Time

### 5.1 Pick the next implementation todo

Mark it as `in_progress`:
```json
{"content": "Implement: [atomic unit 1]", "status": "in_progress", ...}
```

### 5.2 Write the minimum code to pass related tests

- Write only enough code to make the relevant tests pass
- Do not implement beyond what the tests require
- Follow the code patterns referenced in the ticket

### 5.3 Run tests frequently

```bash
# Run tests related to this atomic unit
# Fix any failures
```

### 5.4 Mark complete and repeat

Once the atomic unit is complete and its tests pass:
1. Mark the implementation todo as completed
2. Pick the next implementation todo
3. Repeat until all implementation todos are complete

### 5.5 Final test run

After all implementation todos are complete:
```bash
# Run ALL tests to ensure nothing broke
# All tests must pass before proceeding
```

---

## Step 6: Verify Definition of Done

Before closing the ticket, verify every item in the Definition of Done:

**Common Definition of Done items:**
- [ ] Implementation complete
- [ ] All tests pass (unit, integration, edge cases)
- [ ] No regressions (existing tests still pass)
- [ ] Code follows project conventions
- [ ] No linting errors
- [ ] Performance acceptable (if applicable)
- [ ] Security considerations addressed (if applicable)

**If any item is not met:**
1. Do not close the ticket
2. Add todos for remaining work
3. Complete the work
4. Re-verify

---

## Step 7: Close, Commit, Push

### 7.1 Close the ticket

```bash
./scripts/ticket close task-xxx
```

### 7.2 Create a git commit

```bash
# Stage all changes
git add -A

# Create commit with descriptive message
git commit -m "feat: implement [feature description]

Closes: task-xxx

- [Key change 1]
- [Key change 2]
- [Key change 3]"
```

### 7.3 Push to remote

```bash
git push
```

### 7.4 Add note to ticket (optional)

```bash
./scripts/ticket add-note task-xxx "Completed and pushed. Commit: abc123"
```

---

## Step 8: Repeat

Return to [Step 1](#step-1-select-next-ticket) to select the next ready ticket.

The cycle continues until all tickets in the plan are closed.

---

## Summary Checklist

For each ticket, ensure:

| Step | Action | Done |
|------|--------|------|
| 1 | Selected next ready ticket | ☐ |
| 2 | Broke down into atomic units (todo tool) | ☐ |
| 3 | Wrote ALL tests first (no implementation) | ☐ |
| 4 | Re-analyzed and adjusted breakdown | ☐ |
| 5 | Implemented one unit at a time | ☐ |
| 5 | All tests pass | ☐ |
| 6 | Verified Definition of Done | ☐ |
| 7 | Closed ticket | ☐ |
| 7 | Created git commit | ☐ |
| 7 | Pushed to remote | ☐ |

---

## Working with Dependencies

### Dependency Order Enforcement

The `./scripts/ticket ready` command only shows tickets whose dependencies are closed. This enforces correct ordering automatically.

### If You Must Work Out of Order

**Avoid this if at all possible.** If business needs absolutely require working a ticket before its dependencies:

1. **Document the risk**:
   ```bash
   ./scripts/ticket add-note task-xxx "WARNING: Started before dependency task-yyy closed. May need rework."
   ```

2. **Update dependency** if it turns out to not be a real dependency:
   ```bash
   ./scripts/ticket undep task-xxx task-yyy
   ```

### When Blocked During Execution

If you discover a new blocker mid-task:

1. **Commit any work in progress** (if substantial)
2. **Add the dependency**:
   ```bash
   ./scripts/ticket dep task-xxx newly-discovered-dep
   ```
3. **Add a note**:
   ```bash
   ./scripts/ticket add-note task-xxx "Blocked: Need to complete newly-discovered-dep first"
   ```
4. **Move to another ready ticket**:
   ```bash
   ./scripts/ticket ready
   ```

---

## Session Handoff

### Starting a New Session

1. Check what's in progress:
   ```bash
   ./scripts/ticket ready | grep in_progress
   ```

2. If continuing previous work:
   ```bash
   ./scripts/ticket show task-xxx
   ```

3. If starting fresh:
   ```bash
   ./scripts/ticket ready
   ```

### Ending a Session

1. **Complete or commit work in progress**:
   - If substantial progress: commit with WIP marker
   - If near completion: finish the ticket, close, commit, push

2. **Update ticket status**:
   ```bash
   ./scripts/ticket status task-xxx in_progress
   ```

3. **Add progress note**:
   ```bash
   ./scripts/ticket add-note task-xxx "Completed X and Y. Z remaining. Blocker: [if any]"
   ```

4. **Push any unpushed commits**:
   ```bash
   git push
   ```

---

## Integration Testing Flow

### Testing Requirements

For **CLI applications, TUIs, and long-running processes**:

**MANDATORY:** Use the [test](../../test/) skill. NEVER use raw tmux or blocking sleep commands.

❌ **FORBIDDEN:**
```bash
sleep 15 && tmux -S .tmp/agent.sock capture-pane -p -t test-app
```

✅ **REQUIRED:**
```bash
test wait app "Ready" 30
test capture app
test run app "./myapp" -- --wait "Ready" --expect "Success"
```

### When to Run Integration Tests

After completing a story's tasks:

```bash
# Show story and its children
./scripts/ticket show story-xxx

# Verify all tasks closed
# Then run integration test ticket
./scripts/ticket start task-integ-story-xxx
```

### Integration Test Execution

1. Load integration test ticket
2. Execute each test scenario using test skill commands
3. Document any failures:
   ```bash
   ./scripts/ticket add-note task-integ-xxx "Scenario 2 failed: [details]"
   ```
4. Create bug tickets for failures:
   ```bash
   ./scripts/ticket create "Bug in login redirect" --type bug -d "Scenario 2 failure details"
   ```
5. Close integration test only when all scenarios pass

---

## Completion Verification

### Story Completion

A story is complete when:
1. All child tasks are `closed`
2. Integration test ticket is `closed`
3. All acceptance criteria verified

```bash
# Verify story tasks
./scripts/ticket show story-xxx
```

### Epic Completion

An epic is complete when:
1. All child stories are complete
2. All integration tests pass
3. Completion checklist ticket is `closed`

```bash
# Verify epic structure
./scripts/ticket dep tree epic-xxx

# Check for any open tickets
./scripts/ticket ready | grep epic-xxx
```

---

## Troubleshooting

### No Ready Tickets

If `./scripts/ticket ready` returns empty:
1. Check for blocked tickets: `./scripts/ticket blocked`
2. Check for dependency cycles: `./scripts/ticket dep cycle`
3. Create missing dependencies or unblock by closing dependencies

### Ticket Too Large

If a task ticket feels overwhelming:
1. Split into sub-tasks:
   ```bash
   ./scripts/ticket create "Subtask 1" --type task --parent task-xxx
   ./scripts/ticket create "Subtask 2" --type task --parent task-xxx
   ```
2. Add dependencies between sub-tasks if needed
3. Close original task when sub-tasks complete

### Unexpected Dependencies

If you discover work that should block others:
1. Create the new task
2. Add it as a dependency to affected tickets
3. Check `./scripts/ticket blocked` to see impact

### Need to Reorder

If priorities change mid-execution:
1. Update priority on affected tickets:
   ```bash
   # Priority is in YAML frontmatter
   # Edit directly or recreate with correct priority
   ```
2. Re-run `./scripts/ticket ready` to see updated order
