# HTTP Server Without Express

## Why Learn the Core `http` Module?

Understanding Node's built-in `http` module shows you what Express abstracts away. It helps you:

- Debug low-level HTTP issues.
- Appreciate what middleware and routing frameworks provide.
- Build ultra-lightweight services when Express is overkill.

---

## Creating a Basic Server

```javascript
const http = require("http");

const server = http.createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "text/plain" });
  res.end("Hello, World!");
});

server.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});
```

- `req` — IncomingMessage object (request details).
- `res` — ServerResponse object (what you send back).
- `res.writeHead()` — sets status code and headers.
- `res.end()` — sends the response and closes the connection.

---

## Request Object (`req`)

```javascript
const server = http.createServer((req, res) => {
  console.log(req.method); // "GET", "POST", etc.
  console.log(req.url); // "/users?page=2"
  console.log(req.headers); // { host: "localhost:3000", ... }

  // Parse URL
  const url = new URL(req.url, `http://${req.headers.host}`);
  console.log(url.pathname); // "/users"
  console.log(url.searchParams.get("page")); // "2"

  res.end("OK");
});
```

---

## Reading Request Body

The body comes in chunks (streams) — you must collect them:

```javascript
const server = http.createServer((req, res) => {
  if (req.method === "POST") {
    let body = "";

    req.on("data", (chunk) => {
      body += chunk.toString();
    });

    req.on("end", () => {
      const parsed = JSON.parse(body);
      console.log("Received:", parsed);

      res.writeHead(201, { "Content-Type": "application/json" });
      res.end(JSON.stringify({ received: true, data: parsed }));
    });
  } else {
    res.writeHead(405);
    res.end("Method Not Allowed");
  }
});
```

---

## Manual Routing

```javascript
const http = require("http");

const users = [
  { id: 1, name: "Vikas" },
  { id: 2, name: "Rahul" },
];

const server = http.createServer((req, res) => {
  const url = new URL(req.url, `http://${req.headers.host}`);
  const path = url.pathname;
  const method = req.method;

  // Set JSON header for all responses
  res.setHeader("Content-Type", "application/json");

  // GET /api/users
  if (method === "GET" && path === "/api/users") {
    res.writeHead(200);
    res.end(JSON.stringify(users));
  }

  // GET /api/users/:id
  else if (method === "GET" && path.match(/^\/api\/users\/\d+$/)) {
    const id = parseInt(path.split("/").pop());
    const user = users.find((u) => u.id === id);

    if (user) {
      res.writeHead(200);
      res.end(JSON.stringify(user));
    } else {
      res.writeHead(404);
      res.end(JSON.stringify({ error: "User not found" }));
    }
  }

  // POST /api/users
  else if (method === "POST" && path === "/api/users") {
    let body = "";
    req.on("data", (chunk) => (body += chunk));
    req.on("end", () => {
      const newUser = JSON.parse(body);
      newUser.id = users.length + 1;
      users.push(newUser);

      res.writeHead(201);
      res.end(JSON.stringify(newUser));
    });
  }

  // 404
  else {
    res.writeHead(404);
    res.end(JSON.stringify({ error: "Route not found" }));
  }
});

server.listen(3000, () => console.log("Server on port 3000"));
```

---

## Serving HTML & Static Files

```javascript
const http = require("http");
const fs = require("fs");
const path = require("path");

const MIME_TYPES = {
  ".html": "text/html",
  ".css": "text/css",
  ".js": "text/javascript",
  ".json": "application/json",
  ".png": "image/png",
  ".jpg": "image/jpeg",
  ".svg": "image/svg+xml",
};

const server = http.createServer((req, res) => {
  let filePath = path.join(
    __dirname,
    "public",
    req.url === "/" ? "index.html" : req.url,
  );
  const ext = path.extname(filePath);
  const contentType = MIME_TYPES[ext] || "application/octet-stream";

  fs.readFile(filePath, (err, content) => {
    if (err) {
      if (err.code === "ENOENT") {
        res.writeHead(404, { "Content-Type": "text/html" });
        res.end("<h1>404 — Not Found</h1>");
      } else {
        res.writeHead(500);
        res.end("Internal Server Error");
      }
    } else {
      res.writeHead(200, { "Content-Type": contentType });
      res.end(content);
    }
  });
});

server.listen(3000);
```

---

## Why Express Is Better for Real Apps

| Feature        | Raw `http`                        | Express                         |
| -------------- | --------------------------------- | ------------------------------- |
| Routing        | Manual regex/string matching      | `app.get("/users/:id")`         |
| Body parsing   | Manual chunk collection           | `express.json()` middleware     |
| Static files   | Manual MIME types + `fs.readFile` | `express.static("public")`      |
| Error handling | Manual try/catch in every route   | Centralized error middleware    |
| Middleware     | DIY                               | Built-in + huge ecosystem       |
| Code volume    | ~50 lines for basic routing       | ~5 lines for same functionality |

---

## Summary

- Node's `http` module creates servers from scratch — no dependencies needed.
- Request bodies arrive as streams — collect chunks and parse on `end` event.
- Manual routing uses `req.url` and `req.method` with conditionals.
- This approach is educational but impractical for production — use Express or a similar framework.
- Understanding the raw layer helps debug issues and appreciate framework abstractions.
