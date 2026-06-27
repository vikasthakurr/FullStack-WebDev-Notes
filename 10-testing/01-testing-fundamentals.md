# Testing Fundamentals

## Why Test?

Testing is writing code that verifies your other code works correctly. It might seem like extra effort, but it pays off massively as a project grows.

**Analogy:** Think of tests as the safety net under a trapeze artist. The artist (developer) can perform daring moves (refactors, new features) confidently because they know the net (tests) will catch them if something goes wrong.

### Three Core Reasons to Test

| Reason                    | Explanation                                                                 |
| ------------------------- | --------------------------------------------------------------------------- |
| **Confidence**            | Deploy with peace of mind — you know nothing is silently broken             |
| **Regression Prevention** | Catch bugs introduced by new changes before they reach users                |
| **Living Documentation**  | Tests describe how the code _should_ behave — better than outdated comments |

```js
// This test IS documentation — it tells you the function's expected behavior
describe("calculateDiscount", () => {
  it("applies 10% discount for orders above ₹1000", () => {
    expect(calculateDiscount(1500)).toBe(150);
  });

  it("returns 0 discount for orders below ₹1000", () => {
    expect(calculateDiscount(500)).toBe(0);
  });
});
```

---

## Types of Tests — The Testing Pyramid

The testing pyramid shows how many tests of each type you should ideally have:

```
        /  E2E  \          ← Few (slow, expensive, broad)
       /----------\
      / Integration \      ← Some (moderate speed, moderate scope)
     /----------------\
    /    Unit Tests     \  ← Many (fast, cheap, focused)
   /____________________\
```

| Type            | What It Tests                          | Speed  | Scope  | Example                              |
| --------------- | -------------------------------------- | ------ | ------ | ------------------------------------ |
| **Unit**        | Single function/component in isolation | Fast   | Narrow | Does `formatDate()` return "Jan 1"?  |
| **Integration** | Multiple units working together        | Medium | Medium | Does the form submit to the API?     |
| **E2E**         | Full user journey through the app      | Slow   | Broad  | Can a user sign up, log in, and buy? |

**Analogy:** Imagine testing a car:

- **Unit test** — Does the spark plug fire? Does the fuel pump work?
- **Integration test** — Does the engine start when you turn the key?
- **E2E test** — Can you drive from Delhi to Mumbai without breaking down?

---

## TDD — Test-Driven Development

TDD flips the usual workflow. Instead of writing code first and testing later, you write the test **first**.

### The Red-Green-Refactor Cycle

```
  ┌─────────────────────────────────────┐
  │  1. RED — Write a failing test      │
  │  2. GREEN — Write minimal code to   │
  │     make it pass                    │
  │  3. REFACTOR — Clean up the code    │
  │     (tests still pass)              │
  └─────────────────────────────────────┘
        ↻ Repeat for the next feature
```

```js
// Step 1: RED — test fails (function doesn't exist yet)
test("adds two numbers", () => {
  expect(add(2, 3)).toBe(5);
});

// Step 2: GREEN — write the simplest code to pass
function add(a, b) {
  return a + b;
}

// Step 3: REFACTOR — maybe add input validation, rename, etc.
function add(a, b) {
  if (typeof a !== "number" || typeof b !== "number") {
    throw new Error("Arguments must be numbers");
  }
  return a + b;
}
```

### Benefits of TDD

- Forces you to think about the **interface** before the implementation
- Results in **testable** code by design
- Produces high coverage naturally
- Small, incremental steps reduce debugging time

---

## BDD — Behavior-Driven Development

BDD extends TDD by writing tests in **human-readable** language that describes behavior from the user's perspective.

```gherkin
Feature: Shopping Cart
  Scenario: Adding an item to the cart
    Given the cart is empty
    When the user adds "Laptop" to the cart
    Then the cart should contain 1 item
    And the cart total should be ₹50000
```

BDD uses keywords like **Given**, **When**, **Then** to structure tests. Tools like Cucumber translate these into executable tests. The key takeaway: describe **what** the system does, not **how** it does it.

---

## AAA Pattern — Arrange, Act, Assert

Every well-structured test follows three steps:

```js
test("user can be created with name and email", () => {
  // Arrange — set up the data and conditions
  const userData = { name: "Vikas", email: "vikas@example.com" };

  // Act — perform the action being tested
  const user = createUser(userData);

  // Assert — verify the outcome
  expect(user.name).toBe("Vikas");
  expect(user.email).toBe("vikas@example.com");
  expect(user.id).toBeDefined();
});
```

| Phase       | What You Do                           | Questions to Ask                      |
| ----------- | ------------------------------------- | ------------------------------------- |
| **Arrange** | Set up inputs, mocks, initial state   | What do I need before the action?     |
| **Act**     | Call the function / trigger the event | What single action am I testing?      |
| **Assert**  | Check the result                      | What should be true after the action? |

**Rule:** Keep each test to **one Act** and related assertions. If you're doing multiple actions, you probably need multiple tests.

---

## What Makes a Good Test — F.I.R.S.T Principles

| Principle           | Meaning                                                       |
| ------------------- | ------------------------------------------------------------- |
| **F**ast            | Runs in milliseconds — slow tests don't get run often         |
| **I**solated        | No shared state — one test's outcome never affects another    |
| **R**epeatable      | Same result every time — no dependency on network, time, etc. |
| **S**elf-validating | Passes or fails clearly — no manual checking required         |
| **T**imely          | Written close to the code it tests (ideally before via TDD)   |

