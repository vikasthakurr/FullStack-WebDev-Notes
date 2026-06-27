# Bootstrap 5 Forms

## What Is It

Bootstrap forms provide a comprehensive set of classes for styling form controls -- inputs, selects, checkboxes, radios, switches, textareas, and more. They include layout helpers, floating labels, input groups, and built-in validation states that create polished, accessible forms without custom CSS.

**Analogy:** Raw HTML form elements are like unfurnished apartments -- functional but bare. Bootstrap form classes are like an interior designer who makes every input consistently styled, properly spaced, and visually clear about what the user should do next.

## Why It Matters

- Forms are how users send data to your application -- they must be clear and error-free.
- Consistent styling across inputs, selects, and checkboxes builds user confidence.
- Validation states provide immediate visual feedback (green checkmarks, red warnings).
- Accessibility features (labels, descriptions, ARIA) come built in.

---

## Basic Form Controls

### Text Input

```html
<div class="mb-3">
  <label for="fullName" class="form-label">Full Name</label>
  <input
    type="text"
    class="form-control"
    id="fullName"
    placeholder="John Doe"
  />
</div>
```

### Email Input with Help Text

```html
<div class="mb-3">
  <label for="emailInput" class="form-label">Email address</label>
  <input
    type="email"
    class="form-control"
    id="emailInput"
    placeholder="name@example.com"
    aria-describedby="emailHelp"
  />
  <div id="emailHelp" class="form-text">
    We will never share your email with anyone else.
  </div>
</div>
```

### Password

```html
<div class="mb-3">
  <label for="passwordInput" class="form-label">Password</label>
  <input type="password" class="form-control" id="passwordInput" />
</div>
```

### Textarea

```html
<div class="mb-3">
  <label for="bio" class="form-label">Bio</label>
  <textarea
    class="form-control"
    id="bio"
    rows="3"
    placeholder="Tell us about yourself"
  ></textarea>
</div>
```

### Disabled and Readonly

```html
<input class="form-control" type="text" value="Disabled input" disabled />
<input class="form-control" type="text" value="Readonly input" readonly />
<input
  class="form-control-plaintext"
  type="text"
  value="Readonly plain text"
  readonly
/>
```

---

## Select

```html
<!-- Standard select -->
<div class="mb-3">
  <label for="country" class="form-label">Country</label>
  <select class="form-select" id="country">
    <option selected>Choose a country</option>
    <option value="us">United States</option>
    <option value="uk">United Kingdom</option>
    <option value="in">India</option>
  </select>
</div>

<!-- Multiple select -->
<select class="form-select" multiple size="3">
  <option>Option 1</option>
  <option>Option 2</option>
  <option>Option 3</option>
</select>

<!-- Select sizes -->
<select class="form-select form-select-lg">
  ...
</select>
<select class="form-select form-select-sm">
  ...
</select>
```

---

## Checkboxes and Radios

### Checkboxes

```html
<div class="form-check">
  <input class="form-check-input" type="checkbox" id="agreeTerms" />
  <label class="form-check-label" for="agreeTerms">
    I agree to the terms and conditions
  </label>
</div>

<div class="form-check">
  <input class="form-check-input" type="checkbox" id="newsletter" checked />
  <label class="form-check-label" for="newsletter">
    Subscribe to newsletter
  </label>
</div>
```

### Radio Buttons

```html
<div class="form-check">
  <input
    class="form-check-input"
    type="radio"
    name="plan"
    id="planFree"
    checked
  />
  <label class="form-check-label" for="planFree">Free Plan</label>
</div>
<div class="form-check">
  <input class="form-check-input" type="radio" name="plan" id="planPro" />
  <label class="form-check-label" for="planPro">Pro Plan</label>
</div>
<div class="form-check">
  <input
    class="form-check-input"
    type="radio"
    name="plan"
    id="planEnterprise"
  />
  <label class="form-check-label" for="planEnterprise">Enterprise Plan</label>
</div>
```

### Inline Checkboxes / Radios

