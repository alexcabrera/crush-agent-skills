# Requirements Clarification

This document provides detailed guidance on the iterative Q&A process for clarifying requirements.

## Philosophy

Requirements clarification is not a one-time activity. It's an ongoing conversation that:

1. **Starts with the unknown** - We begin with what we don't know
2. **Uncovers hidden assumptions** - Both user and agent have implicit assumptions
3. **Validates understanding** - Each answer confirms or corrects our mental model
4. **Builds the spec incrementally** - The requirements.md file grows organically

## The Question Cycle

For each question, follow this strict cycle:

```
┌─────────────────────────────────────────────────────────────────┐
│                    QUESTION CYCLE                                │
│                                                                  │
│  1. FORMULATE question                                          │
│         ↓                                                        │
│  2. APPEND question to requirements.md                          │
│         ↓                                                        │
│  3. PRESENT to user                                             │
│         ↓                                                        │
│  4. WAIT for complete response                                  │
│         ↓                                                        │
│  5. APPEND answer to requirements.md                            │
│         ↓                                                        │
│  6. REFLECT on answer → formulate next question                 │
│         ↓                                                        │
│  7. REPEAT until user confirms complete                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Question Categories

### 1. Core Functionality Questions

These establish what the system does.

**Template questions:**
- "What is the primary purpose of [feature]?"
- "Who are the intended users of this feature?"
- "What problem does this solve that isn't solved today?"
- "What does the simplest version of this look like?"
- "What would make this feature complete vs. minimal?"

**What to listen for:**
- Ambiguous terms that need definition
- Assumptions about user behavior
- Implicit dependencies on other systems
- Scope creep indicators

### 2. User Experience Questions

These establish how users interact with the system.

**Template questions:**
- "Walk me through how a user would [accomplish goal]"
- "What does the user see when [scenario]?"
- "What feedback does the user get when [action]?"
- "How does the user know [something succeeded/failed]?"
- "What are the most common user paths?"

**What to listen for:**
- Multiple user types with different needs
- Error states and edge cases
- Accessibility requirements
- Response time expectations

### 3. Technical Constraint Questions

These establish the boundaries of implementation.

**Template questions:**
- "What performance requirements does this have?"
- "How many [users/requests/items] should this handle?"
- "What systems does this need to integrate with?"
- "What platforms/environments must this support?"
- "Are there security or compliance requirements?"
- "What's the expected data volume?"

**What to listen for:**
- Hard limits vs. soft targets
- Scalability concerns
- Technology constraints
- Legacy system integration needs

### 4. Edge Case Questions

These establish what happens outside normal flow.

**Template questions:**
- "What happens if [network fails/timeout/invalid input]?"
- "What's the maximum [size/count/duration] this should handle?"
- "What should happen when [resource is unavailable]?"
- "How should the system behave under [extreme condition]?"
- "What invalid inputs should be rejected, and how?"

**What to listen for:**
- Silent failures vs. visible errors
- Data corruption scenarios
- Partial operation modes
- Recovery expectations

### 5. Success Criteria Questions

These establish how we know when we're done.

**Template questions:**
- "How will you verify this works correctly?"
- "What would make you confident this is production-ready?"
- "What tests would demonstrate this is complete?"
- "What metrics indicate this is successful?"
- "What would a demo of this feature show?"

**What to listen for:**
- Measurable outcomes
- User-observable behavior
- Performance benchmarks
- Integration verification

## Question Quality Criteria

Each question should be:

**Specific:**
- ✗ "How should it work?"
- ✓ "When a user submits the form with invalid data, what feedback do they see?"

**Unbiased:**
- ✗ "Should we use a modal for this?"
- ✓ "How should we present this information to the user?"

**Building on previous answers:**
- ✗ "What about performance?" (after no mention of performance)
- ✓ "You mentioned this needs to handle 1000 users. Is that concurrent or daily?"

**One thing at a time:**
- ✗ "What should the UI look like and how should errors be handled?"
- ✓ "What should the UI look like for the main flow?"

## Recording Format

### requirements.md Template

```markdown
# Requirements: {Feature Name}

## Rough Idea
[Original concept from user]

---

## Clarification Q&A

### Q1: {Question}
**A:** {Answer}

### Q2: {Question}
**A:** {Answer}

...

---

## Summary
[Condensed summary of requirements, updated as clarification progresses]

## Open Questions
- [Question not yet answered]
- [Question that needs more exploration]

## Decisions Made
- {Decision}: {Rationale}
```

## Transition Signals

**Move to research when:**
- User mentions specific technologies to evaluate
- There are questions about what's possible
- Existing solutions or libraries might help
- Technical feasibility is unclear

**Move to design when:**
- Requirements feel complete
- User can describe the full feature
- Success criteria are clear
- No major unknowns remain

## Anti-Patterns

1. **Batching questions** - Asking multiple questions at once loses the iterative benefit
2. **Pre-populating answers** - Don't assume what the user wants
3. **Skipping edge cases** - These are where bugs live
4. **Technical jargon** - Use language the user understands
5. **Rushing to design** - Incomplete requirements lead to rework
6. **Ignoring inconsistencies** - Call out when answers conflict

## Fast Pass Option

If the user wants to move quickly, offer a "fast pass" that covers only:

1. Core purpose (what and why)
2. Primary user flow
3. Success criteria (how we know it's done)
4. Critical constraints (hard limits)

Document that this is a fast pass and may need refinement during implementation.
