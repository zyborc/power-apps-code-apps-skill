# design.md — Design Rules for Power Apps Code Apps

## Design Philosophy

This skill targets a **Claude Code dark-UI aesthetic** — the same visual language used in Claude Code and modern AI developer tooling:
dark navy backgrounds, high contrast, monospace accents, clean layouts, no decorative clutter.

For expressive or creative UIs, pair with the public [`frontend-design` skill](https://github.com/anthropics/skills/tree/main/skills/frontend-design) from Anthropics.

---

## Design Tokens

### Colors
```css
/* Backgrounds */
--bg-base:       #0d1424;   /* page / app shell */
--bg-surface:    #131d30;   /* cards, panels */
--bg-elevated:   #1a2540;   /* dropdowns, modals */
--bg-hover:      #1e2d47;   /* interactive hover */

/* Borders */
--border:        #1e2d47;
--border-subtle: #162035;

/* Text */
--text-primary:   #e2e8f0;
--text-secondary: #94a3b8;
--text-muted:     #64748b;

/* Accent (Claude orange-amber) */
--accent:        #f59e0b;
--accent-hover:  #d97706;

/* Semantic */
--success:  #10b981;
--warning:  #f59e0b;
--error:    #ef4444;
--info:     #3b82f6;
```

### Typography
```css
/* Import in index.css or index.html */
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;600&family=DM+Mono:wght@400;500&display=swap');

--font-sans: 'DM Sans', sans-serif;
--font-mono: 'DM Mono', monospace;
```

**Rules:**
- `DM Sans` — all UI text
- `DM Mono` — numbers, IDs, percentages, code, timestamps

### Spacing & Radius
```css
--radius-sm:  6px;
--radius-md:  10px;
--radius-lg:  14px;

--space-xs:   4px;
--space-sm:   8px;
--space-md:   16px;
--space-lg:   24px;
--space-xl:   32px;
```

### Status Dot Pattern
```tsx
// Use a colored dot + label — never color text alone
<span style={{ display: 'flex', alignItems: 'center', gap: 6 }}>
  <span style={{ width: 8, height: 8, borderRadius: '50%', background: 'var(--success)' }} />
  <span style={{ color: 'var(--text-secondary)', fontFamily: 'var(--font-mono)', fontSize: 12 }}>Active</span>
</span>
```

---

## Tailwind in Code Apps

Unlike Claude Artifacts, the Vite setup runs a full Tailwind compiler.
Custom config and arbitrary values work.

Convention: **Tailwind for layout, inline styles for design tokens.**

```tsx
// ✅ Correct split
<div className="flex items-center gap-4 overflow-hidden"
  style={{ background: "#0d1424", borderRadius: 12, padding: "20px 22px" }}>

// ❌ Avoid — breaks portability across projects and models
<div className="bg-[#0d1424] rounded-xl p-5">
```

Reason: inline token values are portable across projects, teammates,
and AI models without needing the Tailwind config.

---

## Power Apps Specific Constraints

### Root must fill the iframe
Code Apps run inside a managed iframe in the Power Apps host.
Set this in `src/index.css`:

```css
html, body, #root {
  height: 100%;
  margin: 0;
  padding: 0;
  overflow: hidden;
}
```

Scroll only inside `<main>`:
```tsx
<main style={{ flex: 1, overflow: "auto" }}>
```

### No browser storage
```typescript
// ❌ Not supported in Code Apps
localStorage.setItem(...)
sessionStorage.getItem(...)

// ✅ Use instead
// - React state for session data
// - Dataverse for persistent data
```

### Auth context
The logged-in user's info (name, email, AAD groups) is available
through the Power Apps host context after `initialize()` resolves.
The logged-in user's info and app context are available via the `@microsoft/power-apps/app` SDK using the `getContext()` function.
```typescript
import { getContext } from '@microsoft/power-apps/app';
const ctx = await getContext();
// Access: ctx.user.fullName, ctx.user.userPrincipalName, ctx.app.appId, etc.
```
Never prompt for login or implement MSAL — the host handles it.

### Fonts
Google Fonts load correctly — no restrictions.
Always import DM Sans + DM Mono from the design skill.

### Drag and Drop
Standard HTML5 drag-and-drop API works normally inside Code Apps.
`draggable`, `onDragStart`, `onDragOver`, `onDrop` — all supported.

---

## Component Checklist

Run through this before shipping any view:

- [ ] CSS variables applied (colors, spacing, radius — see Design Tokens above)
- [ ] DM Sans + DM Mono imported from Google Fonts
- [ ] Numbers / IDs / percentages use DM Mono
- [ ] Status indicators use semantic color + dot pattern
- [ ] Empty states in all lists and tables
- [ ] Loading state present (guard: isInitialized)
- [ ] No direct API calls — only generated services
- [ ] Root CSS set (height 100%, overflow hidden)
- [ ] Role-based visibility applied where needed
