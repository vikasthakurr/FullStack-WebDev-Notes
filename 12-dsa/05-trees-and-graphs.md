# Trees and Graphs

## Binary Tree Basics

A tree is a hierarchical data structure consisting of nodes connected by edges. A **binary tree** is a tree where each node has **at most two children** (left and right).

```
         10          ← root (depth 0)
        /  \
       5    15       ← depth 1
      / \   / \
     3   7 12  20   ← depth 2 (leaves)
```

### Key Terminology

| Term    | Definition                                                               |
| ------- | ------------------------------------------------------------------------ |
| Root    | Top node of the tree (no parent)                                         |
| Node    | An element containing data and pointers to children                      |
| Leaf    | A node with no children                                                  |
| Edge    | Connection between a parent and child node                               |
| Height  | Longest path from a node down to a leaf (height of tree = root's height) |
| Depth   | Distance from the root to a node (root depth = 0)                        |
| Level   | Set of all nodes at the same depth                                       |
| Subtree | A node and all its descendants                                           |

### Binary Tree Node Implementation

```javascript
class TreeNode {
  constructor(val) {
    this.val = val;
    this.left = null;
    this.right = null;
  }
}

// Building a tree manually
const root = new TreeNode(10);
root.left = new TreeNode(5);
root.right = new TreeNode(15);
root.left.left = new TreeNode(3);
root.left.right = new TreeNode(7);
```

---

## Binary Search Tree (BST)

A BST is a binary tree with an ordering property:

- **Left subtree** contains only nodes with values **less than** the parent
- **Right subtree** contains only nodes with values **greater than** the parent
- Both subtrees are also BSTs

```
         10
        /  \
       5    15       5 < 10, 15 > 10 ✓
      / \   / \
     3   7 12  20   3 < 5, 7 > 5, 12 > 10 ∧ 12 < 15 ✓
```

### BST Operations

```javascript
class BST {
  constructor() {
    this.root = null;
  }

  // Insert — O(log n) average, O(n) worst (skewed tree)
  insert(val) {
    const newNode = new TreeNode(val);
    if (!this.root) {
      this.root = newNode;
      return;
    }

    let current = this.root;
    while (true) {
      if (val < current.val) {
        if (!current.left) {
          current.left = newNode;
          return;
        }
        current = current.left;
      } else {
        if (!current.right) {
          current.right = newNode;
          return;
        }
        current = current.right;
      }
    }
  }

  // Search — O(log n) average, O(n) worst
  search(val) {
    let current = this.root;
    while (current) {
      if (val === current.val) return current;
      if (val < current.val) current = current.left;
      else current = current.right;
    }
    return null; // Not found
  }

  // Delete — O(log n) average
  delete(val) {
    this.root = this._deleteNode(this.root, val);
  }

  _deleteNode(node, val) {
    if (!node) return null;

    if (val < node.val) {
      node.left = this._deleteNode(node.left, val);
    } else if (val > node.val) {
      node.right = this._deleteNode(node.right, val);
    } else {
      // Found the node to delete
      // Case 1: No children (leaf)
      if (!node.left && !node.right) return null;
      // Case 2: One child
      if (!node.left) return node.right;
      if (!node.right) return node.left;
      // Case 3: Two children — replace with inorder successor
      let successor = node.right;
      while (successor.left) successor = successor.left;
      node.val = successor.val;
      node.right = this._deleteNode(node.right, successor.val);
    }
    return node;
  }
}
```

### BST Time Complexity

| Operation | Average  | Worst (skewed) |
| --------- | -------- | -------------- |
| Search    | O(log n) | O(n)           |
| Insert    | O(log n) | O(n)           |
| Delete    | O(log n) | O(n)           |

A skewed BST degenerates into a linked list when elements are inserted in order.

---

## Tree Traversals

### Inorder (Left → Root → Right)

For BST, inorder gives nodes in **sorted order**.

```javascript
// Recursive
function inorder(node, result = []) {
  if (!node) return result;
  inorder(node.left, result);
  result.push(node.val);
  inorder(node.right, result);
  return result;
}

// Iterative (using stack)
function inorderIterative(root) {
  const result = [];
  const stack = [];
  let current = root;

  while (current || stack.length > 0) {
    // Go as far left as possible
    while (current) {
      stack.push(current);
      current = current.left;
    }
    current = stack.pop();
    result.push(current.val);
    current = current.right;
  }
  return result;
}
```

### Preorder (Root → Left → Right)

Used to copy/serialize a tree.

```javascript
// Recursive
function preorder(node, result = []) {
  if (!node) return result;
  result.push(node.val);
  preorder(node.left, result);
  preorder(node.right, result);
  return result;
}

// Iterative
function preorderIterative(root) {
  if (!root) return [];
  const result = [];
  const stack = [root];

  while (stack.length > 0) {
    const node = stack.pop();
    result.push(node.val);
    // Push right first so left is processed first (LIFO)
    if (node.right) stack.push(node.right);
    if (node.left) stack.push(node.left);
  }
  return result;
}
```

### Postorder (Left → Right → Root)

Used to delete a tree (process children before parent).

```javascript
// Recursive
function postorder(node, result = []) {
  if (!node) return result;
  postorder(node.left, result);
  postorder(node.right, result);
  result.push(node.val);
  return result;
}

// Iterative (modified preorder reversed)
function postorderIterative(root) {
  if (!root) return [];
  const result = [];
  const stack = [root];

  while (stack.length > 0) {
    const node = stack.pop();
    result.push(node.val);
    if (node.left) stack.push(node.left);
    if (node.right) stack.push(node.right);
  }
  return result.reverse(); // Reverse of root→right→left = left→right→root
}
```

### Level-Order Traversal (BFS)

Process nodes level by level using a queue.

```javascript
function levelOrder(root) {
  if (!root) return [];
  const result = [];
  const queue = [root];

  while (queue.length > 0) {
    const levelSize = queue.length;
    const currentLevel = [];

    for (let i = 0; i < levelSize; i++) {
      const node = queue.shift();
      currentLevel.push(node.val);
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    result.push(currentLevel);
  }
  return result;
}

// For tree:    10
//            /    \
//           5      15
//          / \    /  \
//         3   7  12   20

// Output: [[10], [5, 15], [3, 7, 12, 20]]
```

### Traversal Order Visualization

```
         10
        /  \
       5    15
      / \
     3   7

Inorder:    3, 5, 7, 10, 15  (sorted for BST!)
Preorder:   10, 5, 3, 7, 15  (root first)
Postorder:  3, 7, 5, 15, 10  (root last)
Level-order: 10, 5, 15, 3, 7 (top to bottom, left to right)
```

---

## Balanced Trees (Concept)

A balanced BST keeps the height at O(log n) to guarantee O(log n) operations.

**AVL Tree:**

- Strict balancing — height difference between left and right subtrees is at most 1
- Rotations after every insert/delete to maintain balance
- Faster lookups (stricter balance) but slower insertions

**Red-Black Tree:**

- Relaxed balancing — uses coloring rules (red/black nodes)
- Fewer rotations than AVL — faster insertions
- Used by Java's TreeMap, C++ std::map

> In interviews, you rarely implement these. Know that they exist and guarantee O(log n) operations. JavaScript's built-in structures don't use them directly — use a library if needed.

---

## Graphs

### Terminology

```
Undirected Graph:          Directed Graph (Digraph):
    A ─── B                    A ──→ B
    |   / |                    ↑   ↗ |
    |  /  |                    |  /  ↓
    C ─── D                    C ←── D
```

| Term          | Definition                                        |
| ------------- | ------------------------------------------------- |
| Vertex (Node) | A point in the graph                              |
| Edge          | A connection between two vertices                 |
| Directed      | Edges have a direction (A → B ≠ B → A)            |
| Undirected    | Edges go both ways (A — B = B — A)                |
| Weighted      | Edges have a cost/distance value                  |
| Degree        | Number of edges connected to a vertex             |
| Path          | Sequence of vertices connected by edges           |
| Cycle         | A path that starts and ends at the same vertex    |
| Connected     | Every vertex is reachable from every other vertex |
| DAG           | Directed Acyclic Graph — directed with no cycles  |

---

### Graph Representations

#### Adjacency Matrix

A 2D array where `matrix[i][j] = 1` if there's an edge from i to j.

```javascript
// Undirected graph: A-B, A-C, B-D, C-D
//    A B C D
// A [0,1,1,0]
// B [1,0,0,1]
// C [1,0,0,1]
// D [0,1,1,0]

const matrix = [
  [0, 1, 1, 0],
  [1, 0, 0, 1],
  [1, 0, 0, 1],
  [0, 1, 1, 0],
];

// Check if edge exists: O(1)
// Space: O(V²)
// Good for: dense graphs, frequent edge lookups
```

#### Adjacency List

Each vertex stores a list of its neighbors. Most common representation.

```javascript
// Using Map
class Graph {
  constructor() {
    this.adjacencyList = new Map();
  }

  addVertex(vertex) {
    if (!this.adjacencyList.has(vertex)) {
      this.adjacencyList.set(vertex, []);
    }
  }

  addEdge(v1, v2) {
    this.adjacencyList.get(v1).push(v2);
    this.adjacencyList.get(v2).push(v1); // Remove for directed graph
  }

  getNeighbors(vertex) {
    return this.adjacencyList.get(vertex) || [];
  }
}

const graph = new Graph();
graph.addVertex("A");
graph.addVertex("B");
graph.addVertex("C");
graph.addEdge("A", "B");
graph.addEdge("A", "C");
graph.addEdge("B", "C");

// A → [B, C]
// B → [A, C]
// C → [A, B]

// Space: O(V + E)
// Check if edge exists: O(degree of vertex)
// Good for: sparse graphs (most real-world graphs)
```

#### Comparison

| Feature            | Adjacency Matrix | Adjacency List |
| ------------------ | ---------------- | -------------- |
| Space              | O(V²)            | O(V + E)       |
| Check edge exists  | O(1)             | O(degree)      |
| Find all neighbors | O(V)             | O(degree)      |
| Add edge           | O(1)             | O(1)           |
| Best for           | Dense graphs     | Sparse graphs  |

---

### Graph Traversals

#### BFS (Breadth-First Search)

Explores neighbors level by level using a queue. Good for finding shortest path in unweighted graphs.

```javascript
function bfs(graph, start) {
  const visited = new Set();
  const queue = [start];
  const result = [];
  visited.add(start);

  while (queue.length > 0) {
    const vertex = queue.shift();
    result.push(vertex);

    for (const neighbor of graph.getNeighbors(vertex)) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        queue.push(neighbor);
      }
    }
  }
  return result;
}
// Time: O(V + E), Space: O(V)
```

#### DFS (Depth-First Search)

Explores as deep as possible before backtracking. Uses a stack (or recursion).

```javascript
// Iterative DFS
function dfs(graph, start) {
  const visited = new Set();
  const stack = [start];
  const result = [];

  while (stack.length > 0) {
    const vertex = stack.pop();
    if (visited.has(vertex)) continue;

    visited.add(vertex);
    result.push(vertex);

    for (const neighbor of graph.getNeighbors(vertex)) {
      if (!visited.has(neighbor)) {
        stack.push(neighbor);
      }
    }
  }
  return result;
}

// Recursive DFS
function dfsRecursive(graph, vertex, visited = new Set(), result = []) {
  visited.add(vertex);
  result.push(vertex);

  for (const neighbor of graph.getNeighbors(vertex)) {
    if (!visited.has(neighbor)) {
      dfsRecursive(graph, neighbor, visited, result);
    }
  }
  return result;
}
// Time: O(V + E), Space: O(V)
```

#### BFS vs DFS

| Feature        | BFS                        | DFS                               |
| -------------- | -------------------------- | --------------------------------- |
| Data structure | Queue                      | Stack / Recursion                 |
| Explores       | Level by level             | Deep before backtracking          |
| Shortest path  | Yes (unweighted)           | No                                |
| Memory         | O(width of graph)          | O(depth of graph)                 |
| Use case       | Shortest path, level order | Cycle detection, topological sort |

---

## Common Interview Problems

### 1. Maximum Depth of Binary Tree

```javascript
function maxDepth(root) {
  if (!root) return 0;
  const leftDepth = maxDepth(root.left);
  const rightDepth = maxDepth(root.right);
  return Math.max(leftDepth, rightDepth) + 1;
}
// Time: O(n), Space: O(h) where h = height

// Iterative (BFS — count levels)
function maxDepthBFS(root) {
  if (!root) return 0;
  const queue = [root];
  let depth = 0;

  while (queue.length > 0) {
    const levelSize = queue.length;
    depth++;
    for (let i = 0; i < levelSize; i++) {
      const node = queue.shift();
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
  }
  return depth;
}
```

---

### 2. Validate BST

Every node must satisfy: left < node < right, considering the entire subtree bounds.

```javascript
function isValidBST(root, min = -Infinity, max = Infinity) {
  if (!root) return true;

  // Current node must be within (min, max)
  if (root.val <= min || root.val >= max) return false;

  // Left subtree: all values must be < current node
  // Right subtree: all values must be > current node
  return (
    isValidBST(root.left, min, root.val) &&
    isValidBST(root.right, root.val, max)
  );
}
// Time: O(n), Space: O(h)
```

```
     10
    /  \
   5    15
       /  \
      6    20    ← 6 < 10, violates BST! (6 is in right subtree of 10)

isValidBST catches this because 6 fails the check: min=10, max=15
```

---

### 3. Level Order Traversal

```javascript
function levelOrder(root) {
  if (!root) return [];
  const result = [];
  const queue = [root];

  while (queue.length > 0) {
    const levelSize = queue.length;
    const currentLevel = [];

    for (let i = 0; i < levelSize; i++) {
      const node = queue.shift();
      currentLevel.push(node.val);
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    result.push(currentLevel);
  }
  return result;
}
// Time: O(n), Space: O(n)
```

---

### 4. Lowest Common Ancestor (BST)

In a BST, use the ordering property: if both values are less, go left. If both greater, go right. Otherwise, current node is the LCA.

```javascript
function lowestCommonAncestor(root, p, q) {
  let current = root;

  while (current) {
    if (p.val < current.val && q.val < current.val) {
      current = current.left; // Both in left subtree
    } else if (p.val > current.val && q.val > current.val) {
      current = current.right; // Both in right subtree
    } else {
      return current; // Split point — this is the LCA
    }
  }
  return null;
}
// Time: O(h), Space: O(1)
```

**For a general binary tree (not BST):**

```javascript
function lcaBinaryTree(root, p, q) {
  if (!root || root === p || root === q) return root;

  const left = lcaBinaryTree(root.left, p, q);
  const right = lcaBinaryTree(root.right, p, q);

  if (left && right) return root; // p and q are on different sides
  return left || right; // Both are on the same side
}
// Time: O(n), Space: O(h)
```

---

### 5. Detect Cycle in Graph (DFS)

For directed graphs, use DFS with a "currently visiting" state to detect back edges.

```javascript
function hasCycle(numVertices, edges) {
  // Build adjacency list
  const graph = new Map();
  for (let i = 0; i < numVertices; i++) graph.set(i, []);
  for (const [from, to] of edges) {
    graph.get(from).push(to);
  }

  // 0 = unvisited, 1 = visiting (in current DFS path), 2 = visited
  const state = new Array(numVertices).fill(0);

  function dfs(node) {
    state[node] = 1; // Mark as visiting

    for (const neighbor of graph.get(node)) {
      if (state[neighbor] === 1) return true; // Back edge — cycle!
      if (state[neighbor] === 0 && dfs(neighbor)) return true;
    }

    state[node] = 2; // Mark as fully visited
    return false;
  }

  // Check all components (graph may not be connected)
  for (let i = 0; i < numVertices; i++) {
    if (state[i] === 0 && dfs(i)) return true;
  }
  return false;
}
// Time: O(V + E), Space: O(V)
```

---

### 6. Shortest Path — BFS (Unweighted Graph)

BFS naturally finds the shortest path in an unweighted graph because it explores level by level.

```javascript
function shortestPath(graph, start, end) {
  const visited = new Set([start]);
  const queue = [[start, [start]]]; // [node, path]

  while (queue.length > 0) {
    const [current, path] = queue.shift();

    if (current === end) return path;

    for (const neighbor of graph.getNeighbors(current)) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        queue.push([neighbor, [...path, neighbor]]);
      }
    }
  }
  return null; // No path exists
}
// Time: O(V + E), Space: O(V)
```

---

## Trie (Prefix Tree)

A trie is a tree-like data structure for efficiently storing and searching strings, typically for autocomplete, spell checking, and prefix matching.

```
Words: "cat", "car", "card", "care", "dog"

        root
       /    \
      c      d
      |      |
      a      o
     / \     |
    t   r    g
       / \
      d   e
```

```javascript
class TrieNode {
  constructor() {
    this.children = {};
    this.isEndOfWord = false;
  }
}

class Trie {
  constructor() {
    this.root = new TrieNode();
  }

  // Insert a word — O(m) where m = word length
  insert(word) {
    let current = this.root;
    for (const char of word) {
      if (!current.children[char]) {
        current.children[char] = new TrieNode();
      }
      current = current.children[char];
    }
    current.isEndOfWord = true;
  }

  // Search for exact word — O(m)
  search(word) {
    let current = this.root;
    for (const char of word) {
      if (!current.children[char]) return false;
      current = current.children[char];
    }
    return current.isEndOfWord;
  }

  // Check if any word starts with prefix — O(m)
  startsWith(prefix) {
    let current = this.root;
    for (const char of prefix) {
      if (!current.children[char]) return false;
      current = current.children[char];
    }
    return true;
  }

  // Autocomplete: find all words with given prefix
  autocomplete(prefix) {
    let current = this.root;
    for (const char of prefix) {
      if (!current.children[char]) return [];
      current = current.children[char];
    }

    const results = [];
    this._dfs(current, prefix, results);
    return results;
  }

  _dfs(node, prefix, results) {
    if (node.isEndOfWord) results.push(prefix);
    for (const [char, childNode] of Object.entries(node.children)) {
      this._dfs(childNode, prefix + char, results);
    }
  }
}

// Usage
const trie = new Trie();
trie.insert("cat");
trie.insert("car");
trie.insert("card");
trie.insert("care");

trie.search("car"); // true
trie.search("ca"); // false (not a complete word)
trie.startsWith("ca"); // true
trie.autocomplete("car"); // ["car", "card", "care"]
```

**Trie Complexity:**

| Operation   | Time | Space                                          |
| ----------- | ---- | ---------------------------------------------- |
| Insert      | O(m) | O(m) worst                                     |
| Search      | O(m) | O(1)                                           |
| StartsWith  | O(m) | O(1)                                           |
| Total space | —    | O(n × m) worst where n = words, m = avg length |

---

## Summary

- **Binary Tree**: Hierarchical structure where each node has at most 2 children.
- **BST**: Binary tree with ordering property — enables O(log n) search, insert, delete (when balanced).
- **Traversals**: Inorder (sorted for BST), Preorder (serialize), Postorder (delete), Level-order (BFS).
- **Balanced trees** (AVL, Red-Black) guarantee O(log n) by preventing skew — know they exist, don't implement in interviews.
- **Graphs** model relationships: vertices + edges. Represented as adjacency matrix (dense) or adjacency list (sparse).
- **BFS** uses a queue, explores level by level — finds shortest path in unweighted graphs.
- **DFS** uses a stack/recursion, goes deep first — useful for cycle detection, topological sort, connected components.
- **Trie** is the go-to for prefix-based problems (autocomplete, word search, spell check).
- Key interview patterns: recursion for trees, BFS for shortest path, DFS for exploration/cycle detection.
