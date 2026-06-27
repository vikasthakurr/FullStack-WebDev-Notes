# Bootstrap 5 Alerts

## What Is It

Alerts are pre-styled notification components that communicate important information to users. They use contextual colors (success, danger, warning, info) to signal the nature of the message -- whether something went right, went wrong, or needs attention.

**Analogy:** Alerts are like the colored lights on a car dashboard. Green means everything is fine, yellow warns you to pay attention, and red says stop and deal with the problem. Users instinctively understand this color language.

## Why It Matters

- Users need feedback: form submitted successfully, session expiring, input invalid.
- Contextual colors eliminate the need for users to read every word to understand severity.
- Dismissible alerts let users clear messages once acknowledged, keeping the UI clean.
- Accessible by default -- Bootstrap applies appropriate ARIA roles.

---

## Basic Alerts

```html
<div class="alert alert-primary" role="alert">
  This is a primary alert -- check it out!
</div>

<div class="alert alert-secondary" role="alert">
  This is a secondary alert -- less emphasis.
</div>

<div class="alert alert-success" role="alert">
  Operation completed successfully.
</div>

<div class="alert alert-danger" role="alert">
  Something went wrong. Please try again.
</div>

<div class="alert alert-warning" role="alert">
  Your session will expire in 5 minutes.
</div>

<div class="alert alert-info" role="alert">
  A new version is available. Refresh to update.
</div>

<div class="alert alert-light" role="alert">
  A light alert for subtle messages.
</div>

<div class="alert alert-dark" role="alert">
  A dark alert for high-contrast contexts.
</div>
```

The `role="alert"` attribute tells assistive technologies to announce the content immediately.

---

## Alert with Links

Use `.alert-link` to style links so they match the alert's color scheme.

```html
<div class="alert alert-success" role="alert">
  Your profile has been updated.
  <a href="/profile" class="alert-link">View your profile</a>.
</div>

<div class="alert alert-danger" role="alert">
  Payment failed.
  <a href="/billing" class="alert-link">Update your payment method</a>.
</div>
```

Without `.alert-link`, links would use the default blue color and look out of place.

---

## Alert with Additional Content

Alerts can contain headings, paragraphs, and dividers.

```html
<div class="alert alert-success" role="alert">
  <h4 class="alert-heading">Well done!</h4>
  <p>
    You successfully submitted your application. We will review it and get back
    to you within 3 business days.
  </p>
  <hr />
  <p class="mb-0">
    Meanwhile, check out our <a href="/faq" class="alert-link">FAQ page</a>
    for common questions.
  </p>
</div>
```

---

## Dismissible Alerts

Users can close dismissible alerts by clicking an X button.

```html
<div class="alert alert-warning alert-dismissible fade show" role="alert">
  <strong>Warning!</strong> Your password expires in 3 days.
  <button
    type="button"
    class="btn-close"
    data-bs-dismiss="alert"
    aria-label="Close"
  ></button>
</div>
```

Key parts:

- `.alert-dismissible` -- adds right padding for the close button.
- `.fade .show` -- enables smooth fade-out animation on dismiss.
- `data-bs-dismiss="alert"` -- tells Bootstrap's JavaScript to close the alert.
- `aria-label="Close"` -- accessibility label for the button.

---

## Alert with Icons

Bootstrap does not include icons by default, but you can pair alerts with Bootstrap Icons or any icon library.

```html
<!-- Using Bootstrap Icons -->
<div class="alert alert-success d-flex align-items-center" role="alert">
  <svg class="bi flex-shrink-0 me-2" width="24" height="24">
    <use xlink:href="#check-circle-fill" />
  </svg>
  <div>Your changes have been saved successfully.</div>
</div>

<div class="alert alert-danger d-flex align-items-center" role="alert">
  <svg class="bi flex-shrink-0 me-2" width="24" height="24">
    <use xlink:href="#exclamation-triangle-fill" />
  </svg>
  <div>An error occurred while processing your request.</div>
</div>

<!-- Using inline SVG shorthand with Bootstrap Icons CDN -->
<div class="alert alert-info d-flex align-items-center" role="alert">
  <i class="bi bi-info-circle-fill me-2"></i>
  <div>New features are available in the latest update.</div>
</div>
```

The `d-flex align-items-center` pattern vertically centers the icon with the text.

---

## Triggering Alerts via JavaScript

### Closing Programmatically

```html
<div
  class="alert alert-info alert-dismissible fade show"
  role="alert"
  id="myAlert"
>
  This alert can be closed via JavaScript.
  <button
    type="button"
    class="btn-close"
    data-bs-dismiss="alert"
    aria-label="Close"
  ></button>
</div>

<script>
  // Get the alert instance
  const alertElement = document.getElementById("myAlert");
  const bsAlert = new bootstrap.Alert(alertElement);

  // Close it after 5 seconds
  setTimeout(() => {
    bsAlert.close();
  }, 5000);
</script>
```

### Listening to Alert Events

```html
<script>
  const alertEl = document.getElementById("myAlert");

  alertEl.addEventListener("close.bs.alert", () => {
    console.log("Alert is about to close");
  });

  alertEl.addEventListener("closed.bs.alert", () => {
    console.log("Alert has been closed and removed from DOM");
  });
</script>
```

Events:

- `close.bs.alert` -- fires immediately when `.close()` is called.
- `closed.bs.alert` -- fires after the fade transition completes and the element is removed.

---

## Dynamic Alert Creation

```html
<div id="alert-container"></div>
<button class="btn btn-primary" onclick="showAlert()">Show Alert</button>

<script>
  function showAlert() {
    const container = document.getElementById("alert-container");
    const alert = document.createElement("div");
    alert.className = "alert alert-success alert-dismissible fade show";
    alert.setAttribute("role", "alert");
    alert.innerHTML = `
      Action completed successfully!
      <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
    `;
    container.appendChild(alert);

    // Auto-dismiss after 3 seconds
    setTimeout(() => {
      const bsAlert = new bootstrap.Alert(alert);
      bsAlert.close();
    }, 3000);
  }
</script>
```

---

## Best Practices

1. Use the right contextual class: `success` for positive outcomes, `danger` for errors, `warning` for caution, `info` for neutral information.
2. Keep alert text concise -- users scan, they do not read novels.
3. Make critical alerts (errors) non-dismissible or ensure users acknowledge them.
4. Use `role="alert"` for important real-time messages; use `role="status"` for less urgent updates.
5. Combine with icons to reinforce meaning for users who may not distinguish colors well (colorblind accessibility).

## Common Mistakes

| Mistake                                        | Why It Is Wrong                                | Fix                                       |
| ---------------------------------------------- | ---------------------------------------------- | ----------------------------------------- |
| Missing `role="alert"`                         | Screen readers will not announce the message   | Always include the role attribute         |
| Forgetting `.fade .show` on dismissible alerts | Alert disappears abruptly instead of fading    | Add both classes                          |
| Using alerts for permanent UI elements         | Alerts imply temporary/actionable messages     | Use cards or callouts for persistent info |
| Overusing alerts on a single page              | Creates "alert fatigue"; users ignore them all | Limit to 1-2 visible alerts at a time     |

---

## Summary

Bootstrap alerts give you a standardized way to communicate feedback to users. With eight contextual color variants, dismissible behavior, icon support, and JavaScript events, you can handle everything from subtle informational notices to critical error messages. The key is choosing the right color for the right context and keeping messages short enough to be read at a glance.
