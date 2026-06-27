# Images and Attributes

## What is the Image Element?

The `<img>` element embeds images into HTML documents. It is a **void element** (self-closing, no closing tag) and a **replaced element** (its content comes from an external resource). Images are fundamental to web content — they communicate information, create visual interest, and can significantly affect page load performance.

**Analogy**: Think of the `<img>` tag as a picture frame on a wall. The `src` attribute is the address where the actual photo is stored. The `alt` attribute is the descriptive label on the back — invisible when viewing normally, but essential if the photo falls off the wall (fails to load) or if someone who cannot see needs to know what it depicts.

## Why It Matters

- Images make up the majority of bytes downloaded on most web pages.
- Proper image handling directly impacts Core Web Vitals (LCP, CLS).
- Alt text is a legal requirement in many jurisdictions for accessibility compliance.
- Responsive images prevent mobile users from downloading unnecessarily large files.

## Basic Syntax

```html
<img src="photo.jpg" alt="A golden retriever playing in a park" />
```

### Required Attributes

| Attribute | Purpose                                                   |
| --------- | --------------------------------------------------------- |
| `src`     | URL/path to the image file                                |
| `alt`     | Alternative text description (required for accessibility) |

## The `alt` Attribute (Critical)

The `alt` attribute provides text that:

- Is read aloud by screen readers for blind users.
- Displays when the image fails to load.
- Is used by search engines to understand image content.

### How to Write Good Alt Text

```html
<!-- GOOD: Descriptive, concise, communicates the content -->
<img
  src="chart.png"
  alt="Bar chart showing 40% increase in Q4 revenue compared to Q3"
/>

<!-- GOOD: Functional description for UI elements -->
<img src="search-icon.svg" alt="Search" />

<!-- BAD: Useless description -->
<img src="chart.png" alt="image" />
<img src="chart.png" alt="chart.png" />

<!-- DECORATIVE: If the image adds no information, use empty alt -->
<img src="decorative-border.png" alt="" />
```

### Alt Text Rules

1. **Informative images**: Describe the content and function. ("Pie chart showing 60% market share for Chrome")
2. **Functional images** (buttons, links): Describe the action. ("Submit form", "Go to homepage")
3. **Decorative images**: Use empty `alt=""` so screen readers skip them.
4. **Complex images** (charts, diagrams): Provide a brief alt + longer description nearby or via `aria-describedby`.
5. **Never start with "Image of..." or "Picture of..."** — screen readers already announce "image."

## Width and Height Attributes

```html
<img src="hero.jpg" alt="Mountain landscape" width="1200" height="800" />
```

Setting explicit `width` and `height` prevents **Cumulative Layout Shift (CLS)** — the annoying page jump that happens when an image loads and pushes content down.

The browser uses these values to calculate the aspect ratio and reserve space before the image downloads:

```mermaid
flowchart LR
    A[Browser reads width/height] --> B[Calculates aspect ratio]
    B --> C[Reserves correct space in layout]
    C --> D[Image loads into reserved space]
    D --> E[No layout shift!]
```

Modern CSS approach — set dimensions in HTML for aspect ratio, then make responsive with CSS:

```html
<img
  src="hero.jpg"
  alt="Mountain landscape"
  width="1200"
  height="800"
  style="width: 100%; height: auto;"
/>
```

## Lazy Loading

```html
<img src="below-fold.jpg" alt="Product photo" loading="lazy" />
```

`loading="lazy"` tells the browser: "Do not download this image until the user scrolls near it." This dramatically improves initial page load time for pages with many images.

| Value   | Behavior                            |
| ------- | ----------------------------------- |
| `lazy`  | Defer loading until near viewport   |
| `eager` | Load immediately (default behavior) |

**When to use lazy loading**:

- Images below the fold (not visible on initial load).
- Image galleries with many items.
- Long article pages with multiple illustrations.

**When NOT to use lazy loading**:

- Hero images / above-the-fold images (they should load immediately).
- LCP (Largest Contentful Paint) images — lazy loading them hurts performance metrics.

## Responsive Images with `srcset`

Different devices have different screen sizes and pixel densities. Sending a 4000px image to a phone with a 400px screen wastes bandwidth.

### Resolution Switching (Different Sizes)

```html
<img
  src="photo-800.jpg"
  srcset="
    photo-400.jpg   400w,
    photo-800.jpg   800w,
    photo-1200.jpg 1200w,
    photo-1600.jpg 1600w
  "
  sizes="(max-width: 600px) 400px,
            (max-width: 900px) 800px,
            1200px"
  alt="City skyline at sunset"
/>
```

**How it works**:

