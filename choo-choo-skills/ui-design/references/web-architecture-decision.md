# Web Architecture Decision Framework

Guide for choosing between hypermedia and single-page application architectures. Use this during the design phase to determine the appropriate web architecture.

---

## TL;DR: Default to Hypermedia

```
┌─────────────────────────────────────────────────────────────────┐
│                    ARCHITECTURE DECISION                         │
│                                                                  │
│                      Start Here                                  │
│                         │                                        │
│                    ┌────▼────┐                                   │
│                    │ HYPERMEDIA│ ◄─── DEFAULT CHOICE             │
│                    └────┬────┘                                   │
│                         │                                        │
│              Does project require:                               │
│              • Real-time collaboration?                          │
│              • Complex client-side visualizations?               │
│              • Offline-first functionality?                      │
│              • Heavy drag-and-drop interactions?                 │
│              • Video/audio editing?                              │
│              • 3D graphics?                                      │
│                         │                                        │
│              ┌──────────┴──────────┐                             │
│              │ NO                  │ YES                         │
│              ▼                     ▼                             │
│        ┌──────────┐          ┌──────────┐                        │
│        │HYPERMEDIA│          │   SPA    │                        │
│        │(proceed) │          │(reconsider│                        │
│        └──────────┘          │hypermedia│                        │
│                              │first)    │                        │
│                              └──────────┘                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Decision Matrix

| Factor | Hypermedia | SPA | Winner |
|--------|------------|-----|--------|
| Initial load speed | Fast (pre-rendered) | Slow (JS bundle) | **Hypermedia** |
| SEO | Built-in | Requires SSR | **Hypermedia** |
| Accessibility | Native HTML | Manual effort | **Hypermedia** |
| Development speed | Faster | Slower | **Hypermedia** |
| Maintenance | Simple | Complex | **Hypermedia** |
| Team requirements | Backend skills | Frontend specialists | **Hypermedia** |
| Bundle size | ~15KB | 50-500KB+ | **Hypermedia** |
| Real-time collab | Limited | Excellent | SPA |
| Offline support | Difficult | Excellent | SPA |
| Complex visualizations | Difficult | Excellent | SPA |
| Drag-and-drop | Basic | Excellent | SPA |
| 3D/Canvas/WebGL | Very limited | Excellent | SPA |

**Count the checkmarks. If hypermedia wins ≥7 factors, use hypermedia.**

---

## Questionnaire

Answer these questions during the design phase:

### 1. Content vs Interaction

**Q: Is this primarily a content site with some interactivity, or primarily an interactive application?**

- Content-heavy (docs, blog, e-commerce catalog) → **Hypermedia**
- Balanced (admin panel, SaaS dashboard) → **Hypermedia**
- Interaction-heavy (design tool, game, editor) → Consider SPA

### 2. Team Composition

**Q: What skills does your team have?**

- Backend-focused (Python, Ruby, Go, PHP) → **Hypermedia**
- Mixed backend/frontend → **Hypermedia**
- Frontend specialist team → Could go either way
- Solo developer → **Hypermedia** (lower complexity)

### 3. SEO Requirements

**Q: How important is search engine visibility?**

- Critical (public marketing site, e-commerce) → **Hypermedia**
- Important (documentation, blog) → **Hypermedia**
- Nice to have → Either
- Not needed (authenticated app, internal tool) → Either

### 4. Offline Requirements

**Q: Must the app work without internet?**

- Yes, critical → Consider SPA (with caveats)
- Yes, nice to have → Either (use Service Workers)
- No → **Hypermedia**

### 5. Real-Time Requirements

**Q: Does the app need real-time collaboration like Google Docs?**

- Yes, simultaneous editing → Consider SPA
- Yes, but just notifications/updates → **Hypermedia** (SSE/WebSockets)
- No → **Hypermedia**

### 6. Visualization Needs

**Q: What kind of data visualization is needed?**

- None or simple charts → **Hypermedia**
- Standard charts (use library) → **Hypermedia**
- Complex interactive visualizations → Consider SPA
- 3D graphics, maps, canvas manipulation → SPA

### 7. Drag-and-Drop

**Q: How important is drag-and-drop functionality?**

- Not needed → **Hypermedia**
- Simple reordering → **Hypermedia** (can use JS library)
- Complex drag-and-drop (design tools) → Consider SPA

### 8. Time to Market

**Q: How quickly do you need to ship?**

- ASAP → **Hypermedia** (faster development)
- Normal timeline → Either
- No rush → Either

### 9. Long-Term Maintenance

**Q: Who will maintain this in 5 years?**

- Uncertain/rotating team → **Hypermedia** (simpler, more stable)
- Same team → Either
- Dedicated frontend team → Either

### 10. Accessibility Requirements

**Q: What accessibility compliance is needed?**

- WCAG AA or higher → **Hypermedia** (easier compliance)
- Basic accessibility → Either
- No specific requirements → Either

---

## Decision Scorecard

Calculate your score:

| Question | Hypermedia Point | SPA Point |
|----------|------------------|-----------|
| Content vs Interaction | Content/Balanced = 1 | Interaction-heavy = 1 |
| Team Composition | Backend/Mixed/Solo = 1 | Frontend specialists = 1 |
| SEO Requirements | Critical/Important = 1 | Not needed = 1 |
| Offline Requirements | No = 1 | Critical = 1 |
| Real-Time Requirements | No/Notifications = 1 | Collaboration = 1 |
| Visualization Needs | None/Standard = 1 | Complex/3D = 1 |
| Drag-and-Drop | Not needed/Simple = 1 | Complex = 1 |
| Time to Market | ASAP = 1 | No rush = 1 |
| Long-Term Maintenance | Uncertain team = 1 | Dedicated team = 1 |
| Accessibility | WCAG required = 1 | No requirements = 1 |

**Score interpretation:**
- **Hypermedia ≥ 7**: Use hypermedia
- **Hypermedia 5-6**: Strong case for hypermedia; reconsider SPA needs
- **Hypermedia 3-4**: Either could work; hypermedia still preferred for simplicity
- **Hypermedia ≤ 2**: SPA may be appropriate (but validate each factor)

---

## Common Scenarios

### E-Commerce Site
- Content-heavy, SEO critical, needs fast loads
- **→ Hypermedia** (with cart JavaScript)

### Admin Dashboard
- CRUD operations, tables, forms
- **→ Hypermedia**

### Documentation Site
- Content-first, SEO important, accessibility matters
- **→ Hypermedia**

### Real-Time Chat
- Continuous updates, immediate feedback
- **→ Hypermedia** (with SSE or WebSockets)

### Project Management (Trello-like)
- Boards, cards, drag-and-drop
- **→ Hypermedia** (drag-and-drop via JS library, not SPA)

### Collaborative Document Editor
- Real-time simultaneous editing
- **→ SPA** (for the editor only; rest can be hypermedia)

### Data Visualization Tool
- Charts, graphs, interactive dashboards
- **→ Hypermedia** (use charting library, not SPA)

### Design Tool (Figma-like)
- Canvas manipulation, complex interactions
- **→ SPA**

### Video Editor
- Timeline, effects, preview
- **→ SPA**

### Internal CRM
- Forms, tables, workflows
- **→ Hypermedia**

---

## Hybrid Architecture

When you need SPA-like features in specific areas:

```
┌─────────────────────────────────────────────────────────────────┐
│                    HYBRID APPROACH                               │
│                                                                  │
│  Most of app: Hypermedia                                         │
│  ├── Navigation (full page or HTMX)                             │
│  ├── Forms and CRUD (server validation)                         │
│  ├── Lists and tables (server rendering)                        │
│  └── Search and filters (HTMX)                                  │
│                                                                  │
│  Specific components: JavaScript widgets                         │
│  ├── Rich text editor (Web Component)                           │
│  ├── Chart dashboard (embed visualization library)              │
│  └── Real-time notifications (SSE)                              │
│                                                                  │
│  The rule: Hypermedia by default, JS only when necessary        │
└─────────────────────────────────────────────────────────────────┘
```

### Embedding JS Widgets in Hypermedia

```html
<!-- Main app is hypermedia -->
<main id="content">
  <!-- Complex widget as Web Component -->
  <rich-text-editor name="content" value="${initialContent}">
  </rich-text-editor>
  
  <!-- Everything else is server-rendered with HTMX -->
  <button hx-post="/save" hx-include="closest form">
    Save
  </button>
