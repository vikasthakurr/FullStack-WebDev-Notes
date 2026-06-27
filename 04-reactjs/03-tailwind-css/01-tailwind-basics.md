# Tailwind CSS Basics

## What Is Utility-First CSS?

Utility-first CSS is an approach where you **style elements by composing small, single-purpose classes** directly in your HTML/JSX, rather than writing custom CSS in separate files. Each class does one thing: `p-4` adds padding, `text-red-500` sets text color, `flex` enables flexbox.

**Analogy:** Traditional CSS is like cooking from scratch — you write custom recipes (classes) for every dish. Tailwind is like having a fully stocked spice rack — you grab pre-made utilities and combine them to create any flavor.

---

## Why Tailwind CSS?

| Traditional CSS Problem                        | Tailwind Solution                        |
| ---------------------------------------------- | ---------------------------------------- |
| Naming classes is hard (`.card-wrapper-inner`) | No custom names — use utilities directly |
| Dead CSS accumulates over time                 | Tree-shaking removes unused utilities    |
| Inconsistent spacing/colors                    | Design system enforced through config    |
| Context switching between CSS and HTML         | Style where you see the markup           |
| Specificity wars                               | Flat utility classes — no nesting        |
| Large CSS bundles                              | Only ships classes you actually use      |

---

## Installation in a React + Vite Project

