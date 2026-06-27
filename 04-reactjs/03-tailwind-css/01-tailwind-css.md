# Tailwind CSS

## What is Tailwind CSS?

Tailwind CSS is a **utility-first CSS framework** — instead of writing custom CSS classes, you compose designs directly in your HTML using small, single-purpose utility classes.

```html
<!-- Traditional CSS approach -->
<button class="btn btn-primary">Click me</button>

<!-- Tailwind CSS approach -->
<button
  class="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 transition-colors"
>
  Click me
</button>
```

Each class does **one thing**: `bg-blue-600` sets background color, `px-4` sets horizontal padding, `rounded-lg` adds border radius. You build up complex designs by combining these utilities.

---

## Why Tailwind vs Traditional CSS vs Bootstrap

| Feature                | Traditional CSS      | Bootstrap            | Tailwind CSS            |
| ---------------------- | -------------------- | -------------------- | ----------------------- |
| Approach               | Write custom classes | Pre-built components | Utility classes         |
| File size (production) | Grows with project   | ~25KB (used CSS)     | ~10KB (purged)          |
| Customization          | Full control         | Override variables   | Full control via config |
| Learning curve         | CSS knowledge        | Class names          | Utility class names     |
| Design consistency     | Manual enforcement   | Opinionated          | Design tokens/config    |
| Component look         | Unique per project   | "Bootstrap look"     | Unique per project      |
| Responsive design      | Write media queries  | Breakpoint classes   | Breakpoint prefixes     |

### Why Choose Tailwind?

- **No context switching** — style without leaving your HTML/JSX.
- **No naming fatigue** — no more inventing class names like `.card-wrapper-inner-header`.
- **Dead code elimination** — unused utilities are purged in production (tiny bundle).
- **Consistent spacing/colors** — design system built into the config.
- **Rapid prototyping** — build UIs without writing a single CSS file.

---

## Setup in a React + Vite Project

### Step 1: Create the Project

```bash
npm create vite@latest my-app -- --template react
cd my-app
```

### Step 2: Install Tailwind CSS

```bash
npm install -D tailwindcss @tailwindcss/vite
```

### Step 3: Configure the Vite Plugin

```javascript
// vite.config.js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [react(), tailwindcss()],
});
```

### Step 4: Import Tailwind in Your CSS

```css
/* src/index.css */
@import "tailwindcss";
```

### Step 5: Start the Dev Server

```bash
npm run dev
```

### Verify It Works

```jsx
function App() {
  return (
    <h1 className="text-3xl font-bold text-blue-600 p-8">
      Tailwind is working!
    </h1>
  );
}
```

---

## Core Concepts

### Utility Classes

Every CSS property has corresponding utility classes:

```html
<!-- Spacing -->
<div class="p-4 m-2 px-6 py-3 mt-8 mb-4"></div>

<!-- Typography -->
<p
  class="text-lg font-semibold text-gray-700 leading-relaxed tracking-wide"
></p>

<!-- Layout -->
<div class="flex items-center justify-between w-full h-screen"></div>

<!-- Borders & Shadows -->
<div class="border border-gray-300 rounded-xl shadow-lg"></div>

<!-- Sizing -->
<div class="w-64 h-32 max-w-lg min-h-screen"></div>
```

### Spacing Scale

Tailwind uses a consistent spacing scale where `1 unit = 0.25rem (4px)`:

| Class  | Value         |
| ------ | ------------- |
| `p-0`  | 0px           |
| `p-1`  | 4px (0.25rem) |
| `p-2`  | 8px (0.5rem)  |
| `p-4`  | 16px (1rem)   |
| `p-6`  | 24px (1.5rem) |
| `p-8`  | 32px (2rem)   |
| `p-12` | 48px (3rem)   |
| `p-16` | 64px (4rem)   |

### Responsive Design

Tailwind uses **mobile-first breakpoint prefixes**. Unprefixed utilities apply to all sizes, prefixed ones apply at that breakpoint **and above**.

```html
<!-- Mobile: stack, Tablet: 2 columns, Desktop: 3 columns -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  <div>Item 1</div>
  <div>Item 2</div>
  <div>Item 3</div>
</div>
```

