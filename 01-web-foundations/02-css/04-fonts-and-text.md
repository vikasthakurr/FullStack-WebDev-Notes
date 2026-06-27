# Fonts and Text in CSS

## What It Is

Typography in CSS covers two related concerns: **fonts** (which typeface, weight, and size to use) and **text** (how that text is aligned, spaced, decorated, and transformed). Good typography is like good handwriting — even if the content is the same, presentation dramatically affects readability and perception.

Think of fonts as choosing the voice, and text properties as choosing how that voice speaks — its volume, pace, and emphasis.

---

## Why It Matters

- Typography accounts for roughly 95% of web design — most of what users interact with is text.
- Poor font choices cause readability issues, slow page loads, and inconsistent experiences.
- Understanding font stacks and fallbacks prevents your site from breaking on systems that lack your chosen font.
- Text properties control whitespace, rhythm, and visual hierarchy — the bones of readable content.

---

## Font Properties

### `font-family`

Defines the typeface. Always provide a **font stack** — a prioritized list with a generic fallback.

```css
body {
  font-family: "Inter", "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
}

code {
  font-family: "Fira Code", "Courier New", Courier, monospace;
}
```

**Generic families:** `serif`, `sans-serif`, `monospace`, `cursive`, `fantasy`, `system-ui`

**Rule:** Always end with a generic family. If all named fonts fail, the browser picks its default for that generic.

---

### `font-size`

Sets the size of text. Accepts various units.

```css
h1 {
  font-size: 2.5rem;
} /* relative to root font size */
h2 {
  font-size: 2rem;
}
body {
  font-size: 1rem;
} /* typically 16px by default */
small {
  font-size: 0.875rem;
}

/* Avoid px for body text — it does not respect user font preferences */
```

---

### `font-weight`

Controls thickness of characters.

```css
.light {
  font-weight: 300;
}
.normal {
  font-weight: 400;
} /* same as 'normal' */
.medium {
  font-weight: 500;
}
.bold {
  font-weight: 700;
} /* same as 'bold' */
.black {
  font-weight: 900;
}
```

**Note:** The font file must include the requested weight. If you load `font-weight: 400` and `700` from Google Fonts, using `500` will synthesize a faux-bold that looks poor.

---

### `font-style`

Sets italic or oblique rendering.

```css
em {
  font-style: italic;
}
.reset {
  font-style: normal;
}
```

---

### `line-height`

Controls the vertical space between lines. Crucial for readability.

```css
body {
  line-height: 1.6; /* unitless multiplier — preferred */
}

h1 {
  line-height: 1.2; /* tighter for headings */
}
```

**Best practice:** Use unitless values. A value of `1.6` means 1.6 times the font size. This scales properly when font-size changes. Avoid `line-height: 24px` — it will not adapt.

---

### `letter-spacing`

Adjusts space between individual characters (tracking in typographic terms).

```css
.uppercase-label {
  letter-spacing: 0.05em; /* slight expansion for uppercase text */
  text-transform: uppercase;
}

h1 {
  letter-spacing: -0.02em; /* tighten large headings */
}
```

---

### `word-spacing`

Adjusts space between words.

```css
.relaxed {
  word-spacing: 0.1em;
}
```

---

## Text Properties

### `text-align`

Horizontal alignment of text within its container.

```css
.center {
  text-align: center;
}
.right {
  text-align: right;
}
.justify {
  text-align: justify;
} /* use carefully — can create uneven gaps */
```

---

### `text-transform`

Controls capitalization without changing the HTML source.

```css
.uppercase {
  text-transform: uppercase;
}
.lowercase {
  text-transform: lowercase;
}
.capitalize {
  text-transform: capitalize;
} /* First letter of each word */
```

---

### `text-decoration`

Adds or removes lines on text.

```css
a {
  text-decoration: none;
} /* remove default underline */

.underline {
  text-decoration: underline;
  text-decoration-color: #3366ff;
  text-decoration-thickness: 2px;
  text-underline-offset: 3px; /* space between text and line */
}

.strikethrough {
  text-decoration: line-through;
}
```

---

### `text-indent`

Indents the first line of a block of text.

