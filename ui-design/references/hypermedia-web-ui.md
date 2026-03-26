# Hypermedia Web UI Best Practices

Comprehensive guide for designing modern hypermedia-based web applications using server-side HTML rendering with progressive enhancement. Focuses on REST/HATEOAS principles, partial updates, and minimal JavaScript.

---

## Core Principles

| Principle | Description |
|-----------|-------------|
| **Server-Driven UI** | Server controls state and renders HTML; client displays it |
| **Progressive Enhancement** | Core functionality works without JavaScript |
| **Hypermedia Controls** | Links and forms drive application state transitions |
| **Minimal JavaScript** | Use only where server-side solutions don't suffice |
| **RESTful Architecture** | Embrace HATEOAS - hypermedia as engine of application state |

### Why Hypermedia Over SPA

| Factor | Hypermedia | SPA |
|--------|------------|-----|
| Initial load | Fast (pre-rendered HTML) | Slow (JS bundle + hydration) |
| SEO | Built-in | Requires extra work (SSR/prerender) |
| Accessibility | Native HTML semantics | Manual implementation |
| Complexity | Low (server-focused) | High (client state management) |
| Maintenance | Simpler mental model | Framework churn, build tooling |
| Team scaling | Backend devs can build UI | Requires frontend specialists |

**Default to hypermedia. Only choose SPA when you have specific, proven needs.**

---

## Architecture

### Hypermedia-Driven Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    HYPERMEDIA ARCHITECTURE                       │
│                                                                  │
│  Browser                                                         │
│  ───────                                                         │
│  1. Request HTML page                                            │
│  2. Server renders full HTML                                     │
│  3. Browser displays immediately                                 │
│  4. User interaction triggers request                            │
│  5. Server renders HTML fragment                                 │
│  6. Browser swaps fragment into DOM                              │
│                                                                  │
│  State lives on server. HTML IS the API.                         │
└─────────────────────────────────────────────────────────────────┘
```

### HTML-Over-The-Wire

Instead of JSON APIs + client-side rendering:

```
Traditional SPA:
Client → JSON API → Client renders HTML

Hypermedia:
Client → HTML API → Client displays HTML directly
```

The server returns HTML fragments that the client inserts directly into the DOM.

---

## Technologies

### HTMX

Declarative AJAX via HTML attributes. The primary tool for hypermedia systems.

| Attribute | Purpose |
|-----------|---------|
| `hx-get` | Issue GET request, swap response into DOM |
| `hx-post` | Issue POST request, swap response |
| `hx-trigger` | Event that triggers request (click, load, etc.) |
| `hx-target` | Element to swap HTML into |
| `hx-swap` | How to swap (innerHTML, outerHTML, beforeend, etc.) |
| `hx-indicator` | Show loading state during request |
| `hx-push-url` | Update browser URL after request |

**Example:**
```html
<button hx-post="/like" 
        hx-target="#like-count"
        hx-swap="innerHTML">
  Like
</button>
<span id="like-count">0</span>
```

Server returns `<span id="like-count">42</span>` and HTMX swaps it in.

### Web Components

Encapsulated, reusable UI elements that work with any server-side stack.

```html
<modal-dialog id="confirm-delete">
  <p>Are you sure you want to delete this item?</p>
  <button slot="actions" onclick="this.closest('modal-dialog').close(true)">
    Delete
  </button>
</modal-dialog>
```

**Use Web Components for:**
- Complex interactive elements (date pickers, modals, tabs)
- Encapsulated widgets used across pages
- Progressive enhancement layers

### Lightweight JavaScript Libraries

| Library | Size | Use When |
|---------|------|----------|
| **HTMX** | ~14KB | Dynamic content updates without JS code |
| **Alpine.js** | ~15KB | Need lightweight reactivity (toggles, state) |
| **Stimulus** | ~10KB | Incremental behavior on server-rendered HTML |
| **Hyperscript** | ~12KB | Want inline scripting without Alpine's reactivity |

These complement HTMX; they don't replace it.

---

## Navigation Patterns

### Full Page Navigation (Default)

For major context changes, use traditional page loads:

```html
<a href="/dashboard">Dashboard</a>
<a href="/settings">Settings</a>
```

This is simpler and more reliable than client-side routing.

### Partial Updates (HTMX)

For in-page interactions:

```html
<nav>
  <a hx-get="/dashboard" hx-target="#main" hx-push-url="true">Dashboard</a>
  <a hx-get="/settings" hx-target="#main" hx-push-url="true">Settings</a>
