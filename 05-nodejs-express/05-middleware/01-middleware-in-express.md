# Middleware in Express

## What Is Middleware?

Middleware functions are functions that have access to the request object (`req`), the response object (`res`), and the `next` function in the request-response cycle. They execute sequentially and can:

- Execute any code.
- Modify `req` and `res` objects.
- End the request-response cycle.
- Call the next middleware with `next()`.

**Analogy:** Middleware is like security checkpoints at an airport. Each checkpoint (middleware) examines the passenger (request), can approve them to proceed (`next()`), reject them (`res.status(403).send()`), or add information to their boarding pass (attach data to `req`).

```mermaid
flowchart LR
    REQ["Request"] --> MW1["Middleware 1"]
    MW1 -->|"next()"| MW2["Middleware 2"]
    MW2 -->|"next()"| MW3["Middleware 3"]
    MW3 -->|"next()"| ROUTE["Route Handler"]
    ROUTE --> RES["Response"]
```

---

## Types of Middleware

### 1. Application-Level Middleware

Applied to the entire app or specific routes:

```javascript
const express = require("express");
const app = express();

// Runs for ALL routes
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path} — ${new Date().toISOString()}`);
  next();
});

// Runs only for /api/* routes
app.use("/api", (req, res, next) => {
  console.log("API request");
  next();
});
```

### 2. Router-Level Middleware

Applied to a specific router:

```javascript
const router = express.Router();

router.use((req, res, next) => {
  console.log("Router middleware");
  next();
});

router.get("/users", (req, res) => {
  res.json([]);
});
```

### 3. Built-in Middleware

```javascript
// Parse JSON request bodies
app.use(express.json());

// Parse URL-encoded form bodies
app.use(express.urlencoded({ extended: true }));

// Serve static files from "public" folder
app.use(express.static("public"));
```

### 4. Third-Party Middleware

```javascript
const cors = require("cors");
const morgan = require("morgan");
const helmet = require("helmet");

app.use(cors()); // Enable Cross-Origin requests
app.use(morgan("dev")); // HTTP request logging
app.use(helmet()); // Security headers
```

---

## Built-in Middleware Deep Dive

### `express.json()`

Parses incoming JSON payloads:

```javascript
app.use(express.json({ limit: "10mb" })); // Max body size

app.post("/api/users", (req, res) => {
  console.log(req.body); // Parsed JSON object
  res.json(req.body);
});
```

Without this middleware, `req.body` is `undefined`.

### `express.urlencoded()`

Parses form submissions (`application/x-www-form-urlencoded`):

```javascript
app.use(express.urlencoded({ extended: true }));
// extended: true — allows nested objects
// extended: false — only simple key-value pairs

app.post("/login", (req, res) => {
  const { username, password } = req.body;
});
```

### `express.static()`

Serves files from a directory:

```javascript
app.use(express.static("public"));
// GET /style.css → serves public/style.css
// GET /images/logo.png → serves public/images/logo.png

// With virtual path prefix
app.use("/assets", express.static("public"));
// GET /assets/style.css → serves public/style.css
```

---

## Writing Custom Middleware

### Request Logger

```javascript
function requestLogger(req, res, next) {
  const start = Date.now();

  // Runs after response is sent
  res.on("finish", () => {
    const duration = Date.now() - start;
    console.log(`${req.method} ${req.path} ${res.statusCode} — ${duration}ms`);
  });

  next();
}

app.use(requestLogger);
```

### Authentication Middleware

```javascript
function authenticate(req, res, next) {
  const token = req.headers.authorization?.split(" ")[1]; // "Bearer <token>"

  if (!token) {
    return res.status(401).json({ error: "No token provided" });
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded; // Attach user info to request
    next();
  } catch (error) {
    return res.status(401).json({ error: "Invalid token" });
  }
}

// Apply to specific routes
app.get("/api/profile", authenticate, (req, res) => {
  res.json({ user: req.user });
});

// Apply to all routes under a router
const protectedRouter = express.Router();
protectedRouter.use(authenticate);
protectedRouter.get("/dashboard", (req, res) => {
  /* ... */
});
```

### Rate Limiting Middleware

```javascript
function rateLimit(maxRequests, windowMs) {
  const requests = new Map();

  return (req, res, next) => {
    const ip = req.ip;
    const now = Date.now();
    const windowStart = now - windowMs;

    // Get requests for this IP within the window
    const ipRequests = requests.get(ip) || [];
    const recentRequests = ipRequests.filter((time) => time > windowStart);

    if (recentRequests.length >= maxRequests) {
      return res.status(429).json({ error: "Too many requests" });
    }

    recentRequests.push(now);
    requests.set(ip, recentRequests);
    next();
  };
}

