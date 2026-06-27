# Bootstrap 5 Typography

## What Is It

Typography in Bootstrap 5 is a set of pre-built utility classes that give you consistent, readable, and visually appealing text styling without writing custom CSS. Think of it as a well-organized wardrobe for your text -- every heading, paragraph, and list has a tailored outfit ready to wear.

Bootstrap resets the browser's default styles and applies its own opinionated base typography using the system font stack, ensuring text looks clean across all platforms.

## Why It Matters

- Consistent text hierarchy helps users scan content quickly.
- Eliminates the need to write repetitive CSS for headings, paragraphs, and lists.
- Responsive by default -- font sizes scale appropriately.
- Follows accessibility best practices with proper contrast and sizing.

---

## Core Concepts

### Headings

Bootstrap styles all standard HTML headings (`<h1>` through `<h6>`) and also provides matching classes when you need heading appearance without the semantic weight.

```html
<h1>Heading 1</h1>
<h2>Heading 2</h2>
<h3>Heading 3</h3>

<!-- Use classes when you need visual hierarchy without semantic meaning -->
<p class="h1">Looks like h1, but is a paragraph</p>
<p class="h4">Looks like h4, but is a paragraph</p>
```

**Analogy:** Semantic headings are like job titles (CEO, Manager, Intern) -- they carry meaning. Heading classes are like wearing the CEO's suit without having the title -- you look the part, but the org chart knows the difference.

### Display Headings

When regular headings are not dramatic enough, display headings give you larger, lighter text -- ideal for hero sections and landing pages.

```html
<h1 class="display-1">Display 1</h1>
<h1 class="display-2">Display 2</h1>
<h1 class="display-3">Display 3</h1>
<h1 class="display-4">Display 4</h1>
<h1 class="display-5">Display 5</h1>
<h1 class="display-6">Display 6</h1>
```

Display headings use a thinner font-weight and larger font-size, making them perfect for first impressions.

### Lead Paragraph

The `.lead` class makes a paragraph stand out -- slightly larger and lighter than body text. Use it for introductory paragraphs.

```html
<p class="lead">
  This is a lead paragraph. It stands out from regular paragraphs by being
  slightly larger and using a lighter font weight.
</p>
```

### Inline Text Elements

```html
<p>You can use <mark>highlight</mark> to mark text.</p>
<p><del>This text is deleted.</del></p>
<p><s>This text has a strikethrough.</s></p>
<p><ins>This text is an addition to the document.</ins></p>
<p><u>This text is underlined.</u></p>
<p><small>This is fine print.</small></p>
<p><strong>This is bold text.</strong></p>
<p><em>This is italicized text.</em></p>
```

---

## Text Utility Classes

Bootstrap provides utility classes for controlling alignment, wrapping, weight, and more.

### Text Alignment

```html
<p class="text-start">Left aligned (default in LTR).</p>
<p class="text-center">Center aligned.</p>
<p class="text-end">Right aligned.</p>

<!-- Responsive alignment -->
<p class="text-md-center">Centered on medium screens and up.</p>
<p class="text-lg-end">Right aligned on large screens and up.</p>
```

### Text Transform

```html
<p class="text-lowercase">LOWERCASED TEXT.</p>
<p class="text-uppercase">uppercased text.</p>
<p class="text-capitalize">capitalize each word.</p>
```

### Font Weight and Italics

```html
<p class="fw-bold">Bold text.</p>
<p class="fw-bolder">Bolder weight (relative to parent).</p>
<p class="fw-semibold">Semibold text.</p>
<p class="fw-normal">Normal weight text.</p>
<p class="fw-light">Light weight text.</p>
<p class="fw-lighter">Lighter weight (relative to parent).</p>
<p class="fst-italic">Italic text.</p>
<p class="fst-normal">Normal style text.</p>
```

### Font Size Utilities

```html
<p class="fs-1">Font size 1 (largest)</p>
<p class="fs-2">Font size 2</p>
<p class="fs-3">Font size 3</p>
<p class="fs-4">Font size 4</p>
<p class="fs-5">Font size 5</p>
<p class="fs-6">Font size 6 (smallest)</p>
```

