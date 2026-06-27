# Stacks and Queues

## Stack (LIFO — Last In, First Out)

A stack is a linear data structure where elements are added and removed from the same end — the **top**. Think of a stack of plates: you can only add or remove from the top.

```
Push 10, 20, 30:

    ┌────┐
    │ 30 │  ← top (last in, first out)
    ├────┤
    │ 20 │
    ├────┤
    │ 10 │
    └────┘

Pop → returns 30 (top element removed)
```

### Stack Operations

| Operation   | Description                         | Time Complexity |
| ----------- | ----------------------------------- | --------------- |
| `push(x)`   | Add element to the top              | O(1)            |
| `pop()`     | Remove and return top element       | O(1)            |
| `peek()`    | Return top element without removing | O(1)            |
| `isEmpty()` | Check if stack is empty             | O(1)            |
| `size()`    | Return number of elements           | O(1)            |

---

### Stack Implementation — Using Array

```javascript
class Stack {
  constructor() {
    this.items = [];
  }

  push(element) {
    this.items.push(element);
  }

  pop() {
    if (this.isEmpty()) return undefined;
    return this.items.pop();
  }

  peek() {
    if (this.isEmpty()) return undefined;
    return this.items[this.items.length - 1];
  }

  isEmpty() {
    return this.items.length === 0;
  }

  size() {
    return this.items.length;
  }
}
```

### Stack Implementation — Using Linked List

Better for large stacks where you want guaranteed O(1) operations (no array resizing).

```javascript
class Node {
  constructor(data) {
    this.data = data;
    this.next = null;
  }
}

class StackLL {
  constructor() {
    this.top = null;
    this.length = 0;
  }

  push(data) {
    const newNode = new Node(data);
    newNode.next = this.top;
    this.top = newNode;
    this.length++;
  }

  pop() {
    if (!this.top) return undefined;
    const data = this.top.data;
    this.top = this.top.next;
    this.length--;
    return data;
  }

  peek() {
    return this.top ? this.top.data : undefined;
  }

  isEmpty() {
    return this.length === 0;
  }

  size() {
    return this.length;
  }
}
```

### Real-World Uses of Stacks

- **Browser history** — back button pops the last visited page
- **Undo/Redo** — each action is pushed, undo pops it
- **Function call stack** — JavaScript execution context
- **Expression evaluation** — parsing mathematical expressions
- **DFS traversal** — explicit stack replaces recursion

---

## Queue (FIFO — First In, First Out)

A queue is a linear data structure where elements are added at the **rear** and removed from the **front**. Think of a line at a store — first person in line gets served first.

```
Enqueue 10, 20, 30:

front                    rear
  │                       │
  ▼                       ▼
┌────┬────┬────┐
│ 10 │ 20 │ 30 │
└────┴────┴────┘

Dequeue → returns 10 (front element removed)
```

### Queue Operations

| Operation    | Description                           | Time Complexity |
| ------------ | ------------------------------------- | --------------- |
| `enqueue(x)` | Add element to the rear               | O(1)            |
| `dequeue()`  | Remove and return front element       | O(1)\*          |
| `front()`    | Return front element without removing | O(1)            |
| `isEmpty()`  | Check if queue is empty               | O(1)            |
| `size()`     | Return number of elements             | O(1)            |

\*O(1) with linked list. Array-based `shift()` is O(n).

---

### Queue Implementation — Using Array (Simple)

```javascript
class QueueSimple {
  constructor() {
    this.items = [];
  }

  enqueue(element) {
    this.items.push(element);
  }

  dequeue() {
    if (this.isEmpty()) return undefined;
    return this.items.shift(); // O(n) — shifts all elements!
  }

  front() {
    return this.items[0];
  }

  isEmpty() {
    return this.items.length === 0;
  }

  size() {
    return this.items.length;
  }
}
```

> **Problem:** `shift()` is O(n) because it re-indexes all elements. For a true O(1) queue, use a linked list or object-based approach.

### Queue Implementation — Using Object (O(1) Dequeue)

```javascript
class Queue {
  constructor() {
    this.items = {};
    this.frontIndex = 0;
    this.rearIndex = 0;
  }

  enqueue(element) {
    this.items[this.rearIndex] = element;
    this.rearIndex++;
  }

  dequeue() {
    if (this.isEmpty()) return undefined;
    const item = this.items[this.frontIndex];
    delete this.items[this.frontIndex];
    this.frontIndex++;
    return item;
  }

  front() {
    return this.items[this.frontIndex];
  }

  isEmpty() {
    return this.rearIndex - this.frontIndex === 0;
  }

  size() {
    return this.rearIndex - this.frontIndex;
  }
}
```

