# Advanced MongoDB — Indexes, Aggregation & Relationships

## What & Why

As your application grows, basic CRUD isn't enough. You need:

- **Indexes** — to speed up queries from seconds to milliseconds
- **Aggregation** — to transform and analyze data (like SQL GROUP BY on steroids)
- **Transactions** — to ensure multiple operations succeed or fail together
- **Relationships** — to model connected data (embedding vs referencing)
- **Pagination** — to efficiently handle large datasets

**Analogy:** If a collection is a book, an index is the table of contents. Without it, you read every page to find what you want. With it, you jump directly to the right page.

---

## Indexes

### Why Indexes Matter

Without indexes, MongoDB performs a **collection scan** — checking every document. With indexes, it uses a B-tree structure to find documents in O(log n) time.

### Creating Indexes

```javascript
// Single field index
db.users.createIndex({ email: 1 }); // 1 = ascending
db.users.createIndex({ age: -1 }); // -1 = descending

// Compound index (multiple fields)
db.users.createIndex({ city: 1, age: -1 });

// Unique index (prevents duplicates)
db.users.createIndex({ email: 1 }, { unique: true });

// Text index (for full-text search)
db.posts.createIndex({ title: "text", content: "text" });

// TTL index (auto-delete after time)
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 });
```

### Using Text Search

```javascript
// Create text index first
db.posts.createIndex({ title: "text", content: "text" });

// Search for text
db.posts.find({ $text: { $search: "mongodb tutorial" } });

// With relevance score
db.posts
  .find({ $text: { $search: "mongodb" } }, { score: { $meta: "textScore" } })
  .sort({ score: { $meta: "textScore" } });
```

### Index Management

```javascript
// View all indexes on a collection
db.users.getIndexes();

// Drop a specific index
db.users.dropIndex({ email: 1 });

// Drop all indexes (except _id)
db.users.dropIndexes();

// Explain query performance
db.users.find({ email: "vikas@test.com" }).explain("executionStats");
// Look for: "stage": "IXSCAN" (good) vs "COLLSCAN" (bad)
```

### Index in Mongoose

```javascript
// In schema definition
const userSchema = new mongoose.Schema({
  email: { type: String, unique: true, index: true },
  name: String,
  city: String,
  age: Number,
});

// Compound index
userSchema.index({ city: 1, age: -1 });

// Text index
userSchema.index({ name: "text", bio: "text" });
```

### Index Performance Trade-offs

| Aspect      | Impact                                     |
| ----------- | ------------------------------------------ |
| Read speed  | ✅ Dramatically faster queries             |
| Write speed | ❌ Slightly slower (index must be updated) |
| Storage     | ❌ Uses additional disk space              |
| Memory      | ❌ Indexes are loaded into RAM             |

> **Rule of thumb:** Index fields that appear in `find()` filters, `sort()`, and `$lookup` conditions. Don't over-index — each write updates all indexes.

---

## Aggregation Framework

The aggregation pipeline processes documents through a sequence of stages. Each stage transforms the data and passes results to the next stage.

**Analogy:** Think of it like an assembly line — raw data enters, passes through multiple transformation stations, and comes out as processed results.

### Pipeline Concept

```javascript
db.collection.aggregate([
  { $match: { ... } },     // Stage 1: Filter
  { $group: { ... } },     // Stage 2: Group
  { $sort: { ... } },      // Stage 3: Sort
  { $project: { ... } }    // Stage 4: Shape output
])
```

### $match — Filter Documents

```javascript
// Filter first (reduces documents for next stages)
db.orders.aggregate([
  { $match: { status: "completed", amount: { $gt: 100 } } },
]);
```

### $group — Group & Aggregate

```javascript
// Total sales per city
db.orders.aggregate([
  {
    $group: {
      _id: "$city", // Group by city
      totalSales: { $sum: "$amount" }, // Sum amounts
      avgOrder: { $avg: "$amount" }, // Average
      count: { $sum: 1 }, // Count documents
      maxOrder: { $max: "$amount" }, // Maximum value
    },
  },
]);

// Output: [{ _id: "Delhi", totalSales: 50000, avgOrder: 500, count: 100 }, ...]
```

### $sort — Sort Results

```javascript
db.orders.aggregate([
  { $group: { _id: "$city", total: { $sum: "$amount" } } },
  { $sort: { total: -1 } }, // Sort by total descending
]);
```

### $project — Reshape Documents

