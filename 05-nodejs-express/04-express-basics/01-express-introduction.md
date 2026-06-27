# Express.js Basics

## What Is Express?

Express is a minimal, flexible Node.js web framework that provides a robust set of features for building web applications and REST APIs. It is the most popular Node.js framework, used by companies like Uber, IBM, and Netflix.

**Analogy:** If Node.js's `http` module is building a house from raw materials (bricks, cement, wood), Express is a prefabricated house kit — you still assemble it yourself, but the structural pieces are pre-made and designed to fit together.

---

## Why Express?

- Node's built-in `http` module is verbose for real apps (manual routing, parsing, etc.).
- Express adds routing, middleware, request/response utilities, and a plugin ecosystem.
- Unopinionated — does not force a specific structure or ORM.
- Huge ecosystem of middleware for auth, validation, logging, etc.

---

## Setup

```bash
mkdir my-api && cd my-api
npm init -y
npm install express
npm install --save-dev nodemon
```

### package.json Scripts

```json
{
  "scripts": {
    "start": "node src/index.js",
    "dev": "nodemon src/index.js"
  }
}
```

---

## First Express Server

```javascript
const express = require("express");
const app = express();
const PORT = 3000;

// Middleware to parse JSON bodies
app.use(express.json());

// Route
app.get("/", (req, res) => {
  res.json({ message: "Hello from Express!" });
});

// Start server
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

---

## Routing

### HTTP Methods

```javascript
// GET — retrieve data
app.get("/users", (req, res) => {
  res.json(users);
});

// POST — create data
app.post("/users", (req, res) => {
  const newUser = req.body;
  users.push(newUser);
  res.status(201).json(newUser);
});

// PUT — replace entire resource
app.put("/users/:id", (req, res) => {
  const { id } = req.params;
  // Replace user with id
  res.json({ updated: true });
});

// PATCH — partial update
app.patch("/users/:id", (req, res) => {
  // Update specific fields
  res.json({ patched: true });
});

// DELETE — remove data
app.delete("/users/:id", (req, res) => {
  res.status(204).send(); // No content
});
```

### Route Parameters

```javascript
// /users/123
app.get("/users/:id", (req, res) => {
  const userId = req.params.id; // "123"
  res.json({ id: userId });
});

// Multiple params: /posts/5/comments/12
app.get("/posts/:postId/comments/:commentId", (req, res) => {
  const { postId, commentId } = req.params;
  res.json({ postId, commentId });
});
```

### Query Strings

```javascript
// /search?q=javascript&page=2&limit=10
app.get("/search", (req, res) => {
  const { q, page = 1, limit = 10 } = req.query;
  res.json({ query: q, page, limit });
});
```

---

## Request Object (`req`)

```javascript
app.post("/users", (req, res) => {
  req.params; // Route parameters (:id)
  req.query; // Query string (?key=value)
  req.body; // Request body (needs express.json() middleware)
  req.headers; // Request headers
  req.method; // "GET", "POST", etc.
  req.path; // URL path
  req.ip; // Client IP address
});
```

## Response Object (`res`)

```javascript
app.get("/demo", (req, res) => {
  // Send JSON
  res.json({ name: "Vikas" });

  // Send plain text
  res.send("Hello");

  // Set status code
  res.status(404).json({ error: "Not found" });

  // Set headers
  res.set("X-Custom-Header", "value");

  // Redirect
  res.redirect("/new-url");
  res.redirect(301, "/permanent-redirect");

  // Send file
  res.sendFile("/path/to/file.pdf");
});
```

---

## Express Router (Modular Routing)

Split routes into separate files:

```javascript
// routes/users.js
const express = require("express");
const router = express.Router();

router.get("/", (req, res) => {
  res.json({ users: [] });
});

router.get("/:id", (req, res) => {
  res.json({ user: req.params.id });
});

router.post("/", (req, res) => {
  res.status(201).json(req.body);
});

module.exports = router;
```

```javascript
// index.js
const express = require("express");
const userRoutes = require("./routes/users");

const app = express();
app.use(express.json());

app.use("/api/users", userRoutes); // Prefix all routes with /api/users

app.listen(3000);
```

---

## MVC Structure

```
src/
├── controllers/    # Request handling logic
│   └── userController.js
├── routes/         # Route definitions
│   └── userRoutes.js
├── models/         # Data models (Mongoose, etc.)
│   └── User.js
├── middleware/     # Custom middleware
│   └── auth.js
├── utils/          # Helper functions
├── config/         # Configuration
└── index.js        # Entry point
```

---

## Middleware

Middleware functions have access to `req`, `res`, and `next()`:

```javascript
// Built-in middleware
app.use(express.json()); // Parse JSON bodies
app.use(express.urlencoded({ extended: true })); // Parse form data
app.use(express.static("public")); // Serve static files

// Custom middleware
function logger(req, res, next) {
  console.log(`${req.method} ${req.path} — ${new Date().toISOString()}`);
  next(); // Pass to next middleware/route
}

app.use(logger); // Apply to all routes

// Error-handling middleware (4 parameters)
function errorHandler(err, req, res, next) {
  console.error(err.stack);
  res.status(500).json({ error: "Something went wrong" });
}

app.use(errorHandler); // Must be after all routes
```

---

## Status Codes & Best Practices

| Code | Meaning               | When to Use                        |
| ---- | --------------------- | ---------------------------------- |
| 200  | OK                    | Successful GET, PUT, PATCH         |
| 201  | Created               | Successful POST (resource created) |
| 204  | No Content            | Successful DELETE                  |
| 400  | Bad Request           | Invalid input / validation error   |
| 401  | Unauthorized          | Not authenticated                  |
| 403  | Forbidden             | Authenticated but not permitted    |
| 404  | Not Found             | Resource does not exist            |
| 500  | Internal Server Error | Unexpected server failure          |

---

## CRUD API Example

```javascript
const express = require("express");
const app = express();
app.use(express.json());

let todos = [
  { id: 1, title: "Learn Express", done: false },
  { id: 2, title: "Build API", done: false },
];

// GET all
app.get("/api/todos", (req, res) => {
  res.json(todos);
});

// GET one
app.get("/api/todos/:id", (req, res) => {
  const todo = todos.find((t) => t.id === parseInt(req.params.id));
  if (!todo) return res.status(404).json({ error: "Todo not found" });
  res.json(todo);
});

// POST
app.post("/api/todos", (req, res) => {
  const { title } = req.body;
  if (!title) return res.status(400).json({ error: "Title is required" });

  const newTodo = { id: Date.now(), title, done: false };
  todos.push(newTodo);
  res.status(201).json(newTodo);
});

// PUT
app.put("/api/todos/:id", (req, res) => {
  const todo = todos.find((t) => t.id === parseInt(req.params.id));
  if (!todo) return res.status(404).json({ error: "Todo not found" });

  todo.title = req.body.title || todo.title;
  todo.done = req.body.done ?? todo.done;
  res.json(todo);
});

// DELETE
app.delete("/api/todos/:id", (req, res) => {
  todos = todos.filter((t) => t.id !== parseInt(req.params.id));
  res.status(204).send();
});

app.listen(3000, () => console.log("Server on port 3000"));
```

---

## Summary

- Express is the standard Node.js web framework — minimal, flexible, and middleware-based.
- Routes map HTTP methods + URL patterns to handler functions.
- `req.params` for route parameters, `req.query` for query strings, `req.body` for request body.
- Middleware processes requests in sequence — use `next()` to pass control.
- Express Router splits routes into modular files for organized code.
- Follow REST conventions: proper HTTP methods, status codes, and resource-based URLs.
