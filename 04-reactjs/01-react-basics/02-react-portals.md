# React Portals

## What Are Portals?

Portals provide a way to **render children into a DOM node that exists outside the parent component's DOM hierarchy**. Normally, React components render into the nearest parent DOM element. Portals break this rule — they let you teleport a component's output to a completely different place in the DOM tree.

**Analogy:** Imagine your component is a letter. Normally it goes into the mailbox (parent DOM node) right next to it. A portal is like a wormhole — the letter appears in a completely different mailbox, but the sender still knows about it.

---

## Why Portals?

| Problem                                  | Portal Solution                        |
| ---------------------------------------- | -------------------------------------- |
| Modal gets clipped by `overflow: hidden` | Renders at document body level         |
| Tooltip cut off by parent container      | Escapes parent stacking context        |
| Dropdown menu hidden behind siblings     | Renders above all other content        |
| z-index wars with nested components      | Removes from parent z-index context    |
| Need to render into a different DOM tree | Portal mounts anywhere in the document |

---

## `createPortal` API

```jsx
import { createPortal } from 'react-dom';

// Syntax
createPortal(child, domNode, key?)
```

- `child` — Any renderable React content (JSX, string, fragment).
- `domNode` — The target DOM element where the child will be mounted.
- `key` — Optional unique key for the portal.

---

## Basic Example — Modal

### HTML Setup

```html
<!-- index.html -->
<body>
  <div id="root"></div>
  <div id="modal-root"></div>
  <!-- Portal target -->
</body>
```

### Modal Component

```jsx
import { createPortal } from "react-dom";

function Modal({ children, isOpen, onClose }) {
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
    document.getElementById("modal-root"),
  );
}

// Usage
function App() {
  const [showModal, setShowModal] = useState(false);

  return (
    <div style={{ overflow: "hidden", height: "200px" }}>
      <button onClick={() => setShowModal(true)}>Open Modal</button>
      <Modal isOpen={showModal} onClose={() => setShowModal(false)}>
        <h2>I escape overflow:hidden!</h2>
        <p>This modal renders at #modal-root, not inside the parent div.</p>
      </Modal>
    </div>
  );
}
```

Even though `Modal` is written inside a div with `overflow: hidden`, it renders at `#modal-root` — completely outside that constraint.

---

## Tooltip Example

```jsx
import { createPortal } from "react-dom";
import { useState, useRef, useEffect } from "react";

function Tooltip({ targetRef, children }) {
  const [position, setPosition] = useState({ top: 0, left: 0 });

  useEffect(() => {
    if (targetRef.current) {
      const rect = targetRef.current.getBoundingClientRect();
      setPosition({
        top: rect.bottom + window.scrollY + 8,
        left: rect.left + window.scrollX,
      });
    }
  }, [targetRef]);

  return createPortal(
    <div
      className="tooltip"
      style={{ position: "absolute", top: position.top, left: position.left }}
    >
      {children}
    </div>,
    document.body,
  );
}

function App() {
  const buttonRef = useRef(null);
  const [showTooltip, setShowTooltip] = useState(false);

  return (
    <>
      <button
        ref={buttonRef}
        onMouseEnter={() => setShowTooltip(true)}
        onMouseLeave={() => setShowTooltip(false)}
      >
        Hover me
      </button>
      {showTooltip && (
        <Tooltip targetRef={buttonRef}>
          This tooltip escapes any overflow container!
        </Tooltip>
      )}
    </>
  );
}
```

---

## Event Bubbling Through Portals

A critical behavior: **events bubble through the React tree, not the DOM tree**. Even though a portal renders somewhere else in the DOM, events still bubble up through the React component hierarchy.

```jsx
function Parent() {
  // This onClick WILL fire when clicking inside the portal!
  return (
    <div onClick={() => console.log("Parent clicked!")}>
      <h1>Parent Component</h1>
      <PortalChild />
    </div>
  );
}

function PortalChild() {
  return createPortal(
    <button onClick={() => console.log("Button clicked!")}>
      Click Me (rendered in portal)
    </button>,
    document.getElementById("modal-root"),
  );
}

// Clicking the button logs:
// "Button clicked!"
// "Parent clicked!" — event bubbled through React tree!
```

