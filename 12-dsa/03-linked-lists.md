# Linked Lists

## What Is a Linked List?

A linked list is a linear data structure where elements (called **nodes**) are stored in non-contiguous memory locations. Each node contains:

1. **Data** — the value stored in the node
2. **Pointer (next)** — a reference to the next node in the sequence

Unlike arrays, linked list elements are not stored in adjacent memory — they can be scattered anywhere in memory, connected through pointers.

```
Array (contiguous memory):
┌───┬───┬───┬───┬───┐
│ 1 │ 2 │ 3 │ 4 │ 5 │
└───┴───┴───┴───┴───┘

Linked List (non-contiguous, connected by pointers):
┌───┬───┐    ┌───┬───┐    ┌───┬──────┐
│ 1 │ ──┼───►│ 2 │ ──┼───►│ 3 │ null │
└───┴───┘    └───┴───┘    └───┴──────┘
 data next    data next    data  next
```

---

## Types of Linked Lists

### Singly Linked List

Each node points to the next node. Traversal is one-directional (forward only).

```
head
 │
 ▼
[10] → [20] → [30] → [40] → null
```

### Doubly Linked List

Each node has pointers to both the next and previous nodes. Traversal is bidirectional.

```
null ← [10] ⇄ [20] ⇄ [30] ⇄ [40] → null
        head                    tail
```

### Circular Linked List

The last node points back to the first node instead of null.

```
┌→ [10] → [20] → [30] → [40] ─┐
│                               │
└───────────────────────────────┘
```

---

## Why Linked Lists? (vs Arrays)

| Feature               | Array                 | Linked List                 |
| --------------------- | --------------------- | --------------------------- |
| Access by index       | O(1)                  | O(n)                        |
| Insert at beginning   | O(n) — shift elements | O(1) — update head pointer  |
| Insert at end         | O(1) amortized        | O(1) with tail pointer      |
| Insert in middle      | O(n) — shift elements | O(1) after finding position |
| Delete from beginning | O(n) — shift elements | O(1) — update head pointer  |
| Delete from middle    | O(n) — shift elements | O(1) after finding position |
| Memory                | Contiguous block      | Scattered, extra pointer    |
| Cache performance     | Excellent (locality)  | Poor (scattered memory)     |
| Dynamic sizing        | Requires resizing     | Naturally dynamic           |

**Use Linked Lists when:**

- Frequent insertions/deletions at the beginning
- You don't know the size in advance and want to avoid resizing
- You don't need random access by index

**Use Arrays when:**

- You need fast random access by index
- Data size is relatively fixed
- Cache performance matters (iteration-heavy workloads)

---

## Implementation in JavaScript

### Node Class

```javascript
class Node {
  constructor(data) {
    this.data = data;
    this.next = null;
  }
}
```

### Singly Linked List Class

```javascript
class LinkedList {
  constructor() {
    this.head = null;
    this.size = 0;
  }

  // Insert at the beginning — O(1)
  insertAtHead(data) {
    const newNode = new Node(data);
    newNode.next = this.head;
    this.head = newNode;
    this.size++;
  }

  // Insert at the end — O(n) without tail pointer
  insertAtTail(data) {
    const newNode = new Node(data);

    if (!this.head) {
      this.head = newNode;
    } else {
      let current = this.head;
      while (current.next) {
        current = current.next;
      }
      current.next = newNode;
    }
    this.size++;
  }

  // Insert at a specific position (0-indexed) — O(n)
  insertAt(data, position) {
    if (position < 0 || position > this.size) return false;

    if (position === 0) {
      this.insertAtHead(data);
      return true;
    }

    const newNode = new Node(data);
    let current = this.head;
    let prev = null;
    let index = 0;

    while (index < position) {
      prev = current;
      current = current.next;
      index++;
    }

    newNode.next = current;
    prev.next = newNode;
    this.size++;
    return true;
  }

  // Delete a node by value — O(n)
  delete(data) {
    if (!this.head) return false;

    // If head is the node to delete
    if (this.head.data === data) {
      this.head = this.head.next;
      this.size--;
      return true;
    }

    let current = this.head;
    while (current.next) {
      if (current.next.data === data) {
        current.next = current.next.next;
        this.size--;
        return true;
      }
      current = current.next;
    }
    return false; // Not found
  }

  // Search for a value — O(n)
  search(data) {
    let current = this.head;
    let index = 0;

    while (current) {
      if (current.data === data) return index;
      current = current.next;
      index++;
    }
    return -1; // Not found
  }

  // Reverse the list in-place — O(n)
  reverse() {
    let prev = null;
    let current = this.head;
    let next = null;

    while (current) {
      next = current.next; // Save next
      current.next = prev; // Reverse pointer
      prev = current; // Move prev forward
      current = next; // Move current forward
    }
    this.head = prev;
  }

  // Print the list
  print() {
    let current = this.head;
    const values = [];
    while (current) {
      values.push(current.data);
      current = current.next;
    }
    console.log(values.join(" → ") + " → null");
  }
}
```

