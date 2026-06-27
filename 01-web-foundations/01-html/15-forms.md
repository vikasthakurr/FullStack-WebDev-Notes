# HTML Forms

## What Are Forms?

Forms are HTML elements that collect user input and send it to a server for processing. They are the primary mechanism for two-way communication between a user and a web application — login pages, search bars, checkout flows, surveys, and contact pages all rely on forms.

**Analogy:** A form is like a paper application at a government office. It has labeled fields (name, address, date of birth), some are required, some are optional, and when you are done you submit it to a clerk (the server) who processes it.

---

## Why Forms Matter

- They are the **only native way** to collect structured user input in HTML.
- They provide built-in browser features: validation, autofill, keyboard navigation, and accessibility.
- They work without JavaScript — a `<form>` can submit data to a server using plain HTTP.
- Screen readers understand form semantics and guide users through fields.

---

## The `<form>` Element

```html
<form action="/submit" method="POST">
  <!-- form controls go here -->
</form>
```

### Attributes

| Attribute      | Purpose                                               | Example                         |
| -------------- | ----------------------------------------------------- | ------------------------------- |
| `action`       | URL where form data is sent                           | `action="/api/login"`           |
| `method`       | HTTP method (`GET` or `POST`)                         | `method="POST"`                 |
| `enctype`      | How form data is encoded (important for file uploads) | `enctype="multipart/form-data"` |
| `novalidate`   | Disables built-in browser validation                  | `novalidate`                    |
| `autocomplete` | Enables/disables browser autofill                     | `autocomplete="off"`            |
| `target`       | Where to display the response                         | `target="_blank"`               |

### `GET` vs `POST`

- **GET** — appends data to the URL as query parameters. Visible in the address bar. Use for searches and filters.
- **POST** — sends data in the request body. Not visible in the URL. Use for login, registration, file uploads, and any sensitive or large data.

---

## Form Controls

### Labels

Always pair inputs with labels. This is critical for accessibility.

```html
<!-- Method 1: for/id association -->
<label for="email">Email:</label>
<input type="email" id="email" name="email" />

<!-- Method 2: wrapping (implicit association) -->
<label>
  Email:
  <input type="email" name="email" />
</label>
```

The `for` attribute must match the input's `id`. Clicking the label focuses the input — essential for small touch targets and screen readers.

---

### `<input>` (Most Common)

```html
<input
  type="text"
  name="username"
  id="username"
  placeholder="Enter username"
  required
/>
```

Refer to the [Input Types](./14-input-types.md) chapter for all `type` values.

---

### `<textarea>`

For multi-line text input.

```html
<label for="message">Message:</label>
<textarea
  id="message"
  name="message"
  rows="5"
  cols="40"
  placeholder="Type your message..."
></textarea>
```

- `rows` and `cols` set the visible size (can be overridden with CSS).
- Content goes between the tags (not in a `value` attribute).

---

### `<select>` and `<option>`

Dropdown menus.

```html
<label for="country">Country:</label>
<select id="country" name="country">
  <option value="" disabled selected>Choose a country</option>
  <option value="in">India</option>
  <option value="us">United States</option>
  <option value="uk">United Kingdom</option>
</select>
```

- `disabled selected` on the first option creates a placeholder.
- Add `multiple` attribute to allow selecting more than one option.
- Group related options with `<optgroup>`:

```html
<select name="car">
  <optgroup label="Swedish Cars">
    <option value="volvo">Volvo</option>
    <option value="saab">Saab</option>
  </optgroup>
  <optgroup label="German Cars">
    <option value="bmw">BMW</option>
    <option value="audi">Audi</option>
  </optgroup>
</select>
```

---

### `<button>`

```html
<button type="submit">Submit</button>
<button type="reset">Reset</button>
<button type="button">Click Me</button>
```

| Type     | Behavior                                     |
| -------- | -------------------------------------------- |
| `submit` | Submits the form (default inside a `<form>`) |
| `reset`  | Resets all fields to their initial values    |
| `button` | Does nothing unless JavaScript handles it    |

**Always specify `type`** on buttons inside forms. Without it, browsers default to `submit`, which may cause accidental form submissions.

---

### `<fieldset>` and `<legend>`

Group related fields visually and semantically.

```html
<fieldset>
  <legend>Personal Information</legend>

  <label for="fname">First Name:</label>
  <input type="text" id="fname" name="fname" />

  <label for="lname">Last Name:</label>
  <input type="text" id="lname" name="lname" />
</fieldset>

<fieldset>
  <legend>Account Details</legend>

  <label for="email">Email:</label>
  <input type="email" id="email" name="email" />

  <label for="password">Password:</label>
  <input type="password" id="password" name="password" />
</fieldset>
```

- `<fieldset>` draws a border around grouped controls.
- `<legend>` provides a caption for the group.
- Screen readers announce the legend when entering the fieldset.

---

### `<datalist>`

Provides autocomplete suggestions for an input.

```html
<label for="browser">Choose your browser:</label>
<input list="browsers" id="browser" name="browser" />

<datalist id="browsers">
  <option value="Chrome"></option>
  <option value="Firefox"></option>
  <option value="Safari"></option>
  <option value="Edge"></option>
  <option value="Opera"></option>
</datalist>
```