app.use("/api", rateLimit(100, 15 * 60 * 1000)); // 100 requests per 15 minutes
```

### Request Validation Middleware

```javascript
function validateBody(schema) {
  return (req, res, next) => {
    const errors = [];

    for (const [field, rules] of Object.entries(schema)) {
      const value = req.body[field];

      if (rules.required && (value === undefined || value === "")) {
        errors.push(`${field} is required`);
      }
      if (rules.type && value !== undefined && typeof value !== rules.type) {
        errors.push(`${field} must be a ${rules.type}`);
      }
      if (rules.minLength && value && value.length < rules.minLength) {
        errors.push(`${field} must be at least ${rules.minLength} characters`);
      }
    }

    if (errors.length > 0) {
      return res.status(400).json({ errors });
    }

    next();
  };
}

// Usage
app.post(
  "/api/users",
  validateBody({
    name: { required: true, type: "string", minLength: 2 },
    email: { required: true, type: "string" },
    age: { required: false, type: "number" },
  }),
  (req, res) => {
    // Body is validated — safe to process
    res.status(201).json(req.body);
  },
);
```

---

## Error-Handling Middleware

Error middleware has **four parameters** — Express uses the parameter count to identify it:

```javascript
// Must be defined AFTER all routes
app.use((err, req, res, next) => {
  console.error(err.stack);

  // Custom error classes
  if (err.name === "ValidationError") {
    return res.status(400).json({ error: err.message });
  }

  if (err.name === "UnauthorizedError") {
    return res.status(401).json({ error: "Invalid credentials" });
  }

  // Default
  res.status(err.status || 500).json({
    error:
      process.env.NODE_ENV === "production"
        ? "Internal Server Error"
        : err.message,
  });
});
```

### Triggering Error Middleware

```javascript
app.get("/api/users/:id", async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) {
      const error = new Error("User not found");
      error.status = 404;
      throw error;
    }
    res.json(user);
  } catch (error) {
    next(error); // Passes to error-handling middleware
  }
});
```

### Async Error Wrapper

```javascript
function asyncHandler(fn) {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
}

// Usage — no try/catch needed
app.get(
  "/api/users",
  asyncHandler(async (req, res) => {
    const users = await User.find(); // If this throws, error middleware catches it
    res.json(users);
  }),
);
```

---

## Middleware Execution Order

Order matters — middleware runs in the order it is defined:

```javascript
// 1. Runs first — applies to all routes
app.use(express.json());
app.use(morgan("dev"));
app.use(cors());

// 2. Runs for /api/* routes
app.use("/api", authenticate);

// 3. Route handlers
app.get("/api/users", getUsers);
app.post("/api/users", createUser);

// 4. 404 handler (no route matched)
app.use((req, res) => {
  res.status(404).json({ error: "Route not found" });
});

// 5. Error handler (must be last)
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.message });
});
```

---

## Best Practices

1. **Keep middleware focused** — each middleware does one thing (logging, auth, validation).
2. **Always call `next()`** — unless you end the response; otherwise the request hangs.
3. **Error middleware must be last** — after all routes and other middleware.
4. **Use `asyncHandler`** for async routes — avoids repetitive try/catch.
5. **Do not leak internal errors** in production — show generic messages to clients.
6. **Apply auth middleware selectively** — not all routes need authentication.

---

## Common Mistakes

| Mistake                        | Why It Is Wrong                               | Fix                                   |
| ------------------------------ | --------------------------------------------- | ------------------------------------- |
| Forgetting `next()`            | Request hangs indefinitely                    | Always call `next()` or send response |
| Error middleware with 3 params | Express treats it as regular middleware       | Must have exactly 4 params            |
| Error handler before routes    | Never reached — order matters                 | Place after all routes                |
| Not parsing body               | `req.body` is `undefined`                     | Add `express.json()` before routes    |
| Async errors not caught        | Unhandled promise rejections crash the server | Use `asyncHandler` wrapper            |

---

## Summary

- Middleware processes requests sequentially — each can modify `req`/`res`, end the cycle, or call `next()`.
- Built-in: `express.json()`, `express.urlencoded()`, `express.static()`.
- Custom middleware handles cross-cutting concerns: auth, logging, validation, rate limiting.
- Error middleware has 4 parameters and must be defined after all routes.
- Use `asyncHandler` to automatically catch errors in async route handlers.
- Order is critical — global middleware first, routes next, 404 handler, then error handler.