- `srcset` lists available image files with their intrinsic widths (`w` descriptor).
- `sizes` tells the browser how wide the image will be displayed at each breakpoint.
- The browser picks the best image based on viewport size AND pixel density.

### Pixel Density Switching

```html
<!-- For icons and logos where you need 2x/3x for Retina screens -->
<img
  src="logo.png"
  srcset="logo.png 1x, logo@2x.png 2x, logo@3x.png 3x"
  alt="Company logo"
  width="200"
  height="50"
/>
```

### The `<picture>` Element (Art Direction)

When you need **different images** (not just different sizes) for different screen sizes:

```html
<picture>
  <!-- Mobile: cropped portrait version -->
  <source media="(max-width: 600px)" srcset="hero-mobile.jpg" />
  <!-- Tablet: cropped landscape version -->
  <source media="(max-width: 1024px)" srcset="hero-tablet.jpg" />
  <!-- Desktop: full wide version -->
  <img src="hero-desktop.jpg" alt="Team collaborating in office" />
</picture>
```

The `<picture>` element can also serve different formats:

```html
<picture>
  <source type="image/avif" srcset="photo.avif" />
  <source type="image/webp" srcset="photo.webp" />
  <img src="photo.jpg" alt="Landscape photo" />
</picture>
```

## `<figure>` and `<figcaption>`

When an image has a caption that belongs semantically with it:

```html
<figure>
  <img
    src="architecture-diagram.png"
    alt="System architecture showing three microservices connected via message queue"
  />
  <figcaption>
    Figure 1: High-level system architecture. Services communicate
    asynchronously through a message broker.
  </figcaption>
</figure>
```

- `<figure>` is a self-contained unit — it could be moved elsewhere without affecting document flow.
- `<figcaption>` provides a visible caption associated with the figure.
- Not just for images — can contain code blocks, tables, diagrams, or quotes.

## Image Formats Guide

| Format | Best For                                   | Transparency | Animation    | Compression                |
| ------ | ------------------------------------------ | ------------ | ------------ | -------------------------- |
| JPEG   | Photos, complex images                     | No           | No           | Lossy                      |
| PNG    | Graphics, screenshots, transparency needed | Yes          | No           | Lossless                   |
| WebP   | Modern replacement for JPEG/PNG            | Yes          | Yes          | Both                       |
| AVIF   | Next-gen format, best compression          | Yes          | Yes          | Both                       |
| SVG    | Icons, logos, illustrations                | Yes          | Yes (CSS/JS) | Vector (scales infinitely) |
| GIF    | Simple animations (prefer video instead)   | Yes (1-bit)  | Yes          | Lossless (limited palette) |

## Best Practices

- Always include meaningful `alt` text (or `alt=""` for decorative images).
- Always set `width` and `height` to prevent layout shift.
- Use `loading="lazy"` for below-the-fold images.
- Serve modern formats (WebP/AVIF) with fallbacks using `<picture>`.
- Use `srcset` and `sizes` for responsive images.
- Compress images before serving — unoptimized images are the most common performance issue.
- Use `<figure>` and `<figcaption>` when images need visible captions.
- Use SVG for icons and logos (they scale perfectly at any size).

## Common Mistakes

| Mistake                              | Why It Is Wrong                                                     | Fix                                            |
| ------------------------------------ | ------------------------------------------------------------------- | ---------------------------------------------- |
| Missing `alt` attribute              | Screen readers say "image" with no context; accessibility violation | Always include `alt`                           |
| `alt="image"` or `alt="photo.jpg"`   | Adds no useful information                                          | Describe the content meaningfully              |
| No width/height                      | Causes layout shift (CLS) as image loads                            | Set dimensions in HTML                         |
| Lazy loading hero image              | Delays the largest visible element; hurts LCP                       | Use `loading="eager"` (default) for above-fold |
| Serving 4000px images to all devices | Wastes bandwidth on mobile                                          | Use `srcset` and `sizes`                       |
| Using JPEG for screenshots with text | Text gets blurry with JPEG compression                              | Use PNG or WebP (lossless)                     |
| Decorative image without `alt=""`    | Screen reader reads filename or "image"                             | Use empty alt: `alt=""`                        |

## Summary

- The `<img>` element requires `src` and `alt` — never skip either.
- `alt` text is the most important accessibility feature for images.
- Set `width` and `height` to reserve layout space and prevent CLS.
- Use `loading="lazy"` to defer below-fold images.
- Use `srcset`/`sizes` for resolution switching and `<picture>` for art direction and format fallbacks.
- Combine `<figure>` with `<figcaption>` for semantically captioned images.
- Image optimization is one of the highest-impact performance improvements you can make on any website.
