# React Testing Library (RTL)

## What Is React Testing Library?

React Testing Library (RTL) is a testing utility for React that encourages testing **behavior** (what users see and do) rather than **implementation details** (internal state, lifecycle methods).

### Philosophy

> "The more your tests resemble the way your software is used, the more confidence they can give you." — Kent C. Dodds

| Traditional Testing                   | RTL Approach                               |
| ------------------------------------- | ------------------------------------------ |
| Test component state directly         | Test what the user sees on screen          |
| Shallow render, mock children         | Full render, test real behavior            |
| Query by class name or component name | Query by role, label, text (accessibility) |
| Assert internal variables             | Assert visible output                      |

### Analogy

Traditional testing is like checking a car by inspecting the engine internals. RTL testing is like driving the car — does the steering wheel turn? Do the brakes stop the car? That is what users care about.

---

## Setup with Vite (Vitest + RTL)

### Installation

```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom
npm install -D @testing-library/user-event jsdom
```

### Configure Vitest

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: "jsdom",
    setupFiles: "./src/test/setup.ts",
  },
});
```

### Setup File

```typescript
// src/test/setup.ts
import "@testing-library/jest-dom";
```

This adds custom matchers like `toBeInTheDocument()`, `toHaveTextContent()`, `toBeVisible()`, etc.

### TypeScript Config

```json
// tsconfig.json (add to compilerOptions)
{
  "compilerOptions": {
    "types": ["vitest/globals", "@testing-library/jest-dom"]
  }
}
```

### Package.json Scripts

```json
{
  "scripts": {
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest --coverage"
  }
}
```

---

## render() and screen Queries

### Basic Test Structure

```tsx
import { render, screen } from "@testing-library/react";
import { Greeting } from "./Greeting";

describe("Greeting", () => {
  it("renders the welcome message", () => {
    render(<Greeting name="Vikas" />);

    expect(screen.getByText("Hello, Vikas!")).toBeInTheDocument();
  });
});
```

### Query Types

RTL provides three categories of queries, differing in behavior when the element is not found:

| Query Type | No Match       | 1 Match         | Multiple Matches | Async?          |
| ---------- | -------------- | --------------- | ---------------- | --------------- |
| `getBy*`   | Throws error   | Returns element | Throws error     | No              |
| `queryBy*` | Returns `null` | Returns element | Throws error     | No              |
| `findBy*`  | Throws error   | Returns element | Throws error     | Yes (awaitable) |

**Plural versions** (`getAllBy*`, `queryAllBy*`, `findAllBy*`) return arrays.

### When to Use Each

```tsx
// getBy — element SHOULD exist
expect(screen.getByRole("button", { name: "Submit" })).toBeInTheDocument();

// queryBy — asserting element does NOT exist
expect(screen.queryByText("Error message")).not.toBeInTheDocument();

// findBy — element appears AFTER async operation
const message = await screen.findByText("Data loaded!");
expect(message).toBeInTheDocument();
```

---

## Query Priority (Accessibility First)

RTL recommends queries in this priority order — matching how users and assistive technologies find elements:

### 1. Accessible to Everyone

```tsx
// getByRole — best choice (how screen readers see the page)
screen.getByRole("button", { name: "Submit" });
screen.getByRole("heading", { level: 2 });
screen.getByRole("textbox", { name: "Email" });
screen.getByRole("checkbox", { name: "Remember me" });

// getByLabelText — for form fields
screen.getByLabelText("Email address");

// getByPlaceholderText — when no label exists
screen.getByPlaceholderText("Search...");

// getByText — for non-interactive elements
screen.getByText("Welcome back!");
```

### 2. Semantic Queries

```tsx
// getByAltText — images
screen.getByAltText("User avatar");

// getByTitle — elements with title attribute
screen.getByTitle("Close");

// getByDisplayValue — current value of input
screen.getByDisplayValue("vikas@example.com");
```

### 3. Test IDs (Last Resort)

```tsx
// getByTestId — only when nothing else works
screen.getByTestId("custom-dropdown");
```

```tsx
// Component must have data-testid attribute
<div data-testid="custom-dropdown">...</div>
```

### Priority Diagram

```
Best ──────────────────────────── Worst
getByRole > getByLabelText > getByText > getByTestId
(accessible)   (form fields)   (visible)   (escape hatch)
```

---

## User Interactions with userEvent

`userEvent` simulates real user interactions (more realistic than `fireEvent`).

### Setup

```tsx
import userEvent from "@testing-library/user-event";