| Prefix | Min Width | Typical Device |
| ------ | --------- | -------------- |
| `sm:`  | 640px     | Large phones   |
| `md:`  | 768px     | Tablets        |
| `lg:`  | 1024px    | Laptops        |
| `xl:`  | 1280px    | Desktops       |
| `2xl:` | 1536px    | Large screens  |

```html
<!-- Text size changes per breakpoint -->
<h1 class="text-xl sm:text-2xl md:text-3xl lg:text-5xl">Responsive Heading</h1>

<!-- Hidden on mobile, visible on desktop -->
<nav class="hidden lg:flex gap-4">
  <a href="/about">About</a>
  <a href="/contact">Contact</a>
</nav>
```

### Hover, Focus, and State Variants

```html
<!-- Hover -->
<button class="bg-blue-500 hover:bg-blue-700 transition-colors">
  Hover me
</button>

<!-- Focus -->
<input
  class="border focus:border-blue-500 focus:ring-2 focus:ring-blue-200 outline-none"
/>

<!-- Active -->
<button class="bg-green-500 active:bg-green-700 active:scale-95">
  Press me
</button>

<!-- Group hover (parent hover affects children) -->
<div class="group cursor-pointer">
  <h3 class="group-hover:text-blue-600">Card Title</h3>
  <p class="group-hover:text-gray-900">Description text</p>
</div>

<!-- First/Last child -->
<ul>
  <li class="first:pt-0 last:pb-0 py-2 border-b last:border-0">Item</li>
</ul>
```

---

## Common Patterns

### Flexbox Layouts

```html
<!-- Center content -->
<div class="flex items-center justify-center min-h-screen">
  <p>Perfectly centered</p>
</div>

<!-- Space between items -->
<header class="flex items-center justify-between px-6 py-4">
  <h1 class="text-xl font-bold">Logo</h1>
  <nav class="flex gap-4">
    <a href="#" class="hover:text-blue-500">Home</a>
    <a href="#" class="hover:text-blue-500">About</a>
  </nav>
</header>

<!-- Vertical stack with gap -->
<div class="flex flex-col gap-4">
  <div class="p-4 bg-white rounded shadow">Card 1</div>
  <div class="p-4 bg-white rounded shadow">Card 2</div>
  <div class="p-4 bg-white rounded shadow">Card 3</div>
</div>
```

### Grid Layouts

```html
<!-- Responsive grid -->
<div
  class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6"
>
  <div class="bg-white p-4 rounded-lg shadow">Product 1</div>
  <div class="bg-white p-4 rounded-lg shadow">Product 2</div>
  <div class="bg-white p-4 rounded-lg shadow">Product 3</div>
  <div class="bg-white p-4 rounded-lg shadow">Product 4</div>
</div>

<!-- Dashboard layout with spanning -->
<div class="grid grid-cols-4 grid-rows-3 gap-4 h-screen p-4">
  <header class="col-span-4 bg-slate-800 rounded-lg p-4">Header</header>
  <aside class="row-span-2 bg-slate-700 rounded-lg p-4">Sidebar</aside>
  <main class="col-span-3 row-span-2 bg-white rounded-lg p-6">
    Main Content
  </main>
</div>
```

### Colors

Tailwind provides a full color palette with shades from 50 (lightest) to 950 (darkest):

```html
<div class="bg-slate-100 text-slate-900">Neutral</div>
<div class="bg-blue-500 text-white">Primary</div>
<div class="bg-red-100 text-red-700 border border-red-300">Error</div>
<div class="bg-green-100 text-green-700">Success</div>
<div class="bg-gradient-to-r from-purple-500 to-pink-500 text-white">
  Gradient
</div>
```

### Typography

```html
<!-- Headings -->
<h1 class="text-4xl font-extrabold tracking-tight text-gray-900">Main Title</h1>
<h2 class="text-2xl font-semibold text-gray-800">Section Title</h2>

<!-- Body text -->
<p class="text-base text-gray-600 leading-relaxed max-w-prose">
  Long paragraph text with comfortable reading width and line height.
</p>

<!-- Truncation -->
<p class="truncate w-48">
  This very long text will be truncated with an ellipsis
</p>

<!-- Line clamping -->
<p class="line-clamp-3">
  This text will be limited to 3 lines and then truncated with an ellipsis at
  the end...
</p>
```