### Queue Implementation — Using Linked List

```javascript
class QueueLL {
  constructor() {
    this.front = null;
    this.rear = null;
    this.length = 0;
  }

  enqueue(data) {
    const newNode = new Node(data);
    if (this.isEmpty()) {
      this.front = newNode;
      this.rear = newNode;
    } else {
      this.rear.next = newNode;
      this.rear = newNode;
    }
    this.length++;
  }

  dequeue() {
    if (this.isEmpty()) return undefined;
    const data = this.front.data;
    this.front = this.front.next;
    if (!this.front) this.rear = null; // Queue is now empty
    this.length--;
    return data;
  }

  peek() {
    return this.front ? this.front.data : undefined;
  }

  isEmpty() {
    return this.length === 0;
  }

  size() {
    return this.length;
  }
}
```

### Real-World Uses of Queues

- **Task scheduling** — CPU job scheduling, printer queue
- **BFS traversal** — graph and tree level-order traversal
- **Message queues** — RabbitMQ, AWS SQS, job processing
- **Rate limiting** — sliding window request tracking
- **Buffer** — keyboard input buffer, streaming data

---

## Circular Queue

A circular queue uses a fixed-size array where the rear wraps around to the front when it reaches the end. Avoids wasted space from dequeuing.

```
Circular Queue (size 5):

After enqueue 10, 20, 30, 40, 50 → full
After dequeue twice → slots 0, 1 are empty

    0     1     2     3     4
┌─────┬─────┬─────┬─────┬─────┐
│     │     │ 30  │ 40  │ 50  │
└─────┴─────┴─────┴─────┴─────┘
              front        rear

Enqueue 60 → wraps to index 0:
    0     1     2     3     4
┌─────┬─────┬─────┬─────┬─────┐
│ 60  │     │ 30  │ 40  │ 50  │
└─────┴─────┴─────┴─────┴─────┘
 rear         front
```

```javascript
class CircularQueue {
  constructor(capacity) {
    this.items = new Array(capacity);
    this.capacity = capacity;
    this.frontIndex = 0;
    this.rearIndex = -1;
    this.length = 0;
  }

  enqueue(element) {
    if (this.isFull()) return false;
    this.rearIndex = (this.rearIndex + 1) % this.capacity; // Wrap around
    this.items[this.rearIndex] = element;
    this.length++;
    return true;
  }

  dequeue() {
    if (this.isEmpty()) return undefined;
    const item = this.items[this.frontIndex];
    this.frontIndex = (this.frontIndex + 1) % this.capacity; // Wrap around
    this.length--;
    return item;
  }

  front() {
    if (this.isEmpty()) return undefined;
    return this.items[this.frontIndex];
  }

  rear() {
    if (this.isEmpty()) return undefined;
    return this.items[this.rearIndex];
  }

  isEmpty() {
    return this.length === 0;
  }

  isFull() {
    return this.length === this.capacity;
  }
}
```

---

## Priority Queue

Elements are dequeued based on **priority**, not insertion order. The highest priority element is served first.

```
Priority Queue (min-priority):
Insert: (task: "email", priority: 3), (task: "bug", priority: 1), (task: "feature", priority: 2)

Dequeue order: "bug" (1) → "feature" (2) → "email" (3)
```

**Implementation approaches:**

| Approach        | Enqueue  | Dequeue  | Best For              |
| --------------- | -------- | -------- | --------------------- |
| Unsorted array  | O(1)     | O(n)     | Rare dequeue          |
| Sorted array    | O(n)     | O(1)     | Rare enqueue          |
| **Binary Heap** | O(log n) | O(log n) | General use (optimal) |

> In interviews, when you need a priority queue, mention that the optimal implementation uses a **binary heap** (Min-Heap or Max-Heap). JavaScript doesn't have a built-in priority queue — you'd implement one with a heap or use a sorted insert for simplicity.

```javascript
// Simple priority queue (sorted insert — O(n) enqueue, O(1) dequeue)
class PriorityQueue {
  constructor() {
    this.items = [];
  }

  enqueue(element, priority) {
    const entry = { element, priority };
    let added = false;

    for (let i = 0; i < this.items.length; i++) {
      if (entry.priority < this.items[i].priority) {
        this.items.splice(i, 0, entry);
        added = true;
        break;
      }
    }
    if (!added) this.items.push(entry);
  }

  dequeue() {
    return this.items.shift()?.element;
  }

  isEmpty() {
    return this.items.length === 0;
  }
}
```

