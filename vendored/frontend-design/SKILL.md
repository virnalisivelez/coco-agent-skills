---
name: frontend-design
description: Build production-grade frontend interfaces with modern frameworks. Use when creating web components, pages, dashboards, or applications with React, Vue, or vanilla HTML/CSS/JS. Focuses on responsive design, accessibility, and clean component architecture.
license: Apache-2.0
metadata:
  original-author: anthropic
  original-repo: anthropics/skills
  vendored-by: good-stories-llc
  version: "1.0"
---

# Frontend Design

Build polished, production-grade frontend interfaces using modern frameworks and best practices. This skill covers component architecture, responsive design, accessibility, performance, and visual polish.

## When to Use

- User wants to build a web page, dashboard, or application UI
- User needs responsive components for React, Vue, or vanilla HTML/CSS/JS
- User asks for a landing page, admin panel, or interactive widget
- User needs help with layout, typography, color systems, or animations

## Design System Foundation

Before writing any component, establish the design system tokens. This ensures visual consistency across the entire interface.

### Color System

Define colors as CSS custom properties on `:root`. Always include light and dark mode:

```css
:root {
  /* Neutrals */
  --color-bg: #FAFAFA;
  --color-surface: #FFFFFF;
  --color-border: #E5E7EB;
  --color-text-primary: #111827;
  --color-text-secondary: #6B7280;
  --color-text-muted: #9CA3AF;

  /* Brand */
  --color-accent: #2563EB;
  --color-accent-hover: #1D4ED8;
  --color-accent-light: #DBEAFE;

  /* Feedback */
  --color-success: #059669;
  --color-warning: #D97706;
  --color-error: #DC2626;

  /* Spacing scale */
  --space-xs: 0.25rem;
  --space-sm: 0.5rem;
  --space-md: 1rem;
  --space-lg: 1.5rem;
  --space-xl: 2rem;
  --space-2xl: 3rem;
  --space-3xl: 4rem;

  /* Typography scale */
  --text-xs: 0.75rem;
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.125rem;
  --text-xl: 1.25rem;
  --text-2xl: 1.5rem;
  --text-3xl: 2rem;
  --text-4xl: 2.5rem;

  /* Radius */
  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;
  --radius-lg: 0.75rem;
  --radius-xl: 1rem;
  --radius-full: 9999px;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
}
```

### Typography

Choose a font pairing: one for headings, one for body. Load via Google Fonts or system stack:

```css
body {
  font-family: 'DM Sans', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  font-size: var(--text-base);
  line-height: 1.6;
  color: var(--color-text-primary);
}

h1, h2, h3, h4 {
  font-family: 'Inter', sans-serif;
  font-weight: 700;
  line-height: 1.2;
  letter-spacing: -0.02em;
}
```

## Responsive Breakpoints

Use mobile-first breakpoints. Write base styles for mobile, then layer on complexity:

```css
/* Mobile-first base styles */
.container {
  width: 100%;
  padding: 0 var(--space-md);
  margin: 0 auto;
}

/* Tablet: 640px+ */
@media (min-width: 640px) {
  .container { max-width: 640px; }
}

/* Desktop: 1024px+ */
@media (min-width: 1024px) {
  .container { max-width: 1024px; }
}

/* Wide: 1280px+ */
@media (min-width: 1280px) {
  .container { max-width: 1280px; }
}
```

### Layout Patterns

Use CSS Grid for page layouts and Flexbox for component internals:

```css
/* Page layout: sidebar + main */
.layout {
  display: grid;
  grid-template-columns: 1fr;
  gap: var(--space-lg);
}

@media (min-width: 1024px) {
  .layout {
    grid-template-columns: 260px 1fr;
  }
}

/* Card grid: auto-fill responsive */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: var(--space-lg);
}
```

## Component Architecture

### Principles

1. **Single responsibility:** Each component does one thing well
2. **Props down, events up:** Data flows downward, actions bubble upward
3. **Composition over inheritance:** Build complex UIs by combining simple components
4. **Stateless when possible:** Prefer presentational components that receive data via props

### React Component Template

```jsx
function Card({ title, description, icon, actions, variant = "default" }) {
  return (
    <div className={`card card--${variant}`} role="article">
      {icon && <div className="card__icon">{icon}</div>}
      <div className="card__body">
        <h3 className="card__title">{title}</h3>
        {description && (
          <p className="card__description">{description}</p>
        )}
      </div>
      {actions && (
        <div className="card__actions">{actions}</div>
      )}
    </div>
  );
}
```

