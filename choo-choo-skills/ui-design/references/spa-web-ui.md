# Single-Page Application (SPA) Web UI Best Practices

Comprehensive guide for designing Single-Page Web Applications that deliver app-like experiences. Focuses on interaction patterns, state management, navigation, and accessibility rather than framework-specific implementation.

---

## Core Principles

| Principle | Description |
|-----------|-------------|
| **Perceived Performance** | Feels instant through skeleton screens, optimistic updates, and smooth transitions |
| **Fluidity** | Seamless navigation without full page reloads |
| **State Consistency** | Client and server state remain synchronized |
| **Accessibility** | Dynamic content accessible to all users including assistive technologies |
| **Progressive Enhancement** | Core functionality works even under suboptimal conditions |

---

## Navigation and Routing

### Client-Side Routing Patterns

| Pattern | Description | Use When |
|---------|-------------|----------|
| **Hash-based** | `#/route` URLs, no server config needed | Simple apps, static hosting |
| **History API** | Clean URLs, requires server support | Production apps, SEO important |
| **Hybrid** | History with hash fallback | Maximum compatibility |

**Routing Best Practices:**
- Support deep linking (every state has a URL)
- Implement proper browser history (back/forward works)
- Lazy load route components for performance
- Handle 404s gracefully with fallback routes

### Navigation Patterns

| Pattern | Example | Use Case |
|---------|---------|----------|
| **Top navigation** | Tabs or horizontal menu | 3-7 primary sections |
| **Side navigation** | Collapsible sidebar | Complex apps with many sections |
| **Breadcrumbs** | `Home > Projects > Settings` | Deep hierarchies |
| **Tab navigation** | In-page content switching | Related content panes |

### Transition Patterns

| Transition | When to Use |
|------------|-------------|
| **Instant swap** | Same context, minimal cognitive load |
| **Fade** | Context change, moderate difference |
| **Slide** | Sequential flow, wizard patterns |
| **Skeleton** | Content loading, maintain layout |

**Guidelines:**
- Keep transitions under 300ms for responsiveness
- Avoid motion for users with `prefers-reduced-motion`
- Maintain scroll position or reset intentionally
- Announce page changes to screen readers

---

## State Management

### State Categories

| Type | Scope | Examples |
|------|-------|----------|
| **URL state** | Global, shareable | Current page, filters, search query |
| **Global state** | App-wide | User session, theme, notifications |
| **Server state** | Synced with backend | User data, lists, records |
| **Local state** | Component-only | Form inputs, UI toggles, hover state |

### Server State Strategies

| Strategy | When to Use | Tools |
|----------|-------------|-------|
| **Optimistic updates** | High confidence, easy rollback | Likes, toggles, simple edits |
| **Pessimistic updates** | Critical data, complex validation | Payments, deletions |
| **Background sync** | Real-time collaboration | Shared documents, chat |
| **Cache-first** | Read-heavy, infrequent changes | Reference data, settings |

**Cache Invalidation Triggers:**
- Mutation response (server confirms change)
- Manual refresh (user action)
- Time-based (stale-after timeout)
- Event-based (WebSocket push)

### Loading States

| State | Pattern | Use Case |
|-------|---------|----------|
| **Initial load** | Skeleton screen | First content paint |
| **Refetch** | Subtle spinner in header | Background refresh |
| **Mutation** | Inline spinner on action button | Save, submit |
| **Lazy load** | Placeholder with fade-in | Below-fold content |

---

## User Feedback

### Feedback Timing

| Timing | Pattern | Example |
|--------|---------|---------|
| **Immediate** | Optimistic UI, local state | Toggle animates instantly |
| **Near-instant** | Loading indicator | Button shows spinner |
| **Delayed** | Progress bar | File upload, import |
| **Background** | Toast notification | "Changes saved" |

### Toast / Notification Patterns

| Type | Duration | Action |
|------|----------|--------|
| **Success** | 3-5 seconds, auto-dismiss | Optional undo |
| **Info** | 5 seconds, auto-dismiss | Optional action link |
| **Warning** | Persistent or 7 seconds | Required acknowledgment |
| **Error** | Persistent | Required action or dismiss |

**Position:** Bottom-right (desktop), top (mobile) to avoid content overlap

### Progress Indicators

| Type | Use Case |
|------|----------|
| **Spinner** | Unknown duration, quick operations |
| **Progress bar** | Known duration, file operations |
| **Step indicator** | Multi-step wizards |
| **Skeleton screen** | Content loading, maintains layout |

---

## Error Handling

