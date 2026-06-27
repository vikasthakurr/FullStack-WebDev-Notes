# Media Queries

## What Are Media Queries?

Media queries are CSS conditionals that apply styles based on the characteristics of the user's device — primarily screen width, but also orientation, resolution, color scheme preference, and more. They are the backbone of **responsive design**.

**Analogy:** Media queries are like having different outfits for different weather. You do not change who you are (the HTML), but you adapt your appearance (CSS) based on the environment (screen size).

---

## Why They Matter

- People browse the web on phones, tablets, laptops, desktops, TVs, and watches.
- A layout designed for a 27-inch monitor is unusable on a 5-inch phone.
- Google uses mobile-friendliness as a ranking factor.
- Media queries let you build **one codebase** that adapts to any screen.

---

## Syntax

```css
@media (condition) {
  /* Styles applied only when condition is true */
}
```

### Basic Width Queries

```css
/* Applies when viewport is 768px or narrower */
@media (max-width: 768px) {
  .sidebar {
    display: none;
  }
}

/* Applies when viewport is 769px or wider */
@media (min-width: 769px) {
  .container {
    max-width: 1200px;
    margin: 0 auto;
  }
}
```

---

## Mobile-First vs Desktop-First

### Mobile-First (Recommended)

Write base styles for mobile, then add complexity for larger screens with `min-width`.

```css
/* Base: mobile styles */
.grid {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

/* Tablet and up */
@media (min-width: 768px) {
  .grid {
    flex-direction: row;
    flex-wrap: wrap;
  }
  .grid > * {
    flex: 1 1 45%;
  }
}

/* Desktop and up */
@media (min-width: 1024px) {
  .grid > * {
    flex: 1 1 30%;
  }
}
```

### Desktop-First

Write base styles for desktop, then simplify for smaller screens with `max-width`.

```css
/* Base: desktop styles */
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
}

/* Tablet */
@media (max-width: 1024px) {
  .grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* Mobile */
@media (max-width: 768px) {
  .grid {
    grid-template-columns: 1fr;
  }
}
```

**Why mobile-first is preferred:**

- Forces you to prioritize content (mobile has less space).
- Results in simpler base styles that progressively enhance.
- Smaller devices download fewer styles (better performance).

---

## Common Breakpoints

| Name        | Min-Width | Typical Devices             |
| ----------- | --------- | --------------------------- |
| Phone       | 0         | Small phones (default base) |
| Large Phone | 480px     | Larger phones (landscape)   |
| Tablet      | 768px     | Tablets, small laptops      |
| Desktop     | 1024px    | Laptops, desktops           |
| Large       | 1200px    | Large desktops              |
| XL          | 1440px    | Ultra-wide monitors         |

> These are **guidelines**, not rules. Set breakpoints where your design breaks, not at specific device widths.

---

## Combining Conditions

### AND (both must be true)

```css
@media (min-width: 768px) and (max-width: 1024px) {
  /* Tablet only */
  .sidebar {
    width: 200px;
  }
}
```

### OR (comma-separated — either can be true)

```css
@media (max-width: 600px), (orientation: portrait) {
  /* Phones OR portrait orientation */
  .hero {
    height: 50vh;
  }
}
```

### NOT (invert the condition)

```css
@media not print {
  /* Everything except print */
  .no-print-badge {
    display: block;
  }
}
```

---

## Media Types

```css
@media screen {
  /* computer screens, phones, tablets */
}
@media print {
  /* when printing the page */
}
@media all {
  /* all devices (default if omitted) */
}
```

### Print Stylesheet Example

```css
@media print {
  nav,
  footer,
  .sidebar,
  .ads {
    display: none;
  }

  body {
    font-size: 12pt;
    color: black;
    background: white;
  }

  a[href]::after {
    content: " (" attr(href) ")";
    font-size: 0.8em;
  }
}
```

---

## Media Features Beyond Width

### Orientation

```css
@media (orientation: portrait) {
  /* Height > Width (phones held vertically) */
}
@media (orientation: landscape) {
  /* Width > Height */
}
```

### Hover Capability

```css
/* Device has a hover mechanism (mouse) */
@media (hover: hover) {
  .card:hover {
    transform: scale(1.02);
  }
}

/* Device has no hover (touchscreen) */
@media (hover: none) {
  .card {
    /* no hover effects — use tap instead */
  }
}
```

### Prefers Color Scheme (Dark Mode)

