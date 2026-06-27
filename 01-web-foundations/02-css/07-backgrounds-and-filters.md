# Backgrounds and Filters in CSS

## What They Are

Background properties control what appears behind an element's content — colors, images, gradients, and how they are positioned. CSS filters modify the visual rendering of an element itself, applying effects like blur, brightness adjustments, and grayscale conversions.

**Analogy:** Backgrounds are like wallpaper in a room — they set the scene behind everything else. Filters are like putting on sunglasses — they change how you _see_ the element itself.

---

## Why They Matter

- Backgrounds are essential for visual design — hero sections, cards, overlays, and patterns.
- Gradients eliminate the need for image files, reducing page weight and improving performance.
- Filters enable effects that previously required Photoshop or canvas manipulation.
- Mastering these properties lets you create sophisticated visual designs with pure CSS.

---

## Background Properties

### `background-color`

Sets a solid color behind the element's content and padding area.

```css
.card {
  background-color: #ffffff;
}

.overlay {
  background-color: rgba(0, 0, 0, 0.6); /* semi-transparent */
}
```

---

### `background-image`

Places an image (or gradient) behind the content.

```css
.hero {
  background-image: url("/images/hero-banner.jpg");
}

/* Multiple backgrounds (layered, first is on top) */
.hero {
  background-image:
    linear-gradient(rgba(0, 0, 0, 0.4), rgba(0, 0, 0, 0.4)),
    url("/images/hero-banner.jpg");
}
```

---

### `background-size`

Controls how the background image is scaled.

```css
.hero {
  background-size: cover; /* scales to fill container, may crop */
  background-size: contain; /* scales to fit inside container, no crop */
  background-size: 100% auto; /* explicit width, auto height */
  background-size: 200px 300px; /* exact dimensions */
}
```

- **`cover`** — image fills the container completely; some parts may be clipped. Most common for hero sections.
- **`contain`** — entire image is visible; may leave empty space.

---

### `background-position`

Sets where the background image is anchored.

```css
.hero {
  background-position: center center; /* horizontally and vertically centered */
  background-position: top right;
  background-position: 50% 30%; /* 50% from left, 30% from top */
  background-position: 20px 40px; /* offset from top-left */
}
```

---

### `background-repeat`

Controls whether and how the image tiles.

```css
.pattern {
  background-repeat: repeat; /* default — tiles both directions */
  background-repeat: no-repeat; /* single instance */
  background-repeat: repeat-x; /* tiles horizontally only */
  background-repeat: repeat-y; /* tiles vertically only */
  background-repeat: space; /* tiles with even spacing, no clipping */
  background-repeat: round; /* tiles and scales to fit without clipping */
}
```

---

### `background-attachment`

Defines whether the background scrolls with the content or stays fixed.

```css
.parallax-section {
  background-attachment: fixed; /* stays in place as page scrolls (parallax effect) */
  background-attachment: scroll; /* default — scrolls with element */
  background-attachment: local; /* scrolls with element's content */
}
```

---

### Background Shorthand

```css
.hero {
  background:
    linear-gradient(rgba(0, 0, 0, 0.5), rgba(0, 0, 0, 0.5)),
    url("/images/hero.jpg") center / cover no-repeat fixed;
  /* image position / size repeat attachment */
}
```

---

## Gradients

Gradients are generated images — they go wherever `background-image` goes.

### Linear Gradient

Creates a transition between colors along a straight line.

```css
.gradient-basic {
  background: linear-gradient(to right, #667eea, #764ba2);
}

/* With angle */
.gradient-angled {
  background: linear-gradient(135deg, #f093fb, #f5576c);
}

/* Multiple color stops */
.gradient-multi {
  background: linear-gradient(
    to bottom,
    #ff6b6b 0%,
    #feca57 33%,
    #48dbfb 66%,
    #ff9ff3 100%
  );
}

/* Hard color stops (no transition) — stripes */
.stripes {
  background: linear-gradient(to right, #333 0%, #333 50%, #fff 50%, #fff 100%);
}
```

---

### Radial Gradient

Creates a transition that radiates from a center point.

```css
.gradient-radial {
  background: radial-gradient(circle, #667eea, #764ba2);
}

/* Elliptical with position */
.spotlight {
  background: radial-gradient(ellipse at top left, #fff, #000);
}

/* Sized radial gradient */
.subtle-glow {
  background: radial-gradient(
    circle 200px at center,
    rgba(255, 255, 255, 0.1),
    transparent
  );
}
```

---

### Conic Gradient

Creates color transitions rotated around a center point (like a pie chart).

```css
.pie-chart {
  background: conic-gradient(
    #ff6b6b 0deg 120deg,
    #48dbfb 120deg 240deg,
    #feca57 240deg 360deg
  );
  border-radius: 50%;
}

.color-wheel {
  background: conic-gradient(red, yellow, green, cyan, blue, magenta, red);
  border-radius: 50%;
}
```

---

