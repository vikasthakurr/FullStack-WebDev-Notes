## Part 1 — Web Foundations

### Module 1 · HTML | Basics of Web Pages

- Introduction to HTML
- HTML Boilerplate & Page Structure
- Headings & Paragraphs
- Text Formatting (`b`, `i`, `strong`, `em`, `mark`, `small`, `del`, `sup`, `sub`)
- Semantic HTML (`header`, `nav`, `main`, `section`, `article`, `footer`, `aside`)
- Audio & Video Tags
- Block vs Inline Elements
- HTML5 Attributes (`required`, `placeholder`, `readonly`, `disabled`, `autofocus`)
- Meta Tags (`viewport`, `description`, `keywords`)
- Adding Images & Attributes
- Lists (`ul`, `ol`, `dl`)
- Links (anchor, mailto, tel)
- Tables
- Input Types
- Forms
- Accessibility Basics
- ARIA Accessibility

---

### Module 2 · CSS | Styling Web Pages

- Types of CSS
- CSS Selectors
- Colors in CSS (RGB, HEX, HSL, HSLA)
- Fonts & Text Properties
- CSS Units (`px`, `%`, `em`, `rem`, `vw`, `vh`)
- Box Model (padding, margin, border, outline, box-sizing)
- Backgrounds & Image Filters
- Flexbox (layout, alignment, gap, wrapping)
- CSS Grid (rows, columns, gaps, alignment)
- Position Property (static, relative, absolute, fixed, sticky)
- Animations & Transitions (keyframes, duration, easing)c
- Media Queries (responsive design)
- 🎨 Exercise: Rothko Painting (CSS exercise)

---

### Module 3 · Bootstrap 5 | Quick Styling

- Typography
- Breakpoints
- Grid
- Tables
- Alerts
- Buttons
- Modals
- Collapse
- Accordion
- Navbar & Collapsible Navbar
- Forms
- Carousel (image slider)
- Toast (notifications)
- Colored Links

---

## Part 2 — JavaScript

### Module 4 · JavaScript | Basics

- Introduction to JavaScript
- How JavaScript Runs (Browser & Node.js)
- Compilation vs Interpretation
- Single-Threaded Nature & Concurrency
- Event Loop
- Installing Node.js & npm
- Using Browser Console
- Editor Setup & Debugging Basics
- Smallest Possible Program
- Memory Management
- Global Execution Context

---

### Module 5 · JavaScript | Core Concepts

- Variables (`var`, `let`, `const`)
- Primitive Types
- Reference Types (objects, arrays, functions)
- Comparison (`==` vs `===`)
- Conditional Statements (`if/else`, `switch`)
- Loops (`for`, `while`, `do...while`, `forEach`, `for...in`)
- Scope & Lexical Environment
- Scope Chaining
- Closures
- 🧩 Two Button Problem (classic problem)
- Objects
- Rest & Spread Operators
- Handling Multiple Arguments
- Freezing Objects, Sealing Objects
- Shallow & Deep Copy
- Callback Functions
- Currying
- Higher-Order Functions
- `map`, `filter`, `reduce`
- `Array.prototype` Basics
- IIFE (Immediately Invoked Function Expression)

---

### Module 6 · JavaScript | Advanced Concepts

- DOM Manipulation & Vanilla JS
- Event Listeners
- Common DOM Events
- Event Bubbling & Capturing
- Event Delegation
- 🛠️ DOM Projects (practice)
- Callback Hell
- Promises
- Fetch API
- Handling API Responses & Destructuring
- Async / Await
- `this` Keyword
- `call`, `apply` and `bind`
- Debounce and Throttle

---

## Part 3 — TypeScript

### Module TS-1 · TypeScript Basics