This means context providers, error boundaries, and event handlers in parent components work seamlessly with portals.

---

## Accessibility Considerations

Portals require extra attention for accessibility since the DOM structure no longer matches the visual/logical structure.

```jsx
function AccessibleModal({ isOpen, onClose, title, children }) {
  const modalRef = useRef(null);

  // Trap focus inside modal
  useEffect(() => {
    if (isOpen && modalRef.current) {
      const focusableElements = modalRef.current.querySelectorAll(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])',
      );
      const firstElement = focusableElements[0];
      firstElement?.focus();

      // Return focus when modal closes
      const previouslyFocused = document.activeElement;
      return () => previouslyFocused?.focus();
    }
  }, [isOpen]);

  // Close on Escape key
  useEffect(() => {
    const handleEscape = (e) => {
      if (e.key === "Escape") onClose();
    };
    document.addEventListener("keydown", handleEscape);
    return () => document.removeEventListener("keydown", handleEscape);
  }, [onClose]);

  if (!isOpen) return null;

  return createPortal(
    <div
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
      ref={modalRef}
    >
      <h2 id="modal-title">{title}</h2>
      {children}
      <button onClick={onClose}>Close</button>
    </div>,
    document.getElementById("modal-root"),
  );
}
```

Key accessibility requirements:

- `role="dialog"` and `aria-modal="true"` on the modal container.
- `aria-labelledby` pointing to the modal title.
- Focus trapping — keyboard users should not tab outside the modal.
- Return focus to the triggering element when the modal closes.
- Escape key should close the modal.

---

## Best Practices

1. **Create a dedicated DOM node** for portals (`#modal-root`, `#tooltip-root`) — do not portal into `#root`.
2. **Clean up on unmount** — React handles this automatically, but ensure event listeners are cleaned up.
3. **Handle accessibility** — focus management, ARIA attributes, and keyboard interactions are your responsibility.
4. **Stop propagation when needed** — since events bubble through React tree, use `e.stopPropagation()` to prevent unexpected parent handlers from firing.
5. **Use portals sparingly** — only when you genuinely need to escape the DOM hierarchy (overflow, z-index issues).
6. **Server-side rendering** — `document.getElementById()` does not exist on the server. Guard with `typeof document !== 'undefined'` or use `useEffect`.

---

## Common Mistakes

| Mistake                                | Why It's Wrong                                 | Fix                                                    |
| -------------------------------------- | ---------------------------------------------- | ------------------------------------------------------ |
| Portaling everything                   | Breaks DOM semantics unnecessarily             | Only portal when escaping overflow/stacking contexts   |
| Forgetting focus management in modals  | Screen reader users get lost                   | Trap focus and return it on close                      |
| Not handling Escape key                | Users expect Escape to close overlays          | Add `keydown` listener for `Escape`                    |
| Assuming events don't reach parent     | Events bubble through React tree, not DOM tree | Use `e.stopPropagation()` when needed                  |
| Using `document.body` as target always | Can conflict with other libraries              | Create dedicated portal root elements                  |
| No null check for portal target        | Crashes if DOM element doesn't exist           | Guard with conditional: `domNode && createPortal(...)` |
| Forgetting `aria-modal` on dialogs     | Assistive tech doesn't know it's a modal       | Add `role="dialog"` and `aria-modal="true"`            |

---

## Summary

- `createPortal(child, domNode)` renders React children into any DOM node outside the parent hierarchy.
- Primary use cases: modals, tooltips, dropdowns, and anything that needs to escape `overflow: hidden` or z-index stacking contexts.
- Event bubbling still follows the React component tree — not the DOM tree — so parent handlers still fire.
- Accessibility requires manual work: focus trapping, ARIA attributes, keyboard handling, and focus restoration.
- Always create dedicated portal target elements and guard against SSR environments.