```html
<div class="form-check form-check-inline">
  <input class="form-check-input" type="checkbox" id="html" value="html" />
  <label class="form-check-label" for="html">HTML</label>
</div>
<div class="form-check form-check-inline">
  <input class="form-check-input" type="checkbox" id="css" value="css" />
  <label class="form-check-label" for="css">CSS</label>
</div>
<div class="form-check form-check-inline">
  <input class="form-check-input" type="checkbox" id="js" value="js" />
  <label class="form-check-label" for="js">JavaScript</label>
</div>
```

---

## Switches

Switches are styled checkboxes that look like toggle switches.

```html
<div class="form-check form-switch">
  <input class="form-check-input" type="checkbox" role="switch" id="darkMode" />
  <label class="form-check-label" for="darkMode">Enable Dark Mode</label>
</div>

<div class="form-check form-switch">
  <input
    class="form-check-input"
    type="checkbox"
    role="switch"
    id="notifications"
    checked
  />
  <label class="form-check-label" for="notifications">Push Notifications</label>
</div>

<div class="form-check form-switch">
  <input
    class="form-check-input"
    type="checkbox"
    role="switch"
    id="maintenance"
    disabled
  />
  <label class="form-check-label" for="maintenance">Maintenance Mode</label>
</div>
```

---

## Input Groups

Extend form controls by adding text, buttons, or icons on either side.

```html
<!-- Text addon -->
<div class="input-group mb-3">
  <span class="input-group-text">@</span>
  <input type="text" class="form-control" placeholder="Username" />
</div>

<!-- Text addon on the right -->
<div class="input-group mb-3">
  <input type="text" class="form-control" placeholder="Amount" />
  <span class="input-group-text">.00</span>
</div>

<!-- Both sides -->
<div class="input-group mb-3">
  <span class="input-group-text">$</span>
  <input type="text" class="form-control" placeholder="Price" />
  <span class="input-group-text">.00</span>
</div>

<!-- Button addon -->
<div class="input-group mb-3">
  <input type="text" class="form-control" placeholder="Search..." />
  <button class="btn btn-outline-secondary" type="button">Go</button>
</div>

<!-- Multiple inputs -->
<div class="input-group mb-3">
  <span class="input-group-text">First and Last Name</span>
  <input type="text" class="form-control" placeholder="First" />
  <input type="text" class="form-control" placeholder="Last" />
</div>
```

### Input Group Sizing

```html
<div class="input-group input-group-sm mb-3">
  <span class="input-group-text">Small</span>
  <input type="text" class="form-control" />
</div>

<div class="input-group input-group-lg mb-3">
  <span class="input-group-text">Large</span>
  <input type="text" class="form-control" />
</div>
```

---

## Floating Labels

Labels that float above the input when focused or filled -- a modern Material Design pattern.

```html
<!-- Text input -->
<div class="form-floating mb-3">
  <input
    type="email"
    class="form-control"
    id="floatingEmail"
    placeholder="name@example.com"
  />
  <label for="floatingEmail">Email address</label>
</div>

<!-- Password -->
<div class="form-floating mb-3">
  <input
    type="password"
    class="form-control"
    id="floatingPassword"
    placeholder="Password"
  />
  <label for="floatingPassword">Password</label>
</div>

<!-- Textarea -->
<div class="form-floating mb-3">
  <textarea
    class="form-control"
    id="floatingTextarea"
    placeholder="Comment"
    style="height: 100px"
  ></textarea>
  <label for="floatingTextarea">Comments</label>
</div>

<!-- Select -->
<div class="form-floating mb-3">
  <select class="form-select" id="floatingSelect">
    <option selected>Select a role</option>
    <option value="admin">Admin</option>
    <option value="editor">Editor</option>
  </select>
  <label for="floatingSelect">Role</label>
</div>
```

Important: The `<input>` must come before the `<label>`, and the `placeholder` attribute is required (even if not visible) for floating behavior to work.

---

## Form Sizing

```html
<!-- Large inputs -->
<input
  class="form-control form-control-lg"
  type="text"
  placeholder="Large input"
/>

<!-- Default -->
<input class="form-control" type="text" placeholder="Default input" />

<!-- Small inputs -->
<input
  class="form-control form-control-sm"
  type="text"
  placeholder="Small input"
/>
```

---

## Validation States

### Client-Side Validation

