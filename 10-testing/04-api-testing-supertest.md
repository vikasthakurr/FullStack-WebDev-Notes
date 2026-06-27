# API Testing with Supertest

## What Is Supertest?

Supertest is an HTTP assertion library that lets you test Express (or any Node.js HTTP server) endpoints **without starting the server on a real port**. It makes HTTP requests to your app in-process and asserts on status codes, headers, and response bodies.

### Analogy

Supertest is like having a direct phone line to your restaurant's kitchen. Instead of walking in the front door (starting a real server, using a real HTTP client), you call the kitchen directly, place an order, and verify the dish comes back correctly.

### Why Supertest?

- Tests run fast (no network overhead).
- No port conflicts between test runs.
- Works with any assertion library (Jest, Vitest).
- Chainable API for clean, readable tests.

---

## Setup with Vitest

### Installation

```bash
npm install -D vitest supertest @types/supertest
```

### Separate App from Server

The key pattern: export your Express `app` separately from the `server.listen()` call.

```typescript
// src/app.ts — Express app (no .listen())
import express from "express";
import { userRouter } from "./routes/users";

const app = express();
app.use(express.json());
app.use("/api/users", userRouter);

export default app;
```

```typescript
// src/server.ts — starts the server (used in production, NOT in tests)
import app from "./app";

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

### Test File Structure

```typescript
// src/routes/users.test.ts
import request from "supertest";
import app from "../app";

describe("GET /api/users", () => {
  it("returns a list of users", async () => {
    const response = await request(app).get("/api/users");

    expect(response.status).toBe(200);
    expect(response.body).toBeInstanceOf(Array);
  });
});
```

---

## Testing GET Endpoints

```typescript
import request from "supertest";
import app from "../app";

describe("GET /api/users", () => {
  it("returns 200 and an array of users", async () => {
    const response = await request(app).get("/api/users");

    expect(response.status).toBe(200);
    expect(response.headers["content-type"]).toMatch(/json/);
    expect(response.body).toEqual([
      { id: 1, name: "Vikas", email: "vikas@test.com" },
      { id: 2, name: "Alice", email: "alice@test.com" },
    ]);
  });

  it("returns a single user by ID", async () => {
    const response = await request(app).get("/api/users/1");

    expect(response.status).toBe(200);
    expect(response.body).toMatchObject({
      id: 1,
      name: "Vikas",
    });
  });

  it("returns 404 for non-existent user", async () => {
    const response = await request(app).get("/api/users/999");

    expect(response.status).toBe(404);
    expect(response.body).toEqual({ message: "User not found" });
  });
});
```

---

## Testing POST Endpoints

```typescript
describe("POST /api/users", () => {
  it("creates a new user and returns 201", async () => {
    const newUser = {
      name: "Bob",
      email: "bob@test.com",
      password: "securePass123",
    };

    const response = await request(app).post("/api/users").send(newUser);

    expect(response.status).toBe(201);
    expect(response.body).toMatchObject({
      id: expect.any(Number),
      name: "Bob",
      email: "bob@test.com",
    });
    // Password should NOT be in response
    expect(response.body).not.toHaveProperty("password");
  });

  it("returns 400 for missing required fields", async () => {
    const response = await request(app)
      .post("/api/users")
      .send({ name: "Bob" }); // missing email and password

    expect(response.status).toBe(400);
    expect(response.body.errors).toContainEqual(
      expect.objectContaining({ field: "email" }),
    );
  });
});
```

---

## Testing PUT and DELETE Endpoints

```typescript
describe("PUT /api/users/:id", () => {
  it("updates user and returns 200", async () => {
    const response = await request(app)
      .put("/api/users/1")
      .send({ name: "Vikas Updated" });

    expect(response.status).toBe(200);
    expect(response.body.name).toBe("Vikas Updated");
  });

  it("returns 404 for non-existent user", async () => {
    const response = await request(app)
      .put("/api/users/999")
      .send({ name: "Ghost" });

    expect(response.status).toBe(404);
  });
});

describe("DELETE /api/users/:id", () => {
  it("deletes user and returns 204", async () => {
    const response = await request(app).delete("/api/users/1");

    expect(response.status).toBe(204);
  });

  it("returns 404 for non-existent user", async () => {
    const response = await request(app).delete("/api/users/999");

    expect(response.status).toBe(404);
  });
});
```

---

## Sending Headers (Authorization)

```typescript
describe("GET /api/profile (protected route)", () => {
  const token = "valid-jwt-token-here";

  it("returns 200 with valid token", async () => {
    const response = await request(app)
      .get("/api/profile")
      .set("Authorization", `Bearer ${token}`);

    expect(response.status).toBe(200);
    expect(response.body).toHaveProperty("name");
  });

  it("returns 401 without token", async () => {
    const response = await request(app).get("/api/profile");

    expect(response.status).toBe(401);
    expect(response.body).toEqual({ message: "Unauthorized" });
  });

  it("returns 401 with invalid token", async () => {
    const response = await request(app)
      .get("/api/profile")
      .set("Authorization", "Bearer invalid-token");

    expect(response.status).toBe(401);
  });
});
```

### Setting Multiple Headers

```typescript
const response = await request(app)
  .post("/api/data")
  .set("Authorization", `Bearer ${token}`)
  .set("X-Request-ID", "test-123")
  .set("Accept", "application/json")
  .send({ key: "value" });
