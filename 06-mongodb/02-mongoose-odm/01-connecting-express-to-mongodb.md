# Mongoose ODM — Connecting Express to MongoDB

## What is an ODM?

An **ODM (Object Document Mapper)** is a library that maps MongoDB documents to JavaScript objects, providing schema enforcement, validation, and a clean API for database operations.

**Analogy:** If MongoDB is a warehouse where you can store anything anywhere, Mongoose is the warehouse manager who enforces labeling rules, checks quality, and knows where everything goes.

**Mongoose** is the most popular ODM for MongoDB + Node.js. It provides:

- Schema definitions with type enforcement
- Built-in validation
- Middleware (hooks)
- Query building
- Population (joining documents)

---

## Installation

```bash
npm install mongoose
```

---

## Connecting to MongoDB

### Basic Connection

```javascript
const mongoose = require("mongoose");

// Connection string (Atlas or local)
const MONGO_URI =
  "mongodb+srv://username:password@cluster0.xxxxx.mongodb.net/myapp?retryWrites=true&w=majority";

mongoose
  .connect(MONGO_URI)
  .then(() => console.log("✅ Connected to MongoDB"))
  .catch((err) => console.error("❌ Connection failed:", err));
```

### Production-Ready Connection (with Express)

```javascript
const express = require("express");
const mongoose = require("mongoose");
require("dotenv").config();

const app = express();
app.use(express.json());

// Connection with options
const connectDB = async () => {
  try {
    await mongoose.connect(process.env.MONGO_URI, {
      // These options are now defaults in Mongoose 6+, but good to know:
      // useNewUrlParser: true,      (default in v6+)
      // useUnifiedTopology: true,   (default in v6+)
    });
    console.log(`✅ MongoDB connected: ${mongoose.connection.host}`);
  } catch (error) {
    console.error("❌ MongoDB connection error:", error.message);
    process.exit(1); // Exit with failure
  }
};

// Connection events
mongoose.connection.on("disconnected", () => {
  console.log("⚠️ MongoDB disconnected");
});

// Start server after DB connects
connectDB().then(() => {
  app.listen(3000, () => console.log("Server running on port 3000"));
});
```

### Environment Variable (.env)

```env
MONGO_URI=mongodb+srv://vikas:mypassword@cluster0.abc123.mongodb.net/myapp?retryWrites=true&w=majority
PORT=3000
```

---

## Schema Definition

A **Schema** defines the structure, types, and rules for documents in a collection.

### Basic Schema

```javascript
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema({
  name: String,
  email: String,
  age: Number,
  isActive: Boolean,
  createdAt: Date,
});
```

### Schema with Types & Options

```javascript
const userSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: [true, "Name is required"],
      trim: true,
      minlength: [2, "Name must be at least 2 characters"],
      maxlength: [50, "Name cannot exceed 50 characters"],
    },
    email: {
      type: String,
      required: [true, "Email is required"],
      unique: true,
      lowercase: true,
      match: [/^\S+@\S+\.\S+$/, "Please enter a valid email"],
    },
    age: {
      type: Number,
      min: [0, "Age cannot be negative"],
      max: [120, "Age seems unrealistic"],
    },
    role: {
      type: String,
      enum: ["user", "admin", "moderator"],
      default: "user",
    },
    skills: {
      type: [String], // Array of strings
      default: [],
    },
    address: {
      city: String,
      state: String,
      zip: String,
    },
    profilePic: {
      type: String,
      default: "default-avatar.png",
    },
  },
  {
    timestamps: true, // Adds createdAt and updatedAt automatically
  },
);
```

### Schema Types Reference

| Type       | Example                 | Notes                                   |
| ---------- | ----------------------- | --------------------------------------- |
| `String`   | `"hello"`               | Trim, lowercase, uppercase, match, enum |
| `Number`   | `42`, `3.14`            | min, max                                |
| `Boolean`  | `true` / `false`        | —                                       |
| `Date`     | `new Date()`            | min, max, default: Date.now             |
| `ObjectId` | `Schema.Types.ObjectId` | Used for references                     |
| `Array`    | `[String]`, `[Number]`  | Array of any type                       |
| `Mixed`    | `Schema.Types.Mixed`    | Any type (no validation)                |
| `Buffer`   | Binary data             | For files                               |
| `Map`      | `Map<String, any>`      | Key-value pairs                         |