### Usage

```javascript
const list = new LinkedList();
list.insertAtHead(30);
list.insertAtHead(20);
list.insertAtHead(10);
list.insertAtTail(40);
list.print(); // 10 → 20 → 30 → 40 → null

list.insertAt(25, 2);
list.print(); // 10 → 20 → 25 → 30 → 40 → null

list.delete(25);
list.print(); // 10 → 20 → 30 → 40 → null

console.log(list.search(30)); // 2
console.log(list.search(99)); // -1

list.reverse();
list.print(); // 40 → 30 → 20 → 10 → null
```

---

## Operations Time Complexity

| Operation          | Singly Linked List | Doubly Linked List |
| ------------------ | ------------------ | ------------------ |
| Insert at head     | O(1)               | O(1)               |
| Insert at tail     | O(n) / O(1)\*      | O(1)               |
| Insert at position | O(n)               | O(n)               |
| Delete head        | O(1)               | O(1)               |
| Delete tail        | O(n)               | O(1)               |
| Delete by value    | O(n)               | O(n)               |
| Search             | O(n)               | O(n)               |
| Access by index    | O(n)               | O(n)               |
| Reverse            | O(n)               | O(n)               |

\*O(1) if you maintain a tail pointer.

---

## Common Interview Problems

### 1. Reverse a Linked List

**Iterative approach — O(n) time, O(1) space:**

```javascript
function reverseList(head) {
  let prev = null;
  let current = head;

  while (current) {
    const next = current.next; // Save
    current.next = prev; // Reverse
    prev = current; // Advance prev
    current = next; // Advance current
  }
  return prev; // New head
}
```

```
Step by step:
Initial: null   1 → 2 → 3 → null
                ↑
              current

Step 1:  null ← 1   2 → 3 → null
                     ↑
                   current

Step 2:  null ← 1 ← 2   3 → null
                         ↑
                       current

Step 3:  null ← 1 ← 2 ← 3
                         ↑
                        prev (new head)
```

**Recursive approach — O(n) time, O(n) space (call stack):**

```javascript
function reverseListRecursive(head) {
  // Base case: empty list or single node
  if (!head || !head.next) return head;

  // Reverse the rest of the list
  const newHead = reverseListRecursive(head.next);

  // Make the next node point back to current
  head.next.next = head;
  head.next = null;

  return newHead;
}
```

---

### 2. Detect Cycle — Floyd's Tortoise and Hare

Use two pointers: slow moves 1 step, fast moves 2 steps. If they meet, there's a cycle.

```
With cycle:
1 → 2 → 3 → 4 → 5
              ↑       ↓
              8 ← 7 ← 6

Slow: 1, 2, 3, 4, 5, 6, 7, 8, 3, 4, 5...
Fast: 1, 3, 5, 7, 3, 5, 7...
They meet at some point inside the cycle!
```

```javascript
function hasCycle(head) {
  if (!head || !head.next) return false;

  let slow = head;
  let fast = head;

  while (fast && fast.next) {
    slow = slow.next; // 1 step
    fast = fast.next.next; // 2 steps

    if (slow === fast) return true; // They met — cycle exists
  }
  return false; // Fast reached end — no cycle
}
// Time: O(n), Space: O(1)
```

**Find where the cycle starts:**

```javascript
function detectCycleStart(head) {
  if (!head || !head.next) return null;

  let slow = head;
  let fast = head;

  // Phase 1: Detect cycle
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow === fast) break;
  }

  if (!fast || !fast.next) return null; // No cycle

  // Phase 2: Find start of cycle
  // Reset one pointer to head, move both at same speed
  slow = head;
  while (slow !== fast) {
    slow = slow.next;
    fast = fast.next;
  }
  return slow; // Start of cycle
}
```

---

### 3. Find Middle Node — Slow/Fast Pointer