it("submits the form", async () => {
  const user = userEvent.setup();
  render(<LoginForm />);

  await user.type(screen.getByLabelText("Email"), "vikas@test.com");
  await user.type(screen.getByLabelText("Password"), "secret123");
  await user.click(screen.getByRole("button", { name: "Login" }));

  expect(screen.getByText("Welcome!")).toBeInTheDocument();
});
```

### Common Actions

```tsx
const user = userEvent.setup();

// Click
await user.click(element);
await user.dblClick(element);

// Type into input
await user.type(screen.getByLabelText("Name"), "Vikas");

// Clear and type
await user.clear(screen.getByLabelText("Name"));
await user.type(screen.getByLabelText("Name"), "New Name");

// Select from dropdown
await user.selectOptions(screen.getByRole("combobox"), "admin");

// Checkbox / Radio
await user.click(screen.getByRole("checkbox", { name: "Agree" }));

// Keyboard
await user.keyboard("{Enter}");
await user.keyboard("{Tab}");

// Hover
await user.hover(element);
await user.unhover(element);

// Upload file
const file = new File(["content"], "test.png", { type: "image/png" });
await user.upload(screen.getByLabelText("Avatar"), file);
```

---

## Testing useState Components

### Component

```tsx
// Counter.tsx
import { useState } from "react";

export function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount((c) => c + 1)}>Increment</button>
      <button onClick={() => setCount((c) => c - 1)}>Decrement</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

### Test

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { Counter } from "./Counter";

describe("Counter", () => {
  it("starts at 0", () => {
    render(<Counter />);
    expect(screen.getByText("Count: 0")).toBeInTheDocument();
  });

  it("increments when clicking Increment", async () => {
    const user = userEvent.setup();
    render(<Counter />);

    await user.click(screen.getByRole("button", { name: "Increment" }));
    expect(screen.getByText("Count: 1")).toBeInTheDocument();

    await user.click(screen.getByRole("button", { name: "Increment" }));
    expect(screen.getByText("Count: 2")).toBeInTheDocument();
  });

  it("resets to 0", async () => {
    const user = userEvent.setup();
    render(<Counter />);

    await user.click(screen.getByRole("button", { name: "Increment" }));
    await user.click(screen.getByRole("button", { name: "Increment" }));
    await user.click(screen.getByRole("button", { name: "Reset" }));

    expect(screen.getByText("Count: 0")).toBeInTheDocument();
  });
});
```

Notice: we **never** check `useState` directly. We test what the user sees and does.

---

## Testing useEffect / Async Components

### Component with Async Data

```tsx
// UserProfile.tsx
import { useState, useEffect } from "react";

export function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<{ name: string } | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then((res) => res.json())
      .then((data) => {
        setUser(data);
        setLoading(false);
      });
  }, [userId]);

  if (loading) return <p>Loading...</p>;
  return <h1>{user?.name}</h1>;
}
```

### Test with waitFor and findBy

```tsx
import { render, screen, waitFor } from "@testing-library/react";
import { UserProfile } from "./UserProfile";

// Mock the global fetch
beforeEach(() => {
  global.fetch = vi.fn(() =>
    Promise.resolve({
      json: () => Promise.resolve({ name: "Vikas" }),
    }),
  ) as any;
});

describe("UserProfile", () => {
  it("shows loading then user name", async () => {
    render(<UserProfile userId="1" />);

    // Initially shows loading
    expect(screen.getByText("Loading...")).toBeInTheDocument();

    // After fetch resolves, shows user name
    expect(await screen.findByText("Vikas")).toBeInTheDocument();

    // Loading is gone
    expect(screen.queryByText("Loading...")).not.toBeInTheDocument();
  });
});
```

### waitFor (for complex assertions)

```tsx
await waitFor(() => {
  expect(screen.getByRole("heading")).toHaveTextContent("Vikas");
});

// With options:
await waitFor(
  () => {
    expect(screen.getByText("Done")).toBeInTheDocument();
  },
  { timeout: 3000 }, // default is 1000ms
);
```

---

## Testing Forms

### Component

```tsx
// ContactForm.tsx
import { useState } from "react";