</main>
```

---

## Anti-Patterns to Avoid

### 1. Choosing SPA Because "It's Modern"

Reality: Hypermedia is experiencing a renaissance. SPAs added complexity without always delivering value.

### 2. Choosing SPA Because "We Might Need It Later"

Reality: YAGNI. Start with hypermedia. SPA migration is possible if truly needed.

### 3. Choosing SPA Because "That's What I Know"

Reality: Learning HTMX takes hours. SPA expertise took months. Consider the learning curve for future maintainers.

### 4. Choosing SPA for "Better UX"

Reality: Hypermedia can deliver excellent UX. Perceived performance is often better with server rendering.

### 5. Choosing Hypermedia for Everything

Reality: Some apps genuinely need SPA. Be honest about requirements.

---

## Making the Decision

During the design phase:

1. **Answer the questionnaire** with stakeholders
2. **Calculate the scorecard** 
3. **Document the decision** in design.md
4. **If hypermedia**: Proceed with hypermedia reference
5. **If SPA**: Document specific requirements that necessitate SPA
6. **If hybrid**: Identify which components need JS widgets

### Document Template

```markdown
## Web Architecture Decision

**Decision:** [Hypermedia / SPA / Hybrid]

**Scorecard:** Hypermedia X/10, SPA Y/10

**Key Factors:**
- [Factor that influenced decision]
- [Another factor]

**SPA Requirements (if applicable):**
- [Specific feature requiring SPA-like behavior]
- [Another specific feature]

**Rationale:**
[2-3 sentences explaining the decision]
```

---

## Quick Reference Card

```
┌─────────────────────────────────────────────────────────────────┐
│            WHEN TO USE HYPERMEDIA (DEFAULT)                      │
├─────────────────────────────────────────────────────────────────┤
│ ✓ Content-heavy sites                                           │
│ ✓ CRUD applications                                             │
│ ✓ SEO matters                                                   │
│ ✓ Accessibility required                                        │
│ ✓ Fast initial load needed                                      │
│ ✓ Backend team                                                  │
│ ✓ Long-term maintainability                                     │
│ ✓ Simple to moderate interactivity                              │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│              WHEN SPA MIGHT BE APPROPRIATE                       │
├─────────────────────────────────────────────────────────────────┤
│ • Real-time collaboration (Google Docs style)                   │
│ • Complex client-side visualizations                            │
│ • Offline-first requirements                                    │
│ • Heavy drag-and-drop (design tools)                            │
│ • Video/audio editing                                           │
│ • 3D graphics / Canvas / WebGL                                  │
└─────────────────────────────────────────────────────────────────┘

Remember: Even for SPA-appropriate features, consider embedding
JavaScript widgets within a hypermedia application first.
```