```

---

## Sending Request Body (JSON)

```typescript
// .send() automatically sets Content-Type: application/json
const response = await request(app)
  .post("/api/orders")
  .send({
    items: [
      { productId: 1, quantity: 2 },
      { productId: 3, quantity: 1 },
    ],
    shippingAddress: {
      street: "123 Main St",
      city: "Mumbai",
      zip: "400001",
    },
  });

// For form-encoded data:
const response = await request(app)
  .post("/api/login")
  .type("form")
  .send("username=vikas&password=secret");

// For multipart (file upload):
const response = await request(app)
  .post("/api/upload")
  .attach("avatar", "./test/fixtures/photo.png")
  .field("name", "Profile Photo");
```

---

## Asserting Status Codes, Body, and Headers

```typescript
it("complete assertion example", async () => {
  const response = await request(app)
    .post("/api/users")
    .send({ name: "Vikas", email: "v@test.com", password: "pass123" });

  // Status code
  expect(response.status).toBe(201);

  // Response headers
  expect(response.headers["content-type"]).toMatch(/application\/json/);
  expect(response.headers["x-request-id"]).toBeDefined();

  // Response body — exact match
  expect(response.body).toEqual({
    id: expect.any(Number),
    name: "Vikas",
    email: "v@test.com",
    createdAt: expect.any(String),
  });

  // Response body — partial match
  expect(response.body).toMatchObject({
    name: "Vikas",
    email: "v@test.com",
  });

  // Array assertions
  expect(response.body.roles).toContain("user");
  expect(response.body.roles).toHaveLength(1);
});
```

### Supertest's Built-in expect() (Alternative)

```typescript
// Supertest also has its own .expect() chainable API:
await request(app)
  .get("/api/users")
  .expect(200)
  .expect("Content-Type", /json/)
  .expect((res) => {
    if (res.body.length < 1) throw new Error("Expected at least one user");
  });
```

---

## Testing with a Real Database

For integration tests that hit a real database, use a separate test database.

### Setup and Teardown

```typescript
// tests/setup.ts
import mongoose from "mongoose";
import { MongoMemoryServer } from "mongodb-memory-server";

let mongoServer: MongoMemoryServer;

beforeAll(async () => {
  mongoServer = await MongoMemoryServer.create();
  await mongoose.connect(mongoServer.getUri());
});

afterEach(async () => {
  // Clear all collections between tests
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    await collections[key].deleteMany({});
  }
});

afterAll(async () => {
  await mongoose.disconnect();
  await mongoServer.stop();
});
```

### Seeding Test Data

```typescript
import { User } from "../models/User";

describe("GET /api/users", () => {
  beforeEach(async () => {
    // Seed database before each test
    await User.insertMany([
      { name: "Vikas", email: "vikas@test.com", password: "hashed" },
      { name: "Alice", email: "alice@test.com", password: "hashed" },
    ]);
  });

  it("returns all users", async () => {
    const response = await request(app).get("/api/users");

    expect(response.status).toBe(200);
    expect(response.body).toHaveLength(2);
  });
});
```

### Test Database Options

| Option                 | Pros                       | Cons                                  |
| ---------------------- | -------------------------- | ------------------------------------- |
| MongoDB Memory Server  | Fast, isolated, no setup   | Only for MongoDB                      |
| SQLite in-memory       | Fast, no files             | Different SQL dialect than production |
| Docker test database   | Matches production exactly | Slower, requires Docker               |
| Separate test database | Real environment           | Cleanup complexity                    |

---

## Testing with Mocked Database

For unit tests where you want to isolate route logic from the database:

```typescript
import request from "supertest";
import app from "../app";
import { User } from "../models/User";

// Mock the entire model
vi.mock("../models/User");