```bash
# Create a Vite project
npm create vite@latest my-app -- --template react
cd my-app

# Install Tailwind CSS v4 (latest)
npm install tailwindcss @tailwindcss/vite

# Or for Tailwind v3 (with PostCSS)
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

### Tailwind v4 Setup (Vite Plugin)

```javascript
// vite.config.js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [react(), tailwindcss()],
});
```

```css
/* src/index.css */
@import "tailwindcss";
```

### Tailwind v3 Setup (PostCSS)

```javascript
// tailwind.config.js
export default {
  content: ["./index.html", "./src/**/*.{js,ts,jsx,tsx}"],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

```css
/* src/index.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

---

## Core Concepts

### Spacing (Padding & Margin)

Tailwind uses a consistent spacing scale. `1` unit = `0.25rem` (4px).

```jsx
<div className="p-4">Padding 1rem (16px) all sides</div>
<div className="px-6 py-2">Horizontal 1.5rem, Vertical 0.5rem</div>
<div className="mt-8 mb-4">Margin-top 2rem, margin-bottom 1rem</div>
<div className="m-auto">Margin auto (centering)</div>
<div className="space-y-4">4px gap between child elements (vertical)</div>
```

| Class  | Value   | Pixels |
| ------ | ------- | ------ |
| `p-1`  | 0.25rem | 4px    |
| `p-2`  | 0.5rem  | 8px    |
| `p-4`  | 1rem    | 16px   |
| `p-8`  | 2rem    | 32px   |
| `p-16` | 4rem    | 64px   |

### Colors

```jsx
<p className="text-blue-500">Blue text</p>
<div className="bg-gray-100">Light gray background</div>
<div className="border border-red-300">Red border</div>
<button className="bg-indigo-600 text-white hover:bg-indigo-700">
  Styled Button
</button>
```

Color scale: `50` (lightest) → `950` (darkest). Example: `blue-50`, `blue-100`, ..., `blue-900`, `blue-950`.

### Typography

```jsx
<h1 className="text-4xl font-bold">Large Bold Heading</h1>
<p className="text-base text-gray-600 leading-relaxed">Body text</p>
<span className="text-sm font-medium uppercase tracking-wide">Label</span>
<p className="text-lg italic underline">Styled paragraph</p>
```

| Class           | Effect                  |
| --------------- | ----------------------- |
| `text-xs`       | 0.75rem font size       |
| `text-sm`       | 0.875rem                |
| `text-base`     | 1rem (default)          |
| `text-lg`       | 1.125rem                |
| `text-2xl`      | 1.5rem                  |
| `font-bold`     | font-weight: 700        |
| `leading-tight` | line-height: 1.25       |
| `tracking-wide` | letter-spacing: 0.025em |

---

### Flexbox & Grid

```jsx
// Flexbox
<div className="flex items-center justify-between gap-4">
  <span>Left</span>
  <span>Right</span>
</div>

<div className="flex flex-col items-center gap-2">
  <p>Stacked vertically</p>
  <p>With gap between</p>
</div>

// Grid
<div className="grid grid-cols-3 gap-6">
  <div>Col 1</div>
  <div>Col 2</div>
  <div>Col 3</div>
</div>

<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
  {/* Responsive grid — 1 col mobile, 2 col tablet, 4 col desktop */}
</div>
```

### Width & Height

```jsx
<div className="w-full">Full width</div>
<div className="w-1/2">50% width</div>
<div className="w-64">16rem (256px) fixed width</div>
<div className="max-w-md mx-auto">Max-width medium, centered</div>
<div className="h-screen">Full viewport height</div>
<div className="min-h-screen">At least full viewport height</div>
```

---

### Responsive Design (Breakpoint Prefixes)

Tailwind is **mobile-first**. Unprefixed utilities apply to all screen sizes. Prefixed utilities apply at that breakpoint and above.

```jsx
<div className="text-sm md:text-base lg:text-lg">
  {/* Small on mobile, base on tablet, large on desktop */}
</div>

<div className="flex flex-col md:flex-row">
  {/* Stacked on mobile, horizontal on tablet+ */}
</div>

<div className="hidden lg:block">
  {/* Hidden on mobile/tablet, visible on desktop */}
</div>

<div className="p-4 md:p-8 lg:p-12">
  {/* Responsive padding */}
</div>
```

| Prefix | Breakpoint  | Min-width |
| ------ | ----------- | --------- |
| `sm:`  | Small       | 640px     |
| `md:`  | Medium      | 768px     |
| `lg:`  | Large       | 1024px    |
| `xl:`  | Extra Large | 1280px    |
| `2xl:` | 2X Large    | 1536px    |

### Hover, Focus & State Variants

```jsx
<button className="bg-blue-500 hover:bg-blue-700 active:bg-blue-800 focus:ring-2 focus:ring-blue-300 transition-colors">
  Interactive Button
</button>

<input className="border border-gray-300 focus:border-blue-500 focus:outline-none focus:ring-2 focus:ring-blue-200" />

<a className="text-blue-600 hover:text-blue-800 hover:underline">
  Hover link
</a>

<div className="opacity-50 group-hover:opacity-100">
  {/* Changes when parent with "group" class is hovered */}
</div>
```

### Dark Mode

```jsx
<div className="bg-white dark:bg-gray-900 text-black dark:text-white">
  <h1 className="text-gray-900 dark:text-gray-100">Adapts to dark mode</h1>
  <p className="text-gray-600 dark:text-gray-400">Subtitle text</p>
</div>
```

Enable dark mode in config (v3):

```javascript
// tailwind.config.js
export default {
  darkMode: "class", // or 'media' for system preference
  // ...
};
```

---

## Practical Example — Card Component

```jsx
function ProductCard({ product }) {
  return (
    <div className="max-w-sm rounded-lg overflow-hidden shadow-lg bg-white dark:bg-gray-800 hover:shadow-xl transition-shadow">
      <img
        src={product.image}
        alt={product.name}
        className="w-full h-48 object-cover"
      />
      <div className="p-6">
        <h2 className="text-xl font-bold text-gray-900 dark:text-white mb-2">
          {product.name}
        </h2>
        <p className="text-gray-600 dark:text-gray-300 text-sm mb-4">
          {product.description}
        </p>
        <div className="flex items-center justify-between">
          <span className="text-2xl font-bold text-indigo-600">
            ${product.price}
          </span>
          <button className="px-4 py-2 bg-indigo-600 text-white rounded-md hover:bg-indigo-700 focus:ring-2 focus:ring-indigo-300 transition-colors">
            Add to Cart
          </button>
        </div>
      </div>
    </div>
  );
}
```

---

## Tailwind vs Traditional CSS

### Traditional CSS

```css
/* styles.css */
.card {
  max-width: 24rem;
  border-radius: 0.5rem;
  overflow: hidden;
  box-shadow: 0 10px 15px rgba(0, 0, 0, 0.1);
  background: white;
}
.card:hover {
  box-shadow: 0 20px 25px rgba(0, 0, 0, 0.15);
}
.card-body {
  padding: 1.5rem;
}
.card-title {
  font-size: 1.25rem;
  font-weight: 700;
  margin-bottom: 0.5rem;
}
```

```jsx
<div className="card">
  <div className="card-body">
    <h2 className="card-title">Title</h2>
  </div>
</div>
```

### Tailwind Equivalent

```jsx
// No separate CSS file needed
<div className="max-w-sm rounded-lg overflow-hidden shadow-lg bg-white hover:shadow-xl">
  <div className="p-6">
    <h2 className="text-xl font-bold mb-2">Title</h2>
  </div>
</div>
```

---

## Best Practices

1. **Use consistent spacing** — stick to the spacing scale (`4`, `6`, `8`) instead of arbitrary values.
2. **Extract components, not classes** — in React, reuse with components, not `@apply` (which defeats the purpose).
3. **Mobile-first** — write base styles for mobile, then add `md:`, `lg:` prefixes for larger screens.
4. **Use design tokens** — customize `tailwind.config.js` for your brand colors, fonts, and spacing.
5. **Avoid arbitrary values** — `w-[347px]` is a code smell. Use the closest scale value or add to config.
6. **Group related utilities** — order: layout → spacing → sizing → typography → visual → interactive.
7. **Use `@apply` sparingly** — only for truly repeated patterns that can't be a component (like base prose styles).
8. **Install Tailwind IntelliSense** — the VS Code extension provides autocomplete, preview, and linting.

---

## Common Mistakes

| Mistake                                  | Why It's Wrong                                | Fix                                               |
| ---------------------------------------- | --------------------------------------------- | ------------------------------------------------- |
| Writing custom CSS alongside Tailwind    | Two systems fighting — inconsistent results   | Use Tailwind utilities, customize via config      |
| Overusing `@apply` to create classes     | Defeats utility-first approach                | Extract React components instead                  |
| Not configuring `content` paths (v3)     | Tailwind can't find classes — they get purged | Include all template file paths in config         |
| Using arbitrary values everywhere        | Breaks design system consistency              | Add values to `tailwind.config.js` theme          |
| Forgetting mobile-first approach         | Styles break on small screens                 | Start with mobile styles, add breakpoint prefixes |
| Not using `transition` for hover effects | State changes feel abrupt                     | Add `transition-colors` or `transition-all`       |
| Ignoring dark mode classes               | App looks bad in dark mode                    | Add `dark:` variants for backgrounds and text     |

---

## Summary

- Tailwind CSS is a utility-first framework — style elements by combining small, single-purpose classes.
- Install with Vite plugin (v4) or PostCSS (v3). Configure `content` paths so unused styles are purged.
- Core utilities: spacing (`p-4`, `m-2`), colors (`text-blue-500`, `bg-gray-100`), typography (`text-lg`, `font-bold`), layout (`flex`, `grid`).
- Responsive design uses breakpoint prefixes: `sm:`, `md:`, `lg:`, `xl:`, `2xl:` (mobile-first).
- State variants: `hover:`, `focus:`, `active:`, `dark:`, `group-hover:` modify styles on interaction.
- In React, extract reusable **components** rather than using `@apply` to create CSS classes.
- Customize the design system in `tailwind.config.js` — colors, spacing, fonts, breakpoints.