```javascript
db.users.aggregate([
  {
    $project: {
      _id: 0,
      fullName: { $concat: ["$firstName", " ", "$lastName"] },
      ageInMonths: { $multiply: ["$age", 12] },
      email: 1, // Include email as-is
    },
  },
]);
```

### $lookup — Join Collections (LEFT OUTER JOIN)

```javascript
// Join orders with user details
db.orders.aggregate([
  {
    $lookup: {
      from: "users", // Collection to join
      localField: "userId", // Field in orders
      foreignField: "_id", // Field in users
      as: "userDetails", // Output array field
    },
  },
  { $unwind: "$userDetails" }, // Flatten the array
]);
```

### $unwind — Deconstruct Arrays

```javascript
// Each array element becomes its own document
db.users.aggregate([
  { $unwind: "$skills" },
  { $group: { _id: "$skills", count: { $sum: 1 } } },
  { $sort: { count: -1 } },
]);
// Shows most popular skills across all users
```

### Full Pipeline Example

```javascript
// Top 5 cities by average order value (completed orders only)
db.orders.aggregate([
  { $match: { status: "completed" } },
  {
    $group: {
      _id: "$city",
      avgOrderValue: { $avg: "$amount" },
      totalOrders: { $sum: 1 },
    },
  },
  { $match: { totalOrders: { $gte: 10 } } }, // At least 10 orders
  { $sort: { avgOrderValue: -1 } },
  { $limit: 5 },
  {
    $project: {
      city: "$_id",
      avgOrderValue: { $round: ["$avgOrderValue", 2] },
      totalOrders: 1,
      _id: 0,
    },
  },
]);
```

### Aggregation in Mongoose

```javascript
const stats = await Order.aggregate([
  { $match: { status: "completed" } },
  { $group: { _id: "$category", total: { $sum: "$amount" } } },
  { $sort: { total: -1 } },
]);
```

---

## Transactions (Multi-Document ACID)

MongoDB supports multi-document ACID transactions (since v4.0). Use them when multiple operations must all succeed or all fail.

```javascript
const session = await mongoose.startSession();
session.startTransaction();

try {
  // Transfer money: debit from sender, credit to receiver
  await Account.updateOne(
    { _id: senderId },
    { $inc: { balance: -amount } },
    { session },
  );

  await Account.updateOne(
    { _id: receiverId },
    { $inc: { balance: amount } },
    { session },
  );

  // Create transaction record
  await Transaction.create(
    [
      {
        from: senderId,
        to: receiverId,
        amount,
        date: new Date(),
      },
    ],
    { session },
  );

  await session.commitTransaction();
  console.log("Transaction successful");
} catch (error) {
  await session.abortTransaction();
  console.error("Transaction failed:", error);
} finally {
  session.endSession();
}
```

> **Note:** Transactions require a replica set (Atlas always has one). They add overhead — use them only when data consistency across documents is critical.

---

## Relationships — Embedding vs Referencing

### Embedding (Denormalization)

Store related data inside the parent document.

```javascript
// Blog post with embedded comments
const postSchema = new mongoose.Schema({
  title: String,
  content: String,
  comments: [
    {
      user: String,
      text: String,
      date: { type: Date, default: Date.now },
    },
  ],
});
```

### Referencing (Normalization)

Store related data in separate collections, linked by ObjectId.

```javascript
// Comment references a Post
const commentSchema = new mongoose.Schema({
  text: String,
  post: { type: mongoose.Schema.Types.ObjectId, ref: "Post" },
  author: { type: mongoose.Schema.Types.ObjectId, ref: "User" },
});

// Post references comments
const postSchema = new mongoose.Schema({
  title: String,
  content: String,
  author: { type: mongoose.Schema.Types.ObjectId, ref: "User" },
  comments: [{ type: mongoose.Schema.Types.ObjectId, ref: "Comment" }],
});
```

### populate() — Joining References

```javascript
// Get post with author and comments populated
const post = await Post.findById(postId)
  .populate("author", "name email") // populate author, select name & email
  .populate({
    path: "comments",
    select: "text date",
    populate: { path: "author", select: "name" }, // nested populate
  });
```

### When to Embed vs Reference

| Factor                       | Embed                 | Reference                     |
| ---------------------------- | --------------------- | ----------------------------- |
| Data accessed together?      | ✅ Yes — embed        | ❌ No — reference             |
| Array can grow unbounded?    | ❌ Don't embed        | ✅ Reference                  |
| Data shared across entities? | ❌ Don't embed        | ✅ Reference                  |
| Need atomic updates?         | ✅ Embed (single doc) | ❌ Need transactions          |
| Document size > 16MB risk?   | ❌ Don't embed        | ✅ Reference                  |
| Read performance priority?   | ✅ Embed (one read)   | ❌ Reference (multiple reads) |

