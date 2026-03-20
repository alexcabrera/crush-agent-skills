---
name: ui-design
description: "Designs user interfaces for applications, focusing on interaction patterns, usability, and accessibility. Use when designing CLI, TUI, or web applications. For web projects, defaults to hypermedia architecture (HTMX, server-side rendering) over SPAs. Invoked during design phase or standalone. Outputs UI specifications including interaction patterns, visual hierarchy, and error handling."
license: MIT
compatibility: Requires bash.
metadata:
  version: "1.2.0"
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
| Building web applications | This skill → **start with web architecture decision** |
| Defining interaction patterns | This skill |
| Planning error messages and feedback | This skill |
| Ensuring accessibility compliance | This skill |
| Architecture-focused design | [design](../design/) skill |

### Invocation Triggers

Activate this skill when:
- User mentions "interface design", "UX", "user experience"
- Designing CLI commands, flags, or arguments
- Creating interactive terminal applications
- Building any web application
- Choosing between web architecture approaches
- Defining error messages and user feedback
- Planning keyboard shortcuts or navigation
- Considering accessibility requirements

---

## Web Architecture Decision (REQUIRED for Web Projects)

**Before designing any web interface, you MUST determine the appropriate architecture.**

Read: [Web Architecture Decision Framework](references/web-architecture-decision.md)

### Quick Decision

```
┌─────────────────────────────────────────────────────────────────┐
│                    DEFAULT: HYPERMEDIA                           │
│                                                                  │
│  Use hypermedia (HTMX, server-side rendering) unless project    │
│  SPECIFICALLY requires:                                         │
│  • Real-time collaboration (Google Docs style)                  │
│  • Complex client-side visualizations (3D, canvas)              │
│  • Offline-first functionality                                  │
│  • Heavy drag-and-drop (design tools)                           │
│                                                                  │
│  When in doubt, choose hypermedia.                              │
└─────────────────────────────────────────────────────────────────┘
```

### Architecture Options

| Architecture | Description | Reference |
|--------------|-------------|-----------|
| **Hypermedia** | Server-rendered HTML with HTMX, progressive enhancement | [hypermedia-web-ui.md](references/hypermedia-web-ui.md) |
| **SPA** | Client-side routing, heavy JavaScript | [spa-web-ui.md](references/spa-web-ui.md) |

**Decision criteria:**
- SEO requirements → Hypermedia
- Accessibility requirements → Hypermedia
- Team expertise (backend) → Hypermedia
- Time to market → Hypermedia
- Real-time collaboration → Consider SPA
- Offline-first → Consider SPA
- 3D/Canvas/WebGL → SPA

Document the architecture decision in design.md with rationale.

---

## Interface Types

| Type | Description | Reference |
|------|-------------|-----------|
| **CLI** | Command-line interfaces with flags, arguments, subcommands | [terminal-ui.md](references/terminal-ui.md) |
| **TUI** | Text-based user interfaces with interactive elements | [terminal-ui.md](references/terminal-ui.md) |
| **Hypermedia Web** | Server-rendered HTML with HTMX (DEFAULT for web) | [hypermedia-web-ui.md](references/hypermedia-web-ui.md) |
| **SPA Web** | Single-page applications with client-side routing | [spa-web-ui.md](references/spa-web-ui.md) |

---

## The UI Design Process

```
┌─────────────────────────────────────────────────────────────────┐
│                    UI DESIGN WORKFLOW                            │
│                                                                  │
│  1. IDENTIFY interface type                                     │
│     (For web: decide hypermedia vs SPA first)                   │
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
- **Web**: Start with [Web Architecture Decision](#web-architecture-decision-required-for-web-projects)

For web projects, this step includes:
1. Review project requirements
2. Complete the architecture questionnaire
3. Calculate decision scorecard
4. Document choice with rationale
5. Load appropriate reference document

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

**For Hypermedia Web (DEFAULT):**
- Server-side rendering with partial updates (HTMX)
- Progressive enhancement (works without JS)
- Forms with server validation
- Navigation patterns (full page vs partial)
- Minimal JavaScript approach

**For SPA Web:**
- Client-side routing and deep linking
- State management (URL, global, server, local)
- Loading states and optimistic updates
- Component patterns (modals, forms, lists)
- Accessibility for dynamic content

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

**For Hypermedia Web:**
- Server returns HTML with inline error messages
- Preserve user input on validation failure
- Use HTMX indicators for loading states

---

## Step 5: Ensure Accessibility

Apply accessibility principles:
- Color contrast (WCAG compliance)
- Screen reader compatibility
- Alternative input methods
- High-contrast/monochrome modes

**For Hypermedia Web:**
- Semantic HTML by default (major advantage)
- Focus management after HTMX swaps
- ARIA live regions for dynamic updates
- Progressive enhancement ensures baseline accessibility

---

## Step 6: Document

If invoked during design skill:
- Add UI sections to `design.md`
- **For web: Document architecture decision with rationale**
- Document interaction patterns in Components section
- Add error scenarios to Error Handling section

If standalone:
- Create `ui-design.md` with:
  - Interface type and rationale
  - **For web: Architecture decision (hypermedia vs SPA) with scorecard**
  - User analysis
  - Interaction patterns
  - Error handling approach
  - Accessibility considerations

---

## Relationship to Other Skills

```
design skill          →  ui-design skill (when UI needed)
                              ↓
                         For web: architecture decision
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

- [Web Architecture Decision](references/web-architecture-decision.md) - **START HERE for web projects**
- [Hypermedia Web UI](references/hypermedia-web-ui.md) - Server-rendered HTML with HTMX (default)
- [SPA Web UI](references/spa-web-ui.md) - Single-page applications
- [Terminal UI](references/terminal-ui.md) - CLI and TUI design patterns
