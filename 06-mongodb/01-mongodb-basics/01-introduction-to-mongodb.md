# Introduction to MongoDB

## What Is MongoDB?

MongoDB is a **NoSQL document-oriented database** that stores data in flexible, JSON-like documents (called BSON). Unlike relational databases (MySQL, PostgreSQL) that use rows and columns in tables, MongoDB stores data as documents in collections.

**Analogy:** A relational database is like a spreadsheet — fixed columns, every row must have the same structure. MongoDB is like a filing cabinet with folders — each document in a folder can have different fields, and you can nest documents inside documents.

---

## Why MongoDB?

| Feature              | Benefit                                            |
| -------------------- | -------------------------------------------------- |
| Flexible schema      | No migrations — add/remove fields anytime          |
| Document model       | Natural match for how objects exist in code        |
| Horizontal scaling   | Sharding distributes data across servers           |
| Rich queries         | Aggregation pipeline, full-text search, geospatial |
| Developer experience | JSON in, JSON out — natural for JavaScript devs    |
| High performance     | In-memory working set, indexing, read replicas     |

---

## SQL vs MongoDB Terminology

| SQL            | MongoDB                |
| -------------- | ---------------------- |
| Database       | Database               |
| Table          | Collection             |
| Row            | Document               |
| Column         | Field                  |
| Primary Key    | `_id` (auto-generated) |
| JOIN           | Embedding or `$lookup` |
| Schema (fixed) | Schema (flexible)      |

---

## BSON vs JSON

MongoDB stores documents in **BSON** (Binary JSON):

```javascript
// JSON (what you write)
{
  "name": "Vikas",
  "age": 25,
  "joinedAt": "2024-01-15T10:30:00Z"
}

// BSON (what MongoDB stores internally)
// - Adds types: Date, ObjectId, Int32, Int64, Decimal128, Binary
// - More efficient for storage and querying
// - Supports types JSON cannot (like Date objects, binary data)
```

BSON adds types that JSON lacks:

- `ObjectId` — unique 12-byte identifiers
- `Date` — stored as milliseconds since epoch
- `Decimal128` — precise decimal numbers (financial data)
- `Binary` — for files, images, binary data

---

## Setup Options

### MongoDB Atlas (Cloud — Recommended for Learning)

