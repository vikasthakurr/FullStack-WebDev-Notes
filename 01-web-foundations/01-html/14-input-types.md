# HTML Input Types

## What Are Input Types?

The `<input>` element is the most versatile form control in HTML. Its behavior changes entirely based on the `type` attribute — from a simple text field to a color picker, date selector, file upload, or range slider. HTML5 dramatically expanded the available types, reducing the need for JavaScript libraries for common form interactions.

**Analogy**: Think of `<input>` as a Swiss Army knife. The `type` attribute determines which tool you pull out. `type="text"` is the basic blade. `type="email"` is the blade with a built-in validator. `type="date"` is a completely different tool — a calendar. Same element, wildly different behavior depending on the type.

## Why Input Types Matter

- **Built-in validation**: Types like `email`, `url`, and `number` validate automatically.
- **Mobile keyboards**: Mobile browsers show contextual keyboards (number pad for `type="tel"`, @ key for `type="email"`).
- **Native UI controls**: Date pickers, color pickers, and range sliders without JavaScript libraries.
- **Accessibility**: Screen readers announce the input type, helping users understand what to enter.
- **Security**: `type="password"` masks characters. `type="hidden"` excludes fields from display.

## Text-Based Inputs

### `type="text"` — Generic Text

```html
<label for="username">Username:</label>
<input
  type="text"
  id="username"
  name="username"
  maxlength="30"
  placeholder="Choose a username"
/>
```

The default. Accepts any text input with no validation.

### `type="password"` — Masked Input

```html
<label for="pwd">Password:</label>
<input type="password" id="pwd" name="password" minlength="8" required />
```

Characters are masked (dots or asterisks). Prevents shoulder-surfing.

### `type="email"` — Email Validation

```html
<label for="email">Email:</label>
<input
  type="email"
  id="email"
  name="email"
  placeholder="you@example.com"
  required
/>
```

- Browser validates format (must contain @, must have domain).
- Mobile: shows keyboard with @ and .com buttons.
- Supports `multiple` attribute for comma-separated emails.

### `type="url"` — URL Validation

```html
<label for="website">Website:</label>
<input
  type="url"
  id="website"
  name="website"
  placeholder="https://example.com"
/>
```

Validates that input starts with a protocol (`http://` or `https://`).

### `type="search"` — Search Field

```html
<label for="query">Search:</label>
<input type="search" id="query" name="q" placeholder="Search articles..." />
```

Functionally similar to text, but browsers may show a clear (X) button and style it differently.

### `type="tel"` — Phone Number

```html
<label for="phone">Phone:</label>
<input
  type="tel"
  id="phone"
  name="phone"
  pattern="[0-9]{10}"
  placeholder="1234567890"
/>
```

Does NOT validate format (phone formats vary globally). But mobile browsers show the numeric keypad.

## Numeric Inputs

### `type="number"` — Numeric Value

```html
<label for="qty">Quantity:</label>
<input
  type="number"
  id="qty"
  name="quantity"
  min="1"
  max="100"
  step="1"
  value="1"
/>
```

Shows up/down spinner arrows. Validates numeric input.

| Attribute | Purpose                                    |
| --------- | ------------------------------------------ |
| `min`     | Minimum allowed value                      |
| `max`     | Maximum allowed value                      |
| `step`    | Increment amount (e.g., 0.01 for decimals) |
| `value`   | Default/initial value                      |

### `type="range"` — Slider

```html
<label for="volume">Volume: <output id="vol-output">50</output></label>
<input
  type="range"
  id="volume"
  name="volume"
  min="0"
  max="100"
  step="5"
  value="50"
  oninput="document.getElementById('vol-output').value = this.value"
/>
```

Displays a horizontal slider. Good for approximate values where precision is not critical.
