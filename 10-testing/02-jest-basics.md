# Jest Basics

## What Is Jest?

Jest is a **zero-configuration** JavaScript testing framework created by Meta (Facebook). It's an all-in-one solution — test runner, assertion library, and mocking framework bundled together.

**Analogy:** If testing frameworks were toolboxes, most give you the box and you buy tools separately. Jest hands you a fully loaded toolbox — screwdrivers, wrenches, measuring tape — all included and ready to go.

### Why Jest?

| Feature              | Benefit                                          |
| -------------------- | ------------------------------------------------ |
| Zero config          | Works out of the box for most JS/TS projects     |
| Fast                 | Runs tests in parallel with intelligent ordering |
| Built-in mocking     | No need for external libraries like Sinon        |
| Snapshot testing     | Catches unexpected UI changes automatically      |
| Code coverage        | Built-in — just add `--coverage` flag            |
| Watch mode           | Re-runs only affected tests on file save         |
| Great error messages | Shows exactly what was expected vs. received     |

---

## Installation and Setup

```bash
# Install Jest as a dev dependency
npm install --save-dev jest

# For TypeScript projects
npm install --save-dev jest ts-jest @types/jest
```

### package.json Configuration

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

### File Naming Convention

Jest automatically finds test files that match these patterns:

- `*.test.js` / `*.test.ts` — e.g., `math.test.js`
- `*.spec.js` / `*.spec.ts` — e.g., `math.spec.js`
- Files inside a `__tests__/` directory

```
src/
├── utils/
│   ├── math.js
│   └── math.test.js     ← Test file next to source
├── __tests__/
│   └── math.test.js     ← Or in __tests__ folder
```

---

## Writing Your First Test

### describe, it/test, expect

```js
// math.js
function add(a, b) {
  return a + b;
}

function multiply(a, b) {
  return a * b;
}

module.exports = { add, multiply };
```

```js
// math.test.js
const { add, multiply } = require("./math");

describe("Math utilities", () => {
  describe("add", () => {
    it("adds two positive numbers", () => {
      expect(add(2, 3)).toBe(5);
    });

    it("handles negative numbers", () => {
      expect(add(-1, -2)).toBe(-3);
    });

    it("handles zero", () => {
      expect(add(5, 0)).toBe(5);
    });
  });

  describe("multiply", () => {
    it("multiplies two numbers", () => {
      expect(multiply(3, 4)).toBe(12);
    });

    it("returns zero when multiplied by zero", () => {
      expect(multiply(5, 0)).toBe(0);
    });
  });
});
```

