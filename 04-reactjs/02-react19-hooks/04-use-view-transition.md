# useViewTransition (React 19)

## What Are View Transitions?

View Transitions are a browser API that enables **smooth animated transitions between UI states**. Instead of content snapping instantly from one state to another, the browser captures a "before" snapshot and a "after" snapshot, then animates between them with crossfade or custom CSS animations.

**Analogy:** Think of a magic trick where one card transforms into another. The View Transition API captures the "before" card and "after" card, then creates a smooth morph between them — all handled by the browser.

---

## Why View Transitions in React?

| Without View Transitions                | With View Transitions                  |
| --------------------------------------- | -------------------------------------- |
| Page changes snap instantly             | Smooth crossfade between pages         |
| List items appear/disappear abruptly    | Items animate in/out gracefully        |
| Layout shifts feel jarring              | Elements morph to new positions        |
| Custom animation libraries needed       | Browser handles it natively with CSS   |
| Complex state management for animations | Declarative with `startViewTransition` |

---

## The `useViewTransition` Hook

React 19 provides `useViewTransition` to integrate the View Transitions API with React's rendering system. It wraps state updates in `document.startViewTransition()` so the browser animates between the old and new DOM states.

```jsx
import { useViewTransition } from "react";

function App() {
  const { startViewTransition } = useViewTransition();

  function handleNavigate(newPage) {
    startViewTransition(() => {
      setCurrentPage(newPage);
    });
  }

  return <div>{/* content */}</div>;
}
```

**How it works:**

1. You call `startViewTransition` with a callback that updates state.
2. React captures the current DOM as the "old" snapshot.
3. React renders the new state.
4. The browser animates from old snapshot to new DOM.

---

## `document.startViewTransition` API (Browser Primitive)

The underlying browser API that React's hook wraps:

```jsx
// Manual usage (without React hook)
document.startViewTransition(() => {
  // Update the DOM
  root.innerHTML = newContent;
});
```

The browser:

1. Takes a screenshot of the current state (old snapshot).
2. Runs your callback (DOM updates).
3. Takes a screenshot of the new state (new snapshot).
4. Animates from old to new using CSS pseudo-elements.

---

## CSS `view-transition-name`

To animate **specific elements** between states (not just a full-page crossfade), assign `view-transition-name` in CSS:

```css
/* Each element with a unique view-transition-name will animate independently */
.page-title {
  view-transition-name: page-title;
}

.hero-image {
  view-transition-name: hero-image;
}

.content-area {
  view-transition-name: main-content;
}
```

```jsx
function PageHeader({ title }) {
  return <h1 style={{ viewTransitionName: "page-title" }}>{title}</h1>;
}
```

**Rule:** Each `view-transition-name` must be unique on the page at any given time. Two elements cannot share the same transition name simultaneously.

---

## Customizing Animations with CSS

```css
/* Default crossfade for all transitions */
::view-transition-old(root) {
  animation: fade-out 0.3s ease-out;
}

::view-transition-new(root) {
  animation: fade-in 0.3s ease-in;
}

/* Custom animation for specific named elements */
::view-transition-old(hero-image) {
  animation: scale-down 0.4s ease-out;
}

::view-transition-new(hero-image) {
  animation: scale-up 0.4s ease-in;
}

/* Slide animation for page content */
::view-transition-old(main-content) {
  animation: slide-out-left 0.3s ease-out;
}

::view-transition-new(main-content) {
  animation: slide-in-right 0.3s ease-in;
}

@keyframes slide-out-left {
  to {
    transform: translateX(-100%);
    opacity: 0;
  }
}

@keyframes slide-in-right {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
}
```

---

## Practical Example — Tab Navigation

```jsx
import { useState } from "react";

function TabPanel() {
  const [activeTab, setActiveTab] = useState("home");

  function switchTab(tab) {
    if (!document.startViewTransition) {
      setActiveTab(tab); // Fallback for unsupported browsers
      return;
    }

    document.startViewTransition(() => {
      setActiveTab(tab);
    });
  }

  return (
    <div>
      <nav>
        <button onClick={() => switchTab("home")}>Home</button>
        <button onClick={() => switchTab("about")}>About</button>
        <button onClick={() => switchTab("contact")}>Contact</button>
      </nav>
      <div style={{ viewTransitionName: "tab-content" }}>
        {activeTab === "home" && <HomePage />}
        {activeTab === "about" && <AboutPage />}
        {activeTab === "contact" && <ContactPage />}
      </div>
    </div>
  );
}
```