describe("GET /api/users (mocked DB)", () => {
  it("returns users from database", async () => {
    // Set up the mock return value
    vi.mocked(User.find).mockResolvedValue([
      { _id: "1", name: "Vikas", email: "vikas@test.com" },
    ]);

    const response = await request(app).get("/api/users");

    expect(response.status).toBe(200);
    expect(response.body[0].name).toBe("Vikas");
    expect(User.find).toHaveBeenCalledOnce();
  });

  it("returns 500 when database fails", async () => {
    vi.mocked(User.find).mockRejectedValue(new Error("DB connection lost"));

    const response = await request(app).get("/api/users");

    expect(response.status).toBe(500);
    expect(response.body.message).toBe("Internal server error");
  });
});
```

---

## Testing Authentication Middleware

### The Middleware

```typescript
// src/middleware/auth.ts
import jwt from "jsonwebtoken";

export function authenticate(req, res, next) {
  const header = req.headers.authorization;
  if (!header?.startsWith("Bearer ")) {
    return res.status(401).json({ message: "Unauthorized" });
  }

  try {
    const token = header.split(" ")[1];
    const decoded = jwt.verify(token, process.env.JWT_SECRET!);
    req.user = decoded;
    next();
  } catch {
    return res.status(401).json({ message: "Invalid token" });
  }
}
```

### Testing the Middleware

```typescript
import request from "supertest";
import jwt from "jsonwebtoken";
import app from "../app";

describe("Auth Middleware", () => {
  const secret = process.env.JWT_SECRET || "test-secret";

  function generateToken(payload: object) {
    return jwt.sign(payload, secret, { expiresIn: "1h" });
  }

  it("allows access with valid token", async () => {
    const token = generateToken({ userId: "1", role: "admin" });

    const response = await request(app)
      .get("/api/profile")
      .set("Authorization", `Bearer ${token}`);

    expect(response.status).toBe(200);
  });

  it("rejects request without Authorization header", async () => {
    const response = await request(app).get("/api/profile");

    expect(response.status).toBe(401);
    expect(response.body.message).toBe("Unauthorized");
  });

  it("rejects expired token", async () => {
    const token = jwt.sign({ userId: "1" }, secret, { expiresIn: "-1h" });

    const response = await request(app)
      .get("/api/profile")
      .set("Authorization", `Bearer ${token}`);

    expect(response.status).toBe(401);
    expect(response.body.message).toBe("Invalid token");
  });

  it("rejects malformed token", async () => {
    const response = await request(app)
      .get("/api/profile")
      .set("Authorization", "Bearer not.a.valid.token");

    expect(response.status).toBe(401);
  });
});
```

---

## Testing Validation Errors (400 Responses)

```typescript
describe("POST /api/users — validation", () => {
  it("rejects empty body", async () => {
    const response = await request(app).post("/api/users").send({});

    expect(response.status).toBe(400);
    expect(response.body.errors).toEqual(
      expect.arrayContaining([
        expect.objectContaining({ field: "name", message: "Name is required" }),
        expect.objectContaining({
          field: "email",
          message: "Email is required",
        }),
      ]),
    );
  });

  it("rejects invalid email format", async () => {
    const response = await request(app)
      .post("/api/users")
      .send({ name: "Vikas", email: "not-an-email", password: "pass123" });

    expect(response.status).toBe(400);
    expect(response.body.errors[0]).toMatchObject({
      field: "email",
      message: "Invalid email format",
    });
  });

  it("rejects short password", async () => {
    const response = await request(app)
      .post("/api/users")
      .send({ name: "Vikas", email: "v@test.com", password: "ab" });

    expect(response.status).toBe(400);
    expect(response.body.errors[0].field).toBe("password");
  });

  it("rejects duplicate email", async () => {
    // First user
    await request(app)
      .post("/api/users")
      .send({ name: "Vikas", email: "v@test.com", password: "pass123" });

    // Duplicate
    const response = await request(app)
      .post("/api/users")
      .send({ name: "Other", email: "v@test.com", password: "pass456" });

    expect(response.status).toBe(409);
    expect(response.body.message).toBe("Email already exists");
  });
});
```

---

## Organizing Test Files

### Option 1: Co-locate with Source (Recommended)

```
src/
├── routes/
│   ├── users.ts
│   ├── users.test.ts      ← test next to route
│   ├── orders.ts
│   └── orders.test.ts
├── middleware/
│   ├── auth.ts
│   └── auth.test.ts
└── models/
    ├── User.ts
    └── User.test.ts
```

### Option 2: Separate `__tests__` Folder

```
src/
├── routes/
│   ├── users.ts
│   └── orders.ts
└── __tests__/
    ├── routes/
    │   ├── users.test.ts
    │   └── orders.test.ts
    ├── middleware/
    │   └── auth.test.ts
    └── setup.ts
```

### Option 3: Top-Level tests Directory

```
project/
├── src/
│   └── ...
└── tests/
    ├── integration/
    │   ├── users.test.ts
    │   └── orders.test.ts
    ├── unit/
    │   └── auth.test.ts
    ├── fixtures/
    │   └── users.json
    └── setup.ts
