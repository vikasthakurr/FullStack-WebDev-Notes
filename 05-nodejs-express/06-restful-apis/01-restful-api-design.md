# RESTful API Design

## What Is REST?

REST (Representational State Transfer) is an architectural style for designing networked applications. A RESTful API uses HTTP methods and URLs to perform CRUD operations on resources.

**Analogy:** REST is like a library system. Resources are books, URLs are shelf locations, HTTP methods are actions (check out, return, add, remove), and status codes are the librarian's responses.

---

## REST Principles

1. **Client-Server** — frontend and backend are separate; communicate over HTTP.
2. **Stateless** — each request contains all information needed; server stores no session state.
3. **Uniform Interface** — consistent URL patterns and HTTP methods.
4. **Resource-Based** — everything is a resource identified by a URL.
5. **Cacheable** — responses indicate if they can be cached.

---

## URL Design

### Resource Naming Conventions

```
✅ Good:
GET    /api/users          → List users
GET    /api/users/123      → Get specific user
POST   /api/users          → Create user
PUT    /api/users/123      → Replace user
PATCH  /api/users/123      → Partial update user
DELETE /api/users/123      → Delete user

❌ Bad:
GET /api/getUsers
POST /api/createUser
GET /api/deleteUser/123
```

**Rules:**

- Use **nouns**, not verbs (the HTTP method is the verb).
- Use **plural** names (`/users`, not `/user`).
- Use **lowercase** with hyphens for multi-word: `/blog-posts`.
- Nest related resources: `/users/123/orders`.

### Nested Resources

```
GET /api/users/123/orders          → Orders for user 123
GET /api/users/123/orders/456      → Specific order for user 123
POST /api/users/123/orders         → Create order for user 123
```

### Filtering, Sorting, Pagination

```
GET /api/users?role=admin&status=active       → Filter
GET /api/users?sort=name&order=asc            → Sort
GET /api/users?page=2&limit=20                → Pagination
GET /api/products?minPrice=10&maxPrice=100    → Range filter
GET /api/users?fields=name,email              → Field selection
```

---

## HTTP Methods & Status Codes

### Methods

| Method | Purpose          | Idempotent | Body |
| ------ | ---------------- | ---------- | ---- |
| GET    | Read resource    | Yes        | No   |
| POST   | Create resource  | No         | Yes  |
| PUT    | Replace resource | Yes        | Yes  |
| PATCH  | Partial update   | No\*       | Yes  |
| DELETE | Remove resource  | Yes        | No   |

\*PATCH can be idempotent depending on implementation.

### Status Codes

| Code | Meaning               | When to Use                                |
| ---- | --------------------- | ------------------------------------------ |
| 200  | OK                    | Successful GET, PUT, PATCH                 |
| 201  | Created               | Successful POST (include created resource) |
| 204  | No Content            | Successful DELETE                          |
| 400  | Bad Request           | Invalid input, validation errors           |
| 401  | Unauthorized          | Missing or invalid authentication          |
| 403  | Forbidden             | Authenticated but lacks permission         |
| 404  | Not Found             | Resource does not exist                    |
| 409  | Conflict              | Duplicate resource, version mismatch       |
| 422  | Unprocessable Entity  | Semantically invalid data                  |
| 429  | Too Many Requests     | Rate limit exceeded                        |
| 500  | Internal Server Error | Unexpected server failure                  |

---

## Complete CRUD API