| Function      | Purpose                                              |
| ------------- | ---------------------------------------------------- |
| `describe`    | Groups related tests together (can be nested)        |
| `it` / `test` | Defines a single test case (they're identical)       |
| `expect`      | Creates an assertion — the value you're checking     |
| `.toBe()`     | Matcher — defines what you expect the value to equal |

---

## Matchers

Matchers are the assertion methods chained after `expect()`. Jest has many built-in matchers.

### Equality Matchers

```js
// toBe — strict equality (===), use for primitives
expect(2 + 2).toBe(4);
expect("hello").toBe("hello");

// toEqual — deep equality, use for objects and arrays
expect({ name: "Vikas" }).toEqual({ name: "Vikas" });
expect([1, 2, 3]).toEqual([1, 2, 3]);

// ❌ toBe fails for objects (different references)
expect({ a: 1 }).toBe({ a: 1 }); // FAILS!
// ✅ toEqual checks value equality
expect({ a: 1 }).toEqual({ a: 1 }); // PASSES!
```

### Truthiness Matchers

```js
expect(null).toBeNull();
expect(undefined).toBeUndefined();
expect("value").toBeDefined();
expect(true).toBeTruthy();
expect(0).toBeFalsy();
```

### Number Matchers

```js
expect(10).toBeGreaterThan(5);
expect(10).toBeGreaterThanOrEqual(10);
expect(5).toBeLessThan(10);
expect(0.1 + 0.2).toBeCloseTo(0.3); // Handles floating point!
```

### String Matchers

```js
expect("Hello World").toContain("World");
expect("team@example.com").toMatch(/^[^@]+@[^@]+\.[^@]+$/);
```

### Array / Object Matchers

```js
const fruits = ["apple", "banana", "mango"];

expect(fruits).toContain("banana");
expect(fruits).toHaveLength(3);

const user = { name: "Vikas", age: 25, city: "Delhi" };
expect(user).toMatchObject({ name: "Vikas", age: 25 }); // Partial match
expect(user).toHaveProperty("city", "Delhi");
```

### Exception Matchers

```js
function divide(a, b) {
  if (b === 0) throw new Error("Cannot divide by zero");
  return a / b;
}

// Must wrap in a function for toThrow to catch it
expect(() => divide(10, 0)).toThrow();
expect(() => divide(10, 0)).toThrow("Cannot divide by zero");
expect(() => divide(10, 0)).toThrow(/zero/);
```

### Negation with .not

```js
expect(5).not.toBe(3);
expect([1, 2, 3]).not.toContain(4);
expect("hello").not.toMatch(/xyz/);
```

---

## Setup and Teardown

Use lifecycle hooks to run code before/after tests — avoids repetition and ensures clean state.

```js
describe("Database operations", () => {
  let db;

  // Runs ONCE before all tests in this describe block
  beforeAll(async () => {
    db = await connectToTestDatabase();
  });

  // Runs ONCE after all tests in this describe block
  afterAll(async () => {
    await db.disconnect();
  });

  // Runs before EACH test
  beforeEach(async () => {
    await db.clear(); // Clean slate for each test
    await db.seed({ users: [{ name: "Vikas" }] });
  });

  // Runs after EACH test
  afterEach(() => {
    jest.restoreAllMocks(); // Clean up mocks
  });

  it("finds a user by name", async () => {
    const user = await db.findUser("Vikas");
    expect(user).toBeDefined();
  });

  it("returns null for unknown user", async () => {
    const user = await db.findUser("Ghost");
    expect(user).toBeNull();
  });
});
```

| Hook         | Runs                     | Use Case                            |
| ------------ | ------------------------ | ----------------------------------- |
| `beforeAll`  | Once before all tests    | Database connection, server startup |
| `afterAll`   | Once after all tests     | Disconnect, cleanup                 |
| `beforeEach` | Before every single test | Reset state, seed data, clear mocks |
| `afterEach`  | After every single test  | Restore mocks, clear timers         |

---

## Mocking Functions

### jest.fn() — Create a Mock Function

```js
const mockCallback = jest.fn();

// Call it
mockCallback("hello");
mockCallback("world");

// Assert how it was called
expect(mockCallback).toHaveBeenCalled();
expect(mockCallback).toHaveBeenCalledTimes(2);
expect(mockCallback).toHaveBeenCalledWith("hello");
expect(mockCallback).toHaveBeenLastCalledWith("world");
```

### mockReturnValue — Control What It Returns

```js
const getPrice = jest.fn();

// Always return the same value
getPrice.mockReturnValue(99);
expect(getPrice()).toBe(99);

// Return different values on successive calls
getPrice.mockReturnValueOnce(10).mockReturnValueOnce(20).mockReturnValue(30); // Default after that

expect(getPrice()).toBe(10); // First call
expect(getPrice()).toBe(20); // Second call
expect(getPrice()).toBe(30); // Third and beyond
```

### mockImplementation — Custom Logic

```js
const calculateTax = jest.fn().mockImplementation((amount) => {
  return amount * 0.18; // 18% GST
});

expect(calculateTax(100)).toBe(18);
expect(calculateTax(500)).toBe(90);
```

### Practical Example — Testing a Function That Uses a Callback

```js
function fetchUserData(userId, callback) {
  // In real code, this would call an API
  const user = { id: userId, name: "Vikas" };
  callback(user);
}

test("calls callback with user data", () => {
  const mockCallback = jest.fn();

  fetchUserData(1, mockCallback);

  expect(mockCallback).toHaveBeenCalledWith({ id: 1, name: "Vikas" });
});
```

---

## Mocking Modules

### jest.mock() — Replace an Entire Module

```js
// userService.js
const axios = require("axios");

async function getUser(id) {
  const response = await axios.get(`/api/users/${id}`);
  return response.data;
}

module.exports = { getUser };
```

```js
// userService.test.js
const axios = require("axios");
const { getUser } = require("./userService");

// Mock the entire axios module
jest.mock("axios");

describe("getUser", () => {
  it("fetches user by ID", async () => {
    // Configure the mock
    axios.get.mockResolvedValue({
      data: { id: 1, name: "Vikas" },
    });

    const user = await getUser(1);

    expect(user).toEqual({ id: 1, name: "Vikas" });
    expect(axios.get).toHaveBeenCalledWith("/api/users/1");
  });

  it("handles API errors", async () => {
    axios.get.mockRejectedValue(new Error("Network Error"));

    await expect(getUser(1)).rejects.toThrow("Network Error");
  });
});
```

### Partial Module Mocks

```js
// Mock only one export, keep the rest real
jest.mock("./utils", () => ({
  ...jest.requireActual("./utils"),
  fetchData: jest.fn(), // Only mock this one
}));
```

---

## Testing Async Code

### async/await Pattern (Recommended)

```js
async function fetchTodo(id) {
  const response = await fetch(`https://api.example.com/todos/${id}`);
  if (!response.ok) throw new Error("Not found");
  return response.json();
}

test("fetches a todo by ID", async () => {
  const todo = await fetchTodo(1);
  expect(todo).toHaveProperty("title");
});
```

### resolves / rejects Matchers

```js
test("resolves with user data", async () => {
  await expect(getUser(1)).resolves.toEqual({ id: 1, name: "Vikas" });
});

test("rejects when user not found", async () => {
  await expect(getUser(999)).rejects.toThrow("Not found");
});
```

### Testing Timers

```js
jest.useFakeTimers();