</nav>
<main id="main">
  <!-- Server renders content here -->
</main>
```

### When to Use Each

| Pattern | Use For |
|---------|---------|
| Full page load | Major sections, deep links, SEO-critical pages |
| Partial update | Filters, tabs, pagination, search results |
| Inline update | Buttons, toggles, form submissions |

---

## State Management

### Server as Source of Truth

State lives on the server. The HTML response encodes current state and available actions.

```html
<!-- Server renders different HTML based on state -->
<div id="task-123">
  <span>Buy groceries</span>
  <button hx-post="/tasks/123/complete">Complete</button>
</div>

<!-- After clicking, server returns: -->
<div id="task-123">
  <span><s>Buy groceries</s></span>
  <button hx-post="/tasks/123/uncomplete">Undo</button>
</div>
```

No client-side state synchronization. No stale data. No complex state libraries.

### Session State

Use server sessions for:
- User authentication
- Shopping carts
- Wizard/multi-step form progress
- User preferences

### URL State

Use URLs for:
- Current page/section
- Search queries
- Filters (when shareable)
- Pagination

```html
<!-- URL contains state -->
<a hx-get="/products?category=electronics&page=2" 
   hx-push-url="true"
   hx-target="#products">
  Page 2
</a>
```

---

## Forms and Validation

### Server-Side Validation (Preferred)

Validate on the server, return updated HTML with error messages:

```html
<form hx-post="/users" hx-target="#form-container">
  <input name="email" value="invalid-email">
  <button>Submit</button>
</form>

<!-- Server returns form with errors: -->
<form hx-post="/users" hx-target="#form-container">
  <input name="email" value="invalid-email" class="error">
  <span class="error-message">Please enter a valid email address</span>
  <button>Submit</button>
</form>
```

### Progressive Enhancement

Forms work without JavaScript:

```html
<form action="/users" method="POST">
  <input name="email">
  <button>Submit</button>
</form>

<!-- Enhanced with HTMX: -->
<form hx-post="/users" hx-target="#main">
  <input name="email">
  <button>Submit</button>
</form>
```

Without HTMX, form submits normally. With HTMX, it submits via AJAX.

### Inline Validation

For immediate feedback while preserving server validation:

```html
<input name="username" 
       hx-get="/check-username" 
       hx-trigger="change delay:500ms"
       hx-target="next .validation-message">
<span class="validation-message"></span>
```

---

## User Feedback

### Loading States

Use HTMX indicators:

```html
<button hx-post="/save" hx-indicator="#saving">
  Save
</button>
<span id="saving" class="htmx-indicator">
  Saving...
</span>
```

```css
.htmx-indicator {
  opacity: 0;
  transition: opacity 200ms;
}
.htmx-request .htmx-indicator {
  opacity: 1;
}
```

### Success Feedback

```html
<form hx-post="/save" 
      hx-target="#result"
      hx-swap="innerHTML">
  <!-- form fields -->
</form>
<div id="result">
  <!-- Server returns success message: -->
  <div class="alert success">Saved successfully!</div>
</div>
```

### Error Feedback

```html
<form hx-post="/save" 
      hx-target="#form"
      hx-swap="outerHTML">
  <!-- Server returns form with errors inline -->
</form>
```

Use `hx-ext="response-targets"` for different targets on success/error:

```html
<form hx-post="/save"
      hx-target="#result"
      hx-target-5*="#errors">
  <!-- 4xx/5xx responses go to #errors -->