```css
p {
  text-indent: 2em; /* like a traditional paragraph indent */
}
```

---

### `text-overflow`

Controls how overflowing text is displayed.

```css
.truncate {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis; /* shows "..." when text overflows */
}
```

---

### `text-shadow`

Adds shadow behind text for depth or glow effects.

```css
h1 {
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
  /* offset-x | offset-y | blur-radius | color */
}
```

---

## Loading Custom Fonts

### Google Fonts Integration

The simplest way to add web fonts. Add the `<link>` in your HTML `<head>`:

```html
<head>
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link
    href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700&display=swap"
    rel="stylesheet"
  />
</head>
```

Then use in CSS:

```css
body {
  font-family: "Inter", sans-serif;
}
```

**The `display=swap` parameter** ensures text is visible immediately using a fallback font, then swaps to the custom font once loaded (avoids invisible text during load).

---

### `@font-face` (Self-Hosted Fonts)

For full control, host font files yourself:

```css
@font-face {
  font-family: "CustomFont";
  src:
    url("/fonts/CustomFont-Regular.woff2") format("woff2"),
    url("/fonts/CustomFont-Regular.woff") format("woff");
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: "CustomFont";
  src:
    url("/fonts/CustomFont-Bold.woff2") format("woff2"),
    url("/fonts/CustomFont-Bold.woff") format("woff");
  font-weight: 700;
  font-style: normal;
  font-display: swap;
}

body {
  font-family: "CustomFont", sans-serif;
}
```

**Key points:**

- Use `woff2` as the primary format — best compression, widest modern support.
- `font-display: swap` prevents Flash of Invisible Text (FOIT).
- Define separate `@font-face` blocks for each weight/style combination.

---

## Font Shorthand

Combine multiple font properties in one declaration:

```css
p {
  font:
    italic 500 1rem/1.6 "Inter",
    sans-serif;
  /* font: style weight size/line-height family */
}
```

**Required:** `font-size` and `font-family` must be present. Order matters.

---

## Responsive Typography

```css
/* Fluid typography using clamp() */
h1 {
  font-size: clamp(1.5rem, 4vw, 3rem);
  /* minimum | preferred | maximum */
}

body {
  font-size: clamp(1rem, 1.2vw, 1.25rem);
}
```

This automatically scales text between a minimum and maximum size based on viewport width — no media queries needed.

---

## Best Practices

1. **Use `rem` for font sizes** — respects user browser settings and scales predictably.
2. **Limit font families to 2-3 per project** — one for headings, one for body, optionally one for code.
3. **Load only the weights you need** — each weight adds to download size.
4. **Use `font-display: swap`** — users should see content immediately, not a blank page.
5. **Set `line-height` on `body`** — establish a vertical rhythm that all text inherits.
6. **Use unitless `line-height`** — it scales with font-size changes.
7. **Preconnect to font origins** — `<link rel="preconnect">` reduces latency for Google Fonts.

---

## Common Mistakes

| Mistake                                       | Why It Is Wrong                                                 |
| --------------------------------------------- | --------------------------------------------------------------- |
| Not providing fallback fonts                  | If the custom font fails to load, users get the browser default |
| Using `px` for body font-size                 | Overrides user accessibility preferences                        |
| Loading too many font weights                 | Each weight is a separate file — hurts load time                |
| Setting `line-height` in `px`                 | Does not scale with font-size — leads to cramped or loose text  |
| Forgetting `font-display`                     | Causes FOIT — users see invisible text until font loads         |
| Using `text-align: justify` without `hyphens` | Creates ugly rivers of whitespace                               |

---

## Summary

- **Font properties** (`font-family`, `font-size`, `font-weight`, `font-style`) control what the text looks like.
- **Text properties** (`text-align`, `text-transform`, `text-decoration`, `line-height`, `letter-spacing`) control how text behaves and flows.
- Load custom fonts via **Google Fonts** (easy) or **`@font-face`** (full control).
- Use **`rem` units** and **`clamp()`** for responsive, accessible typography.
- Always provide a **font stack** with a generic fallback family.
- Good typography is invisible — readers notice bad typography, but great typography just makes content feel effortless to read.