- What is TypeScript? (typed superset of JavaScript)
- Installing TypeScript & `tsc` compiler
- `tsconfig.json` overview
- Type Annotations (`string`, `number`, `boolean`, `any`, `unknown`, `never`, `void`)
- Type Inference
- Arrays & Tuples
- Enums (numeric & string)
- Union & Intersection Types
- Type Aliases vs Interfaces
- Optional & Default Properties
- Readonly & Const Assertions

---

### Module TS-2 · TypeScript with Functions & OOP

- Function Types & Return Types
- Optional & Rest Parameters
- Overloading Functions
- Classes in TypeScript
- Access Modifiers (`public`, `private`, `protected`)
- Abstract Classes & Interfaces
- Generics (`T`, `K`, `V`)
- Generic Functions & Generic Interfaces
- Decorators (intro)

---

### Module TS-3 · TypeScript Advanced Types

- Utility Types (`Partial`, `Required`, `Pick`, `Omit`, `Record`, `Readonly`)
- Mapped Types
- Conditional Types
- Template Literal Types
- Discriminated Unions
- Type Guards (`typeof`, `instanceof`, `in`)
- Narrowing & Control Flow Analysis
- TypeScript with DOM
- TypeScript in a React + Vite project

---

## Part 4 — ReactJS | Frontend Framework

### Module 7 · React Basics

- Real DOM vs Virtual DOM
- Setting up React with Vite
- First React App
- Components
- React Portals
- Event Handlers
- State in React
- `useState` Hook
- Props (passing data)
- Props Drilling
- `useEffect` Hook
- `useRef` Hook
- `useContext` & Context API
- `useMemo` Hook
- `useCallback` Hook
- `React.memo` (optimization)
- Custom Hooks
- Conditional Rendering
- React Router DOM
- Link vs Anchor
- Nested Routes & Params
- Skeleton UI
- Lazy and Suspense
- Redux (Global State Management)

---

### Module 8 · React 19 | New Hooks

> Hooks introduced in React 19

- `useOptimistic`
- `useActionState` (`useAction`)
- `useViewTransition`
- `useFormStatus`
- `useResult`

---

### Module 9 · React UI & Styling | Tailwind CSS

- Tailwind CSS Basics & Demo
- Tailwind Setup in React
- 🛠️ Project: Travel Website

---

## Part 5 — Node.js & Express

### Module 10 · Getting Started with Node.js

- What is Node.js? (event-driven, non-blocking I/O)
- Installing Node.js & npm
- Node.js REPL & First Program
- Node.js Architecture (event loop, single-threaded nature)
- Node.js vs Browser JavaScript

---

### Module 11 · Node.js Core Modules

- CommonJS vs ES Modules
- File System (`fs`)

---

### Module 12 · HTTP & Core Server

- HTTP Module Basics
- Creating a Server without Express
- Handling Requests & Responses
- Serving JSON & HTML
- Manual Routing

---

### Module 13 · Express Basics

- What is Express & Why Use It?
- Setting up an Express Server
- Express App Structure (MVC overview)
- Routes (`GET`, `POST`, `PUT`, `DELETE`)
- Route Parameters & Query Strings

---

### Module 14 · Middleware in Express

- What is Middleware?
- Built-in Middleware (`express.json()`, `urlencoded`, `static`)
- Custom Middleware
- Error-handling Middleware

---

### Module 15 · RESTful APIs with Express

- Designing RESTful APIs
- CRUD APIs
- Status Codes & Best Practices
- Postman / Thunder Client Testing

---

### Module 16 · Advanced Express Features

- Express Router (modular routing)
- Request Validation (`express-validator`, Joi)
- File Uploads with Multer
- Logging with Morgan / Winston
- Sending Emails (Nodemailer)

---

## Part 6 — MongoDB & Mongoose

### Module 17 · MongoDB Basics

- What is NoSQL? (Document-based DB)
- Installing MongoDB locally / using Atlas
- Mongo Shell & Compass Overview
- CRUD Operations (insert, find, update, delete)
- BSON vs JSON

---