---

## Dark Mode

Tailwind supports dark mode using the `dark:` variant. By default, it uses the `prefers-color-scheme` media query (follows system settings).

### Class-Based Dark Mode (Manual Toggle)

```css
/* src/index.css */
@import "tailwindcss";

@custom-variant dark (&:where(.dark, .dark *));
```

### Usage in Components

```html
<div
  class="bg-white dark:bg-gray-900 text-gray-900 dark:text-gray-100 min-h-screen"
>
  <h1 class="text-2xl font-bold">Welcome</h1>
  <p class="text-gray-600 dark:text-gray-400">This text adapts to dark mode.</p>

  <div
    class="bg-gray-100 dark:bg-gray-800 rounded-lg p-4 border border-gray-200 dark:border-gray-700"
  >
    A card that respects the current theme.
  </div>
</div>
```

### Dark Mode Toggle in React

```jsx
import { useState, useEffect } from "react";

function DarkModeToggle() {
  const [dark, setDark] = useState(() =>
    document.documentElement.classList.contains("dark"),
  );

  useEffect(() => {
    if (dark) {
      document.documentElement.classList.add("dark");
      localStorage.setItem("theme", "dark");
    } else {
      document.documentElement.classList.remove("dark");
      localStorage.setItem("theme", "light");
    }
  }, [dark]);

  return (
    <button
      onClick={() => setDark(!dark)}
      className="p-2 rounded-lg bg-gray-200 dark:bg-gray-700"
    >
      {dark ? "☀️ Light" : "🌙 Dark"}
    </button>
  );
}
```

---

## Component Examples

### Card Component

```jsx
function Card({ title, description, image, tags }) {
  return (
    <div className="bg-white dark:bg-gray-800 rounded-xl shadow-md overflow-hidden hover:shadow-xl transition-shadow duration-300">
      <img src={image} alt={title} className="w-full h-48 object-cover" />
      <div className="p-6">
        <h3 className="text-xl font-semibold text-gray-900 dark:text-white mb-2">
          {title}
        </h3>
        <p className="text-gray-600 dark:text-gray-300 mb-4">{description}</p>
        <div className="flex flex-wrap gap-2">
          {tags.map((tag) => (
            <span
              key={tag}
              className="px-3 py-1 bg-blue-100 dark:bg-blue-900 text-blue-700 dark:text-blue-200 text-sm rounded-full"
            >
              {tag}
            </span>
          ))}
        </div>
      </div>
    </div>
  );
}
```

### Navbar Component

```jsx
import { useState } from "react";

function Navbar() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <nav className="bg-white dark:bg-gray-900 shadow-sm border-b border-gray-200 dark:border-gray-700">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex items-center justify-between h-16">
          {/* Logo */}
          <a href="/" className="text-xl font-bold text-blue-600">
            MyApp
          </a>

          {/* Desktop Nav */}
          <div className="hidden md:flex items-center gap-6">
            <a
              href="#"
              className="text-gray-700 dark:text-gray-200 hover:text-blue-600 transition-colors"
            >
              Home
            </a>
            <a
              href="#"
              className="text-gray-700 dark:text-gray-200 hover:text-blue-600 transition-colors"
            >
              About
            </a>
            <a
              href="#"
              className="text-gray-700 dark:text-gray-200 hover:text-blue-600 transition-colors"
            >
              Contact
            </a>
            <button className="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 transition-colors">
              Sign Up
            </button>
          </div>

          {/* Mobile Hamburger */}
          <button
            className="md:hidden p-2"
            onClick={() => setIsOpen(!isOpen)}
            aria-label="Toggle navigation menu"
          >
            <svg
              className="w-6 h-6"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
            >
              {isOpen ? (
                <path
                  strokeLinecap="round"
                  strokeLinejoin="round"
                  strokeWidth={2}
                  d="M6 18L18 6M6 6l12 12"
                />
              ) : (
                <path
                  strokeLinecap="round"
                  strokeLinejoin="round"
                  strokeWidth={2}
                  d="M4 6h16M4 12h16M4 18h16"
                />
              )}
            </svg>
          </button>
        </div>

        {/* Mobile Menu */}
        {isOpen && (
          <div className="md:hidden py-4 space-y-2 border-t border-gray-200 dark:border-gray-700">
            <a
              href="#"
              className="block py-2 text-gray-700 dark:text-gray-200 hover:text-blue-600"
            >
              Home
            </a>
            <a
              href="#"
              className="block py-2 text-gray-700 dark:text-gray-200 hover:text-blue-600"
            >
              About
            </a>
            <a
              href="#"
              className="block py-2 text-gray-700 dark:text-gray-200 hover:text-blue-600"
            >
              Contact
            </a>
          </div>
        )}
      </div>
    </nav>
  );
}
```

