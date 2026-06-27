# Mongoose ODM

## What Is Mongoose?

Mongoose is an ODM (Object Data Modeling) library for MongoDB and Node.js. It provides schema definitions, validation, middleware, and a clean API for interacting with MongoDB from your Express application.

**Analogy:** If MongoDB is a flexible filing cabinet (no rules about what goes in), Mongoose is adding labeled folder tabs, required fields stamps, and a filing clerk who rejects improperly filled documents.

---

## Why Mongoose?

- **Schema enforcement** — define the shape of documents despite MongoDB being schema-less.
- **Validation** — built-in and custom validators.
- **Middleware (hooks)** — run logic before/after save, find, update, delete.
- **Type casting** — automatically converts `"25"` to `25` for number fields.
- **Population** — resolve references between documents (like JOINs).
- **Plugins** — extend functionality (timestamps, pagination, etc.).

---

## Setup

```bash
npm install mongoose
```

### Connecting to MongoDB

```javascript
// config/db.js
const mongoose = require("mongoose");

async function connectDB() {
  try {
    await mongoose.connect(process.env.MONGO_URI, {
      // Options are mostly automatic in Mongoose 7+
    });
    console.log("MongoDB connected");
  } catch (error) {
    console.error("MongoDB connection error:", error.message);
    process.exit(1);
  }
}

module.exports = connectDB;
```

```javascript
// server.js
require("dotenv").config();
const app = require("./app");
const connectDB = require("./config/db");

connectDB().then(() => {
  app.listen(3000, () => console.log("Server running on port 3000"));
});
```

---

## Schema & Model

### Defining a Schema

```javascript
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: [true, "Name is required"],
      trim: true,
      minlength: [2, "Name must be at least 2 characters"],
    },
    email: {
      type: String,
      required: [true, "Email is required"],
      unique: true,
      lowercase: true,
      match: [/^\S+@\S+\.\S+$/, "Please provide a valid email"],
    },
    age: {
      type: Number,
      min: [18, "Must be at least 18"],
      max: 120,
    },
    role: {
      type: String,
      enum: ["user", "admin", "moderator"],
      default: "user",
    },
    skills: [String], // Array of strings
    address: {
      city: String,
      country: { type: String, default: "India" },
    },
    isActive: {
      type: Boolean,
      default: true,
    },
  },
  {
    timestamps: true, // Adds createdAt and updatedAt automatically
  },
);

// Create model from schema
const User = mongoose.model("User", userSchema);
// Collection name in MongoDB will be "users" (lowercase plural)

module.exports = User;
```

### Schema Types

| Type       | Example                                  | Notes                          |
| ---------- | ---------------------------------------- | ------------------------------ |
| `String`   | `name: String`                           | Supports trim, lowercase, etc. |
| `Number`   | `age: Number`                            | Supports min, max              |
| `Boolean`  | `isActive: Boolean`                      | -                              |
| `Date`     | `createdAt: Date`                        | Supports min, max              |
| `ObjectId` | `author: mongoose.Schema.Types.ObjectId` | References another document    |
| `Array`    | `tags: [String]`                         | Array of any type              |
| `Mixed`    | `metadata: mongoose.Schema.Types.Mixed`  | Any structure                  |
| `Buffer`   | `photo: Buffer`                          | Binary data                    |

---

## CRUD with Mongoose

### Create

```javascript
// Method 1: new + save
const user = new User({ name: "Vikas", email: "vikas@example.com", age: 25 });
await user.save();

// Method 2: create (shorthand)
const user = await User.create({
  name: "Rahul",
  email: "rahul@example.com",
  age: 30,
});

// Create many
await User.insertMany([
  { name: "User1", email: "u1@example.com" },
  { name: "User2", email: "u2@example.com" },
]);
```

### Read

```javascript
// Find all
const users = await User.find();

// Find with filter
const admins = await User.find({ role: "admin" });

// Find one
const user = await User.findOne({ email: "vikas@example.com" });

// Find by ID
const user = await User.findById("65a1b2c3d4e5f6a7b8c9d0e1");

// Select specific fields
const users = await User.find().select("name email -_id");

// Sort, limit, skip
const users = await User.find()
  .sort({ createdAt: -1 }) // Newest first
  .limit(10)
  .skip(20); // Page 3

// Count
const count = await User.countDocuments({ role: "admin" });
```

### Update

```javascript
// Find and update (returns updated document with { new: true })
const user = await User.findByIdAndUpdate(
  id,
  { $set: { name: "New Name", age: 26 } },
  { new: true, runValidators: true },
);

// Update one
await User.updateOne(
  { email: "vikas@example.com" },
  { $set: { isActive: false } },
);

// Update many
await User.updateMany(
  { lastLogin: { $lt: thirtyDaysAgo } },
  { $set: { isActive: false } },
);
```

### Delete

```javascript
// Find and delete (returns the deleted document)
const deleted = await User.findByIdAndDelete(id);

// Delete one
await User.deleteOne({ email: "old@example.com" });

// Delete many
await User.deleteMany({ isActive: false });
```

