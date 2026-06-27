# Node.js Core Modules — File System

## CommonJS vs ES Modules

### CommonJS (Traditional)

```javascript
// Importing
const fs = require("fs");
const { readFile, writeFile } = require("fs/promises");

// Exporting
module.exports = { myFunction };
module.exports.helper = () => {};
```

### ES Modules (Modern)

```javascript
// Importing
import fs from "fs";
import { readFile, writeFile } from "fs/promises";

// Exporting
export function myFunction() {}
export default class MyClass {}
```

To enable ESM in Node.js:

- Use `.mjs` extension, OR
- Add `"type": "module"` in `package.json`

| Feature         | CommonJS                       | ES Modules               |
| --------------- | ------------------------------ | ------------------------ |
| Syntax          | `require()` / `module.exports` | `import` / `export`      |
| Loading         | Synchronous                    | Asynchronous             |
| Top-level await | ❌                             | ✅                       |
| Tree shaking    | ❌                             | ✅                       |
| `__dirname`     | ✅ Available                   | ❌ Use `import.meta.url` |
| Default in Node | Yes (without config)           | Requires config          |

---

## The `fs` Module

The file system module provides APIs to interact with files and directories.

### Three Flavors

```javascript
const fs = require("fs");
const fsPromises = require("fs/promises");

// 1. Synchronous (blocking — avoid in servers)
const data = fs.readFileSync("file.txt", "utf-8");

// 2. Callback-based (non-blocking)
fs.readFile("file.txt", "utf-8", (err, data) => {
  if (err) throw err;
  console.log(data);
});

// 3. Promise-based (modern — recommended)
const data = await fsPromises.readFile("file.txt", "utf-8");
```

**Rule:** Use promise-based (`fs/promises`) for async code. Use sync only for scripts or startup config loading.

---

## Reading Files

```javascript
import { readFile, readdir } from "fs/promises";

// Read text file
const content = await readFile("./data/config.json", "utf-8");
const config = JSON.parse(content);

// Read binary file (no encoding = Buffer)
const imageBuffer = await readFile("./images/logo.png");

// Read directory contents
const files = await readdir("./data");
console.log(files); // ["file1.txt", "file2.txt", "subfolder"]

// Read directory with details
const entries = await readdir("./data", { withFileTypes: true });
entries.forEach((entry) => {
  console.log(`${entry.name} — ${entry.isDirectory() ? "dir" : "file"}`);
});
```

---

## Writing Files

```javascript
import { writeFile, appendFile } from "fs/promises";

// Write (creates or overwrites)
await writeFile("./output/result.txt", "Hello, World!", "utf-8");

// Write JSON
const data = { name: "Vikas", age: 25 };
await writeFile("./data/user.json", JSON.stringify(data, null, 2));

// Append to file
await appendFile(
  "./logs/app.log",
  `[${new Date().toISOString()}] Server started\n`,
);
```

---

## File Operations

```javascript
import { rename, unlink, copyFile, stat } from "fs/promises";

// Rename / Move
await rename("./old-name.txt", "./new-name.txt");
await rename("./file.txt", "./archive/file.txt"); // Move to another dir

// Delete file
await unlink("./temp/cache.json");

// Copy file
await copyFile("./source.txt", "./backup/source.txt");

// Get file info
const info = await stat("./data/config.json");
console.log(info.size); // Size in bytes
console.log(info.isFile()); // true
console.log(info.isDirectory()); // false
console.log(info.mtime); // Last modified time
```

---

## Directory Operations

```javascript
import { mkdir, rmdir, rm } from "fs/promises";

// Create directory (recursive creates parent dirs too)
await mkdir("./uploads/images/thumbnails", { recursive: true });

// Remove empty directory
await rmdir("./empty-folder");

// Remove directory with contents (like rm -rf)
await rm("./temp", { recursive: true, force: true });
```

---

## Checking If File/Directory Exists

```javascript
import { access, constants } from "fs/promises";

async function fileExists(path) {
  try {
    await access(path, constants.F_OK);
    return true;
  } catch {
    return false;
  }
}

if (await fileExists("./config.json")) {
  // File exists
}
```

---

## Watching Files

```javascript
import { watch } from "fs/promises";

// Watch for changes
const watcher = watch("./src", { recursive: true });

for await (const event of watcher) {
  console.log(`${event.eventType}: ${event.filename}`);
}
```

---

## The `path` Module

```javascript
import path from "path";

path.join("src", "utils", "helper.js"); // "src/utils/helper.js" (OS-aware)
path.resolve("./src", "index.js"); // Absolute path
path.basename("/home/user/file.txt"); // "file.txt"
path.dirname("/home/user/file.txt"); // "/home/user"
path.extname("app.config.js"); // ".js"
path.parse("/home/user/file.txt");
// { root: "/", dir: "/home/user", base: "file.txt", ext: ".txt", name: "file" }
```

**Always use `path.join()`** instead of string concatenation — it handles OS-specific separators (`/` vs `\`).

---

## Practical Example: JSON File Database

```javascript
import { readFile, writeFile } from "fs/promises";
import path from "path";

const DB_PATH = path.join(process.cwd(), "data", "users.json");

async function getUsers() {
  const data = await readFile(DB_PATH, "utf-8");
  return JSON.parse(data);
}

async function saveUsers(users) {
  await writeFile(DB_PATH, JSON.stringify(users, null, 2));
}

async function addUser(user) {
  const users = await getUsers();
  user.id = Date.now();
  users.push(user);
  await saveUsers(users);
  return user;
}
```

---

## Error Handling

```javascript
import { readFile } from "fs/promises";

try {
  const data = await readFile("./nonexistent.txt", "utf-8");
} catch (error) {
  if (error.code === "ENOENT") {
    console.error("File not found");
  } else if (error.code === "EACCES") {
    console.error("Permission denied");
  } else {
    throw error; // Re-throw unexpected errors
  }
}
```

Common error codes:

- `ENOENT` — file/directory not found
- `EACCES` — permission denied
- `EEXIST` — file already exists
- `EISDIR` — expected file but got directory

---

## Best Practices

1. **Use `fs/promises`** — cleaner than callbacks, non-blocking unlike sync.
2. **Always use `path.join()`** — never concatenate paths with `/` manually.
3. **Handle errors explicitly** — check error codes for graceful handling.
4. **Use `recursive: true`** for `mkdir` — avoids errors if parent dirs are missing.
5. **Never trust user input for file paths** — validate and sanitize to prevent path traversal attacks.
6. **Use streams for large files** — `readFile` loads the entire file into memory.

---

## Summary

- Node.js supports CommonJS (`require`) and ES Modules (`import`) — prefer ESM for new projects.
- The `fs` module comes in three flavors: sync, callback, and promise-based — use `fs/promises`.
- Core operations: read, write, append, rename, delete, copy, stat.
- The `path` module handles file paths in an OS-agnostic way.
- Always handle errors with `try/catch` and check error codes like `ENOENT`.
- For large files, use streams instead of loading everything into memory.