```javascript
const express = require("express");
const { v4: uuidv4 } = require("uuid");
const app = express();

app.use(express.json());

let users = [
  { id: "1", name: "Vikas", email: "vikas@example.com", role: "admin" },
  { id: "2", name: "Rahul", email: "rahul@example.com", role: "user" },
];

// GET all users (with filtering and pagination)
app.get("/api/users", (req, res) => {
  let result = [...users];

  // Filtering
  if (req.query.role) {
    result = result.filter((u) => u.role === req.query.role);
  }

  // Pagination
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  const startIndex = (page - 1) * limit;
  const endIndex = startIndex + limit;

  const paginatedResult = result.slice(startIndex, endIndex);

  res.json({
    data: paginatedResult,
    pagination: {
      total: result.length,
      page,
      limit,
      totalPages: Math.ceil(result.length / limit),
    },
  });
});

// GET single user
app.get("/api/users/:id", (req, res) => {
  const user = users.find((u) => u.id === req.params.id);

  if (!user) {
    return res.status(404).json({ error: "User not found" });
  }

  res.json({ data: user });
});

// POST create user
app.post("/api/users", (req, res) => {
  const { name, email, role } = req.body;

  // Validation
  if (!name || !email) {
    return res.status(400).json({
      error: "Validation failed",
      details: [
        ...(!name ? ["name is required"] : []),
        ...(!email ? ["email is required"] : []),
      ],
    });
  }

  // Check duplicate
  if (users.find((u) => u.email === email)) {
    return res.status(409).json({ error: "Email already exists" });
  }

  const newUser = { id: uuidv4(), name, email, role: role || "user" };
  users.push(newUser);

  res.status(201).json({ data: newUser });
});

// PUT replace user
app.put("/api/users/:id", (req, res) => {
  const index = users.findIndex((u) => u.id === req.params.id);

  if (index === -1) {
    return res.status(404).json({ error: "User not found" });
  }

  const { name, email, role } = req.body;
  if (!name || !email) {
    return res.status(400).json({ error: "name and email are required" });
  }

  users[index] = { id: req.params.id, name, email, role: role || "user" };
  res.json({ data: users[index] });
});

// PATCH partial update
app.patch("/api/users/:id", (req, res) => {
  const user = users.find((u) => u.id === req.params.id);

  if (!user) {
    return res.status(404).json({ error: "User not found" });
  }

  const allowedFields = ["name", "email", "role"];
  for (const field of allowedFields) {
    if (req.body[field] !== undefined) {
      user[field] = req.body[field];
    }
  }

  res.json({ data: user });
});

// DELETE user
app.delete("/api/users/:id", (req, res) => {
  const index = users.findIndex((u) => u.id === req.params.id);

  if (index === -1) {
    return res.status(404).json({ error: "User not found" });
  }

  users.splice(index, 1);
  res.status(204).send();
});

app.listen(3000, () => console.log("API running on port 3000"));
```

---

## Response Format

### Consistent JSON Response Structure

```javascript
// Success
{
  "data": { ... },         // or [...] for lists
  "pagination": { ... },   // for paginated lists
  "message": "User created successfully"
}

// Error
{
  "error": "Validation failed",
  "details": ["name is required", "email must be valid"],
  "statusCode": 400
}
```

---

## Testing APIs

### With Postman / Thunder Client

1. Set request method (GET, POST, etc.).
2. Enter URL (`http://localhost:3000/api/users`).
3. For POST/PUT — set Body tab → raw → JSON.
4. Add headers if needed (Authorization, Content-Type).
5. Send and inspect response.

### With `curl`

```bash
# GET
curl http://localhost:3000/api/users

# POST
curl -X POST http://localhost:3000/api/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Priya", "email": "priya@example.com"}'

# DELETE
curl -X DELETE http://localhost:3000/api/users/1
```

---

## Best Practices

1. **Use proper HTTP methods and status codes** — not everything is a 200 OK.
2. **Validate all input** — never trust client data.
3. **Return the created/updated resource** in POST/PUT/PATCH responses.
4. **Use consistent response format** — always wrap in `{ data: ... }` or `{ error: ... }`.
5. **Version your API** — `/api/v1/users` allows breaking changes without affecting existing clients.
6. **Implement pagination** for list endpoints — never return unbounded data.
7. **Use UUID or similar** for IDs — not sequential integers (prevents enumeration attacks).

---

## Common Mistakes

| Mistake                         | Why It Is Wrong                                | Fix                                          |
| ------------------------------- | ---------------------------------------------- | -------------------------------------------- |
| Using verbs in URLs             | REST uses HTTP methods as verbs                | `/users` not `/getUsers`                     |
| Returning 200 for everything    | Client cannot distinguish success from failure | Use proper status codes                      |
| Not validating input            | Security vulnerabilities, crashes              | Validate and sanitize all input              |
| Returning full objects always   | Performance and security issues                | Allow field selection, omit sensitive fields |
| No pagination on list endpoints | Large datasets crash or timeout                | Always paginate with defaults                |
| Sequential integer IDs          | Attackers can enumerate all resources          | Use UUID or similar                          |

---

## Summary

- REST uses HTTP methods as verbs on noun-based resource URLs.
- Follow conventions: plural nouns, proper status codes, consistent response shapes.
- Implement filtering, sorting, and pagination for list endpoints.
- Validate all input, return appropriate error details, and version your API.
- Test with Postman, Thunder Client, or curl before writing frontend code.