```

### Recommendation

Co-locate tests with source for smaller projects. As the project grows, separate integration and unit tests into their own directories for clarity.

---

## Test Database Patterns (Lifecycle Hooks)

### Pattern 1: In-Memory Database (MongoDB)

```typescript
import { MongoMemoryServer } from "mongodb-memory-server";
import mongoose from "mongoose";

let mongo: MongoMemoryServer;

beforeAll(async () => {
  mongo = await MongoMemoryServer.create();
  await mongoose.connect(mongo.getUri());
});

afterEach(async () => {
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    await collections[key].deleteMany({});
  }
});

afterAll(async () => {
  await mongoose.disconnect();
  await mongo.stop();
});
```

### Pattern 2: Transaction Rollback (PostgreSQL)

```typescript
import { Pool } from "pg";

const pool = new Pool({ connectionString: process.env.TEST_DATABASE_URL });

let client;

beforeEach(async () => {
  client = await pool.connect();
  await client.query("BEGIN"); // Start transaction
});

afterEach(async () => {
  await client.query("ROLLBACK"); // Undo everything
  client.release();
});

afterAll(async () => {
  await pool.end();
});
```

### Pattern 3: Seed and Clean (General)

```typescript
import { seedDatabase, cleanDatabase } from "./helpers";

beforeAll(async () => {
  await connectToTestDB();
});

beforeEach(async () => {
  await seedDatabase(); // Insert test data
});

afterEach(async () => {
  await cleanDatabase(); // Truncate all tables
});

afterAll(async () => {
  await disconnectFromDB();
});
```

### Fixture Files

```json
// tests/fixtures/users.json
[
  {
    "name": "Vikas",
    "email": "vikas@test.com",
    "role": "admin"
  },
  {
    "name": "Alice",
    "email": "alice@test.com",
    "role": "user"
  }
]
```

```typescript
import users from "../fixtures/users.json";

beforeEach(async () => {
  await User.insertMany(users);
});
```

---

## Best Practices

1. **Separate app from server** — export the Express app without `.listen()` for Supertest.
2. **Test the contract** — assert status codes, response shape, and headers.
3. **Use `toMatchObject`** over `toEqual` — ignore dynamic fields (id, createdAt) in assertions.
4. **Isolate tests** — each test should set up its own data and clean up after.
5. **Test error paths** — 400, 401, 403, 404, 500 are as important as 200.
6. **Use factories for test data** — avoids repetition and keeps tests DRY.
7. **Keep integration and unit tests separate** — integration tests hit real/memory DB, unit tests mock everything.
8. **Test middleware independently** — auth, validation, and rate-limiting deserve their own test suites.
9. **Use environment variables for test config** — `NODE_ENV=test` and `TEST_DATABASE_URL`.
10. **Run tests in CI** — Supertest tests should be part of your PR checks.

---

## Common Mistakes

| Mistake                                   | Why It Is a Problem                 | Fix                                            |
| ----------------------------------------- | ----------------------------------- | ---------------------------------------------- |
| Starting server with `.listen()` in tests | Port conflicts, slow tests          | Export app separately, use `request(app)`      |
| Shared state between tests                | Tests pass alone but fail together  | Use `beforeEach`/`afterEach` for setup/cleanup |
| Not testing error responses               | Bugs in error handling go unnoticed | Test 400, 401, 404, 500 paths                  |
| Hardcoding test data in assertions        | Brittle when DB generates IDs       | Use `expect.any(Number)` or `toMatchObject`    |
| Testing with production database          | Accidentally deletes real data      | Always use a test database or in-memory DB     |
| No timeout handling                       | Tests hang forever on failure       | Set test timeout: `vitest --timeout 10000`     |
| Forgetting to `await` the request         | Assertions run before response      | Always `await request(app).get(...)`           |

---

## Summary

- **Supertest** makes HTTP requests to your Express app in-process — fast and no port needed.
- **Separate `app.ts` from `server.ts`** — export the app for testing, only listen in production.
- Test all HTTP methods (**GET, POST, PUT, DELETE**) and assert status codes, headers, and body.
- **Send headers** with `.set()` and **body** with `.send()`.
- Use **real databases** (in-memory or Docker) for integration tests and **mocks** for unit tests.
- Test **authentication middleware** with valid tokens, expired tokens, and missing tokens.
- Test **validation errors** thoroughly — 400 responses protect your API from bad input.
- Use **lifecycle hooks** (`beforeEach`, `afterEach`) to seed and clean test data.
- Keep tests **isolated** — no test should depend on another test's state.
