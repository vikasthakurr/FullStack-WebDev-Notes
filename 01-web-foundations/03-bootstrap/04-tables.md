# Bootstrap 5 Tables

## What Is It

Bootstrap's table classes enhance standard HTML tables with clean styling, contextual colors, hover effects, and responsive behavior. Instead of writing dozens of CSS rules for borders, stripes, and spacing, you add a few classes and get production-ready tables instantly.

**Analogy:** A raw HTML table is like an unformatted spreadsheet -- functional but hard to read. Bootstrap table classes are like applying a professional Excel theme with one click: alternating row colors, clean borders, and proper spacing all appear automatically.

## Why It Matters

- Tables are essential for displaying structured data (user lists, invoices, analytics).
- Without styling, tables are visually dense and difficult to scan.
- Bootstrap handles cross-browser consistency, hover states, and responsive overflow.
- Accessibility-friendly -- proper semantic structure is preserved.

---

## Basic Table

```html
<table class="table">
  <thead>
    <tr>
      <th scope="col">#</th>
      <th scope="col">Name</th>
      <th scope="col">Email</th>
      <th scope="col">Role</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">1</th>
      <td>Alice Johnson</td>
      <td>alice@example.com</td>
      <td>Admin</td>
    </tr>
    <tr>
      <th scope="row">2</th>
      <td>Bob Smith</td>
      <td>bob@example.com</td>
      <td>Editor</td>
    </tr>
    <tr>
      <th scope="row">3</th>
      <td>Carol Lee</td>
      <td>carol@example.com</td>
      <td>Viewer</td>
    </tr>
  </tbody>
</table>
```

The `.table` class is required. Without it, none of the other table modifiers work.

---

## Table Variants

### Striped Rows

Alternating row backgrounds improve scanability for long tables.

```html
<table class="table table-striped">
  <!-- rows will alternate between white and light gray -->
</table>
```

### Striped Columns (Bootstrap 5.3+)

```html
<table class="table table-striped-columns">
  <!-- columns alternate colors instead of rows -->
</table>
```

### Bordered Table

Adds borders on all sides and cells.

```html
<table class="table table-bordered">
  <!-- full borders everywhere -->
</table>
```

### Borderless Table

Removes all borders for a minimal look.

```html
<table class="table table-borderless">
  <!-- clean, no-border aesthetic -->
</table>
```

### Hover Effect

Highlights rows on mouse hover -- useful for large datasets.

```html
<table class="table table-hover">
  <!-- rows highlight on hover -->
</table>
```

### Compact/Small Table

Reduces cell padding for denser data display.

```html
<table class="table table-sm">
  <!-- tighter padding, more data in less space -->
</table>
```

### Combining Modifiers

Classes are composable -- stack them as needed.

```html
<table class="table table-striped table-hover table-bordered table-sm">
  <!-- striped + hoverable + bordered + compact -->
</table>
```

---

## Dark Table

Invert the color scheme for dark-themed interfaces.

```html
<table class="table table-dark">
  <thead>
    <tr>
      <th>#</th>
      <th>Name</th>
      <th>Status</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>Server A</td>
      <td>Online</td>
    </tr>
  </tbody>
</table>

<!-- Dark + striped + hover -->
<table class="table table-dark table-striped table-hover">
  <!-- dark themed with stripes and hover -->
</table>
```

---

## Contextual Row and Cell Colors

Apply semantic colors to individual rows or cells.

```html
<table class="table">
  <tbody>
    <tr class="table-primary">
      <td>Primary row</td>
    </tr>
    <tr class="table-success">
      <td>Success row</td>
    </tr>
    <tr class="table-danger">
      <td>Danger row</td>
    </tr>
    <tr class="table-warning">
      <td>Warning row</td>
    </tr>
    <tr class="table-info">
      <td>Info row</td>
    </tr>
    <tr class="table-light">
      <td>Light row</td>
    </tr>
    <tr class="table-dark">
      <td>Dark row</td>
    </tr>
    <tr>
      <td class="table-success">Individual green cell</td>
      <td class="table-danger">Individual red cell</td>
    </tr>
  </tbody>
</table>
```

---

## Table Head Variants