- The `list` attribute on `<input>` points to the `<datalist>` id.
- Unlike `<select>`, the user can still type a custom value.

---

## Form Validation (Built-in)

HTML5 provides native validation without JavaScript.

### Validation Attributes

| Attribute     | Purpose                       | Example                         |
| ------------- | ----------------------------- | ------------------------------- |
| `required`    | Field must not be empty       | `required`                      |
| `minlength`   | Minimum character count       | `minlength="8"`                 |
| `maxlength`   | Maximum character count       | `maxlength="100"`               |
| `min` / `max` | Minimum/maximum numeric value | `min="1" max="100"`             |
| `pattern`     | Regex the value must match    | `pattern="[A-Za-z]{3,}"`        |
| `type`        | Built-in format validation    | `type="email"` validates format |
| `step`        | Allowed number increments     | `step="0.01"`                   |

### Example with Validation

```html
<form action="/register" method="POST">
  <label for="username">Username (3–15 chars, letters only):</label>
  <input
    type="text"
    id="username"
    name="username"
    required
    minlength="3"
    maxlength="15"
    pattern="[A-Za-z]+"
    title="Only letters allowed, 3 to 15 characters"
  />

  <label for="age">Age (18–120):</label>
  <input type="number" id="age" name="age" required min="18" max="120" />

  <label for="email">Email:</label>
  <input type="email" id="email" name="email" required />

  <button type="submit">Register</button>
</form>
```

- The `title` attribute shows as a tooltip explaining the pattern.
- The browser blocks submission and shows error messages automatically.

---

## Complete Form Example

```html
<form action="/api/contact" method="POST">
  <fieldset>
    <legend>Contact Us</legend>

    <label for="name">Full Name:</label>
    <input type="text" id="name" name="name" required placeholder="John Doe" />

    <label for="email">Email:</label>
    <input
      type="email"
      id="email"
      name="email"
      required
      placeholder="john@example.com"
    />

    <label for="subject">Subject:</label>
    <select id="subject" name="subject" required>
      <option value="" disabled selected>Select a subject</option>
      <option value="general">General Inquiry</option>
      <option value="support">Technical Support</option>
      <option value="billing">Billing</option>
    </select>

    <label for="message">Message:</label>
    <textarea
      id="message"
      name="message"
      rows="5"
      required
      minlength="10"
      placeholder="Write your message..."
    ></textarea>

    <label>
      <input type="checkbox" name="agree" required />
      I agree to the terms and conditions
    </label>

    <button type="submit">Send Message</button>
  </fieldset>
</form>
```

---

## How Form Data Is Sent

### GET Request

```
GET /search?q=javascript&category=tutorials HTTP/1.1
```

Data appears in the URL as key-value pairs.

### POST Request

```
POST /api/contact HTTP/1.1
Content-Type: application/x-www-form-urlencoded

name=John+Doe&email=john%40example.com&message=Hello+there
```

Data is in the request body, URL-encoded by default.

### For File Uploads

```html
<form action="/upload" method="POST" enctype="multipart/form-data">
  <input type="file" name="avatar" accept="image/*" />
  <button type="submit">Upload</button>
</form>
```

`enctype="multipart/form-data"` is **required** for file uploads.

---

## Best Practices

1. **Always use `<label>`** — never rely on placeholder text alone as a label.
2. **Use `name` attributes** on all inputs — without `name`, the data is not sent on submission.
3. **Group related fields** with `<fieldset>` and `<legend>`.
4. **Use appropriate `type` values** — `email`, `tel`, `url` trigger correct mobile keyboards.
5. **Add `autocomplete` attributes** — helps browsers autofill correctly (e.g., `autocomplete="email"`).
6. **Validate on the server too** — client-side validation is for UX; it can be bypassed.
7. **Provide clear error messages** — use the `title` attribute with `pattern` for guidance.

---

## Common Mistakes

| Mistake                               | Why It Is Wrong                                          | Fix                                        |
| ------------------------------------- | -------------------------------------------------------- | ------------------------------------------ |
| Missing `name` attribute              | Data will not be included in form submission             | Add `name` to every input                  |
| Placeholder as the only label         | Disappears when typing; invisible to some screen readers | Always use `<label>`                       |
| Not specifying button `type`          | May accidentally submit form                             | Use `type="button"` for non-submit buttons |
| Using `GET` for sensitive data        | Passwords/tokens appear in URL and browser history       | Use `POST` for sensitive data              |
| Forgetting `enctype` for file uploads | Files will not be transmitted correctly                  | Add `enctype="multipart/form-data"`        |
| Relying only on client validation     | Can be bypassed by disabling JavaScript or editing HTML  | Always validate server-side                |

---

## Summary

- `<form>` is the container for all user input — it defines where and how data is sent.
- Use `method="POST"` for sensitive or large data, `method="GET"` for searches and bookmarkable queries.
- Label every input, group related fields with `<fieldset>`, and use appropriate input types.
- HTML5 validation (`required`, `pattern`, `min`, `max`) provides instant feedback without JavaScript.
- Always validate on the server — client-side validation is a UX feature, not a security feature.
- The `name` attribute is what makes data submittable — without it, the input is invisible to the server.
