# Research Procedures

This document provides guidance on conducting and documenting research during the PDD process.

## When to Research

Research is appropriate when:

1. **Technology selection is needed** - Multiple options exist
2. **Feasibility is unclear** - Can we actually build this?
3. **Existing patterns exist** - What can we learn from others?
4. **Integration is required** - How do external systems work?
5. **Best practices are needed** - What's the right approach?

## Research Plan

Before diving in, propose a research plan to the user:

```markdown
## Research Plan

I'll investigate:
1. [Topic 1] - Why: [reason]
2. [Topic 2] - Why: [reason]
3. [Topic 3] - Why: [reason]

Are there other topics I should research? Any I can skip?
```

## Research Categories

### 1. Technology Research

**Goal:** Select appropriate technologies and understand their capabilities.

**Topics to investigate:**
- Available libraries/frameworks
- Version compatibility
- Performance characteristics
- Community support and maintenance
- License considerations
- Learning curve

**Documentation format:**
```markdown
# {Technology Name}

## Overview
[What it is and why we're considering it]

## Capabilities
- [Capability 1]
- [Capability 2]

## Limitations
- [Limitation 1]
- [Limitation 2]

## Integration Notes
[How it fits with our stack]

## References
- [Link to documentation]
- [Link to relevant article]

## Verdict
[Recommended or not, with rationale]
```

### 2. Existing Code Research

**Goal:** Understand what exists and what can be reused.

**Topics to investigate:**
- Similar features in the codebase
- Existing patterns and conventions
- Reusable components/utilities
- Technical debt to avoid

**Documentation format:**
```markdown
# Existing Code Analysis: {Area}

## Current State
[What exists now]

## Patterns Found
- [Pattern 1]: [Where it's used]
- [Pattern 2]: [Where it's used]

## Reusable Components
- [Component]: [Location and capability]

## Technical Debt
- [Issue]: [Impact on new feature]

## Recommendations
[What to follow, what to avoid]
```

### 3. API/Integration Research

**Goal:** Understand external systems we need to work with.

**Topics to investigate:**
- Available endpoints/methods
- Authentication requirements
- Rate limits
- Data formats
- Error responses
- Versioning/deprecation

**Documentation format:**
```markdown
# Integration: {Service Name}

## Overview
[What this service provides]

## Authentication
[How to authenticate]

## Endpoints

### {Endpoint Name}
- **Method:** [GET/POST/etc]
- **URL:** [pattern]
- **Parameters:** [list]
- **Response:** [format]
- **Errors:** [possible errors]

## Rate Limits
[Limits and how to handle them]

## References
- [API documentation link]

## Integration Notes
[Special considerations]
```

### 4. Best Practices Research

**Goal:** Understand established approaches for the problem domain.

**Topics to investigate:**
- Industry standards
- Common patterns
- Known pitfalls
- Security considerations

**Documentation format:**
```markdown
# Best Practices: {Domain}

## Standard Approaches
1. [Approach 1]: [Description]
2. [Approach 2]: [Description]

## Security Considerations
- [Consideration 1]
- [Consideration 2]

## Common Pitfalls
- [Pitfall 1]: [How to avoid]
- [Pitfall 2]: [How to avoid]

## References
- [Source 1]
- [Source 2]

## Recommended Approach
[What we should do and why]
```

## Research Process

```
┌─────────────────────────────────────────────────────────────────┐
│                    RESEARCH CYCLE                                │
│                                                                  │
│  1. IDENTIFY research topics                                    │
│         ↓                                                        │
│  2. PROPOSE plan to user                                        │
│         ↓                                                        │
│  3. INVESTIGATE each topic                                      │
│         ↓                                                        │
│  4. DOCUMENT findings in research/                              │
│         ↓                                                        │
│  5. CHECK IN with user (share findings, confirm direction)      │
│         ↓                                                        │
│  6. ITERATE if more research needed                             │
│         ↓                                                        │
│  7. SUMMARIZE key findings                                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Check-in Frequency

Check in with the user:

- **After each major topic** - Share what you found, confirm it's useful
- **When you hit a blocker** - Can't find information, need clarification
- **When findings change direction** - Research reveals new requirements
- **Before concluding** - Summarize findings, get approval to proceed

## Check-in Format

```markdown
## Research Update: {Topic}

### What I Found
- [Finding 1]
- [Finding 2]

### Implications
[What this means for the design]

### Questions for You
- [Question 1]
- [Question 2]

### Next Steps
[What I plan to research next]
```

## Handling Uncertainty

When research is inconclusive:

1. **Document what you know** - Facts, capabilities, limitations
2. **Document what you don't know** - Uncertainties, gaps
3. **Propose options** - Multiple paths forward with tradeoffs
4. **Recommend a path** - Your best judgment with rationale
5. **Note the risk** - What could go wrong if you're wrong

```markdown
## Uncertainty: {Topic}

### What We Know
- [Known fact 1]
- [Known fact 2]

### What We Don't Know
- [Uncertainty 1]
- [Uncertainty 2]

### Options
1. **{Option 1}**: Pros/Cons
2. **{Option 2}**: Pros/Cons

### Recommendation
[Recommended option and why]

### Risk
[What happens if we're wrong]
```

## Source Quality

Prioritize sources:
1. Official documentation
2. Source code (for libraries)
3. Reputable technical blogs
4. Stack Overflow / community discussions
5. General web search

Always cite sources:
```markdown
## References
- [Official docs](url) - Primary source
- [Blog post](url) - Explains pattern
- [SO answer](url) - Community consensus
```

## Anti-Patterns

1. **Endless research** - Set time limits, make decisions with available info
2. **Single source** - Verify important claims across multiple sources
3. **No documentation** - Future you (and others) need to know why decisions were made
4. **Research without synthesis** - Findings must translate to design decisions
5. **Ignoring contradictions** - Reconcile conflicting information explicitly