```css
/* User prefers dark mode */
@media (prefers-color-scheme: dark) {
  :root {
    --bg: #1a1a2e;
    --text: #e0e0e0;
  }
}

/* User prefers light mode */
@media (prefers-color-scheme: light) {
  :root {
    --bg: #ffffff;
    --text: #1a1a1a;
  }
}
```

### Prefers Reduced Motion

```css
/* User wants minimal animations */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

### Resolution (High DPI / Retina)

```css
@media (min-resolution: 2dppx) {
  .logo {
    background-image: url("logo@2x.png");
  }
}
```

---

## Responsive Design Patterns

### Responsive Navigation

```css
/* Mobile: hamburger menu */
.nav-links {
  display: none;
  flex-direction: column;
}
.nav-links.active {
  display: flex;
}
.hamburger {
  display: block;
}

/* Desktop: horizontal nav */
@media (min-width: 768px) {
  .nav-links {
    display: flex;
    flex-direction: row;
    gap: 2rem;
  }
  .hamburger {
    display: none;
  }
}
```

### Responsive Typography

```css
html {
  font-size: 14px;
}

@media (min-width: 768px) {
  html {
    font-size: 16px;
  }
}

@media (min-width: 1200px) {
  html {
    font-size: 18px;
  }
}

/* Modern approach: fluid typography with clamp() */
html {
  font-size: clamp(14px, 1.5vw, 18px);
}
```

### Responsive Images

```css
img {
  max-width: 100%;
  height: auto;
}
```

```html
<!-- HTML approach with srcset -->
<img
  src="image-800.jpg"
  srcset="image-400.jpg 400w, image-800.jpg 800w, image-1200.jpg 1200w"
  sizes="(max-width: 600px) 400px, (max-width: 1024px) 800px, 1200px"
  alt="Responsive image"
/>
```

---

## Container Queries (Modern CSS)

Container queries allow components to respond to their **container's size** rather than the viewport.

```css
.card-container {
  container-type: inline-size;
  container-name: card;
}

@container card (min-width: 400px) {
  .card {
    display: flex;
    flex-direction: row;
  }
}

@container card (max-width: 399px) {
  .card {
    display: flex;
    flex-direction: column;
  }
}
```

This is useful for reusable components that might appear in sidebars (narrow) or main content areas (wide).

---

## Best Practices

1. **Design mobile-first** — start simple, add complexity for larger screens.
2. **Set breakpoints based on content** — not specific devices. When the layout breaks, add a breakpoint.
3. **Use relative units** (`rem`, `em`, `%`) — they scale better than `px` at different screen sizes.
4. **Test on real devices** — emulators miss touch behavior, font rendering, and performance.
5. **Combine media queries with modern CSS** — `clamp()`, `min()`, `max()`, and container queries reduce the need for breakpoints.
6. **Respect user preferences** — `prefers-reduced-motion` and `prefers-color-scheme` are not optional nice-to-haves.
7. **Keep media queries close to the code they modify** — easier to maintain than a separate section at the bottom.

---

## Common Mistakes

| Mistake                                       | Why It Is Wrong                                      | Fix                                                                        |
| --------------------------------------------- | ---------------------------------------------------- | -------------------------------------------------------------------------- |
| Using device-specific breakpoints             | New devices constantly launch with new sizes         | Break where your design breaks                                             |
| Hiding content on mobile with `display: none` | Content is still downloaded; hurts performance       | Restructure layout or load conditionally                                   |
| Forgetting the viewport meta tag              | Mobile browsers render at desktop width by default   | Add `<meta name="viewport" content="width=device-width, initial-scale=1">` |
| Using only `max-width` (desktop-first)        | Results in overriding styles; harder to maintain     | Prefer `min-width` (mobile-first)                                          |
| Not testing between breakpoints               | Layouts can break at widths between your breakpoints | Resize slowly and fix any gaps                                             |
| Fixed widths instead of fluid                 | Content overflows on small screens                   | Use `%`, `fr`, `max-width`, or `flex`                                      |

---

## Summary

- Media queries apply CSS conditionally based on viewport size, orientation, user preferences, and device capabilities.
- **Mobile-first** (`min-width`) is the recommended approach — build up from simple to complex.
- Set breakpoints where your design breaks, not at arbitrary device widths.
- Beyond width: use `prefers-color-scheme` for dark mode, `prefers-reduced-motion` for accessibility, and `hover` for touch vs. mouse.
- Container queries are the modern evolution — components respond to their container, not the viewport.
- Always include `<meta name="viewport">` in your HTML, or media queries will not work correctly on mobile.