</form>
```

---

## Error Handling

### Server Error Response Pattern

```html
<!-- Server returns on error: -->
<div id="content" class="error-state">
  <div class="error-banner">
    <strong>Unable to save changes</strong>
    <p>Your session expired. <a href="/login">Sign in again</a></p>
  </div>
  <!-- Preserve user's form data -->
  <form>
    <input name="title" value="User's typed content preserved">
  </form>
</div>
```

### HTMX Error Events

```javascript
document.body.addEventListener('htmx:responseError', function(evt) {
  // Server returned 4xx/5xx
  // Response HTML is still swapped per hx-target
});

document.body.addEventListener('htmx:sendError', function(evt) {
  // Network error - no response
  // Show generic error message
});
```

### Graceful Degradation

Always provide fallback for JavaScript failures:

```html
<!-- Works without HTMX -->
<form action="/submit" method="POST">
  <input name="data">
  <button>Submit</button>
</form>

<!-- Enhanced with HTMX -->
<form action="/submit" method="POST"
      hx-post="/submit"
      hx-target="#result">
  <input name="data">
  <button>Submit</button>
</form>
```

---

## Real-Time Updates

### Server-Sent Events (SSE)

For server-initiated updates:

```html
<div hx-ext="sse" sse-connect="/notifications">
  <div sse-swap="message" hx-swap="innerHTML">
    <!-- Server pushes updates here -->
  </div>
</div>
```

**Use for:**
- Notifications
- Live feeds
- Status updates
- Progress indicators

### WebSockets

For bidirectional real-time:

```html
<div hx-ext="ws" ws-connect="/chat">
  <div id="messages" ws-swap>
    <!-- Messages appear here -->
  </div>
  <form ws-send>
    <input name="message">
    <button>Send</button>
  </form>
</div>
```

**Use for:**
- Chat applications
- Collaboration tools
- Live gaming

---

## Accessibility

### Semantic HTML (Built-In)

Hypermedia systems use semantic HTML by default:

```html
<nav aria-label="Main navigation">
  <ul>
    <li><a href="/home" aria-current="page">Home</a></li>
    <li><a href="/about">About</a></li>
  </ul>
</nav>

<main>
  <h1>Page Title</h1>
  <article>
    <h2>Article Title</h2>
    <!-- content -->
  </article>
</main>
```

### Focus Management

After HTMX swaps content:

```html
<div hx-get="/search" 
     hx-trigger="keyup changed delay:500ms"
     hx-target="#results"
     hx-on::after-swap="document.getElementById('results').focus()">
</div>
```

Or use `hx-focus`:
```html
<div hx-get="/page" hx-target="#content" hx-focus="#content">
```

### ARIA for Dynamic Content

```html
<div hx-get="/notifications" 
     hx-trigger="every 30s"
     hx-target="#notif-list"
     role="log"
     aria-live="polite"
     aria-label="Notifications">
  <ul id="notif-list">
    <!-- Notifications load here -->
  </ul>
</div>
```

---

## Performance

### Initial Load

| Optimization | How |
|--------------|-----|
| Server rendering | HTML arrives ready to display |
| Minimal JavaScript | Only HTMX + lightweight libs |
| No hydration | No "interactive" delay |
| Streaming | Send HTML as it's generated |

### Partial Updates

Only request and render what changed:

```html
<!-- Only update the counter, not the whole page -->
<button hx-post="/like/123" 
        hx-target="#count-123"
        hx-swap="innerHTML">
  Like
</button>
<span id="count-123">0</span>
```

### Caching

- Cache HTML responses (HTTP caching headers)
- Use ETags for conditional requests
- Cache partial fragments independently

```html
<!-- Response cached by browser/CDN -->
<section hx-get="/trending" 
         hx-trigger="load"
         hx-target="this">
  Loading...
</section>
```

---

## Common Patterns

### Infinite Scroll

```html
<div hx-get="/items?page=1" 
     hx-trigger="revealed"
     hx-swap="beforeend"
     hx-target="this">
  <!-- Next page prepended as user scrolls -->
</div>
```

### Click to Edit

```html
<div id="item-123">
  <span>John Doe</span>
  <button hx-get="/items/123/edit" 
          hx-target="#item-123">
    Edit
  </button>
</div>

