# Links (Anchor Element)

## What Are Links?

The anchor element (`<a>`) creates hyperlinks — the connective tissue of the web. Links let users navigate between pages, download files, send emails, make phone calls, and jump to specific sections within a page.

**Analogy**: If the web is a city, links are the roads connecting buildings (pages). Some roads take you to other buildings in the same neighborhood (relative links). Others take you across the country to a different city entirely (absolute links). The `<a>` tag is how you build these roads.

## Why Links Matter

- They are the fundamental mechanism of the web — "HyperText" in HTML literally means linked text.
- Navigation would not exist without them.
- Search engines use links to discover pages and determine authority (PageRank).
- Accessibility depends on properly labeled, focusable links for keyboard navigation.

## Basic Syntax

```html
<a href="https://example.com">Visit Example</a>
```

The `href` (Hypertext REFerence) attribute specifies the destination.

## Link Types and `href` Values

### External Links (Absolute URLs)

```html
<a href="https://developer.mozilla.org">MDN Web Docs</a>
<a href="https://github.com/username/repo">My GitHub Repo</a>
```

Full URL including protocol (`https://`). Takes the user to a completely different website.

### Internal Links (Relative URLs)

```html
<!-- Same directory -->
<a href="about.html">About</a>

<!-- Subdirectory -->
<a href="blog/first-post.html">First Post</a>

<!-- Parent directory -->
<a href="../index.html">Back to Home</a>

<!-- Root-relative (from site root) -->
<a href="/contact">Contact Us</a>
```

Relative paths do not include the domain — they navigate within your own site.

### Absolute vs Relative Comparison

| Type              | Example                    | Use Case                                                     |
| ----------------- | -------------------------- | ------------------------------------------------------------ |
| Absolute          | `https://example.com/page` | Linking to external sites                                    |
| Root-relative     | `/about/team`              | Internal links (site-wide, regardless of current page depth) |
| Document-relative | `../images/logo.png`       | Internal links (relative to current file)                    |

### Fragment Links (Same Page)

```html
<!-- Jump to a section on the same page -->
<a href="#pricing">Jump to Pricing</a>

<!-- The target section -->
<section id="pricing">
  <h2>Pricing</h2>
  <p>Our plans start at...</p>
</section>

<!-- Jump to top of page -->
<a href="#">Back to top</a>
```

### Email Links (`mailto:`)

```html
<a href="mailto:hello@example.com">Send us an email</a>

<!-- Pre-fill subject and body -->
<a
  href="mailto:support@example.com?subject=Bug%20Report&body=I%20found%20an%20issue..."
>
  Report a Bug
</a>

<!-- Multiple recipients -->
<a href="mailto:person1@example.com,person2@example.com">Email Both</a>
```

### Phone Links (`tel:`)

```html
<a href="tel:+15551234567">Call Us: (555) 123-4567</a>

<!-- On mobile, this triggers the phone dialer -->
<a href="tel:+919876543210">+91 98765 43210</a>
```

Best practice: Use international format with country code (no spaces or dashes in the `href`).

### SMS Links

```html
<a href="sms:+15551234567">Send us a text</a>
<a href="sms:+15551234567?body=Hello">Text with pre-filled message</a>
```

## The `target` Attribute

Controls where the linked page opens.

```html
<!-- Opens in the same tab (default) -->
<a href="/about">About Us</a>

<!-- Opens in a new tab -->
<a href="https://external-site.com" target="_blank">External Resource</a>
```

| Value     | Behavior                               |
| --------- | -------------------------------------- |
| `_self`   | Same tab (default)                     |
| `_blank`  | New tab/window                         |
| `_parent` | Parent frame (for iframes)             |
| `_top`    | Full window (breaks out of all frames) |

### Security with `target="_blank"`

When opening external links in new tabs, always add `rel="noopener noreferrer"`:

```html
<a href="https://external.com" target="_blank" rel="noopener noreferrer">
  External Site
</a>
```

- `noopener`: Prevents the new page from accessing `window.opener` (security risk).
- `noreferrer`: Prevents sending the referrer header (privacy).

Modern browsers handle `noopener` automatically for `target="_blank"`, but explicit declaration is still good practice.

## The `download` Attribute

Forces the browser to download the file instead of navigating to it.