---

## Deque (Double-Ended Queue)

A deque allows insertion and removal from **both ends** in O(1).

```
┌────────────────────────────────────┐
│  addFront / removeFront ←  DEQUE  → addRear / removeRear │
└────────────────────────────────────┘
```

```javascript
class Deque {
  constructor() {
    this.items = {};
    this.frontIndex = 0;
    this.rearIndex = 0;
  }

  addFront(element) {
    this.frontIndex--;
    this.items[this.frontIndex] = element;
  }

  addRear(element) {
    this.items[this.rearIndex] = element;
    this.rearIndex++;
  }

  removeFront() {
    if (this.isEmpty()) return undefined;
    const item = this.items[this.frontIndex];
    delete this.items[this.frontIndex];
    this.frontIndex++;
    return item;
  }

  removeRear() {
    if (this.isEmpty()) return undefined;
    this.rearIndex--;
    const item = this.items[this.rearIndex];
    delete this.items[this.rearIndex];
    return item;
  }

  peekFront() {
    return this.items[this.frontIndex];
  }

  peekRear() {
    return this.items[this.rearIndex - 1];
  }

  isEmpty() {
    return this.rearIndex - this.frontIndex === 0;
  }

  size() {
    return this.rearIndex - this.frontIndex;
  }
}
```

**Uses:** Sliding window maximum, palindrome checking, work-stealing algorithms.

---

## Common Interview Problems

### 1. Valid Parentheses (Balanced Brackets)

Use a stack to match opening brackets with their corresponding closing brackets.

```javascript
function isValid(s) {
  const stack = [];
  const map = {
    ")": "(",
    "}": "{",
    "]": "[",
  };

  for (const char of s) {
    if (char === "(" || char === "{" || char === "[") {
      stack.push(char);
    } else {
      // Closing bracket — check if it matches top of stack
      if (stack.length === 0 || stack.pop() !== map[char]) {
        return false;
      }
    }
  }
  return stack.length === 0; // All brackets matched
}
// Time: O(n), Space: O(n)

isValid("()[]{}"); // true
isValid("(]"); // false
isValid("([)]"); // false
isValid("{[]}"); // true
```

---

### 2. Implement Queue Using Two Stacks

One stack for enqueue, one for dequeue. Transfer elements when needed.

```javascript
class MyQueue {
  constructor() {
    this.pushStack = []; // For enqueue
    this.popStack = []; // For dequeue
  }

  enqueue(x) {
    this.pushStack.push(x);
  }

  dequeue() {
    if (this.popStack.length === 0) {
      // Transfer all from pushStack to popStack (reverses order)
      while (this.pushStack.length > 0) {
        this.popStack.push(this.pushStack.pop());
      }
    }
    return this.popStack.pop();
  }

  front() {
    if (this.popStack.length === 0) {
      while (this.pushStack.length > 0) {
        this.popStack.push(this.pushStack.pop());
      }
    }
    return this.popStack[this.popStack.length - 1];
  }

  isEmpty() {
    return this.pushStack.length === 0 && this.popStack.length === 0;
  }
}
// Amortized O(1) per operation
```

```
Enqueue 1, 2, 3:
pushStack: [1, 2, 3]    popStack: []

Dequeue:
Transfer → pushStack: []  popStack: [3, 2, 1]
Pop from popStack → returns 1 (FIFO order!)

Enqueue 4:
pushStack: [4]           popStack: [3, 2]

Dequeue → returns 2 (from popStack — no transfer needed)
```

---

### 3. Implement Stack Using Two Queues

```javascript
class MyStack {
  constructor() {
    this.q1 = [];
    this.q2 = [];
  }

  push(x) {
    // Push to q2, then move all from q1 to q2, then swap
    this.q2.push(x);
    while (this.q1.length > 0) {
      this.q2.push(this.q1.shift());
    }
    [this.q1, this.q2] = [this.q2, this.q1]; // Swap
  }

  pop() {
    return this.q1.shift();
  }

  top() {
    return this.q1[0];
  }

  isEmpty() {
    return this.q1.length === 0;
  }
}
// Push: O(n), Pop: O(1)
```

---

### 4. Min Stack — Get Minimum in O(1)

Maintain a second stack that tracks the minimum at each level.