export function ContactForm({ onSubmit }: { onSubmit: (data: any) => void }) {
  const [errors, setErrors] = useState<Record<string, string>>({});

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    const name = formData.get("name") as string;
    const email = formData.get("email") as string;

    const newErrors: Record<string, string> = {};
    if (!name) newErrors.name = "Name is required";
    if (!email) newErrors.email = "Email is required";
    else if (!email.includes("@")) newErrors.email = "Invalid email";

    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }

    onSubmit({ name, email });
  };

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Name
        <input name="name" />
      </label>
      {errors.name && <span role="alert">{errors.name}</span>}

      <label>
        Email
        <input name="email" type="email" />
      </label>
      {errors.email && <span role="alert">{errors.email}</span>}

      <button type="submit">Send</button>
    </form>
  );
}
```

### Tests

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { ContactForm } from "./ContactForm";

describe("ContactForm", () => {
  const mockSubmit = vi.fn();

  beforeEach(() => {
    mockSubmit.mockClear();
  });

  it("submits with valid data", async () => {
    const user = userEvent.setup();
    render(<ContactForm onSubmit={mockSubmit} />);

    await user.type(screen.getByLabelText("Name"), "Vikas");
    await user.type(screen.getByLabelText("Email"), "vikas@test.com");
    await user.click(screen.getByRole("button", { name: "Send" }));

    expect(mockSubmit).toHaveBeenCalledWith({
      name: "Vikas",
      email: "vikas@test.com",
    });
  });

  it("shows validation errors for empty fields", async () => {
    const user = userEvent.setup();
    render(<ContactForm onSubmit={mockSubmit} />);

    await user.click(screen.getByRole("button", { name: "Send" }));

    expect(screen.getByText("Name is required")).toBeInTheDocument();
    expect(screen.getByText("Email is required")).toBeInTheDocument();
    expect(mockSubmit).not.toHaveBeenCalled();
  });

  it("shows error for invalid email", async () => {
    const user = userEvent.setup();
    render(<ContactForm onSubmit={mockSubmit} />);

    await user.type(screen.getByLabelText("Name"), "Vikas");
    await user.type(screen.getByLabelText("Email"), "not-an-email");
    await user.click(screen.getByRole("button", { name: "Send" }));

    expect(screen.getByText("Invalid email")).toBeInTheDocument();
    expect(mockSubmit).not.toHaveBeenCalled();
  });
});
```

---

## Mocking API Calls

### Option 1: Mock Service Worker (MSW) — Recommended

MSW intercepts requests at the network level — components use real `fetch` calls.

```bash
npm install -D msw
```

```typescript
// src/mocks/handlers.ts
import { http, HttpResponse } from "msw";

export const handlers = [
  http.get("/api/users", () => {
    return HttpResponse.json([
      { id: 1, name: "Vikas" },
      { id: 2, name: "Alice" },
    ]);
  }),

  http.post("/api/users", async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({ id: 3, ...body }, { status: 201 });
  }),
];
```

```typescript
// src/mocks/server.ts
import { setupServer } from "msw/node";
import { handlers } from "./handlers";

export const server = setupServer(...handlers);
```

```typescript
// src/test/setup.ts
import "@testing-library/jest-dom";
import { server } from "../mocks/server";

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

```tsx
// Override handler for a specific test
import { http, HttpResponse } from "msw";
import { server } from "../mocks/server";

it("shows error when API fails", async () => {
  server.use(
    http.get("/api/users", () => {
      return HttpResponse.json({ message: "Server error" }, { status: 500 });
    }),
  );

  render(<UserList />);
  expect(await screen.findByText("Failed to load users")).toBeInTheDocument();
});
```

### Option 2: vi.mock (Quick and Simple)

```tsx
// Mock a module
vi.mock("../api/userService", () => ({
  fetchUsers: vi.fn(() => Promise.resolve([{ id: 1, name: "Vikas" }])),
}));
```

### When to Use Each

| Approach | Best For                                                     |
| -------- | ------------------------------------------------------------ |
| MSW      | Integration tests, testing real fetch behavior, shared mocks |
| vi.mock  | Unit tests, simple cases, mocking specific modules           |

---

## Testing Custom Hooks with renderHook

```tsx
// useCounter.ts
import { useState, useCallback } from "react";

