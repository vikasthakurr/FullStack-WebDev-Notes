# MongoDB CRUD Operations

## What & Why

CRUD stands for **Create, Read, Update, Delete** — the four fundamental operations for any database. MongoDB provides intuitive methods for each, with a powerful query language that supports filtering, projection, and data manipulation operators.

**Analogy:** CRUD operations are like managing a filing cabinet — you can add new files (Create), look through them (Read), modify existing files (Update), or throw them away (Delete).

---

## Setup: Sample Collection

All examples assume a `users` collection:

```javascript
// Sample documents we'll work with
db.users.insertMany([
  {
    name: "Vikas",
    age: 25,
    city: "Delhi",
    skills: ["JS", "React"],
    active: true,
  },
  {
    name: "Priya",
    age: 28,
    city: "Mumbai",
    skills: ["Python", "Django"],
    active: true,
  },
  {
    name: "Rahul",
    age: 22,
    city: "Delhi",
    skills: ["JS", "Node"],
    active: false,
  },
  {
    name: "Anita",
    age: 30,
    city: "Bangalore",
    skills: ["Java", "Spring"],
    active: true,
  },
  {
    name: "Karan",
    age: 26,
    city: "Mumbai",
    skills: ["JS", "Vue"],
    active: false,
  },
]);
```

---

## CREATE — Inserting Documents

### insertOne()

```javascript
// Insert a single document
db.users.insertOne({
  name: "Amit",
  age: 27,
  city: "Pune",
  skills: ["Go", "Docker"],
  active: true,
});

// Returns: { acknowledged: true, insertedId: ObjectId("...") }
```

### insertMany()

```javascript
// Insert multiple documents at once
db.users.insertMany([
  { name: "Sneha", age: 24, city: "Chennai", skills: ["Ruby"], active: true },
  { name: "Dev", age: 29, city: "Delhi", skills: ["Rust"], active: false },
]);

// Returns: { acknowledged: true, insertedIds: { '0': ObjectId("..."), '1': ObjectId("...") } }
```

> **Note:** If `_id` is not provided, MongoDB auto-generates an ObjectId.

---

## READ — Querying Documents

### find() — Get Multiple Documents

```javascript
// Find ALL documents
db.users.find();

// Find with a filter (all users in Delhi)
db.users.find({ city: "Delhi" });

// Find active users in Mumbai
db.users.find({ city: "Mumbai", active: true });
```

### findOne() — Get a Single Document

```javascript
// Returns the FIRST matching document (or null)
db.users.findOne({ name: "Vikas" });

// Find by _id
db.users.findOne({ _id: ObjectId("64a7f2b3c9e1a2d4e8f01234") });
```

### Projection — Select Specific Fields

```javascript
// Only return name and city (exclude _id)
db.users.find({}, { name: 1, city: 1, _id: 0 });

// Exclude specific fields
db.users.find({}, { skills: 0, active: 0 });
```

> **Rule:** You cannot mix inclusion (1) and exclusion (0) in the same projection, except for `_id`.

---

## Comparison Operators

| Operator | Meaning               | Example                                  |
| -------- | --------------------- | ---------------------------------------- |
| `$eq`    | Equal to              | `{ age: { $eq: 25 } }`                   |
| `$ne`    | Not equal             | `{ city: { $ne: "Delhi" } }`             |
| `$gt`    | Greater than          | `{ age: { $gt: 25 } }`                   |
| `$gte`   | Greater than or equal | `{ age: { $gte: 25 } }`                  |
| `$lt`    | Less than             | `{ age: { $lt: 30 } }`                   |
| `$lte`   | Less than or equal    | `{ age: { $lte: 28 } }`                  |
| `$in`    | Matches any in array  | `{ city: { $in: ["Delhi", "Mumbai"] } }` |
| `$nin`   | Not in array          | `{ city: { $nin: ["Pune"] } }`           |

```javascript
// Users older than 25
db.users.find({ age: { $gt: 25 } });

// Users aged between 22 and 28 (inclusive)
db.users.find({ age: { $gte: 22, $lte: 28 } });

// Users in Delhi or Mumbai
db.users.find({ city: { $in: ["Delhi", "Mumbai"] } });
```

---

## Logical Operators

| Operator | Meaning                           | Usage                        |
| -------- | --------------------------------- | ---------------------------- |
| `$and`   | All conditions must match         | `{ $and: [{...}, {...}] }`   |
| `$or`    | At least one condition must match | `{ $or: [{...}, {...}] }`    |
| `$not`   | Negates the condition             | `{ field: { $not: {...} } }` |
| `$nor`   | None of the conditions match      | `{ $nor: [{...}, {...}] }`   |

```javascript
// Users in Delhi AND age > 23
db.users.find({ $and: [{ city: "Delhi" }, { age: { $gt: 23 } }] });

// Shorthand (implicit AND — same field object)
db.users.find({ city: "Delhi", age: { $gt: 23 } });

// Users in Delhi OR age > 28
db.users.find({ $or: [{ city: "Delhi" }, { age: { $gt: 28 } }] });

// Users whose age is NOT greater than 25
db.users.find({ age: { $not: { $gt: 25 } } });
```

---

## UPDATE — Modifying Documents

### updateOne()

```javascript
// Update the first matching document
db.users.updateOne(
  { name: "Vikas" }, // filter
  { $set: { age: 26 } }, // update
);
```

### updateMany()

```javascript
// Update ALL matching documents
db.users.updateMany({ city: "Delhi" }, { $set: { active: true } });
```

### Update Operators