---

## Creating Models

A **Model** is a compiled version of a Schema — it provides the interface to the database.

```javascript
// Schema → Model → Document
const User = mongoose.model("User", userSchema);
// 'User' → MongoDB creates a 'users' collection (lowercase + plural)

module.exports = User;
```

### File Structure (Recommended)

```
models/
  User.js
  Post.js
  Comment.js
```

```javascript
// models/User.js
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema(
  {
    name: { type: String, required: true },
    email: { type: String, required: true, unique: true },
  },
  { timestamps: true },
);

module.exports = mongoose.model("User", userSchema);
```

---

## CRUD with Mongoose

### CREATE

```javascript
const User = require("./models/User");

// Method 1: Model.create()
const newUser = await User.create({
  name: "Vikas",
  email: "vikas@example.com",
  age: 25,
});

// Method 2: new + save()
const user = new User({ name: "Priya", email: "priya@example.com" });
await user.save();
```

### READ

```javascript
// Find all users
const users = await User.find();

// Find with filter
const delhiUsers = await User.find({ city: "Delhi" });

// Find one by ID
const user = await User.findById("64a7f2b3c9e1a2d4e8f01234");

// Find one by condition
const admin = await User.findOne({ role: "admin" });

// With query chaining
const results = await User.find({ age: { $gte: 18 } })
  .select("name email -_id") // projection
  .sort({ name: 1 }) // sort ascending
  .skip(0) // pagination offset
  .limit(10); // pagination limit
```

### UPDATE

```javascript
// Find by ID and update (returns the UPDATED document with { new: true })
const updatedUser = await User.findByIdAndUpdate(
  "64a7f2b3c9e1a2d4e8f01234",
  { $set: { age: 26, role: "admin" } },
  { new: true, runValidators: true }, // new: true returns updated doc
);

// Update one
await User.updateOne({ email: "vikas@example.com" }, { $inc: { age: 1 } });

// Update many
await User.updateMany({ role: "user" }, { $set: { isActive: true } });
```

### DELETE

```javascript
// Find by ID and delete
const deletedUser = await User.findByIdAndDelete("64a7f2b3c9e1a2d4e8f01234");

// Delete one
await User.deleteOne({ email: "old@example.com" });

// Delete many
await User.deleteMany({ isActive: false });
```

---

## Express Route Example (Full CRUD)

```javascript
const express = require("express");
const router = express.Router();
const User = require("../models/User");

// GET all users
router.get("/users", async (req, res) => {
  try {
    const users = await User.find().select("-__v");
    res.json(users);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// GET single user
router.get("/users/:id", async (req, res) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) return res.status(404).json({ message: "User not found" });
    res.json(user);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// POST create user
router.post("/users", async (req, res) => {
  try {
    const user = await User.create(req.body);
    res.status(201).json(user);
  } catch (error) {
    res.status(400).json({ message: error.message });
  }
});

// PUT update user
router.put("/users/:id", async (req, res) => {
  try {
    const user = await User.findByIdAndUpdate(req.params.id, req.body, {
      new: true,
      runValidators: true,
    });
    if (!user) return res.status(404).json({ message: "User not found" });
    res.json(user);
  } catch (error) {
    res.status(400).json({ message: error.message });
  }
});

// DELETE user
router.delete("/users/:id", async (req, res) => {
  try {
    const user = await User.findByIdAndDelete(req.params.id);
    if (!user) return res.status(404).json({ message: "User not found" });
    res.json({ message: "User deleted" });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

module.exports = router;
```

---

## Schema Validation

### Built-in Validators

```javascript
const productSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, "Product name is required"],
    trim: true,
    minlength: [3, "Name too short"],
    maxlength: [100, "Name too long"],
  },
  price: {
    type: Number,
    required: true,
    min: [0, "Price cannot be negative"],
  },
  category: {
    type: String,
    enum: {
      values: ["electronics", "clothing", "food", "books"],
      message: "{VALUE} is not a valid category",
    },
  },
  sku: {
    type: String,
    unique: true, // Not a validator — creates a unique index
  },
});
```

