# Introduction to NoSQL & MongoDB

## What is NoSQL?

NoSQL (Not Only SQL) databases are non-relational data stores designed for flexible schemas, horizontal scaling, and specific data models (document, key-value, graph, column-family). Unlike SQL databases that enforce rigid table structures, NoSQL lets you store data in the shape your application actually uses.

**Analogy:** Think of SQL as a spreadsheet with fixed columns — every row must conform. NoSQL (document-based) is like a filing cabinet where each folder can contain different types of documents with varying structures.

## Why MongoDB?

MongoDB is a **document-based** NoSQL database that stores data as JSON-like documents (BSON internally). It's the "M" in the MERN stack and is widely used for web applications that need flexible, evolving schemas.

---

## SQL vs NoSQL Comparison

| Concept        | SQL (Relational)           | MongoDB (NoSQL)                  |
| -------------- | -------------------------- | -------------------------------- |
| Data Storage   | Tables with rows & columns | Collections with documents       |
| Schema         | Fixed, predefined          | Flexible, dynamic                |
| Relationships  | JOINs across tables        | Embedded documents or references |
| Scaling        | Vertical (bigger server)   | Horizontal (more servers)        |
| Query Language | SQL                        | MongoDB Query Language (MQL)     |
| Example        | PostgreSQL, MySQL          | MongoDB, CouchDB                 |

---

## When to Use NoSQL (MongoDB)

✅ **Use MongoDB when:**

- Your data schema changes frequently (agile development)
- You need horizontal scaling for large datasets
- Your data is naturally hierarchical/nested (e.g., blog posts with comments)
- You need high write throughput
- Rapid prototyping with evolving requirements

❌ **Stick with SQL when:**

- You need complex multi-table transactions (banking)
- Data has many relationships (social graphs — consider graph DB instead)
- You need strict data integrity and ACID compliance across tables
- Your schema is well-defined and stable

---

## MongoDB's Document Model

### BSON & JSON

MongoDB stores data in **BSON** (Binary JSON) — a binary-encoded serialization of JSON-like documents. You interact with it using JSON syntax.

```json
// A MongoDB Document (stored in a collection)
{
  "_id": ObjectId("64a7f2b3c9e1a2d4e8f01234"),
  "name": "Vikas",
  "email": "vikas@example.com",
  "age": 25,
  "skills": ["JavaScript", "React", "Node.js"],
  "address": {
    "city": "Delhi",
    "country": "India"
  },
  "createdAt": ISODate("2024-01-15T10:30:00Z")
}
```

### Key Concepts

| SQL Term    | MongoDB Term          | Description                        |
| ----------- | --------------------- | ---------------------------------- |
| Database    | Database              | Container for collections          |
| Table       | Collection            | Group of documents                 |
| Row         | Document              | A single record (JSON object)      |
| Column      | Field                 | A key-value pair in a document     |
| Primary Key | `_id`                 | Unique identifier (auto-generated) |
| JOIN        | Embedding / `$lookup` | Combining related data             |

---

## Installing MongoDB

### Option 1: Local Installation (MongoDB Community Server)

1. Download from [mongodb.com/try/download/community](https://www.mongodb.com/try/download/community)
2. Choose your OS, install with default settings
3. Add MongoDB `bin` folder to your system PATH
4. Verify installation:

```bash
mongod --version
# MongoDB server version

mongosh --version
# MongoDB Shell version
```

5. Start the MongoDB server:

```bash
# Start the server (default port 27017)
mongod

# In a new terminal, connect with the shell
mongosh
```

### Option 2: MongoDB Atlas (Cloud - Recommended for Learning)

1. Sign up at [mongodb.com/atlas](https://www.mongodb.com/atlas)
2. Create a free M0 cluster (512MB, shared)
3. Set up a database user (username + password)
4. Whitelist your IP address (or allow all with `0.0.0.0/0` for development)
5. Get your connection string:

```
mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/<dbname>?retryWrites=true&w=majority
```

---

## MongoDB Compass (GUI)

MongoDB Compass is the official GUI for visually exploring your data.

- Download from [mongodb.com/products/compass](https://www.mongodb.com/products/compass)
- Paste your connection string to connect
- Features: browse collections, run queries, view indexes, analyze schema

---

## Mongo Shell (mongosh) Basics

```javascript
// Show all databases
show dbs

// Switch to (or create) a database
use myapp

// Show collections in current database
show collections

// Create a collection implicitly by inserting
db.users.insertOne({ name: "Vikas", age: 25 })

// Find all documents in a collection
db.users.find()

// Find with pretty formatting
db.users.find().pretty()

// Count documents
db.users.countDocuments()

// Drop a collection
db.users.drop()

// Drop the current database
db.dropDatabase()
```

### Useful Shell Commands

```javascript
// Show current database
db;

// Get server status
db.serverStatus();

// Get collection stats
db.users.stats();

// Clear the shell
cls;
```

---

## BSON Data Types

| Type           | Example                    | Description               |
| -------------- | -------------------------- | ------------------------- |
| String         | `"hello"`                  | UTF-8 text                |
| Number (int32) | `42`                       | 32-bit integer            |
| Number (int64) | `NumberLong("9999999999")` | 64-bit integer            |
| Double         | `3.14`                     | 64-bit floating point     |
| Boolean        | `true` / `false`           | True or false             |
| ObjectId       | `ObjectId("...")`          | 12-byte unique identifier |
| Date           | `ISODate("2024-01-01")`    | Date/time                 |
| Array          | `[1, 2, 3]`                | Ordered list              |
| Object         | `{ key: "value" }`         | Embedded document         |
| Null           | `null`                     | Null value                |

---

## Best Practices

1. **Use Atlas for learning** — no setup headaches, free tier is generous
2. **Always set up authentication** — never run MongoDB without auth in production
3. **Design your schema around your queries** — not around your data relationships
4. **Use `_id` wisely** — MongoDB auto-generates ObjectId, but you can use custom values
5. **Embed related data** when it's accessed together (denormalization)
6. **Reference data** when documents would grow unbounded or data is shared across entities

---

## Common Mistakes

| Mistake                                     | Why It's Wrong                   | Fix                                          |
| ------------------------------------------- | -------------------------------- | -------------------------------------------- |
| Running `mongod` without auth in production | Anyone can access your data      | Enable `--auth` flag and create admin user   |
| Using MongoDB for everything                | Not all data fits document model | Use SQL for heavy relational data            |
| Not indexing queried fields                 | Full collection scans are slow   | Create indexes on frequently queried fields  |
| Storing huge arrays in documents            | 16MB document size limit         | Reference instead of embedding               |
| Forgetting to whitelist IP in Atlas         | Connection refused errors        | Add IP to Network Access in Atlas dashboard  |
| Not handling connection errors              | App crashes silently             | Always add error handling for DB connections |

---

## Summary

- **NoSQL** databases offer flexible schemas and horizontal scaling — MongoDB is the most popular document-based NoSQL DB
- MongoDB stores data as **JSON-like documents** (BSON internally) in **collections**
- **Collections** are like tables, **documents** are like rows, but each document can have a different structure
- Use **Atlas** for cloud hosting (free tier available) or install **MongoDB Community** locally
- **MongoDB Compass** provides a GUI for visual data exploration
- **mongosh** is the interactive shell for running queries directly
- Design your schema around **how your application reads data**, not how a relational model would normalize it