```html
<form class="needs-validation" novalidate>
  <div class="mb-3">
    <label for="validationName" class="form-label">Name</label>
    <input type="text" class="form-control" id="validationName" required />
    <div class="valid-feedback">Looks good!</div>
    <div class="invalid-feedback">Please provide your name.</div>
  </div>

  <div class="mb-3">
    <label for="validationEmail" class="form-label">Email</label>
    <input type="email" class="form-control" id="validationEmail" required />
    <div class="invalid-feedback">Please provide a valid email.</div>
  </div>

  <div class="mb-3 form-check">
    <input type="checkbox" class="form-check-input" id="agreeCheck" required />
    <label class="form-check-label" for="agreeCheck">Agree to terms</label>
    <div class="invalid-feedback">You must agree before submitting.</div>
  </div>

  <button class="btn btn-primary" type="submit">Submit</button>
</form>

<script>
  // Bootstrap validation script
  const forms = document.querySelectorAll(".needs-validation");
  Array.from(forms).forEach((form) => {
    form.addEventListener(
      "submit",
      (event) => {
        if (!form.checkValidity()) {
          event.preventDefault();
          event.stopPropagation();
        }
        form.classList.add("was-validated");
      },
      false,
    );
  });
</script>
```

### Server-Side Validation (Manual Classes)

```html
<div class="mb-3">
  <label for="serverName" class="form-label">Username</label>
  <input
    type="text"
    class="form-control is-valid"
    id="serverName"
    value="alice123"
  />
  <div class="valid-feedback">Username is available!</div>
</div>

<div class="mb-3">
  <label for="serverEmail" class="form-label">Email</label>
  <input
    type="email"
    class="form-control is-invalid"
    id="serverEmail"
    value="not-an-email"
  />
  <div class="invalid-feedback">Please provide a valid email address.</div>
</div>
```

---

## Form Layout

### Vertical (Default)

Standard stacking with `mb-3` spacing.

### Horizontal Form

```html
<form>
  <div class="row mb-3">
    <label for="hEmail" class="col-sm-2 col-form-label">Email</label>
    <div class="col-sm-10">
      <input type="email" class="form-control" id="hEmail" />
    </div>
  </div>
  <div class="row mb-3">
    <label for="hPassword" class="col-sm-2 col-form-label">Password</label>
    <div class="col-sm-10">
      <input type="password" class="form-control" id="hPassword" />
    </div>
  </div>
  <div class="row">
    <div class="col-sm-10 offset-sm-2">
      <button type="submit" class="btn btn-primary">Sign in</button>
    </div>
  </div>
</form>
```

### Inline Form

```html
<form class="row row-cols-lg-auto g-3 align-items-center">
  <div class="col-12">
    <input type="text" class="form-control" placeholder="Username" />
  </div>
  <div class="col-12">
    <input type="password" class="form-control" placeholder="Password" />
  </div>
  <div class="col-12">
    <button type="submit" class="btn btn-primary">Login</button>
  </div>
</form>
```

---

## Best Practices

1. Always pair inputs with `<label>` elements -- never rely on placeholder alone.
2. Use `aria-describedby` to link help text to its input for screen readers.
3. Use `.form-text` for hint text below inputs, not paragraphs or spans.
4. Prefer client-side validation with `.needs-validation` for instant feedback.
5. Use floating labels for clean, modern forms -- but ensure placeholder is set.

## Common Mistakes

| Mistake                                               | Why It Is Wrong                                      | Fix                                     |
| ----------------------------------------------------- | ---------------------------------------------------- | --------------------------------------- |
| Using placeholder instead of labels                   | Placeholder disappears on focus; fails accessibility | Always use `<label class="form-label">` |
| Forgetting `for` / `id` pairing                       | Clicking label does not focus the input              | Match `for` attribute with input `id`   |
| Missing `novalidate` with custom validation           | Browser native validation competes with Bootstrap's  | Add `novalidate` to the form            |
| Putting `<label>` before `<input>` in floating labels | Floating effect will not work                        | Input must come first                   |

---

## Summary

Bootstrap forms give you a complete toolkit for building accessible, well-styled forms. From basic inputs with labels and help text, to switches, input groups, floating labels, and validation states, everything works together cohesively. The key rule: always use semantic HTML (`<label>`, `<fieldset>`, proper input types) and let Bootstrap classes handle the visual presentation.
