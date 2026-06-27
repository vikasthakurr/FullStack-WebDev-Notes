# React Portals

## What Are Portals?

Portals let you render a component's children into a **different DOM node** that exists outside the parent component's DOM hierarchy. The component still lives in the React tree (events bubble up, context works), but its HTML is rendered elsewhere in the actual DOM.

**Analogy:** Imagine a TV studio (your component tree). Portals are like a live broadcast — the news anchor is in the studio (React tree), but their face appears on millions of TVs across the country (a different DOM node). They're still part of the studio team, but their output appears elsewhere.

---

## Why Portals?

Without portals, everything renders inside `<div id="root">`. This causes problems when:

| Problem                    | Cause                                               | Portal Solution                       |
| -------------------------- | --------------------------------------------------- | ------------------------------------- |
| Modal behind other content | Parent has `overflow: hidden` or `z-index` stacking | Render modal at document body level   |
| Tooltip gets clipped       | Container has `overflow: hidden`                    | Render tooltip outside the container  |
| Dropdown cut off           | Parent's CSS restricts visibility                   | Render dropdown at a higher DOM level |

---

## createPortal Syntax

```jsx
import { createPortal } from "react-dom";

function MyComponent() {
  return createPortal(
    <div>This renders outside the parent DOM!</div>, // What to render
    document.getElementById("portal-root"), // Where to render it
  );
}
```

### Setup: Add a Portal Target in HTML

```html
<!-- index.html -->
<body>
  <div id="root"></div>
  <!-- Normal React app -->
  <div id="portal-root"></div>
  <!-- Portal destination -->
</body>
```

---

## Modal Example

```jsx
import { createPortal } from "react-dom";

function Modal({ isOpen, onClose, children }) {
  if (!isOpen) return null;

  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={(e) => e.stopPropagation()}>
        <button className="modal-close" onClick={onClose}>
          ×
        </button>
        {children}
      </div>
    </div>,
    document.getElementById("portal-root"),
  );
}

// Usage
function App() {
  const [showModal, setShowModal] = useState(false);

  return (
    <div style={{ overflow: "hidden", height: "200px" }}>
      <button onClick={() => setShowModal(true)}>Open Modal</button>

      {/* Even though parent has overflow:hidden, modal is NOT clipped */}
      <Modal isOpen={showModal} onClose={() => setShowModal(false)}>
        <h2>Delete Item?</h2>
        <p>This action cannot be undone.</p>
        <button onClick={() => setShowModal(false)}>Cancel</button>
        <button>Confirm Delete</button>
      </Modal>
    </div>
  );
}
```

---

## Event Bubbling Works Through Portals

Even though the portal renders in a different DOM location, React events still bubble up through the **React tree** (not the DOM tree):

```jsx
function Parent() {
  // This catches clicks from the portal child!
  const handleClick = () => console.log("Parent caught click from portal");

  return (
    <div onClick={handleClick}>
      <h1>Parent Component</h1>
      <PortalChild />
    </div>
  );
}

function PortalChild() {
  return createPortal(
    <button>Click Me (I'm in a portal)</button>,
    document.getElementById("portal-root"),
  );
}
// Clicking the button logs "Parent caught click from portal"
```

---

## Common Use Cases

- **Modals / Dialogs** — render above all content without z-index issues
- **Tooltips** — avoid clipping from parent containers
- **Dropdowns / Popovers** — position freely without overflow constraints
- **Notification toasts** — render at a fixed position on the page
- **Full-screen overlays** — loading screens, image lightboxes

---

## Best Practices

1. **Manage focus** — when a modal opens, focus the first interactive element inside it. Return focus on close.
2. **Add `aria-modal="true"`** and proper ARIA attributes for accessibility.
3. **Handle Escape key** — close portals on `Escape` keypress.
4. **Prevent body scroll** when a modal is open (`document.body.style.overflow = 'hidden'`).
5. **Clean up** the portal target if created dynamically.

---

## Summary

- `createPortal(jsx, domNode)` renders children into a DOM node outside the parent hierarchy.
- Useful for **modals, tooltips, dropdowns** that need to escape CSS constraints like `overflow: hidden` or `z-index`.
- React events **still bubble through the React tree**, not the DOM tree — context and event handlers work as expected.
- The component is still logically part of its parent in React — only the HTML output goes elsewhere.