### Custom Validators

```javascript
const userSchema = new mongoose.Schema({
  phone: {
    type: String,
    validate: {
      validator: function (v) {
        return /^\d{10}$/.test(v);
      },
      message: (props) => `${props.value} is not a valid 10-digit phone number`,
    },
  },
  confirmPassword: {
    type: String,
    validate: {
      validator: function (v) {
        // 'this' refers to the document (only works on create/save)
        return v === this.password;
      },
      message: "Passwords do not match",
    },
  },
});
```

### Handling Validation Errors

```javascript
try {
  await User.create({ name: "", email: "invalid" });
} catch (error) {
  if (error.name === "ValidationError") {
    const messages = Object.values(error.errors).map((e) => e.message);
    // messages: ['Name is required', 'Please enter a valid email']
  }
}
```

---

## Virtuals

Virtuals are computed properties that don't get stored in MongoDB.

```javascript
const userSchema = new mongoose.Schema({
  firstName: String,
  lastName: String,
});

// Virtual property
userSchema.virtual("fullName").get(function () {
  return `${this.firstName} ${this.lastName}`;
});

// Ensure virtuals appear in JSON output
userSchema.set("toJSON", { virtuals: true });
userSchema.set("toObject", { virtuals: true });

const User = mongoose.model("User", userSchema);

const user = new User({ firstName: "Vikas", lastName: "Kumar" });
console.log(user.fullName); // "Vikas Kumar"
// But fullName is NOT saved to the database
```

---

## Timestamps Option

```javascript
const postSchema = new mongoose.Schema(
  {
    title: String,
    content: String,
  },
  {
    timestamps: true,
    // Automatically adds:
    // createdAt: Date (set on creation)
    // updatedAt: Date (updated on every save)
  },
);
```

---

## Best Practices

1. **Always use `async/await` with try/catch** — Mongoose operations return promises
2. **Set `runValidators: true`** on updates — validators don't run on updates by default
3. **Use `{ new: true }`** with findByIdAndUpdate to get the updated document
4. **Keep schemas in separate files** (`models/` directory)
5. **Use `.select()` to exclude sensitive fields** (e.g., password) from responses
6. **Set `timestamps: true`** on every schema — always useful for debugging
7. **Handle duplicate key errors** (code 11000) separately from validation errors
8. **Use `.lean()`** when you only need plain objects (faster, no Mongoose methods)

---

## Common Mistakes

| Mistake                                     | Why It's Wrong                                        | Fix                                                |
| ------------------------------------------- | ----------------------------------------------------- | -------------------------------------------------- |
| Not awaiting Mongoose operations            | Operations return promises; results will be undefined | Always use `await` or `.then()`                    |
| Forgetting `{ new: true }` on update        | Returns the OLD document before update                | Add `{ new: true }` as options                     |
| Validators not running on update            | By design — only run on `save()` by default           | Add `{ runValidators: true }`                      |
| Using `unique` as a validator               | It creates a DB index, not a Mongoose validator       | Handle duplicate key error (code 11000) separately |
| Connecting inside route handlers            | Creates new connection per request                    | Connect once at app startup                        |
| Not handling cast errors (invalid ObjectId) | App crashes on invalid ID format                      | Wrap in try/catch, check `CastError`               |
| Circular `populate()`                       | Infinite loops, memory issues                         | Limit populate depth, use `select`                 |

---

## Summary

- **Mongoose** is an ODM that provides schema enforcement and a clean API over MongoDB
- **Connect once** at application startup using `mongoose.connect()` with your Atlas/local URI
- **Schemas** define structure, types, and validation rules; **Models** compile schemas into usable classes
- Use `Model.create()`, `Model.find()`, `Model.findByIdAndUpdate()`, `Model.findByIdAndDelete()` for CRUD
- **Validation** includes built-in (required, min, max, enum) and custom validators
- **Virtuals** are computed fields not stored in the database
- **`timestamps: true`** automatically manages `createdAt` and `updatedAt` fields