export function useCounter(initial = 0) {
  const [count, setCount] = useState(initial);

  const increment = useCallback(() => setCount((c) => c + 1), []);
  const decrement = useCallback(() => setCount((c) => c - 1), []);
  const reset = useCallback(() => setCount(initial), [initial]);

  return { count, increment, decrement, reset };
}
```

```tsx
// useCounter.test.ts
import { renderHook, act } from "@testing-library/react";
import { useCounter } from "./useCounter";

describe("useCounter", () => {
  it("starts with initial value", () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });

  it("increments", () => {
    const { result } = renderHook(() => useCounter(0));

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });

  it("resets to initial value", () => {
    const { result } = renderHook(() => useCounter(5));

    act(() => {
      result.current.increment();
      result.current.increment();
      result.current.reset();
    });

    expect(result.current.count).toBe(5);
  });
});
```

**`act()`** is required when calling functions that trigger state updates outside of user events.

---

## Snapshot Testing

Snapshots capture the rendered output and compare against a stored file.

### Creating a Snapshot

```tsx
import { render } from "@testing-library/react";
import { Card } from "./Card";

it("matches snapshot", () => {
  const { container } = render(<Card title="Hello" description="World" />);

  expect(container).toMatchSnapshot();
});
```

First run creates `__snapshots__/Card.test.tsx.snap`. Subsequent runs compare output.

### Updating Snapshots

```bash
vitest --update  # or press 'u' in watch mode
```

### When to Use Snapshots

✅ **Good for:**

- Small, presentational components (icons, badges, static layouts)
- Detecting unintentional UI changes

❌ **Bad for:**

- Large components (snapshots become unreadable)
- Components with dynamic data (dates, IDs)
- As a replacement for behavior tests

### Inline Snapshots

```tsx
it("renders correctly", () => {
  const { container } = render(<Badge status="active" />);

  expect(container).toMatchInlineSnapshot(`
    <div>
      <span class="badge badge-active">Active</span>
    </div>
  `);
});
```

---

## Best Practices

1. **Query by role first** — `getByRole` mirrors how screen readers see the page.
2. **Use `userEvent` over `fireEvent`** — `userEvent` simulates real browser behavior (focus, keyboard events, etc.).
3. **Test behavior, not implementation** — do not assert on state variables or internal methods.
4. **One concept per test** — each `it()` block should test one thing.
5. **Use `findBy*` for async** — do not use `waitFor` + `getBy` when `findBy` works.
6. **Mock at the network level** (MSW) — component code stays untouched.
7. **Write accessible components** — if you cannot find an element with `getByRole`, your component may have accessibility issues.
8. **Avoid testing third-party libraries** — test your logic, not Material UI's button.
9. **Use `screen`** instead of destructuring from `render()` — more readable, no stale references.
10. **Set up `userEvent.setup()` once per test** — do not call it inside assertions.

---

## Common Mistakes

| Mistake                         | Why It Is Wrong                       | Fix                                                    |
| ------------------------------- | ------------------------------------- | ------------------------------------------------------ |
| Using `getByTestId` everywhere  | Ignores accessibility, fragile        | Use `getByRole`, `getByLabelText`                      |
| Testing implementation details  | Tests break when refactoring          | Test what users see and do                             |
| Not awaiting `userEvent`        | Race conditions, flaky tests          | Always `await user.click(...)`                         |
| Using `container.querySelector` | Bypasses RTL's accessible queries     | Use `screen.getBy*` queries                            |
| Snapshot testing everything     | Unreadable, always breaking, no value | Use snapshots only for small presentational components |
| Not cleaning up mocks           | Tests leak state to each other        | Use `afterEach`, `mockClear()`                         |
| `waitFor` with side effects     | `waitFor` re-runs the callback        | Put only assertions inside `waitFor`                   |
| Missing `act()` warnings        | State updates outside React's control | Wrap state-triggering calls in `act()`                 |

---

## Summary

- **React Testing Library** tests components from the user's perspective — behavior over implementation.
- Use **`getByRole`** as your primary query — it promotes accessible components.
- **`userEvent`** simulates real interactions more accurately than `fireEvent`.
- Use **`findBy*`** and **`waitFor`** for async rendering (API calls, `useEffect`).
- **MSW** (Mock Service Worker) is the recommended way to mock API calls at the network level.
- **`renderHook`** tests custom hooks in isolation when they are not easily tested through a component.
- **Snapshots** are useful for small presentational components but should not replace behavior tests.
- If you cannot find an element with accessible queries, that is a signal to improve your component's accessibility.