```html
<!-- Download with original filename -->
<a href="/files/report.pdf" download>Download Report</a>

<!-- Download with custom filename -->
<a href="/files/report-2024-q1.pdf" download="Q1-Report.pdf"
  >Download Q1 Report</a
>

<!-- Download an image -->
<a href="/images/infographic.png" download="infographic.png"
  >Download Infographic</a
>
```

**Note**: The `download` attribute only works for same-origin URLs. Cross-origin downloads may be blocked by the browser.

## The `rel` Attribute

Defines the relationship between the current page and the linked page.

```html
<!-- Do not pass SEO authority to this link -->
<a href="https://sponsored.com" rel="nofollow">Sponsored Link</a>

<!-- External link with security attributes -->
<a href="https://other-site.com" target="_blank" rel="noopener noreferrer">
  Visit Other Site
</a>

<!-- Mark as sponsored content -->
<a href="https://product.com" rel="sponsored">Check out this product</a>

<!-- Mark as user-generated content -->
<a href="https://user-blog.com" rel="ugc">User's blog</a>
```

| Value        | Purpose                                           |
| ------------ | ------------------------------------------------- |
| `nofollow`   | Tell search engines not to pass ranking authority |
| `noopener`   | Prevent new page from accessing opener window     |
| `noreferrer` | Do not send referrer information                  |
| `sponsored`  | Mark paid/sponsored links (Google SEO)            |
| `ugc`        | Mark user-generated content links                 |
| `external`   | Indicate link goes to another domain              |

## Link States (CSS)

Links have four distinct states that can be styled:

```css
/* Unvisited link */
a:link {
  color: blue;
}

/* Visited link */
a:visited {
  color: purple;
}

/* Mouse hovering over link */
a:hover {
  color: darkblue;
  text-decoration: underline;
}

/* Link being clicked (active/pressed) */
a:active {
  color: red;
}

/* Link focused via keyboard (Tab key) */
a:focus {
  outline: 2px solid blue;
  outline-offset: 2px;
}
```

**Important**: Always maintain a visible `:focus` style for keyboard accessibility. Never use `outline: none` without providing an alternative focus indicator.

## Accessible Link Text

```html
<!-- BAD: Non-descriptive link text -->
<a href="/pricing">Click here</a>
<a href="/docs">Read more</a>

<!-- GOOD: Descriptive link text -->
<a href="/pricing">View our pricing plans</a>
<a href="/docs">Read the API documentation</a>

<!-- If link text must be short, use aria-label -->
<a href="/pricing" aria-label="View pricing plans for all tiers">Pricing</a>
```

Screen reader users often navigate by tabbing through links. They hear link text out of context, so "click here" and "read more" are meaningless without surrounding text.

## Best Practices

- Use descriptive link text that makes sense out of context.
- Use relative URLs for internal links (easier to maintain, works across environments).
- Always add `rel="noopener noreferrer"` with `target="_blank"`.
- Use the `download` attribute for file downloads, not JavaScript hacks.
- Make links visually distinguishable from surrounding text (color + underline).
- Ensure all links are keyboard-focusable with a visible focus indicator.
- Do not use links for actions that do not navigate (use `<button>` instead).

## Common Mistakes

| Mistake                          | Why It Is Wrong                          | Fix                                |
| -------------------------------- | ---------------------------------------- | ---------------------------------- |
| `<a href="#">` as a button       | Links navigate; buttons perform actions  | Use `<button>` for actions         |
| "Click here" link text           | Meaningless for screen reader users      | Write descriptive link text        |
| Missing `href` attribute         | Element is not focusable or clickable    | Always include `href`              |
| No focus styles                  | Keyboard users cannot see where they are | Keep or replace `outline` styles   |
| `target="_blank"` without `rel`  | Security vulnerability (opener access)   | Add `rel="noopener noreferrer"`    |
| Using link for non-navigation JS | Confuses semantics                       | Use `<button>` with event listener |
| Broken relative paths            | Links 404 when site structure changes    | Use root-relative paths (`/about`) |

## Summary

- The `<a>` element is the hyperlink — the fundamental connector of the web.
- `href` accepts URLs (absolute/relative), fragments (`#id`), `mailto:`, `tel:`, and `sms:` protocols.
- `target="_blank"` opens in new tab but requires `rel="noopener noreferrer"` for security.
- The `download` attribute forces file download instead of navigation.
- Links must have descriptive text, visible focus styles, and proper `rel` attributes for SEO and security.
- If it navigates somewhere, use `<a>`. If it performs an action, use `<button>`. This distinction is fundamental to accessible web development.