### Vanilla HTML/CSS Component

```html
<article class="card" role="article">
  <div class="card__icon">
    <svg><!-- icon --></svg>
  </div>
  <div class="card__body">
    <h3 class="card__title">Title</h3>
    <p class="card__description">Description text</p>
  </div>
  <div class="card__actions">
    <button class="btn btn--primary">Action</button>
  </div>
</article>
```

Use BEM naming (`block__element--modifier`) for CSS classes in vanilla projects.

## Accessibility Checklist

Every interface must meet WCAG 2.1 AA. Check these before shipping:

### Structure
- [ ] Semantic HTML: use `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>`
- [ ] Heading hierarchy: exactly one `<h1>`, logical `<h2>`-`<h6>` nesting
- [ ] Landmark regions: `<main>`, `<nav>`, `<header>`, `<footer>` present
- [ ] Skip-to-content link as the first focusable element

### Interactive Elements
- [ ] All interactive elements are focusable and have visible focus styles
- [ ] Buttons use `<button>`, links use `<a>` -- never swap roles
- [ ] Custom controls have ARIA roles, states, and properties
- [ ] Keyboard navigation works: Tab, Shift+Tab, Enter, Escape, Arrow keys

### Visual
- [ ] Color contrast ratio: 4.5:1 for normal text, 3:1 for large text
- [ ] Information not conveyed by color alone (use icons, text, patterns)
- [ ] Text resizable to 200% without horizontal scroll
- [ ] Reduced motion: respect `prefers-reduced-motion`

### Forms
- [ ] Every input has a visible `<label>` (not just placeholder)
- [ ] Error messages are associated with inputs via `aria-describedby`
- [ ] Required fields marked with `aria-required="true"`
- [ ] Form validation errors announced to screen readers

### Media
- [ ] Images have meaningful `alt` text (or `alt=""` for decorative)
- [ ] Videos have captions/transcripts
- [ ] No auto-playing audio or video

## Performance Optimization

### Critical Rendering Path
1. Inline critical CSS in `<head>` (above-the-fold styles, under 14KB)
2. Defer non-critical CSS with `<link rel="preload" as="style">`
3. Load fonts with `font-display: swap` to prevent invisible text
4. Defer JavaScript: `<script defer src="app.js">`

### Images
- Use `loading="lazy"` on images below the fold
- Provide `width` and `height` attributes to prevent layout shift
- Use responsive images with `srcset` and `sizes`
- Prefer modern formats: WebP with JPEG fallback

### Bundle Size
- Tree-shake unused code (modern bundlers do this by default)
- Code-split routes: load only what the current page needs
- Avoid importing entire libraries for one utility function

## Animations and Transitions

Use CSS transitions for simple state changes, CSS animations for repeating effects:

```css
/* Transition: smooth hover effect */
.card {
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}
.card:hover {
  transform: translateY(-2px);
  box-shadow: var(--shadow-lg);
}

/* Respect reduced motion preference */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}

/* Fade-in on scroll (use with IntersectionObserver) */
.fade-up {
  opacity: 0;
  transform: translateY(20px);
  transition: opacity 0.5s ease, transform 0.5s ease;
}
.fade-up.visible {
  opacity: 1;
  transform: translateY(0);
}
```

## Dark Mode

Support dark mode via CSS custom properties and media query:

```css
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #0F172A;
    --color-surface: #1E293B;
    --color-border: #334155;
    --color-text-primary: #F1F5F9;
    --color-text-secondary: #94A3B8;
    --color-text-muted: #64748B;
  }
}
```

Optionally add a toggle button that sets `data-theme="dark"` on `<html>` and stores preference in `localStorage`.

## Output Expectations

When building a frontend:

1. **Deliver complete, runnable code** -- not fragments. Include HTML, CSS, and JS
2. **Use semantic HTML** with proper ARIA attributes
3. **Mobile-first responsive** -- test at 375px, 768px, 1024px, 1440px
4. **Include hover/focus/active states** for all interactive elements
5. **Handle empty states** -- what does the UI look like with no data?
6. **Handle loading states** -- skeleton screens or spinners
7. **Handle error states** -- user-friendly error messages
8. **Include comments** on non-obvious design decisions