### Repeating Gradients

```css
.repeating-stripes {
  background: repeating-linear-gradient(
    45deg,
    #606dbc,
    #606dbc 10px,
    #465298 10px,
    #465298 20px
  );
}
```

---

## CSS Filters

Filters apply visual effects to an entire element and its children.

### Available Filters

```css
.element {
  /* Blur — Gaussian blur in pixels */
  filter: blur(5px);

  /* Brightness — 1 is normal, <1 darker, >1 brighter */
  filter: brightness(0.7);

  /* Contrast — 1 is normal */
  filter: contrast(1.5);

  /* Grayscale — 0 to 1 (0% to 100%) */
  filter: grayscale(1);

  /* Hue rotation — degrees */
  filter: hue-rotate(90deg);

  /* Invert — 0 to 1 */
  filter: invert(1);

  /* Opacity — like the opacity property but can be hardware-accelerated */
  filter: opacity(0.5);

  /* Saturate — 1 is normal, >1 more vivid, <1 less vivid */
  filter: saturate(2);

  /* Sepia — 0 to 1 */
  filter: sepia(0.8);

  /* Drop shadow — like box-shadow but follows the shape (great for PNGs) */
  filter: drop-shadow(4px 4px 8px rgba(0, 0, 0, 0.3));
}

/* Combine multiple filters */
.vintage-photo {
  filter: sepia(0.4) contrast(1.1) brightness(0.9) saturate(1.2);
}
```

---

### Common Filter Patterns

```css
/* Image hover effect */
.image-card img {
  transition: filter 0.3s ease;
}

.image-card img:hover {
  filter: brightness(1.1) saturate(1.2);
}

/* Disabled/inactive state */
.disabled {
  filter: grayscale(1) opacity(0.6);
  pointer-events: none;
}

/* Loading placeholder blur */
.image-loading {
  filter: blur(20px);
  transition: filter 0.5s ease;
}

.image-loaded {
  filter: blur(0);
}
```

---

### `backdrop-filter`

Applies filter effects to the area **behind** an element (what is visible through it). Creates frosted-glass effects.

```css
.glass-card {
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 12px;
}

.navbar-frosted {
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(8px) saturate(1.5);
  position: sticky;
  top: 0;
}
```

---

## Practical Examples

### Hero Section with Overlay

```css
.hero {
  position: relative;
  min-height: 80vh;
  background:
    linear-gradient(to bottom, rgba(0, 0, 0, 0.3), rgba(0, 0, 0, 0.7)),
    url("/images/hero.jpg") center / cover no-repeat;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
}
```

### Gradient Text

```css
.gradient-text {
  background: linear-gradient(135deg, #667eea, #764ba2);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  color: transparent; /* fallback */
}
```

### Frosted Glass Modal

```css
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.4);
  backdrop-filter: blur(4px);
  display: grid;
  place-items: center;
}

.modal-content {
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(10px);
  padding: 2rem;
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.3);
}
```

---

## Best Practices

1. **Always pair background-image with background-color** — if the image fails to load, the color provides a fallback.
2. **Use `cover` for hero images** — ensures no empty space regardless of viewport size.
3. **Compress background images** — large images slow page load significantly.
4. **Use gradients instead of images** when possible — zero network requests, infinitely scalable.
5. **Prefer `backdrop-filter`** for glassmorphism — it affects only the area behind, not children.
6. **Use `will-change: filter`** for animated filters — hints the browser to optimize rendering.
7. **Test `backdrop-filter` cross-browser** — older browsers may not support it.

---

## Common Mistakes

| Mistake                                                                              | Why It Is Wrong                                                                                 |
| ------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------- |
| Using `background` shorthand and accidentally overriding other background properties | Shorthand resets unset sub-properties to defaults                                               |
| Forgetting `background-size: cover` on responsive hero images                        | Image will display at natural size, causing overflow or tiling                                  |
| Applying `filter` to a parent and expecting children to be unaffected                | `filter` applies to the entire element tree — use `backdrop-filter` for background-only effects |
| Not providing fallback for `backdrop-filter`                                         | Older browsers show nothing — always set a semi-opaque `background-color` as fallback           |
| Using huge unoptimized images as backgrounds                                         | Destroys page load performance — compress and use appropriate dimensions                        |
| Stacking too many CSS filters                                                        | Hurts rendering performance — each filter adds a paint operation                                |

---

## Summary

- Background properties control what appears behind content: colors, images, gradients, positioning.
- **`background-size: cover`** is your go-to for responsive hero images.
- **Gradients** (linear, radial, conic) create smooth color transitions without any file downloads.
- **CSS filters** apply real-time visual effects: blur, grayscale, brightness, contrast, and more.
- **`backdrop-filter`** creates frosted-glass effects on the area behind an element.
- Layer multiple backgrounds with gradients for overlays — no extra HTML elements needed.
- Always consider performance: compress images, limit filter stacking, and use `will-change` for animated effects.