1. Go to [https://cloud.mongodb.com](https://cloud.mongodb.com)
2. Create a free cluster (M0 tier — 512MB free).
3. Create a database user.
4. Whitelist your IP (or allow from anywhere: `0.0.0.0/0` for dev).
5. Get connection string: `mongodb+srv://user:pass@cluster.mongodb.net/dbname`

### Local Installation

```bash
# After installing MongoDB Community Server
mongosh    # Opens Mongo Shell
```

### MongoDB Compass (GUI)

Desktop application for visual database exploration. Connect with same URI.

---

## Mongo Shell Basics (`mongosh`)

```javascript
// Show all databases
show dbs

// Switch/create database
use myapp

// Show collections in current database
show collections

// Drop database
db.dropDatabase()
```

---

## CRUD Operations

### Create (Insert)

```javascript
// Insert one document
db.users.insertOne({
  name: "Vikas",
  email: "vikas@example.com",
  age: 25,
  skills: ["JavaScript", "Node.js", "React"],
  address: {
    city: "Delhi",
    country: "India",
  },
  createdAt: new Date(),
});

// Insert many documents
db.users.insertMany([
  { name: "Rahul", email: "rahul@example.com", age: 30 },
  { name: "Priya", email: "priya@example.com", age: 28 },
]);
```

Every document automatically gets a unique `_id` field (ObjectId).

### Read (Find)

```javascript
// Find all documents
db.users.find();

// Find with filter
db.users.find({ age: 25 });

// Find one
db.users.findOne({ email: "vikas@example.com" });

// Query operators
db.users.find({ age: { $gt: 25 } }); // Greater than
db.users.find({ age: { $gte: 18, $lte: 30 } }); // Range
db.users.find({ name: { $in: ["Vikas", "Rahul"] } }); // In array
db.users.find({ skills: "React" }); // Array contains
db.users.find({ "address.city": "Delhi" }); // Nested field

// Projection (select specific fields)
db.users.find({}, { name: 1, email: 1, _id: 0 });

// Sorting
db.users.find().sort({ age: -1 }); // Descending

// Limit and skip (pagination)
db.users.find().skip(10).limit(10); // Page 2, 10 per page

// Count
db.users.countDocuments({ age: { $gt: 25 } });
```

### Common Query Operators

| Operator  | Meaning               | Example                                        |
| --------- | --------------------- | ---------------------------------------------- |
| `$eq`     | Equal                 | `{ age: { $eq: 25 } }`                         |
| `$ne`     | Not equal             | `{ status: { $ne: "deleted" } }`               |
| `$gt`     | Greater than          | `{ age: { $gt: 18 } }`                         |
| `$gte`    | Greater than or equal | `{ age: { $gte: 18 } }`                        |
| `$lt`     | Less than             | `{ price: { $lt: 100 } }`                      |
| `$lte`    | Less than or equal    | `{ price: { $lte: 100 } }`                     |
| `$in`     | In array of values    | `{ role: { $in: ["admin", "mod"] } }`          |
| `$nin`    | Not in array          | `{ status: { $nin: ["deleted"] } }`            |
| `$and`    | Logical AND           | `{ $and: [{age: {$gt:18}}, {age: {$lt:30}}] }` |
| `$or`     | Logical OR            | `{ $or: [{city: "Delhi"}, {city: "Mumbai"}] }` |
| `$exists` | Field exists          | `{ email: { $exists: true } }`                 |
| `$regex`  | Pattern match         | `{ name: { $regex: /^V/i } }`                  |

### Update

```javascript
// Update one document
db.users.updateOne(
  { _id: ObjectId("...") }, // Filter
  { $set: { age: 26, role: "admin" } }, // Update
);

// Update many
db.users.updateMany({ age: { $lt: 18 } }, { $set: { status: "minor" } });

// Update operators
db.users.updateOne(
  { name: "Vikas" },
  {
    $set: { email: "new@email.com" }, // Set field value
    $inc: { loginCount: 1 }, // Increment
    $push: { skills: "MongoDB" }, // Add to array
    $pull: { skills: "jQuery" }, // Remove from array
    $unset: { tempField: "" }, // Remove field
    $rename: { oldName: "newName" }, // Rename field
  },
);

// Replace entire document (except _id)
db.users.replaceOne(
  { _id: ObjectId("...") },
  { name: "Vikas", email: "vikas@new.com", age: 26 },
);
```

### Delete

```javascript
// Delete one
db.users.deleteOne({ _id: ObjectId("...") });

// Delete many
db.users.deleteMany({ status: "inactive" });

// Delete all documents in collection
db.users.deleteMany({});
```

---

## Document Structure Best Practices

### Embedding (Denormalization)

Store related data within the same document:

```javascript
// User with embedded address — good for 1:1 or 1:few relationships
{
  name: "Vikas",
  address: {
    street: "123 Main St",
    city: "Delhi",
    zip: "110001"
  },
  orders: [
    { id: 1, product: "Laptop", amount: 999 },
    { id: 2, product: "Phone", amount: 699 }
  ]
}
```

**When to embed:** Data is always accessed together, 1:1 or 1:few relationships, data does not change independently.

### Referencing (Normalization)

Store a reference (ObjectId) to another collection:

```javascript
// User document
{ _id: ObjectId("user1"), name: "Vikas" }

// Order documents (separate collection)
{ userId: ObjectId("user1"), product: "Laptop", amount: 999 }
{ userId: ObjectId("user1"), product: "Phone", amount: 699 }
```

**When to reference:** 1:many or many:many relationships, data is large or changes frequently, data is accessed independently.

---

## Best Practices

1. **Design schema around query patterns** — not around data relationships.
2. **Embed for read performance** — fewer queries, atomic operations.
3. **Reference for write independence** — when subdocuments change frequently.
4. **Always create indexes** for fields you query on frequently.
5. **Use meaningful field names** — they are stored in every document (disk cost).
6. **Set `_id` explicitly** if you have a natural unique key.

---

## Summary

- MongoDB is a document-oriented NoSQL database — data is stored as flexible JSON-like documents.
- Documents live in collections (like tables), and can have different shapes (flexible schema).
- CRUD: `insertOne`/`insertMany`, `find`/`findOne`, `updateOne`/`updateMany`, `deleteOne`/`deleteMany`.
- Query operators (`$gt`, `$in`, `$regex`, etc.) enable powerful filtering.
- Choose embedding for tightly coupled data, referencing for loosely coupled data.
- MongoDB Atlas provides a free cloud-hosted tier perfect for development and learning.
