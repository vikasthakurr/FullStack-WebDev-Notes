# Bootstrap 5 Buttons

## What Is It

Bootstrap buttons are pre-styled interactive elements that provide visual consistency and contextual meaning through color variants, sizes, and states. They work on `<button>`, `<a>`, and `<input>` elements, giving you a unified look regardless of the underlying HTML tag.

**Analogy:** Buttons are like traffic signals for your interface. Green (success) says "go ahead," red (danger) says "be careful," and the size of the signal tells you how important the action is. Bootstrap gives you a full set of these signals, ready to use.

## Why It Matters

- Buttons are the primary way users take action in any web application.
- Consistent styling builds user trust and reduces cognitive load.
- Contextual colors communicate intent (delete = danger, save = success).
- Built-in states (hover, focus, active, disabled) ensure proper interaction feedback.

---

## Button Variants

### Solid Buttons

```html
<button class="btn btn-primary">Primary</button>
<button class="btn btn-secondary">Secondary</button>
<button class="btn btn-success">Success</button>
<button class="btn btn-danger">Danger</button>
<button class="btn btn-warning">Warning</button>
<button class="btn btn-info">Info</button>
<button class="btn btn-light">Light</button>
<button class="btn btn-dark">Dark</button>
<button class="btn btn-link">Link</button>
```

### Outline Buttons

Outline buttons have transparent backgrounds with colored borders. They are less visually heavy -- ideal for secondary actions.

```html
<button class="btn btn-outline-primary">Primary</button>
<button class="btn btn-outline-secondary">Secondary</button>
<button class="btn btn-outline-success">Success</button>
<button class="btn btn-outline-danger">Danger</button>
<button class="btn btn-outline-warning">Warning</button>
<button class="btn btn-outline-info">Info</button>
<button class="btn btn-outline-light">Light</button>
<button class="btn btn-outline-dark">Dark</button>
```

**When to use outline vs solid:** Use solid for primary/main actions (Submit, Save) and outline for secondary actions (Cancel, Back) to establish visual hierarchy.

---

## Button Tags

Bootstrap button classes work on multiple HTML elements:

```html
<!-- Standard button -->
<button class="btn btn-primary" type="button">Button</button>

<!-- Anchor styled as button -->
<a class="btn btn-primary" href="/dashboard" role="button">Link Button</a>

<!-- Input as button -->
<input class="btn btn-primary" type="submit" value="Submit" />
<input class="btn btn-secondary" type="reset" value="Reset" />
```

Use `<button>` for actions, `<a>` for navigation styled as buttons, and `<input>` inside forms.

---

## Button Sizes

```html
<!-- Large button -->
<button class="btn btn-primary btn-lg">Large Button</button>

<!-- Default size (no extra class needed) -->
<button class="btn btn-primary">Default Button</button>

<!-- Small button -->
<button class="btn btn-primary btn-sm">Small Button</button>
```

### Block Buttons (Full Width)

In Bootstrap 5, block buttons use the grid/flex utilities instead of the old `.btn-block` class:

```html
<!-- Full-width button using d-grid -->
<div class="d-grid gap-2">
  <button class="btn btn-primary" type="button">Full Width Button</button>
  <button class="btn btn-secondary" type="button">Another Full Width</button>
</div>

<!-- Full-width only on mobile, inline on larger screens -->
<div class="d-grid gap-2 d-md-flex">
  <button class="btn btn-primary" type="button">Responsive Block</button>
  <button class="btn btn-secondary" type="button">Another</button>
</div>
```

---

## Disabled State

```html
<!-- Disabled button -->
<button class="btn btn-primary" disabled>Disabled Button</button>

<!-- Disabled anchor (requires additional attributes) -->
<a
  class="btn btn-primary disabled"
  href="#"
  role="button"
  tabindex="-1"
  aria-disabled="true"
>
  Disabled Link Button
</a>
```

For `<a>` elements, add `.disabled` class, `tabindex="-1"`, and `aria-disabled="true"` since `<a>` tags do not support the `disabled` attribute natively.

---

## Button Groups

Group related buttons together into a toolbar-like component.

```html
<!-- Basic button group -->
<div class="btn-group" role="group" aria-label="Basic example">
  <button type="button" class="btn btn-primary">Left</button>
  <button type="button" class="btn btn-primary">Middle</button>
  <button type="button" class="btn btn-primary">Right</button>
</div>

<!-- Mixed styles -->
<div class="btn-group" role="group">
  <button type="button" class="btn btn-danger">Delete</button>
  <button type="button" class="btn btn-warning">Archive</button>
  <button type="button" class="btn btn-success">Approve</button>
</div>

<!-- Button group sizing -->
<div class="btn-group btn-group-lg" role="group">
  <button type="button" class="btn btn-outline-primary">Large</button>
  <button type="button" class="btn btn-outline-primary">Group</button>
</div>

<div class="btn-group btn-group-sm" role="group">
  <button type="button" class="btn btn-outline-secondary">Small</button>
  <button type="button" class="btn btn-outline-secondary">Group</button>
</div>
```

### Vertical Button Group