### Button Variants

```jsx
function Button({ variant = "primary", size = "md", children, ...props }) {
  const baseClasses = "inline-flex items-center justify-center font-medium rounded-lg transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2";

  const variants = {
    primary: "bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500",
    secondary: "bg-gray-200 text-gray-800 hover:bg-gray-300 focus:ring-gray-500 dark:bg-gray-700 dark:text-gray-200",
    danger: "bg-red-600 text-white hover:bg-red-700 focus:ring-red-500",
    outline: "border-2 border-blue-600 text-blue-600 hover:bg-blue-50 focus:ring-blue-500 dark:hover:bg-blue-950",
    ghost: "text-gray-700 hover:bg-gray-100 focus:ring-gray-500 dark:text-gray-200 dark:hover:bg-gray-800",
  };

  const sizes = {
    sm: "px-3 py-1.5 text-sm",
    md: "px-4 py-2 text-base",
    lg: "px-6 py-3 text-lg",
  };

  return (
    <button
      className={`${baseClasses} ${variants[variant]} ${sizes[size]}`}
      {...props}
    >
      {children}
    </button>
  );
}

// Usage
<Button variant="primary">Save</Button>
<Button variant="danger" size="sm">Delete</Button>
<Button variant="outline" size="lg">Learn More</Button>
<Button variant="ghost">Cancel</Button>
```

---

## Custom Configuration

Tailwind is fully customizable through its configuration. You can extend the default theme or override it entirely.

### Extending the Theme

```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
export default {
  content: ["./index.html", "./src/**/*.{js,ts,jsx,tsx}"],
  theme: {
    extend: {
      colors: {
        brand: {
          50: "#eff6ff",
          100: "#dbeafe",
          200: "#bfdbfe",
          300: "#93c5fd",
          400: "#60a5fa",
          500: "#3b82f6",
          600: "#2563eb",
          700: "#1d4ed8",
          800: "#1e40af",
          900: "#1e3a8a",
          950: "#172554",
        },
      },
      fontFamily: {
        sans: ["Inter", "system-ui", "sans-serif"],
        mono: ["JetBrains Mono", "monospace"],
      },
      spacing: {
        18: "4.5rem",
        88: "22rem",
        128: "32rem",
      },
      borderRadius: {
        "4xl": "2rem",
      },
      animation: {
        "fade-in": "fadeIn 0.5s ease-in-out",
        "slide-up": "slideUp 0.3s ease-out",
      },
      keyframes: {
        fadeIn: {
          "0%": { opacity: "0" },
          "100%": { opacity: "1" },
        },
        slideUp: {
          "0%": { transform: "translateY(10px)", opacity: "0" },
          "100%": { transform: "translateY(0)", opacity: "1" },
        },
      },
    },
  },
  plugins: [],
};
```

### Using Custom Values

```html
<!-- Custom colors -->
<button class="bg-brand-600 hover:bg-brand-700 text-white">Brand Button</button>

<!-- Custom spacing -->
<div class="p-18 max-w-128">Custom padding and max-width</div>

<!-- Custom animations -->
<div class="animate-fade-in">Fades in on mount</div>
```

### Arbitrary Values

When you need a one-off value not in your theme:

```html
<!-- Arbitrary values with square brackets -->
<div
  class="w-[327px] h-[200px] bg-[#1da1f2] text-[13px] grid-cols-[1fr_2fr_1fr]"
>
  Custom values
</div>

<!-- Arbitrary properties for unsupported CSS -->
<div class="[mask-type:luminance] [--custom-var:12px]">Any CSS property</div>
```

---

## Best Practices

1. **Extract repeated patterns into components** — if you find yourself copying the same 10 classes, make it a React component.

```jsx
// Instead of repeating these classes everywhere
function Badge({ children, color = "blue" }) {
  const colors = {
    blue: "bg-blue-100 text-blue-700",
    green: "bg-green-100 text-green-700",
    red: "bg-red-100 text-red-700",
  };

  return (
    <span
      className={`px-2 py-1 text-xs font-medium rounded-full ${colors[color]}`}
    >
      {children}
    </span>
  );
}
```

2. **Use `@apply` sparingly** — only for base styles you cannot extract into components (global elements, third-party library overrides).

```css
/* Acceptable: global base styles */
@layer base {
  h1 {
    @apply text-3xl font-bold text-gray-900 dark:text-white;
  }
}

/* Avoid: component-level styles (use React components instead) */
/* .btn-primary { @apply bg-blue-500 text-white px-4 py-2 rounded; } */
```

3. **Use consistent spacing** — stick to the default scale (4, 6, 8, 12, 16) rather than arbitrary values.

4. **Mobile-first design** — write base styles for mobile, then add breakpoint prefixes for larger screens.

```html
<!-- Good: mobile-first -->
<div class="text-sm md:text-base lg:text-lg">Responsive text</div>

<!-- Avoid: desktop-first (doesn't work well with Tailwind) -->
```

5. **Group related utilities logically** — layout → sizing → spacing → typography → colors → effects.

```html
<!-- Organized -->
<div
  class="flex items-center gap-4 w-full p-4 text-sm text-gray-700 bg-white rounded-lg shadow-sm"
>
  <!-- Harder to read: random order -->
  <div
    class="shadow-sm text-sm p-4 bg-white flex rounded-lg items-center text-gray-700 gap-4 w-full"
  ></div>
</div>
```

6. **Use the Tailwind CSS IntelliSense VS Code extension** — provides autocomplete, linting, and class sorting.

7. **Install `prettier-plugin-tailwindcss`** — automatically sorts class names into a consistent order.

```bash
npm install -D prettier prettier-plugin-tailwindcss
```

---

## Common Mistakes

| Mistake                              | Why It Fails                                            | Fix                                                                        |
| ------------------------------------ | ------------------------------------------------------- | -------------------------------------------------------------------------- |
| Dynamically constructing class names | `bg-${color}-500` is not detectable by Tailwind's purge | Use complete class names: `color === "red" ? "bg-red-500" : "bg-blue-500"` |
| Overusing `@apply`                   | Defeats the purpose of utility-first; CSS file grows    | Extract React components instead                                           |
| Not setting `content` paths          | Classes are purged in production                        | Ensure all template files are listed in `content`                          |
| Fighting specificity                 | Adding `!important` everywhere                          | Use Tailwind's `!` prefix: `!text-red-500` (only as last resort)           |
| Ignoring dark mode from the start    | Retrofitting dark mode is painful                       | Add `dark:` variants as you build                                          |

---

## Summary

- **Tailwind CSS** is a utility-first framework — you compose styles using small, single-purpose classes directly in markup.
- **No custom CSS files needed** for most projects — utilities cover layout, spacing, typography, colors, responsive design, and animations.
- **Responsive design** uses mobile-first breakpoint prefixes (`sm:`, `md:`, `lg:`, `xl:`, `2xl:`).
- **State variants** (`hover:`, `focus:`, `active:`, `group-hover:`, `dark:`) handle interactivity without writing CSS.
- **Dark mode** is built-in with the `dark:` variant — use class-based toggling for manual control.
- **Custom configuration** lets you extend colors, spacing, fonts, and animations to match your design system.
- **Production builds are tiny** — Tailwind purges unused utilities, resulting in CSS files under 10KB.
- **Prefer React components over `@apply`** for extracting repeated patterns — this is the utility-first way.