### Error Message Structure

```
Context: What were you trying to do?
Problem: What went wrong?
Solution: How can you fix it?
```

**Example:**
```
Unable to save changes

Your session expired while editing. 
[Sign in again] to continue, or [copy your changes] to save them locally.
```

### Error Patterns

| Error Type | Pattern | Recovery |
|------------|---------|----------|
| **Network** | "Unable to connect" with retry | Retry button |
| **Validation** | Inline field errors | Show near field, focus on error |
| **Authorization** | "You don't have permission" | Explain why, offer alternative |
| **Not found** | Helpful 404 with navigation | Links to relevant areas |
| **Server** | "Something went wrong" | Report option, retry |

### Form Validation

| Timing | Pattern |
|--------|---------|
| **On blur** | Validate when user leaves field |
| **On submit** | Validate all before submission |
| **On change** | Real-time for specific fields (password strength) |

**Error Display:**
- Show error message below or beside field
- Use color + icon + text (not color alone)
- Focus first error field on submit
- Clear errors as user corrects them

---

## Accessibility (a11y)

### Dynamic Content

| Challenge | Solution |
|-----------|----------|
| Content changes | Use ARIA live regions (`aria-live="polite"`) |
| Page navigation | Announce new page title |
| Modal opens | Trap focus, announce modal |
| Loading states | Announce "Loading" and "Loaded" |

### Focus Management

| Scenario | Focus Behavior |
|----------|----------------|
| Page load | Focus on main content or first heading |
| Route change | Focus on page title or main |
| Modal opens | Focus on modal title or first interactive |
| Modal closes | Return focus to trigger element |
| Toast appears | Do not move focus (announce only) |
| Error appears | Focus on error message or field |

### Semantic Structure

```html
<!-- Landmark regions -->
<header role="banner">
<nav role="navigation" aria-label="Main">
<main role="main">
<aside role="complementary">
<footer role="contentinfo">

<!-- Page structure -->
<h1>Page Title</h1>
<section aria-labelledby="section-heading">
  <h2 id="section-heading">Section</h2>
</section>
```

### Keyboard Navigation

| Key | Action |
|-----|--------|
| Tab | Navigate forward through interactive elements |
| Shift+Tab | Navigate backward |
| Enter/Space | Activate buttons, links |
| Escape | Close modals, cancel actions |
| Arrow keys | Navigate lists, menus, grids |

**Guidelines:**
- All interactive elements focusable via keyboard
- Visible focus indicators (not just color change)
- Skip links for repetitive navigation
- Logical tab order matching visual order

### WCAG Compliance Checklist

- [ ] Color contrast ratio ≥ 4.5:1 (text), ≥ 3:1 (UI components)
- [ ] Text resizable to 200% without loss of functionality
- [ ] All images have alt text (decorative use `alt=""`)
- [ ] Forms have associated labels
- [ ] Error messages linked to fields via `aria-describedby`
- [ ] Motion respects `prefers-reduced-motion`
- [ ] No content relies solely on color

---

## Interactive Elements

### Modals and Dialogs

| Element | Requirement |
|---------|-------------|
| **Trigger** | Opens modal, receives focus on close |
| **Overlay** | Click to dismiss (optional), blocks background |
| **Title** | Clear, concise, focused on first open |
| **Content** | Modal content, scrollable if needed |
| **Actions** | Primary action right-aligned, escape to cancel |
| **Close** | X button (optional), Escape key, overlay click |

**Accessibility:**
- `role="dialog"` and `aria-modal="true"`
- `aria-labelledby` pointing to title
- Focus trapped within modal
- Escape key closes modal

### Forms

| Element | Best Practice |
|---------|---------------|
| **Labels** | Always visible, above or beside field |
| **Placeholders** | Hints only, never replace labels |
| **Required** | Mark with asterisk + `(required)` text |
| **Help text** | Below field, linked via `aria-describedby` |
| **Errors** | Inline, clear, actionable |

### Buttons and Actions

| Type | Style | Use |
|------|-------|-----|
| **Primary** | Filled, prominent | Main action (Save, Submit) |
| **Secondary** | Outlined or muted | Alternative actions (Cancel) |
| **Destructive** | Red/danger styling | Irreversible actions (Delete) |
| **Ghost/Text** | No border | Tertiary actions (Learn more) |

**Guidelines:**
- One primary action per context
- Destructive actions require confirmation
- Disabled state shows why when possible
- Loading state shows spinner, disables click

### Lists and Tables