### Text Wrapping and Overflow

```html
<p class="text-wrap" style="width: 6rem;">This text will wrap normally.</p>
<p class="text-nowrap" style="width: 6rem;">This text will not wrap.</p>
<p class="text-truncate" style="width: 150px;">
  This long text will be truncated with an ellipsis.
</p>
```

### Text Decoration

```html
<a href="#" class="text-decoration-none">Link without underline</a>
<p class="text-decoration-underline">Underlined text</p>
<p class="text-decoration-line-through">Strikethrough text</p>
```

---

## Blockquotes

Blockquotes get a clean, minimal style in Bootstrap. Use the `<figure>` element for proper attribution.

```html
<figure>
  <blockquote class="blockquote">
    <p>A well-known quote, contained in a blockquote element.</p>
  </blockquote>
  <figcaption class="blockquote-footer">
    Someone famous in <cite title="Source Title">Source Title</cite>
  </figcaption>
</figure>
```

### Aligned Blockquotes

```html
<figure class="text-center">
  <blockquote class="blockquote">
    <p>A centered blockquote.</p>
  </blockquote>
  <figcaption class="blockquote-footer">Author Name</figcaption>
</figure>

<figure class="text-end">
  <blockquote class="blockquote">
    <p>A right-aligned blockquote.</p>
  </blockquote>
  <figcaption class="blockquote-footer">Author Name</figcaption>
</figure>
```

---

## Lists

### Unstyled Lists

Remove bullets and left margin from a list.

```html
<ul class="list-unstyled">
  <li>Item one</li>
  <li>Item two</li>
  <li>
    Nested items (not affected):
    <ul>
      <li>Nested item one</li>
      <li>Nested item two</li>
    </ul>
  </li>
</ul>
```

### Inline Lists

Display list items horizontally.

```html
<ul class="list-inline">
  <li class="list-inline-item">First</li>
  <li class="list-inline-item">Second</li>
  <li class="list-inline-item">Third</li>
</ul>
```

### Description Lists

```html
<dl class="row">
  <dt class="col-sm-3">Term</dt>
  <dd class="col-sm-9">Definition for the term above.</dd>

  <dt class="col-sm-3">Another term</dt>
  <dd class="col-sm-9">Another definition.</dd>

  <dt class="col-sm-3 text-truncate">Truncated long term that overflows</dt>
  <dd class="col-sm-9">Definition for truncated term.</dd>
</dl>
```

---

## Best Practices

1. Use semantic headings (`h1`-`h6`) for document structure; use heading classes only for visual overrides.
2. Limit display headings to hero/landing sections -- overusing them dilutes their impact.
3. Use `.lead` for the first paragraph of a section to draw readers in.
4. Combine text utilities with responsive prefixes (e.g., `text-md-center`) for adaptive layouts.
5. Prefer utility classes over custom CSS when Bootstrap already provides the solution.

## Common Mistakes

| Mistake                                                       | Why It Is Wrong                                                   | Fix                                           |
| ------------------------------------------------------------- | ----------------------------------------------------------------- | --------------------------------------------- |
| Using `<h1>` multiple times for visual size                   | Screen readers interpret multiple h1 tags as multiple main topics | Use `.h1` class or display headings instead   |
| Skipping heading levels (h1 then h3)                          | Breaks document outline for accessibility                         | Follow sequential order                       |
| Using `text-truncate` without a fixed width                   | Truncation only works when the container constrains the text      | Add a width or `max-width`                    |
| Applying `.list-unstyled` expecting it to affect nested lists | Only affects immediate children                                   | Apply the class to each nested `<ul>` as well |

---

## Summary

Bootstrap typography gives you a complete type system out of the box. From display headings for impact to inline utilities for fine control, you can handle virtually any text layout without writing custom CSS. The key principle: use semantic HTML for meaning, Bootstrap classes for presentation. This separation keeps your markup accessible and your styles maintainable.