Slow moves 1 step, fast moves 2 steps. When fast reaches the end, slow is at the middle.

```javascript
function findMiddle(head) {
  let slow = head;
  let fast = head;

  while (fast && fast.next) {
    slow = slow.next; // 1 step
    fast = fast.next.next; // 2 steps
  }
  return slow; // Middle node
}
// Time: O(n), Space: O(1)
```

```
List: 1 → 2 → 3 → 4 → 5
       s       s       s
       f           f           f (null)
                   ↑
                 middle (3)

List: 1 → 2 → 3 → 4 → 5 → 6
       s       s       s
       f           f           f (null.next fails)
                       ↑
               middle (4 — second middle for even length)
```

---

### 4. Merge Two Sorted Lists

```javascript
function mergeTwoLists(l1, l2) {
  // Use a dummy node to simplify edge cases
  const dummy = new Node(0);
  let current = dummy;

  while (l1 && l2) {
    if (l1.data <= l2.data) {
      current.next = l1;
      l1 = l1.next;
    } else {
      current.next = l2;
      l2 = l2.next;
    }
    current = current.next;
  }

  // Attach remaining nodes
  current.next = l1 || l2;

  return dummy.next; // Skip dummy, return real head
}
// Time: O(n + m), Space: O(1)
```

```
l1: 1 → 3 → 5
l2: 2 → 4 → 6

Result: 1 → 2 → 3 → 4 → 5 → 6
```

---

### 5. Remove Nth Node From End

Use two pointers with a gap of n between them. When the fast pointer reaches the end, slow is just before the node to remove.

```javascript
function removeNthFromEnd(head, n) {
  const dummy = new Node(0);
  dummy.next = head;

  let fast = dummy;
  let slow = dummy;

  // Move fast n+1 steps ahead
  for (let i = 0; i <= n; i++) {
    fast = fast.next;
  }

  // Move both until fast reaches end
  while (fast) {
    slow = slow.next;
    fast = fast.next;
  }

  // slow.next is the node to remove
  slow.next = slow.next.next;

  return dummy.next;
}
// Time: O(n), Space: O(1)
```

```
Remove 2nd from end: 1 → 2 → 3 → 4 → 5

Step 1 — gap of n+1=3:
dummy → 1 → 2 → 3 → 4 → 5 → null
 slow              fast

Step 2 — move both:
dummy → 1 → 2 → 3 → 4 → 5 → null
               slow              fast

slow.next (4) is removed → 1 → 2 → 3 → 5
```

---

## Doubly Linked List Implementation

```javascript
class DoublyNode {
  constructor(data) {
    this.data = data;
    this.next = null;
    this.prev = null;
  }
}

class DoublyLinkedList {
  constructor() {
    this.head = null;
    this.tail = null;
    this.size = 0;
  }

  // Insert at head — O(1)
  insertAtHead(data) {
    const newNode = new DoublyNode(data);

    if (!this.head) {
      this.head = newNode;
      this.tail = newNode;
    } else {
      newNode.next = this.head;
      this.head.prev = newNode;
      this.head = newNode;
    }
    this.size++;
  }

  // Insert at tail — O(1)
  insertAtTail(data) {
    const newNode = new DoublyNode(data);

    if (!this.tail) {
      this.head = newNode;
      this.tail = newNode;
    } else {
      newNode.prev = this.tail;
      this.tail.next = newNode;
      this.tail = newNode;
    }
    this.size++;
  }

  // Delete a node — O(1) if you have the node reference
  deleteNode(node) {
    if (node.prev) node.prev.next = node.next;
    else this.head = node.next;

    if (node.next) node.next.prev = node.prev;
    else this.tail = node.prev;

    this.size--;
  }
}
```

---

## Summary

- A **linked list** stores elements in nodes connected by pointers — not contiguous memory.
- **Singly linked list**: each node points to the next. Simple but can only traverse forward.
- **Doubly linked list**: nodes point both ways. O(1) delete at tail, bidirectional traversal.
- **O(1) insert/delete at head** is the main advantage over arrays — no shifting required.
- **O(n) access by index** is the main disadvantage — no random access.
- Key interview patterns: **two pointers** (slow/fast), **dummy node** (simplifies edge cases), **reversing pointers**.
- Floyd's cycle detection uses O(1) space by moving slow (1 step) and fast (2 steps) pointers.
- Always consider edge cases: empty list, single node, operation on head/tail.
