---
name: ui-design
description: Designs user interfaces for applications, focusing on interaction patterns, usability, and accessibility. Use when designing CLI, TUI, or other interface types. Invoked during design phase or standalone. Outputs UI specifications including interaction patterns, visual hierarchy, and error handling.
license: MIT
compatibility: Requires bash.
metadata:
  version: "1.0.0"
  author: choo-choo-skills
---

# UI Design Skill

Design user interfaces with a focus on interaction patterns, usability, and accessibility. This skill specializes in creating UI specifications that guide implementation, complementing the architecture-focused design skill.

Part of the [choo-choo-skills](../) collection.

## When to Use This Skill

| Scenario | Use |
|----------|-----|
| Designing command-line interfaces | This skill → terminal reference |
| Creating text-based user interfaces (TUI) | This skill → terminal reference |
| Defining interaction patterns | This skill |
| Planning error messages and feedback | This skill |
| Ensuring accessibility compliance | This skill |
| Architecture-focused design | [design](../design/) skill |

### Invocation Triggers

Activate this skill when:
- User mentions "interface design", "UX", "user experience"
- Designing CLI commands, flags, or arguments
- Creating interactive terminal applications
- Defining error messages and user feedback
- Planning keyboard shortcuts or navigation
- Considering accessibility requirements

---

## Interface Types

| Type | Description | Reference |
|------|-------------|-----------|
| **CLI** | Command-line interfaces with flags, arguments, subcommands | [terminal-ui.md](references/terminal-ui.md) |
| **TUI** | Text-based user interfaces with interactive elements | [terminal-ui.md](references/terminal-ui.md) |

Additional interface types can be added as references as needed.

---

## The UI Design Process

```
┌─────────────────────────────────────────────────────────────────┐
│                    UI DESIGN WORKFLOW                            │
│                                                                  │
│  1. IDENTIFY interface type (CLI, TUI, etc.)                    │
│         ↓                                                        │
│  2. DEFINE users and their goals                                │
│         ↓                                                        │
│  3. DESIGN interaction patterns                                 │
│         ↓                                                        │
│  4. PLAN feedback and error handling                            │
│         ↓                                                        │
│  5. ENSURE accessibility                                        │
│         ↓                                                        │
│  6. DOCUMENT in design.md or standalone spec                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Step 1: Identify Interface Type

Determine the interface type based on:
- **CLI**: Single commands, scripts, automation, pipes
- **TUI**: Interactive dashboards, multi-pane layouts, real-time updates

Load the appropriate reference document for detailed patterns.

---

## Step 2: Define Users and Goals

**User Analysis:**
- Who are the primary users? (developers, sysadmins, end users)
- What is their expertise level? (beginner, intermediate, expert)
- What are their goals? (complete task, get information, monitor state)

**Goal Articulation:**
- What should users accomplish?
- What are the success criteria?
- What friction points exist in current solutions?

---

## Step 3: Design Interaction Patterns

Based on interface type, apply appropriate patterns from references:

**For CLI:**
- Command structure (verb-noun, noun-verb)
- Flag conventions (short/long forms, positional args)
- Subcommand hierarchies
- Input validation approach

**For TUI:**
- Navigation model (modal, menu-driven, command-driven)
- Keyboard shortcuts
- Layout and visual hierarchy
- Interactive elements (prompts, menus, progress)

---

## Step 4: Plan Feedback and Error Handling

**User Feedback:**
- Progress indicators for long operations
- Success confirmations for destructive actions
- Status visibility for ongoing processes

**Error Handling:**
- Visibility: Display errors prominently
- Clarity: Use plain language, avoid jargon
- Constructive: Suggest specific recovery steps
- Prevention: Validate input early

---

## Step 5: Ensure Accessibility

Apply accessibility principles:
- Color contrast (WCAG compliance)
- Screen reader compatibility
- Alternative input methods
- High-contrast/monochrome modes

---

## Step 6: Document

If invoked during design skill:
- Add UI sections to `design.md`
- Document interaction patterns in Components section
- Add error scenarios to Error Handling section

If standalone:
- Create `ui-design.md` with:
  - Interface type and rationale
  - User analysis
  - Interaction patterns
  - Error handling approach
  - Accessibility considerations

---

## Relationship to Other Skills

```
design skill          →  ui-design skill (when UI needed)
                              ↓
                         UI specifications
                              ↓
plan skill            ←  design.md + ui decisions
```

The ui-design skill is typically invoked:
1. During design skill when UI components arise
2. After design skill for detailed UI work
3. Standalone for UI-only projects

---

## File References

- [Terminal UI Best Practices](references/terminal-ui.md) - CLI and TUI design patterns