**Rules of thumb:**

- Embed: 1-to-few relationships (user → addresses)
- Reference: 1-to-many or many-to-many (user → orders, student ↔ courses)
- Embed: data that doesn't change independently
- Reference: data that's updated from multiple places

---

## Pagination

### Skip/Limit Pagination (Simple but Slow for Large Offsets)

```javascript
// Express route with pagination
router.get("/posts", async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  const skip = (page - 1) * limit;

  const posts = await Post.find()
    .sort({ createdAt: -1 })
    .skip(skip)
    .limit(limit);

  const total = await Post.countDocuments();

  res.json({
    data: posts,
    currentPage: page,
    totalPages: Math.ceil(total / limit),
    totalItems: total,
  });
});
```

### Cursor-Based Pagination (Better for Large Datasets)

```javascript
// Uses the last document's _id or timestamp as cursor
router.get("/posts", async (req, res) => {
  const limit = parseInt(req.query.limit) || 10;
  const cursor = req.query.cursor; // last document's _id or date

  const query = cursor ? { createdAt: { $lt: new Date(cursor) } } : {};

  const posts = await Post.find(query)
    .sort({ createdAt: -1 })
    .limit(limit + 1); // Fetch one extra to check if there's a next page

  const hasMore = posts.length > limit;
  const results = hasMore ? posts.slice(0, limit) : posts;
  const nextCursor = hasMore ? results[results.length - 1].createdAt : null;

  res.json({
    data: results,
    nextCursor,
    hasMore,
  });
});
```

| Method       | Pros                                                 | Cons                                         |
| ------------ | ---------------------------------------------------- | -------------------------------------------- |
| skip/limit   | Simple, supports random page access                  | Slow at high offsets (skip scans)            |
| cursor-based | Fast at any position, consistent with real-time data | No random page access, slightly more complex |

---

## Best Practices

1. **Place `$match` early in pipelines** — reduces documents processed by later stages
2. **Index fields used in `$match` and `$sort`** within aggregation pipelines
3. **Avoid skip/limit for large datasets** — cursor-based pagination scales better
4. **Use `explain()` to analyze queries** — look for IXSCAN vs COLLSCAN
5. **Don't over-index** — each index slows writes and uses memory
6. **Embed data that's read together** — avoid unnecessary `$lookup` operations
7. **Use transactions sparingly** — they add latency; design schema to minimize need
8. **Limit `populate()` depth** — deep nesting causes N+1 query problems

---

## Common Mistakes

| Mistake                                     | Why It's Wrong                                          | Fix                                              |
| ------------------------------------------- | ------------------------------------------------------- | ------------------------------------------------ |
| No index on frequently queried fields       | Full collection scans on every query                    | Run `explain()`, add indexes for COLLSCAN fields |
| Putting `$sort` before `$match` in pipeline | Sorts all documents before filtering                    | Always `$match` first to reduce dataset          |
| Embedding unbounded arrays                  | Document hits 16MB limit, performance degrades          | Reference with separate collection               |
| Using transactions for everything           | Unnecessary overhead, defeats MongoDB's design          | Only use when multiple docs MUST be atomic       |
| `populate()` without `select()`             | Fetches entire referenced documents                     | Always `.populate('field', 'needed fields')`     |
| skip(10000) on large collections            | MongoDB must scan and discard 10,000 docs               | Use cursor-based pagination                      |
| Creating indexes on low-cardinality fields  | Index on `{ gender: 1 }` barely helps (only 2-3 values) | Index high-cardinality fields (email, userId)    |
| Not using compound indexes correctly        | Query on `{ city, age }` doesn't use index `{ age: 1 }` | Match index prefix: `{ city: 1, age: 1 }`        |

---

## Summary

- **Indexes** dramatically improve read performance but cost write speed and storage — use `explain()` to verify
- **Aggregation pipelines** are stages that transform data: `$match` → `$group` → `$sort` → `$project` → `$lookup`
- **`$lookup`** performs left outer joins between collections; `$unwind` flattens arrays
- **Transactions** provide ACID guarantees across multiple documents — use only when needed
- **Embed** for 1-to-few, read-together data; **Reference** for 1-to-many, shared, or unbounded data
- **Cursor-based pagination** outperforms skip/limit at scale — use the last document's sort field as the cursor