```js
// ❌ Bad test — not isolated, depends on shared state
let counter = 0;
test("increments counter", () => {
  counter++;
  expect(counter).toBe(1);
});
test("increments counter again", () => {
  counter++;
  expect(counter).toBe(2); // Fails if run alone!
});

// ✅ Good test — each test is independent
test("increments counter from 0", () => {
  let counter = 0;
  counter++;
  expect(counter).toBe(1);
});
test("increments counter from 5", () => {
  let counter = 5;
  counter++;
  expect(counter).toBe(6);
});
```

---

## Test Coverage

Test coverage measures the percentage of your code that is executed during tests.

### Types of Coverage

| Type          | What It Measures                     |
| ------------- | ------------------------------------ |
| **Line**      | Percentage of code lines executed    |
| **Branch**    | Percentage of if/else branches taken |
| **Function**  | Percentage of functions called       |
| **Statement** | Percentage of statements executed    |

### Why 100% Coverage Isn't the Goal

```js
// 100% coverage but terrible test — proves nothing useful
function divide(a, b) {
  if (b === 0) throw new Error("Cannot divide by zero");
  return a / b;
}

test("divide works", () => {
  divide(10, 2); // Line is "covered" but we never checked the result!
});
```

**What matters is testing the right things, not hitting a number.**

- **80% coverage** with meaningful assertions > **100% coverage** with weak assertions
- Focus on testing **behavior and edge cases**, not chasing a metric
- Coverage shows what's **not tested** — that's its real value

---

## Mocking, Stubbing, and Spying

When testing a unit in isolation, you need to replace its dependencies with controlled substitutes.

**Analogy:** In a movie, stunt doubles replace actors for dangerous scenes. The scene (test) still happens, but with a controlled substitute that you can direct precisely.

| Concept  | What It Does                                   | When to Use                                    |
| -------- | ---------------------------------------------- | ---------------------------------------------- |
| **Mock** | Replaces a function/module with a fake version | Replacing API calls, databases, timers         |
| **Stub** | Returns a predetermined value (no real logic)  | Providing controlled data to the unit          |
| **Spy**  | Wraps the real function, records calls to it   | Verifying a function was called with arguments |

```js
// Mock — replace the whole function
const fetchUser = jest.fn().mockResolvedValue({ name: "Vikas" });

// Stub — hardcoded return value
const getConfig = () => ({ apiUrl: "http://test-api.com" });

// Spy — watch the real function
const consoleSpy = jest.spyOn(console, "log");
greet("Vikas");
expect(consoleSpy).toHaveBeenCalledWith("Hello, Vikas!");
consoleSpy.mockRestore();
```

---

## When NOT to Test

Not everything needs a test. Testing the wrong things wastes time and creates brittle tests.

### Don't Test

| What to Skip                       | Why                                                       |
| ---------------------------------- | --------------------------------------------------------- |
| **Implementation details**         | Tests break on refactor even if behavior is unchanged     |
| **Third-party library internals**  | That's the library author's job — trust their tests       |
| **Trivial code** (getters/setters) | No logic = nothing to verify                              |
| **Framework behavior**             | Don't test that React renders or Express routes — they do |
| **Private methods directly**       | Test them through the public API that uses them           |

```js
// ❌ Testing implementation details — brittle!
test("uses array internally", () => {
  const cart = new ShoppingCart();
  expect(cart._items).toBeInstanceOf(Array); // testing private state
});

// ✅ Testing behavior — resilient to refactors
test("cart starts empty", () => {
  const cart = new ShoppingCart();
  expect(cart.getItemCount()).toBe(0);
});
```

### Do Test

- Business logic and calculations
- Edge cases and boundary conditions
- Error handling paths
- User-facing behavior
- Integration points between modules

---

## Best Practices

1. **Follow the testing pyramid** — many unit tests, fewer integration, fewest E2E.
2. **Test behavior, not implementation** — ask "what should happen?" not "how does it work internally?"
3. **One assertion concept per test** — tests with one purpose are easier to debug when they fail.
4. **Use descriptive test names** — `"returns null when user is not found"` beats `"test case 3"`.
5. **Keep tests independent** — no test should depend on another test running first.
6. **Don't test implementation details** — if you refactor without changing behavior, tests shouldn't break.
7. **Use the AAA pattern** — clear structure makes tests readable and maintainable.
8. **Treat test code like production code** — refactor, remove duplication, keep it clean.

---

## Common Mistakes

| Mistake                                        | Why It's Wrong                                                       |
| ---------------------------------------------- | -------------------------------------------------------------------- |
| Testing implementation instead of behavior     | Tests break on refactors even when the feature still works correctly |
| Writing tests after the entire feature is done | Harder to write testable code retroactively — consider TDD           |
| Shared mutable state between tests             | Tests pass/fail depending on execution order — impossible to debug   |
| No assertions in a test                        | Test always passes — gives false confidence                          |
| Overly specific assertions                     | `toEqual(exact huge object)` breaks when irrelevant fields change    |
| Testing third-party code                       | You're testing someone else's responsibility — waste of time         |
| Chasing 100% coverage                          | Leads to meaningless tests just to hit a number                      |
| Huge "god tests" that test everything at once  | When they fail, you have no idea what broke — keep tests focused     |

---

## Summary

- **Testing** gives confidence, prevents regressions, and documents expected behavior.
- The **testing pyramid** suggests many unit tests, fewer integration tests, and the fewest E2E tests.
- **TDD** means Red → Green → Refactor — write the test first, then make it pass.
- Every test should follow **AAA**: Arrange (setup), Act (execute), Assert (verify).
- Good tests are **Fast, Isolated, Repeatable, Self-validating, and Timely** (F.I.R.S.T).
- **Coverage** shows what's untested — but 100% isn't the goal; meaningful assertions are.
- Use **mocks** to isolate units, **stubs** for controlled data, and **spies** to verify calls.
- Don't test implementation details, third-party code, or trivial logic — test **behavior** and **edge cases**.