### Module 18 · Mongoose ODM

- Connecting Express to MongoDB
- Schema & Model Design (Mongoose / Prisma)
- CRUD with Mongoose
- Schema Validation

---

### Module 19 · Advanced MongoDB

- Indexes & Performance Optimization
- Aggregation Framework
- Transactions in MongoDB
- Relationships: Embedding vs Referencing
- Pagination & Filtering

---

### Module 20 · Authentication & Security

- User Registration & Login
- Password Hashing with bcrypt
- JWT Authentication in Express
- Role-Based Access Control
- Security Best Practices (Helmet, CORS, rate limiting, dotenv)

---

## Part 7 — System Design

### Module 21 · System Design Fundamentals

- Monolith vs Microservices Architecture
- CAP Theorem (Consistency, Availability, Partition Tolerance)
- BASE Consistency (Basically Available, Soft State, Eventual Consistency)
- Throughput vs Latency
- High Availability & Fault Tolerance
- Proxy vs Reverse Proxy

---

### Module 22 · High Level Design (HLD) for Full Stack Web

- Scalability (Vertical vs Horizontal)
- Load Balancing (L4 vs L7, Algorithms)
- Caching Strategies (Redis, CDN, Browser Caching)
- Database Design (SQL vs NoSQL, Sharding, Replication, Indexing)
- Message Queues & Event-Driven Architecture (RabbitMQ, Kafka)
- API Gateway & Rate Limiting
- Service Discovery
- Disaster Recovery & Backup

---

### Module 23 · Advanced Web Performance & Optimization

- Critical Rendering Path (CRP): DOM, CSSOM, Render Tree
- Resource Prioritization: Preload, Prefetch, Preconnect
- CORS & Preflight Requests (OPTIONS)
- Windowing / List Virtualization (handling large datasets)
- Image Optimization & Lazy Loading
- Core Web Vitals (LCP, FID, CLS)

---

### Module 24 · Real-Time Communication

- HTTP Polling vs Long Polling
- WebSockets (Bi-directional communication)
- Server-Sent Events (SSE)
- WebRTC (Introduction)

---

### Module 25 · Micro Frontend System Design

- Core Principles of Micro Frontends
- Integration Patterns (Build-time, Run-time, Server-side)
- Module Federation (Webpack 5 / Vite)
- Communication between Micro Frontends (Custom Events, Shared State)
- Shared Component Libraries & Design Systems
- Independent Deployment & CI/CD Pipelines
- Routing & Deep Linking in Micro Frontends
- Performance & Security Considerations

## Deployment

### Module 27 · Deployment

- Hosting Node.js Apps (Render, Railway, Heroku, AWS)
- Connecting to Cloud MongoDB (MongoDB Atlas)
- Environment Variables in Production
- CI/CD Basics (GitHub Actions)
- Deployment Best Practices

---

## Part 8 — Git & GitHub

### Module 28 · Git

- Git Basics (init, add, commit, status, log)
- Branching & Merging
- Remote Repos & GitHub
- Pull Requests & Code Review
- Advanced Git (rebase, cherry-pick, stash, bisect)

---

## Part 9 — Testing

### Module 29 · Testing

- Testing Fundamentals (unit, integration, e2e, TDD)
- Jest Basics
- React Testing Library
- API Testing with Supertest
- E2E Testing with Playwright

---

## Part 10 — Next.js

### Module 30 · Next.js

- Introduction to Next.js
- App Router & File Conventions
- Server Components vs Client Components
- Data Fetching Patterns
- Server Actions
- Deployment on Vercel

---

## Part 11 — Data Structures & Algorithms (Interviews)

### Module 31 · DSA

- Big-O Notation
- Arrays & Strings
- Linked Lists
- Stacks & Queues
- Trees & Graphs
- Sorting Algorithms
- Searching Algorithms
- Common Patterns (Sliding Window, Two Pointers, DP, Backtracking)
