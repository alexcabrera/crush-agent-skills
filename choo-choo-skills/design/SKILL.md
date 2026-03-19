---
name: design
description: Transforms rough ideas into detailed design documents through iterative requirements clarification, research, and architecture design. Use when starting with a vague concept, exploring a new feature, or when requirements need clarification. Outputs specs that feed into plan skill for decomposition.
license: MIT
compatibility: Requires bash, git.
metadata:
  version: "1.0.0"
  author: choo-choo-skills
---

# Design Skill

Transform a rough idea into a detailed design with an implementation plan. This skill handles the exploration phase — clarifying requirements, researching technologies, and designing architecture — before handing off to the plan skill for decomposition and execution.

Part of the [choo-choo-skills](../) collection.

## When to Use This Skill

| Scenario | Use |
|----------|-----|
| Starting with a vague idea or concept | This skill (design) |
| Requirements need clarification | This skill |
| Need to research technologies/approaches | This skill |
| Architecture design needed | This skill |
| Have clear requirements, ready to implement | plan skill |
| Breaking down work into trackable units | plan skill |

### Invocation Triggers

Activate this skill when:
- User says "I want to build..." or "I have an idea for..."
- Requirements are unclear or incomplete
- User asks to "explore" or "design" a feature
- User mentions needing to "figure out" how something should work
- Starting a major new feature or project

---

## Output Artifacts

PDD creates a specification directory that serves as the source of truth:

```
specs/{task-name}/
├── rough-idea.md      # Original concept (preserved)
├── requirements.md    # Q&A clarification record
├── research/          # Technology research notes
│   ├── {topic-1}.md
│   └── {topic-2}.md
├── design.md          # Architecture and component design
└── plan.md            # High-level implementation steps
```

These artifacts feed directly into the plan skill.

---

## The PDD Process

```
┌─────────────────────────────────────────────────────────────────┐
│                    PDD WORKFLOW                                  │
│                                                                  │
│  1. CAPTURE rough idea                                          │
│         ↓                                                        │
│  2. CLARIFY requirements (iterative Q&A)                        │
│         ↓                                                        │
│  3. RESEARCH technologies and approaches                        │
│         ↓                                                        │
│  4. DESIGN architecture and components                          │
│         ↓                                                        │
│  5. PLAN high-level implementation                              │
│         ↓                                                        │
│  6. HANDOFF to plan skill                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Step 1: Initialize Project Structure

Use the init-spec script to create the specification directory:

```bash
./scripts/init-spec "build a rate limiter"
# Creates: specs/rate-limiter/ with initial files
```

Or create manually:

```bash
# Derive task-name as kebab-case from the idea
task_name="rate-limiter"  # from "build a rate limiter"

mkdir -p specs/$task_name/research
echo "[rough idea content]" > specs/$task_name/rough-idea.md
touch specs/$task_name/requirements.md
```

---

## Step 2: Requirements Clarification

Guide the user through iterative Q&A to refine the idea into a thorough specification.

**Process Rules:**
- Ask ONE question at a time
- Append each question to `requirements.md` before asking
- Wait for complete response
- Append answer to `requirements.md`
- Continue until user confirms requirements are complete

**Question Categories:**

1. **Core Functionality**
   - What is the primary purpose?
   - Who are the users?
   - What problem does this solve?

2. **User Experience**
   - What does success look like for the user?
   - What are the main user flows?
   - What feedback/error states are needed?

3. **Technical Constraints**
   - Performance requirements?
   - Scale expectations?
   - Integration points?
   - Platform/environment constraints?

4. **Edge Cases**
   - What could go wrong?
   - What are the boundary conditions?
   - What invalid inputs should be handled?

5. **Success Criteria**
   - How do we know when this is done?
   - What metrics define success?
   - What acceptance tests will verify completion?

See [Requirements Clarification](references/requirements-clarification.md) for detailed guidance.

---

## Step 3: Research

Conduct research on technologies, libraries, or existing code to inform the design.

**Research Focus Areas:**

1. **Technology Selection**
   - What libraries/frameworks exist for this?
   - What are the tradeoffs between options?
   - What does the current codebase use?

2. **Existing Patterns**
   - How have similar features been implemented?
   - What patterns exist in the codebase?
   - What can be reused?

3. **External Dependencies**
   - What APIs or services are needed?
   - What are their capabilities and limits?
   - What authentication/authorization is required?

**Documentation:**
- Create separate files in `research/` for each topic
- Include links and references
- Summarize key findings
- Note any limitations or concerns

See [Research Procedures](references/research-procedures.md) for detailed guidance.

---

## Step 4: Design

Create `design.md` as a standalone specification document.

**Required Sections:**

```markdown
# {Feature Name} Design

