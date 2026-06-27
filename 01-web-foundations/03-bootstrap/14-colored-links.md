# Bootstrap Colored Links

## What Are Colored Links?

Bootstrap provides utility classes for styling links with theme colors. These colored links automatically adjust their shade on hover and focus states, providing visual feedback without writing custom CSS.

---

## Available Classes

```html
<a href="#" class="link-primary">Primary link</a>
<a href="#" class="link-secondary">Secondary link</a>
<a href="#" class="link-success">Success link</a>
<a href="#" class="link-danger">Danger link</a>
<a href="#" class="link-warning">Warning link</a>
<a href="#" class="link-info">Info link</a>
<a href="#" class="link-light">Light link</a>
<a href="#" class="link-dark">Dark link</a>
<a href="#" class="link-body-emphasis">Body emphasis link</a>
```

Each class:

- Sets the link color to the corresponding theme color.
- Darkens the color on `:hover` and `:focus` (using `color-mix()` or shade functions).
- Includes the underline by default.

---

## Link Opacity

Control link opacity with utility classes:

```html
<a href="#" class="link-primary link-opacity-10">10% opacity</a>
<a href="#" class="link-primary link-opacity-25">25% opacity</a>
<a href="#" class="link-primary link-opacity-50">50% opacity</a>
<a href="#" class="link-primary link-opacity-75">75% opacity</a>
<a href="#" class="link-primary link-opacity-100">100% opacity</a>
```

Hover variants:

```html
<a href="#" class="link-primary link-opacity-50 link-opacity-100-hover">
  Fades in on hover
</a>
```

---

## Link Underline Utilities

### Underline Color

```html
<a href="#" class="link-underline-primary">Primary underline</a>
<a href="#" class="link-underline-danger">Danger underline</a>
```

### Underline Opacity

```html
<a href="#" class="link-underline-opacity-0">No underline</a>
<a href="#" class="link-underline-opacity-50">50% underline</a>
<a href="#" class="link-underline-opacity-100">Full underline</a>
```

### Underline Offset

```html
<a href="#" class="link-offset-1">Offset 1</a>
<a href="#" class="link-offset-2">Offset 2</a>
<a href="#" class="link-offset-3">Offset 3</a>
```

---

## Combining Utilities

```html
<a
  href="#"
  class="link-success link-offset-2 link-underline-opacity-25 link-underline-opacity-100-hover"
>
  Styled success link
</a>
```

This creates a green link with a subtle underline that becomes fully visible on hover.

---

## Practical Use Cases

```html
<!-- Navigation-style links -->
<p>
  Read our
  <a href="/terms" class="link-dark link-offset-2 link-underline-opacity-25"
    >Terms of Service</a
  >
  and
  <a href="/privacy" class="link-dark link-offset-2 link-underline-opacity-25"
    >Privacy Policy</a
  >.
</p>

<!-- Status-colored links -->
<ul class="list-unstyled">
  <li><a href="#" class="link-success">✓ Completed tasks</a></li>
  <li><a href="#" class="link-warning">⚠ Pending review</a></li>
  <li><a href="#" class="link-danger">✕ Failed items</a></li>
</ul>
```

---

## Summary

- Use `link-{color}` classes to theme links with Bootstrap colors.
- Control opacity with `link-opacity-{value}` and hover variants.
- Adjust underlines with `link-underline-{color}`, `link-underline-opacity-{value}`, and `link-offset-{value}`.
- Combine utility classes for fine-grained control without custom CSS.
- These utilities maintain accessible hover/focus states automatically.