| Feature | Implementation |
|---------|----------------|
| **Sorting** | Click column header, show indicator |
| **Filtering** | Search input, filter chips |
| **Pagination** | Show current/total, prev/next |
| **Selection** | Checkbox column, select all option |
| **Actions** | Row actions menu or inline buttons |

---

## Responsive Design

### Breakpoints

| Name | Min Width | Target |
|------|-----------|--------|
| Mobile | 0px | Phones |
| Tablet | 768px | Tablets, small laptops |
| Desktop | 1024px | Laptops, desktops |
| Wide | 1440px | Large monitors |

### Layout Patterns

| Pattern | Mobile | Desktop |
|---------|--------|---------|
| **Navigation** | Hamburger menu, bottom bar | Top nav, sidebar |
| **Forms** | Single column | Multi-column possible |
| **Tables** | Card list or horizontal scroll | Full table |
| **Modals** | Full-screen or bottom sheet | Centered, max-width |
| **Sidebars** | Overlay or off-canvas | Persistent or collapsible |

### Touch Considerations

| Element | Minimum Size |
|---------|--------------|
| Tap target | 44×44px (iOS), 48×48px (Android) |
| Tap spacing | 8px between targets |
| Swipe areas | Full-width for carousels |

---

## Performance Patterns

### Loading Strategies

| Strategy | When to Use |
|----------|-------------|
| **Eager load** | Above-fold, critical path |
| **Lazy load** | Below-fold, heavy components |
| **Preload** | Next likely navigation |
| **Prefetch** | Likely future pages (hover, idle) |

### Code Splitting

```
Initial bundle:
- App shell (navigation, layout)
- Current route
- Critical shared components

On navigation:
- Load new route chunk
- Load route-specific dependencies
```

### Data Fetching

| Approach | Use Case |
|----------|----------|
| **Parallel fetch** | Independent data needs |
| **Waterfall** | Dependent data (user → profile) |
| **Prefetch** | On hover, anticipation |
| **Background** | Stale-while-revalidate |

---

## Real-World Examples

### Gmail (SPA)

**Strengths:**
- Optimistic updates (send, archive)
- Keyboard shortcuts for power users
- Offline support with service worker
- Progressive disclosure of features

**Patterns:**
- Left sidebar for navigation
- List-detail split view
- Toast notifications for actions
- Undo for destructive operations

### Notion (SPA)

**Strengths:**
- Real-time collaboration
- Smooth block-based editing
- slash commands for quick actions
- Clean, minimal interface

**Patterns:**
- Sidebar with page tree
- Inline editing without modes
- Hover actions on blocks
- Breadcrumbs for navigation

### Linear (SPA)

**Strengths:**
- Keyboard-first navigation
- Command palette (Cmd+K)
- Optimistic everywhere
- Instant transitions

**Patterns:**
- Sidebar navigation
- List view with quick actions
- Modal for details
- Toast with undo

---

## Design Checklist

Use this checklist when designing SPA interfaces:

### Navigation
- [ ] Deep linking supported (URL reflects state)
- [ ] Back/forward browser buttons work
- [ ] Page transitions are smooth
- [ ] Navigation state visible (active item)
- [ ] Breadcrumbs for deep hierarchies

### State Management
- [ ] Loading states for all async operations
- [ ] Optimistic updates where appropriate
- [ ] Error boundaries for failures
- [ ] Stale data handling defined
- [ ] Offline behavior considered

### User Feedback
- [ ] Immediate feedback on interactions
- [ ] Progress indicators for long operations
- [ ] Toast notifications for completed actions
- [ ] Undo option for destructive actions
- [ ] Empty states are helpful

### Error Handling
- [ ] Network errors handled gracefully
- [ ] Form validation is inline and clear
- [ ] Error messages are actionable
- [ ] Retry mechanisms available
- [ ] Fallback UI for critical failures

### Accessibility
- [ ] Screen reader announces changes
- [ ] Focus management on navigation
- [ ] Keyboard navigation complete
- [ ] Color contrast meets WCAG
- [ ] Motion respects user preference

### Responsive
- [ ] Touch targets appropriately sized
- [ ] Mobile navigation accessible
- [ ] Tables adapt to small screens
- [ ] Forms single column on mobile
- [ ] Modals mobile-friendly

---

## References

- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)
- [React Documentation - Accessibility](https://react.dev/learn/accessibility)
- [Vue.js - Accessibility](https://vuejs.org/guide/best-practices/accessibility.html)
- [web.dev - Performance](https://web.dev/performance/)
- [Refactoring UI](https://www.refactoringui.com/)