## Overview
[1-2 paragraph summary of what this is and why]

## Detailed Requirements
[Consolidated from requirements.md]

## Architecture Overview
[High-level system design with mermaid diagram]

## Components and Interfaces
[Each component with its interface/contract]

## Data Models
[Schemas, data structures, relationships]

## Error Handling
[Error scenarios and how they're handled]

## Acceptance Criteria
[Given-When-Then format for verification]

## Testing Strategy
[How this will be tested]

## Appendices
### Technology Choices
[Rationale for technology decisions]

### Research Findings
[Summary of research]

### Alternative Approaches
[Options considered and why they weren't chosen]
```

**Design Constraints:**
- Must be understandable without reading other files
- Must consolidate all requirements from requirements.md
- Must include architecture diagrams (mermaid)
- Must have acceptance criteria in Given-When-Then format

See [Design Templates](references/design-templates.md) for examples.

---

## Step 5: Implementation Plan

Create `plan.md` with incremental implementation steps.

**Plan Principles:**
- Each step builds on previous steps
- Each step results in working, demoable functionality
- Core end-to-end functionality available early
- Every step ends with integration (no orphaned code)

**Plan Format:**

```markdown
# Implementation Plan

## Progress
- [ ] Step 1: [Name]
- [ ] Step 2: [Name]
- [ ] Step 3: [Name]
...

## Step 1: [Name]

**Objective:** [What this step accomplishes]

**Implementation:**
- [Key implementation points]

**Tests Required:**
- [Test cases that must pass]

**Integration:**
- [How this integrates with existing system]

**Demo:**
- [What can be demonstrated after this step]

---

## Step 2: [Name]
...
```

---

## Step 6: Handoff to Plan Skill

When design.md and plan.md are complete and approved by the user:

**Offer the handoff:**

> "The design is complete. Would you like me to create tickets from this design for implementation?"

**If yes:**
1. Reference the plan skill
2. The design.md becomes the source of truth
3. Plan skill will decompose the plan.md into hierarchical tickets
4. Documentation tickets will reference design.md rather than duplicating it

**Transition summary:**

```
Design Output                    →    Plan Input
─────────────────────────────────────────────────────────
specs/{task-name}/design.md   →    Source of truth for requirements
specs/{task-name}/plan.md     →    Decomposed into tickets
                              →    .tickets/epic-{name}.md
                              →    .tickets/story-{name}.md
                              →    .tickets/task-{name}.md
```

---

## Important Rules

These rules apply across ALL steps:

1. **User-driven flow:** Never proceed to the next step without explicit user confirmation. At each transition, ask the user what they want to do next.

2. **Iterative:** The user can move between requirements clarification and research at any time. Always offer this option at phase transitions.

3. **Record as you go:** Append questions, answers, and findings to project files in real time — don't batch-write at the end.

4. **Mermaid diagrams:** Include diagrams for architectures, data flows, and component relationships.

5. **Sources:** Cite references and links in research documents when based on external materials.

---

## Troubleshooting

**Requirements stall:** Suggest switching to a different aspect, provide examples, or pivot to research to unblock decisions.

**Research limitations:** Document what's missing, suggest alternatives with available information, ask user for additional context. Don't block progress.

**Design complexity:** Break into smaller components, focus on core functionality first, suggest phased implementation, return to requirements to re-prioritize.

**User wants to skip ahead:** Remind them that unclear requirements lead to rework. Offer to do a "fast pass" that covers the critical questions only.

---

## File References

- [Requirements Clarification](references/requirements-clarification.md) - Detailed Q&A guidance
- [Research Procedures](references/research-procedures.md) - How to conduct and document research
- [Design Templates](references/design-templates.md) - Examples and patterns for design.md