| Operator    | Purpose                                   | Example                               |
| ----------- | ----------------------------------------- | ------------------------------------- |
| `$set`      | Set field value (create if doesn't exist) | `{ $set: { age: 30 } }`               |
| `$unset`    | Remove a field                            | `{ $unset: { active: "" } }`          |
| `$inc`      | Increment by value                        | `{ $inc: { age: 1 } }`                |
| `$push`     | Add element to array                      | `{ $push: { skills: "MongoDB" } }`    |
| `$pull`     | Remove element from array                 | `{ $pull: { skills: "React" } }`      |
| `$addToSet` | Add to array (no duplicates)              | `{ $addToSet: { skills: "JS" } }`     |
| `$rename`   | Rename a field                            | `{ $rename: { "city": "location" } }` |

```javascript
// Increment age by 1
db.users.updateOne({ name: "Vikas" }, { $inc: { age: 1 } });

// Add a skill to the array
db.users.updateOne({ name: "Vikas" }, { $push: { skills: "MongoDB" } });

// Remove a skill from the array
db.users.updateOne({ name: "Vikas" }, { $pull: { skills: "React" } });

// Add multiple skills at once
db.users.updateOne(
  { name: "Vikas" },
  { $push: { skills: { $each: ["TypeScript", "Next.js"] } } },
);

// Remove a field entirely
db.users.updateOne({ name: "Vikas" }, { $unset: { active: "" } });
```

### Upsert (Update or Insert)

```javascript
// If no document matches, INSERT a new one
db.users.updateOne(
  { name: "NewUser" },
  { $set: { age: 20, city: "Jaipur" } },
  { upsert: true },
);
```

---

## DELETE — Removing Documents

### deleteOne()

```javascript
// Delete the first matching document
db.users.deleteOne({ name: "Karan" });

// Returns: { acknowledged: true, deletedCount: 1 }
```

### deleteMany()

```javascript
// Delete ALL matching documents
db.users.deleteMany({ active: false });

// Delete ALL documents in collection (dangerous!)
db.users.deleteMany({});
```

---

## Sorting, Limiting, and Skipping

### sort()

```javascript
// Sort by age ascending (1)
db.users.find().sort({ age: 1 });

// Sort by age descending (-1)
db.users.find().sort({ age: -1 });

// Sort by city (A-Z), then by age (descending)
db.users.find().sort({ city: 1, age: -1 });
```

### limit()

```javascript
// Get only the first 3 documents
db.users.find().limit(3);

// Top 2 oldest users
db.users.find().sort({ age: -1 }).limit(2);
```

### skip() — Pagination

```javascript
// Skip the first 2 documents
db.users.find().skip(2);

// Pagination: page 2, 3 items per page
const page = 2;
const pageSize = 3;
db.users
  .find()
  .skip((page - 1) * pageSize)
  .limit(pageSize);
```

### Chaining Order

```javascript
// MongoDB processes in this order regardless of chain order:
// 1. filter → 2. sort → 3. skip → 4. limit
db.users.find({ active: true }).sort({ age: 1 }).skip(1).limit(2);
```

---

## Counting Documents

```javascript
// Count all documents
db.users.countDocuments();

// Count with a filter
db.users.countDocuments({ city: "Delhi" });

// Estimated count (faster for large collections, no filter)
db.users.estimatedDocumentCount();
```

---

## Best Practices

1. **Use `findOne()` when you expect a single result** — avoids returning a cursor
2. **Always use update operators** (`$set`, `$inc`) — never replace entire documents accidentally
3. **Be specific with filters** — `deleteMany({})` with an empty filter deletes EVERYTHING
4. **Use projection** to return only the fields you need — reduces network payload
5. **Index fields you frequently query or sort by** — improves read performance
6. **Use `upsert` cautiously** — can create unexpected documents if filters are too broad
7. **Prefer `$addToSet` over `$push`** when you need unique array values

---

## Common Mistakes

| Mistake                                      | Why It's Wrong                                               | Fix                                         |
| -------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------- |
| `db.users.update({}, { name: "X" })`         | Replaces the entire document (removes all other fields)      | Use `$set`: `{ $set: { name: "X" } }`       |
| Mixing inclusion/exclusion in projection     | MongoDB throws an error                                      | Use only `1`s or only `0`s (except `_id`)   |
| `deleteMany({})` without confirmation        | Deletes all documents in collection                          | Always double-check your filter             |
| Not using `$each` with `$push` for arrays    | `$push` adds the array itself as one element                 | `{ $push: { skills: { $each: [...] } } }`   |
| Forgetting `skip/limit` order doesn't matter | Developers chain them thinking order matters                 | MongoDB always applies: sort → skip → limit |
| Using `$inc` with a string                   | Throws a type error                                          | Ensure the field is a number                |
| Not indexing sorted fields                   | Large collection sorts without index crash (32MB sort limit) | Create index on sorted fields               |

---

## Summary

- **Create:** `insertOne()` for single docs, `insertMany()` for bulk inserts
- **Read:** `find()` returns a cursor, `findOne()` returns a single document; use projection to limit fields
- **Update:** Always use operators (`$set`, `$inc`, `$push`, `$pull`); `updateOne()` for single, `updateMany()` for bulk
- **Delete:** `deleteOne()` removes first match, `deleteMany()` removes all matches — empty filter `{}` means ALL
- **Comparison operators** (`$gt`, `$lt`, `$in`, etc.) go inside the field value object
- **Logical operators** (`$and`, `$or`) wrap arrays of conditions
- **sort → skip → limit** is the execution order for pagination