---

## Schema Validation

```javascript
const productSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, "Product name is required"],
    trim: true,
    maxlength: [100, "Name cannot exceed 100 characters"],
  },
  price: {
    type: Number,
    required: true,
    min: [0, "Price cannot be negative"],
    validate: {
      validator: function (value) {
        return value > 0;
      },
      message: "Price must be greater than 0",
    },
  },
  category: {
    type: String,
    enum: {
      values: ["electronics", "clothing", "books", "food"],
      message: "{VALUE} is not a valid category",
    },
  },
  sku: {
    type: String,
    unique: true,
    validate: {
      validator: function (v) {
        return /^[A-Z]{3}-\d{4}$/.test(v);
      },
      message: "SKU must match format ABC-1234",
    },
  },
});
```

---

## Mongoose Middleware (Hooks)

```javascript
// Pre-save hook (runs before document is saved)
userSchema.pre("save", async function (next) {
  if (this.isModified("password")) {
    this.password = await bcrypt.hash(this.password, 12);
  }
  next();
});

// Post-save hook (runs after document is saved)
userSchema.post("save", function (doc) {
  console.log(`User ${doc.name} was saved`);
});

// Pre-find hook
userSchema.pre(/^find/, function (next) {
  this.find({ isActive: { $ne: false } }); // Exclude inactive by default
  next();
});

// Pre-remove hook
userSchema.pre("deleteOne", { document: true }, async function (next) {
  await Order.deleteMany({ user: this._id }); // Cascade delete
  next();
});
```

---

## Instance Methods & Statics

```javascript
// Instance method (on a document)
userSchema.methods.comparePassword = async function (candidatePassword) {
  return bcrypt.compare(candidatePassword, this.password);
};

const user = await User.findOne({ email });
const isMatch = await user.comparePassword("password123");

// Static method (on the model)
userSchema.statics.findByEmail = function (email) {
  return this.findOne({ email: email.toLowerCase() });
};

const user = await User.findByEmail("VIKAS@EXAMPLE.COM");

// Virtual property (not stored in DB)
userSchema.virtual("fullName").get(function () {
  return `${this.firstName} ${this.lastName}`;
});
```

---

## Population (References)

```javascript
const postSchema = new mongoose.Schema({
  title: String,
  content: String,
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: "User", // References the User model
    required: true,
  },
});

const Post = mongoose.model("Post", postSchema);

// Create with reference
const post = await Post.create({
  title: "Hello World",
  content: "My first post",
  author: userId, // ObjectId of a user
});

// Populate — resolve the reference
const post = await Post.findById(postId).populate("author", "name email");
// post.author is now { _id: ..., name: "Vikas", email: "..." }
// Without populate, post.author would just be an ObjectId
```

---

## Express + Mongoose Pattern

```javascript
// controllers/userController.js
const User = require("../models/User");

exports.getAll = async (req, res, next) => {
  try {
    const { page = 1, limit = 10, role } = req.query;
    const filter = role ? { role } : {};

    const users = await User.find(filter)
      .select("-password")
      .sort({ createdAt: -1 })
      .limit(Number(limit))
      .skip((Number(page) - 1) * Number(limit));

    const total = await User.countDocuments(filter);

    res.json({
      data: users,
      pagination: { total, page: Number(page), limit: Number(limit) },
    });
  } catch (error) {
    next(error);
  }
};

exports.create = async (req, res, next) => {
  try {
    const user = await User.create(req.body);
    res.status(201).json({ data: user });
  } catch (error) {
    if (error.code === 11000) {
      return res.status(409).json({ error: "Email already exists" });
    }
    if (error.name === "ValidationError") {
      const messages = Object.values(error.errors).map((e) => e.message);
      return res
        .status(400)
        .json({ error: "Validation failed", details: messages });
    }
    next(error);
  }
};
```

---

## Best Practices

1. **Always use `{ new: true, runValidators: true }`** with `findByIdAndUpdate`.
2. **Index fields** you query frequently — especially `email`, foreign keys, and sort fields.
3. **Use `select("-password")`** to exclude sensitive fields from query results.
4. **Handle duplicate key errors** (`error.code === 11000`) gracefully.
5. **Use `timestamps: true`** in schema options — automatic `createdAt`/`updatedAt`.
6. **Keep schemas in separate files** under a `models/` directory.

---

## Summary

- Mongoose adds schema enforcement, validation, and middleware to MongoDB's flexible documents.
- Define schemas with types, required fields, validators, and defaults; create models from schemas.
- CRUD: `create`/`save`, `find`/`findById`, `findByIdAndUpdate`, `findByIdAndDelete`.
- Middleware (hooks) run logic before/after database operations (hashing passwords, cascading deletes).
- Population resolves ObjectId references — the closest MongoDB gets to SQL JOINs.
- Combine with Express in a controller pattern for clean, maintainable APIs.