```html
<!-- Light header -->
<table class="table">
  <thead class="table-light">
    <tr>
      <th>Name</th>
      <th>Age</th>
    </tr>
  </thead>
</table>

<!-- Dark header -->
<table class="table">
  <thead class="table-dark">
    <tr>
      <th>Name</th>
      <th>Age</th>
    </tr>
  </thead>
</table>
```

---

## Table Caption

Captions describe the table's purpose. By default, they appear below the table.

```html
<table class="table">
  <caption>
    List of registered users as of 2024
  </caption>
  <thead>
    <tr>
      <th>ID</th>
      <th>Name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>Alice</td>
    </tr>
  </tbody>
</table>

<!-- Caption on top -->
<table class="table caption-top">
  <caption>
    Monthly revenue breakdown
  </caption>
  <!-- ... -->
</table>
```

---

## Responsive Tables

When tables exceed the viewport width, they need horizontal scrolling. Wrap the table in a responsive container.

```html
<!-- Always responsive -->
<div class="table-responsive">
  <table class="table">
    <!-- wide table content -->
  </table>
</div>

<!-- Responsive only below a specific breakpoint -->
<div class="table-responsive-md">
  <table class="table">
    <!-- scrolls horizontally only on screens smaller than md -->
  </table>
</div>
```

Available responsive breakpoints: `table-responsive-sm`, `table-responsive-md`, `table-responsive-lg`, `table-responsive-xl`, `table-responsive-xxl`.

---

## Table with Vertical Alignment

```html
<table class="table align-middle">
  <tbody>
    <tr>
      <td>Vertically centered content in all cells</td>
      <td>Another cell</td>
    </tr>
  </tbody>
</table>

<!-- Per-cell alignment -->
<tr>
  <td class="align-top">Top</td>
  <td class="align-middle">Middle</td>
  <td class="align-bottom">Bottom</td>
</tr>
```

---

## Complete Example

```html
<div class="table-responsive">
  <table class="table table-striped table-hover table-bordered align-middle">
    <caption class="caption-top">
      Employee Directory
    </caption>
    <thead class="table-dark">
      <tr>
        <th scope="col">#</th>
        <th scope="col">Name</th>
        <th scope="col">Department</th>
        <th scope="col">Status</th>
        <th scope="col">Actions</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <th scope="row">1</th>
        <td>Alice Johnson</td>
        <td>Engineering</td>
        <td><span class="badge bg-success">Active</span></td>
        <td><button class="btn btn-sm btn-outline-primary">Edit</button></td>
      </tr>
      <tr class="table-warning">
        <th scope="row">2</th>
        <td>Bob Smith</td>
        <td>Marketing</td>
        <td><span class="badge bg-warning text-dark">On Leave</span></td>
        <td><button class="btn btn-sm btn-outline-primary">Edit</button></td>
      </tr>
    </tbody>
  </table>
</div>
```

---

## Best Practices

1. Always use `<thead>`, `<tbody>`, and `scope` attributes for accessibility.
2. Wrap wide tables in `.table-responsive` to prevent layout breakage on mobile.
3. Use contextual colors sparingly -- limit them to rows that genuinely need attention.
4. Prefer `table-sm` for data-dense dashboards; use default spacing for general content.
5. Use `caption` or an `aria-label` so screen readers can announce the table's purpose.

## Common Mistakes

| Mistake                                                         | Why It Is Wrong                                 | Fix                                           |
| --------------------------------------------------------------- | ----------------------------------------------- | --------------------------------------------- |
| Forgetting the `.table` base class                              | Modifier classes have no effect without it      | Always start with `class="table"`             |
| Using tables for layout                                         | Tables are for tabular data, not page structure | Use the grid system for layout                |
| Not wrapping in `.table-responsive`                             | Wide tables overflow and break mobile layouts   | Always add responsive wrapper for data tables |
| Applying `table-dark` to `<thead>` and `<table>` simultaneously | Creates confusing double-dark appearance        | Choose one level to darken                    |

---

## Summary

Bootstrap tables transform raw HTML tables into readable, professional data displays with minimal effort. The composable class system (striped, bordered, hover, dark, responsive) lets you mix and match styles to suit any data presentation need. Wrap in `.table-responsive` for mobile safety, use semantic markup for accessibility, and you have tables that work everywhere.