function delayedGreeting(callback) {
  setTimeout(() => callback("Hello!"), 3000);
}

test("calls callback after 3 seconds", () => {
  const mockCb = jest.fn();

  delayedGreeting(mockCb);

  // Fast-forward time
  jest.advanceTimersByTime(3000);

  expect(mockCb).toHaveBeenCalledWith("Hello!");
});
```

---

## Snapshot Testing

Snapshot testing captures the output of a component or function and saves it. On subsequent runs, it compares the current output to the saved snapshot.

```js
const { render } = require("@testing-library/react");
const Button = require("./Button");

test("Button renders correctly", () => {
  const { container } = render(<Button label="Click Me" />);
  expect(container).toMatchSnapshot();
});
```

First run: Jest creates a `.snap` file with the rendered output.
Next runs: Jest compares current output with the saved snapshot.

### Updating Snapshots

```bash
# When you intentionally change the UI
jest --updateSnapshot
# or shorthand
jest -u
```

### When to Use Snapshots

| Good For                       | Not Good For                             |
| ------------------------------ | ---------------------------------------- |
| Catching unintended UI changes | Testing logic or behavior                |
| Small, stable components       | Large, frequently changing components    |
| Serializable outputs (JSON)    | Dynamic content (timestamps, random IDs) |

---

## Code Coverage Reports

```bash
# Run tests with coverage report
npx jest --coverage
```

Jest generates a report showing:

```
--------------------|---------|----------|---------|---------|
File                | % Stmts | % Branch | % Funcs | % Lines |
--------------------|---------|----------|---------|---------|
All files           |   85.71 |    66.67 |     100 |   83.33 |
 math.js            |     100 |      100 |     100 |     100 |
 userService.js     |   71.43 |    33.33 |     100 |   66.67 |
--------------------|---------|----------|---------|---------|
```

### Configuration in package.json

```json
{
  "jest": {
    "collectCoverageFrom": ["src/**/*.{js,ts}", "!src/index.js"],
    "coverageThreshold": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    }
  }
}
```

---

## Watch Mode

Watch mode re-runs tests automatically when you save files — essential during development.

```bash
npx jest --watch      # Re-runs tests related to changed files
npx jest --watchAll   # Re-runs all tests on any change
```

### Watch Mode Controls

When in watch mode, press these keys:

| Key     | Action                      |
| ------- | --------------------------- |
| `a`     | Run all tests               |
| `f`     | Run only failed tests       |
| `p`     | Filter by filename pattern  |
| `t`     | Filter by test name pattern |
| `q`     | Quit watch mode             |
| `Enter` | Trigger a test run          |

---

## Best Practices

1. **One concept per test** — each `it()` block tests exactly one thing.
2. **Descriptive test names** — read like a sentence: `"returns null when user is not found"`.
3. **Use `beforeEach` for setup** — don't duplicate setup code across tests.
4. **Clean up mocks** — use `jest.restoreAllMocks()` in `afterEach` to avoid leaks.
5. **Prefer `toEqual` for objects** — `toBe` checks reference equality, which fails for objects.
6. **Don't mock what you don't own** — mock your own adapters instead of third-party internals.
7. **Keep tests close to source** — `math.test.js` next to `math.js` for easy navigation.
8. **Use `--coverage` in CI** — ensure coverage doesn't drop on pull requests.

---

## Common Mistakes

| Mistake                                  | Why It's Wrong                                                       |
| ---------------------------------------- | -------------------------------------------------------------------- |
| Using `toBe` for objects/arrays          | Compares references, not values — use `toEqual` instead              |
| Forgetting `await` in async tests        | Test passes before assertion runs — gives false positive             |
| Not wrapping thrown functions in `() =>` | `expect(fn()).toThrow()` calls `fn` immediately, Jest can't catch it |
| Mocking too much                         | Test passes but proves nothing — only mock external dependencies     |
| Not cleaning up mocks between tests      | Mock from one test bleeds into another — unpredictable results       |
| Huge snapshot files                      | Painful to review — use small, focused snapshots                     |
| Testing internal/private implementation  | Tests break on refactor — test the public interface                  |
| Ignoring watch mode during development   | Running full suite manually each time is slow and frustrating        |

---

## Summary

- **Jest** is an all-in-one testing framework — runner, assertions, mocking, and coverage built in.
- Use `describe` to group, `it`/`test` to define cases, and `expect` with matchers to assert.
- **Matchers** like `toBe`, `toEqual`, `toContain`, `toThrow`, and `toMatchObject` cover most needs.
- **Lifecycle hooks** (`beforeEach`, `afterEach`, etc.) keep setup DRY and tests isolated.
- Mock functions with `jest.fn()`, control returns with `mockReturnValue`, and replace modules with `jest.mock()`.
- Test async code with `async/await` and use `resolves`/`rejects` matchers for promise assertions.
- **Snapshot testing** catches unintended changes but should be used sparingly for stable outputs.
- Run `--coverage` to identify untested code and `--watch` for fast development feedback.