```html
<div class="btn-group-vertical" role="group" aria-label="Vertical group">
  <button type="button" class="btn btn-primary">Top</button>
  <button type="button" class="btn btn-primary">Middle</button>
  <button type="button" class="btn btn-primary">Bottom</button>
</div>
```

### Button Toolbar

Combine multiple button groups into a toolbar.

```html
<div class="btn-toolbar" role="toolbar" aria-label="Toolbar">
  <div class="btn-group me-2" role="group">
    <button type="button" class="btn btn-primary">1</button>
    <button type="button" class="btn btn-primary">2</button>
    <button type="button" class="btn btn-primary">3</button>
  </div>
  <div class="btn-group me-2" role="group">
    <button type="button" class="btn btn-secondary">4</button>
    <button type="button" class="btn btn-secondary">5</button>
  </div>
</div>
```

---

## Toggle Buttons

Buttons that maintain an active/pressed state.

```html
<!-- Single toggle button -->
<button type="button" class="btn btn-primary" data-bs-toggle="button">
  Toggle Me
</button>

<!-- Active by default -->
<button
  type="button"
  class="btn btn-primary active"
  data-bs-toggle="button"
  aria-pressed="true"
>
  Active Toggle
</button>

<!-- Toggle with checkbox behavior -->
<input type="checkbox" class="btn-check" id="btn-check" autocomplete="off" />
<label class="btn btn-outline-primary" for="btn-check">Toggle Checkbox</label>

<!-- Toggle with radio behavior -->
<div class="btn-group" role="group">
  <input
    type="radio"
    class="btn-check"
    name="options"
    id="option1"
    autocomplete="off"
    checked
  />
  <label class="btn btn-outline-primary" for="option1">Option 1</label>

  <input
    type="radio"
    class="btn-check"
    name="options"
    id="option2"
    autocomplete="off"
  />
  <label class="btn btn-outline-primary" for="option2">Option 2</label>

  <input
    type="radio"
    class="btn-check"
    name="options"
    id="option3"
    autocomplete="off"
  />
  <label class="btn btn-outline-primary" for="option3">Option 3</label>
</div>
```

---

## Loading State Pattern

Bootstrap does not have a built-in loading state, but here is the standard pattern:

```html
<button class="btn btn-primary" type="button" id="loadBtn">Submit</button>

<script>
  const btn = document.getElementById("loadBtn");
  btn.addEventListener("click", () => {
    btn.disabled = true;
    btn.innerHTML = `
      <span class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
      Loading...
    `;

    // Simulate async operation
    setTimeout(() => {
      btn.disabled = false;
      btn.innerHTML = "Submit";
    }, 2000);
  });
</script>
```

### Loading with Spinner Variants

```html
<!-- Growing spinner -->
<button class="btn btn-primary" type="button" disabled>
  <span
    class="spinner-grow spinner-grow-sm"
    role="status"
    aria-hidden="true"
  ></span>
  Processing...
</button>

<!-- Spinner only (no text) -->
<button class="btn btn-primary" type="button" disabled>
  <span
    class="spinner-border spinner-border-sm"
    role="status"
    aria-hidden="true"
  ></span>
  <span class="visually-hidden">Loading...</span>
</button>
```

---

## Button with Dropdown

```html
<div class="btn-group">
  <button
    type="button"
    class="btn btn-primary dropdown-toggle"
    data-bs-toggle="dropdown"
    aria-expanded="false"
  >
    Actions
  </button>
  <ul class="dropdown-menu">
    <li><a class="dropdown-item" href="#">Edit</a></li>
    <li><a class="dropdown-item" href="#">Duplicate</a></li>
    <li><hr class="dropdown-divider" /></li>
    <li><a class="dropdown-item text-danger" href="#">Delete</a></li>
  </ul>
</div>
```

---

## Best Practices

1. Use `.btn-primary` for the main action on a page; avoid multiple primary buttons in the same view.
2. Pair solid buttons with outline buttons to create clear visual hierarchy.
3. Always add `type="button"` to prevent accidental form submissions.
4. Use `aria-label` or visible text -- never rely on color alone to communicate meaning.
5. Disable buttons during async operations and show a spinner for feedback.

## Common Mistakes

| Mistake                                       | Why It Is Wrong                                  | Fix                                          |
| --------------------------------------------- | ------------------------------------------------ | -------------------------------------------- |
| Multiple primary buttons in one section       | Confuses users about what the main action is     | Use one primary, others as secondary/outline |
| Missing `type="button"` on non-submit buttons | Inside forms, buttons default to `type="submit"` | Explicitly set `type="button"`               |
| Using color alone to convey meaning           | Colorblind users cannot distinguish variants     | Add text labels or icons                     |
| Forgetting `role="group"` on button groups    | Screen readers cannot announce the group         | Always add role and aria-label               |

---

## Summary

Bootstrap buttons provide a complete system for interactive elements: eight color variants, outline alternatives, three sizes, block-level layout, groups, toggles, and loading states. The key to effective button design is hierarchy -- one primary action stands out, secondary actions recede, and dangerous actions warn through color. Pair these with proper accessibility attributes and you have buttons that look good and work for everyone.