---

## With React Router

```jsx
import { useNavigate } from "react-router-dom";

function NavLink({ to, children }) {
  const navigate = useNavigate();

  function handleClick(e) {
    e.preventDefault();

    if (!document.startViewTransition) {
      navigate(to);
      return;
    }

    document.startViewTransition(() => {
      navigate(to);
    });
  }

  return (
    <a href={to} onClick={handleClick}>
      {children}
    </a>
  );
}

// Usage
function Header() {
  return (
    <nav>
      <NavLink to="/">Home</NavLink>
      <NavLink to="/products">Products</NavLink>
      <NavLink to="/about">About</NavLink>
    </nav>
  );
}
```

### Shared Element Transitions (Card → Detail Page)

```jsx
// Product card on list page
function ProductCard({ product }) {
  return (
    <NavLink to={`/products/${product.id}`}>
      <div className="product-card">
        <img
          src={product.image}
          alt={product.name}
          style={{ viewTransitionName: `product-image-${product.id}` }}
        />
        <h3 style={{ viewTransitionName: `product-title-${product.id}` }}>
          {product.name}
        </h3>
      </div>
    </NavLink>
  );
}

// Product detail page — same view-transition-names create morph effect
function ProductDetail({ product }) {
  return (
    <div>
      <img
        src={product.image}
        alt={product.name}
        style={{ viewTransitionName: `product-image-${product.id}` }}
      />
      <h1 style={{ viewTransitionName: `product-title-${product.id}` }}>
        {product.name}
      </h1>
      <p>{product.description}</p>
    </div>
  );
}
```

The browser detects matching `view-transition-name` values and morphs the element from its old position/size to its new position/size.

---

## Feature Detection & Fallback

Always check for browser support — View Transitions are not available everywhere yet:

```jsx
function useTransitionNavigate() {
  const navigate = useNavigate();

  function transitionTo(path) {
    if (!document.startViewTransition) {
      navigate(path); // Instant navigation as fallback
      return;
    }

    document.startViewTransition(() => {
      navigate(path);
    });
  }

  return transitionTo;
}
```

---

## Best Practices

1. **Always feature-detect** — check `document.startViewTransition` exists before using.
2. **Use unique `view-transition-name` values** — duplicates on the same page cause errors.
3. **Keep animations short** — 200-400ms feels snappy. Longer feels sluggish.
4. **Prefer CSS for animation definitions** — use `::view-transition-old` and `::view-transition-new` pseudo-elements.
5. **Respect reduced motion** — wrap animations in `@media (prefers-reduced-motion: no-preference)`.
6. **Dynamic names for lists** — use element IDs in transition names: `product-image-${id}`.
7. **Don't overuse** — animate meaningful transitions (page changes, detail views), not every state change.

---

## Common Mistakes

| Mistake                                      | Why It's Wrong                                     | Fix                                                     |
| -------------------------------------------- | -------------------------------------------------- | ------------------------------------------------------- |
| Not checking browser support                 | Crashes in unsupported browsers                    | Feature-detect with `if (document.startViewTransition)` |
| Duplicate `view-transition-name` on page     | Browser can't match elements — animation breaks    | Ensure names are unique at any moment                   |
| Animations too long (> 500ms)                | Feels slow, blocks user interaction                | Keep transitions 200-400ms                              |
| Ignoring `prefers-reduced-motion`            | Accessibility violation for motion-sensitive users | Disable animations when reduced motion is set           |
| Applying to every state change               | Overwhelming, distracting UX                       | Only transition meaningful navigation/layout changes    |
| Using inline styles for all transition names | Hard to maintain                                   | Use CSS classes with `view-transition-name`             |

---

## Summary

- View Transitions enable smooth, animated transitions between UI states using browser-native APIs.
- React 19's `useViewTransition` hook wraps state updates in `document.startViewTransition()` for seamless integration.
- Assign `view-transition-name` in CSS/inline styles to animate specific elements between states.
- Matching `view-transition-name` values across pages create shared element morph animations (card → detail page).
- Customize animations with `::view-transition-old()` and `::view-transition-new()` CSS pseudo-elements.
- Always feature-detect, respect reduced motion preferences, and keep animations brief (200-400ms).
- Works with React Router for page transitions and with any state change that alters visible DOM content.