```javascript
class MinStack {
  constructor() {
    this.stack = [];
    this.minStack = []; // Tracks min at each level
  }

  push(val) {
    this.stack.push(val);
    // Push the new minimum (current val or existing min)
    const currentMin =
      this.minStack.length === 0
        ? val
        : Math.min(val, this.minStack[this.minStack.length - 1]);
    this.minStack.push(currentMin);
  }

  pop() {
    this.stack.pop();
    this.minStack.pop();
  }

  top() {
    return this.stack[this.stack.length - 1];
  }

  getMin() {
    return this.minStack[this.minStack.length - 1];
  }
}
// All operations: O(1) time, O(n) space
```

```
Push: 5, 3, 7, 2, 8
stack:    [5, 3, 7, 2, 8]
minStack: [5, 3, 3, 2, 2]  ← min at each level

getMin() → 2
pop() → removes 8
stack:    [5, 3, 7, 2]
minStack: [5, 3, 3, 2]
getMin() → 2 (still correct!)
```

---

### 5. Next Greater Element

For each element in array, find the next element that is greater (to its right). Use a stack to track elements waiting for their next greater.

```javascript
function nextGreaterElement(nums) {
  const result = new Array(nums.length).fill(-1);
  const stack = []; // Stores indices

  for (let i = 0; i < nums.length; i++) {
    // While current element is greater than elements stack is waiting for
    while (stack.length > 0 && nums[i] > nums[stack[stack.length - 1]]) {
      const index = stack.pop();
      result[index] = nums[i];
    }
    stack.push(i);
  }
  return result;
}
// Time: O(n), Space: O(n)

nextGreaterElement([4, 5, 2, 25]);
// [5, 25, 25, -1]

nextGreaterElement([13, 7, 6, 12]);
// [-1, 12, 12, -1]
```

```
Array: [4, 5, 2, 25]

i=0: stack=[0(4)]
i=1: 5 > 4 → result[0]=5, stack=[1(5)]
i=2: 2 < 5 → stack=[1(5), 2(2)]
i=3: 25 > 2 → result[2]=25, 25 > 5 → result[1]=25, stack=[3(25)]

Result: [5, 25, 25, -1]
```

---

### 6. Sliding Window Maximum (Deque Approach)

Find the maximum in every window of size k. Use a deque to maintain indices of useful elements in decreasing order.

```javascript
function maxSlidingWindow(nums, k) {
  const result = [];
  const deque = []; // Stores indices, front is always the max for current window

  for (let i = 0; i < nums.length; i++) {
    // Remove indices outside current window
    while (deque.length > 0 && deque[0] <= i - k) {
      deque.shift();
    }

    // Remove indices of elements smaller than current
    // (they'll never be the max while current exists in window)
    while (deque.length > 0 && nums[deque[deque.length - 1]] <= nums[i]) {
      deque.pop();
    }

    deque.push(i);

    // Window is fully formed (i >= k-1)
    if (i >= k - 1) {
      result.push(nums[deque[0]]); // Front of deque is the max
    }
  }
  return result;
}
// Time: O(n), Space: O(k)

maxSlidingWindow([1, 3, -1, -3, 5, 3, 6, 7], 3);
// [3, 3, 5, 5, 6, 7]
```

```
Window [1, 3, -1] → max = 3
Window [3, -1, -3] → max = 3
Window [-1, -3, 5] → max = 5
Window [-3, 5, 3] → max = 5
Window [5, 3, 6] → max = 6
Window [3, 6, 7] → max = 7
```

---

## Comparison Table

| Feature        | Stack              | Queue              | Deque            |
| -------------- | ------------------ | ------------------ | ---------------- |
| Access pattern | LIFO               | FIFO               | Both ends        |
| Insert         | Top — O(1)         | Rear — O(1)        | Both ends — O(1) |
| Remove         | Top — O(1)         | Front — O(1)       | Both ends — O(1) |
| Use case       | DFS, undo, parsing | BFS, scheduling    | Sliding window   |
| JS built-in    | Array (push/pop)   | Array (push/shift) | No built-in      |

---

## Summary

- **Stack (LIFO):** Push and pop from the same end. Used for DFS, undo, expression parsing, backtracking.
- **Queue (FIFO):** Enqueue at rear, dequeue from front. Used for BFS, task scheduling, buffering.
- **Circular Queue:** Fixed-size queue where rear wraps to front — avoids wasted space.
- **Priority Queue:** Elements served by priority, not order. Best implemented with a heap (O(log n) operations).
- **Deque:** Insert and remove from both ends in O(1). Key to sliding window maximum.
- In JavaScript, arrays work as stacks (`push`/`pop`) but are inefficient as queues (`shift` is O(n)). Use object-based or linked list implementations for true O(1) queue operations.
- Common interview pattern: use a **stack** whenever you need to match/track something that follows a "most recent" rule (brackets, next greater, undo).