<!-- Server returns edit form: -->
<div id="item-123">
  <form hx-put="/items/123" hx-target="#item-123">
    <input name="name" value="John Doe">
    <button>Save</button>
    <button hx-get="/items/123" hx-target="#item-123">Cancel</button>
  </form>
</div>
```

### Active Search

```html
<input type="search" 
       name="q"
       hx-get="/search" 
       hx-trigger="input changed delay:300ms, search"
       hx-target="#search-results"
       placeholder="Search...">
<div id="search-results"></div>
```

### Bulk Actions

```html
<form hx-post="/items/bulk-delete" hx-target="#items">
  <table>
    <tbody id="items">
      <tr>
        <td><input type="checkbox" name="ids" value="1"></td>
        <td>Item 1</td>
      </tr>
    </tbody>
  </table>
  <button>Delete Selected</button>
</form>
```

---

## Decision Framework

### When Hypermedia Excels

- Content-heavy sites (blogs, docs, e-commerce)
- CRUD applications (admin panels, dashboards)
- Forms and data entry
- SEO-critical pages
- Teams with backend expertise
- Projects needing long-term maintainability
- Accessibility requirements

### When to Consider SPA

- Real-time collaboration (Google Docs style)
- Complex client-side visualizations (charts, maps, games)
- Offline-first requirements
- Heavy drag-and-drop interactions
- Video/audio editing
- 3D graphics

### Hybrid Approach

Use hypermedia for most of the app, embed SPA-like components where needed:

```html
<!-- Mostly hypermedia -->
<main>
  <!-- Use a JS widget for complex visualization -->
  <data-visualization data-endpoint="/api/metrics"></data-visualization>
  
  <!-- Everything else is hypermedia -->
  <nav hx-boost="true">
    <a href="/dashboard">Dashboard</a>
    <a href="/reports">Reports</a>
  </nav>
</main>
```

---

## Design Checklist

### Architecture
- [ ] Server renders HTML (not JSON + client rendering)
- [ ] Progressive enhancement (works without JS)
- [ ] HTMX attributes for dynamic updates
- [ ] Minimal JavaScript dependencies

### Navigation
- [ ] Full page loads for major sections
- [ ] Partial updates for in-page interactions
- [ ] URL reflects current state (hx-push-url)
- [ ] Back button works correctly

### Forms
- [ ] Forms work without JavaScript
- [ ] Server-side validation with error messages in HTML
- [ ] Loading states during submission
- [ ] User input preserved on error

### Feedback
- [ ] Loading indicators for requests
- [ ] Success confirmations
- [ ] Error messages near relevant elements
- [ ] Actionable recovery steps

### Accessibility
- [ ] Semantic HTML structure
- [ ] Focus management after updates
- [ ] ARIA live regions for dynamic content
- [ ] Keyboard navigation works

### Performance
- [ ] Minimal JavaScript bundle
- [ ] HTTP caching headers
- [ ] Partial updates (not full page)
- [ ] Lazy loading for below-fold content

---

## Real-World Examples

### GitHub (HTML-over-the-wire migration)

GitHub is progressively adopting hypermedia patterns:
- Server-rendered HTML for content
- PJAX/Hotwire-style navigation
- Progressive enhancement

### Basecamp (Hotwire)

Uses Turbo + Stimulus:
- Server renders all HTML
- Turbo for navigation and updates
- Stimulus for sprinkled interactivity

### Phoenix LiveView

Elixir/Phoenix framework:
- Server-side rendering via WebSocket
- Real-time updates without JavaScript
- EEx templates render HTML

---

## References

- [HTMX Documentation](https://htmx.org/)
- [HTMX Essays](https://htmx.org/essays/) - Philosophy and patterns
- [Hypermedia Systems (Book)](https://hypermedia.systems/) - Free online book
- [Hotwire (Turbo + Stimulus)](https://hotwired.dev/)
- [Phoenix LiveView](https://hexdocs.pm/phoenix_live_view/)
- [HATEOAS](https://restfulapi.net/hateoas/)
- [Web Components](https://www.webcomponents.org/)
