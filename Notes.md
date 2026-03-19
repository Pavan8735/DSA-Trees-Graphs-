# Trees & Graphs — Deep Study Guide

> Every concept includes: What it is → Why it matters → Visual walkthrough → Code → Common mistakes → Interview tips

---

## Table of Contents

1. Core Data Structures (TreeNode, Graph, WeightedGraph)
2. Tree Height
3. Tree Diameter
4. Tree Traversals — Pre / In / Post / Level-order
5. Lowest Common Ancestor (LCA)
6. Check if a Tree is Balanced
7. Mirror / Invert a Tree
8. Path Sum
9. Count Nodes and Leaves
10. Maximum Path Sum
11. Graph BFS
12. Graph DFS
13. Detect Cycle — Undirected Graph
14. Detect Cycle — Directed Graph
15. Topological Sort
16. Connected Components
17. Bipartite Graph Check
18. Dijkstra — Shortest Path (Weighted)
19. Bellman-Ford — Shortest Path with Negative Weights
20. Floyd-Warshall — All-Pairs Shortest Path
21. Prim's Minimum Spanning Tree
22. Kruskal's Minimum Spanning Tree
23. Complexity Cheat Sheet

---

---

## 1. Core Data Structures

### What is a Binary Tree?

A **binary tree** is a hierarchical data structure where every node has at most two children — called the **left child** and the **right child**. It is not linear like an array or linked list; instead, it branches out like an actual tree (but drawn upside down in CS).

Real-world analogy: Think of a family tree. You are a node, your two children are your left and right subtrees. Your parent is the node above you. The topmost ancestor with no parent is the **root**.

Key vocabulary:
- **Root**: The topmost node (no parent)
- **Leaf**: A node with no children
- **Height**: Number of edges on the longest path from root to a leaf
- **Depth**: Number of edges from the root to a given node
- **Subtree**: Any node and all of its descendants

### What is a Graph?

A **graph** is a more general structure. It has **vertices** (nodes) and **edges** (connections between nodes). Unlike trees, graphs can have cycles, multiple paths between nodes, and disconnected parts.

Real-world analogy: Cities (vertices) connected by roads (edges). Facebook friends — each person is a node, each friendship is an edge.

Key vocabulary:
- **Directed graph**: Edges go one way (like Twitter follows)
- **Undirected graph**: Edges go both ways (like Facebook friends)
- **Weighted graph**: Each edge has a cost/distance (like road maps)
- **Adjacency list**: Most common way to store graphs — each node stores a list of its neighbors

```python
# ── Binary Tree Node ──────────────────────────────────────────────────────────
class TreeNode:
    def __init__(self, val, left=None, right=None):
        self.val   = val
        self.left  = left
        self.right = right

# ── Unweighted Graph ──────────────────────────────────────────────────────────
class Graph:
    def __init__(self, vertices):
        self.vertices       = vertices
        self.adjacency_list = {i: [] for i in range(vertices)}

    def add_edge(self, u, v):
        self.adjacency_list[u].append(v)
        self.adjacency_list[v].append(u)   # Both directions = undirected

    def get_neighbors(self, vertex):
        return self.adjacency_list[vertex]

# ── Weighted Graph ────────────────────────────────────────────────────────────
class WeightedGraph:
    def __init__(self, vertices):
        self.vertices       = vertices
        self.adjacency_list = {i: [] for i in range(vertices)}

    def add_edge(self, u, v, weight):
        self.adjacency_list[u].append((v, weight))
        self.adjacency_list[v].append((u, weight))

    def get_neighbors(self, vertex):
        return self.adjacency_list[vertex]

# ── Sample Tree used throughout this guide ────────────────────────────────────
#          1          <- Root (depth 0)
#        /   \
#       2     3       <- depth 1
#      / \   / \
#     4   5 6   7     <- Leaves (depth 2)
tree1 = TreeNode(1,
            TreeNode(2, TreeNode(4), TreeNode(5)),
            TreeNode(3, TreeNode(6), TreeNode(7)))
```

---

## 2. Tree Height

### What is the Problem?

Find how **tall** the tree is — the number of edges (or nodes) on the longest path from the root down to any leaf. This is one of the most fundamental tree problems and forms the base of many harder problems.

### The Key Insight

The height of any node = 1 + the taller of its two subtrees.
If the node is None (empty), height is 0. That is your base case.

You can't know the height of the root until you know the heights of the left and right subtrees. So you go all the way down to the leaves first, then combine results as you come back up. This is called **post-order recursion** — process children before the parent.

### Step-by-Step Dry Run

```
Tree:       1
           / \
          2   3
         / \
        4   5
```

- height(4) → no children → return 0 + 1 = 1
- height(5) → no children → return 0 + 1 = 1
- height(2) → max(height(4), height(5)) + 1 = max(1,1) + 1 = 2
- height(3) → no children → return 0 + 1 = 1
- height(1) → max(height(2), height(3)) + 1 = max(2,1) + 1 = 3
```

### Code

```python
def height_of_tree(node):
    if node is None:       # Base case: empty subtree has height 0
        return 0
    left_height  = height_of_tree(node.left)
    right_height = height_of_tree(node.right)
    return max(left_height, right_height) + 1   # +1 counts current node

print(height_of_tree(tree1))   # Output: 3
```

### Common Mistakes
- Returning -1 for None instead of 0 (mixes up edge-count vs node-count definitions — pick one and be consistent)
- Forgetting the +1 (counts edges only, misses the root itself)

### When to Use
Any time a problem asks about depth, levels, or requires knowing how deep a subtree goes.

---

## 3. Tree Diameter

### What is the Problem?

Find the **longest path between any two nodes** in the tree. This path does NOT have to pass through the root.

### The Key Insight

At any node, the longest path passing through it = height of its left subtree + height of its right subtree.

The trick is: while computing height (which we already know), also track the maximum diameter seen at any node. We use a list `[0]` instead of a plain variable because Python closures can read outer variables but can't reassign them without `nonlocal`. The list is a workaround.

### Step-by-Step Dry Run

```
Tree:       1
           / \
          2   3
         / \
        4   5
```

At node 2: left_h=1, right_h=1 → path through 2 = 1+1 = 2 (path: 4→2→5)
At node 1: left_h=2 (subtree of 2), right_h=1 → path through 1 = 2+1 = 3 (path: 4→2→1→3)

Answer = 3

But for tree1 (perfectly balanced):
At node 1: left_h=2, right_h=2 → path = 2+2 = 4 (path: 4→2→1→3→6 or similar)

### Code

```python
def diameter_of_tree(node):
    max_diameter = [0]   # Use list so nested function can modify it

    def height(node):
        if node is None:
            return 0
        left_h  = height(node.left)
        right_h = height(node.right)
        # The path THROUGH this node spans both subtrees
        max_diameter[0] = max(max_diameter[0], left_h + right_h)
        return max(left_h, right_h) + 1   # Still return height for parent

    height(node)
    return max_diameter[0]

print(diameter_of_tree(tree1))  # Output: 4
```

### Why Not Just Compute Height of Left + Height of Right at Root?

Because the longest path might not go through the root at all. Example:

```
    1
   /
  2
 / \
3   4
     \
      5
```

The longest path is 3→2→4→5, which never touches node 1.

### Common Mistakes
- Only checking the diameter at the root — misses paths in subtrees
- Calling height() separately (causing O(n^2)) instead of computing diameter inside height in one pass

---

## 4. Tree Traversals

### What are Traversals?

A traversal visits every node in the tree exactly once. The ORDER in which you visit nodes depends on the traversal type. Different orderings are useful for different tasks.

### Pre-order — Root First

**Order**: Visit current node → recurse left → recurse right

**Real use**: Copying a tree. Serializing a tree to disk. If you want to process a parent before its children.

**Memory trick**: "Pre" = Root comes PRE (before) everything else.

```
Tree:       1
           / \
          2   3
         / \
        4   5

Visit: 1 → go left → 2 → go left → 4 → back up → 5 → back up → back to 1 → go right → 3
Result: [1, 2, 4, 5, 3]
```

```python
def preorder_traversal(node):
    if node is None:
        return []
    return [node.val] + preorder_traversal(node.left) + preorder_traversal(node.right)

print(preorder_traversal(tree1))  # Output: [1, 2, 4, 5, 3, 6, 7]
```

### In-order — Root in the Middle

**Order**: Recurse left → visit current node → recurse right

**Real use**: For a Binary Search Tree (BST), in-order traversal gives nodes in SORTED order. This is the most important property of BSTs.

**Memory trick**: "In" = Root comes IN the middle.

```
Tree:       1
           / \
          2   3
         / \
        4   5

All the way left → 4 → back to 2 → 5 → back to 1 → 3
Result: [4, 2, 5, 1, 3]
```

```python
def inorder_traversal(node):
    if node is None:
        return []
    return inorder_traversal(node.left) + [node.val] + inorder_traversal(node.right)

print(inorder_traversal(tree1))   # Output: [4, 2, 5, 1, 6, 3, 7]
```

### Post-order — Root Last

**Order**: Recurse left → recurse right → visit current node

**Real use**: Deleting a tree (must delete children before parent). Computing size/height of subtrees. File system — compute folder sizes (you need to know all subfolders first).

**Memory trick**: "Post" = Root comes POST (after) everything else.

```python
def postorder_traversal(node):
    if node is None:
        return []
    return postorder_traversal(node.left) + postorder_traversal(node.right) + [node.val]

print(postorder_traversal(tree1)) # Output: [4, 5, 2, 6, 7, 3, 1]
```

### Level-order BFS — Row by Row

**Order**: Visit all nodes at depth 0, then depth 1, then depth 2, etc.

**Real use**: Finding the shortest path in an unweighted tree. Printing a tree level by level. When you care about the distance from the root.

**Uses a queue** (FIFO): enqueue the root, then for each node dequeued, enqueue its children.

```
Level 0:  [1]
Level 1:  [2, 3]
Level 2:  [4, 5, 6, 7]
```

```python
from collections import deque

def level_order_traversal(node):
    if node is None:
        return []
    result = []
    queue  = deque([node])       # Start with root in queue
    while queue:
        current = queue.popleft()        # Process front of queue
        result.append(current.val)
        if current.left:                 # Enqueue children for next level
            queue.append(current.left)
        if current.right:
            queue.append(current.right)
    return result

print(level_order_traversal(tree1))  # Output: [1, 2, 3, 4, 5, 6, 7]
```

### Quick Comparison Table

| Traversal   | Order               | Key Use Case             |
|-------------|---------------------|--------------------------|
| Pre-order   | Root → Left → Right | Tree serialization/copy  |
| In-order    | Left → Root → Right | Sorted output from BST   |
| Post-order  | Left → Right → Root | Delete tree, compute size|
| Level-order | Row by row (BFS)    | Shortest path, levels    |

---

## 5. Lowest Common Ancestor (LCA)

### What is the Problem?

Given two nodes p and q in a tree, find their **Lowest Common Ancestor** — the deepest node that is an ancestor of both p and q.

Real-world analogy: In a company org chart, the LCA of two employees is their most immediate shared manager.

### The Key Insight

Do a DFS. At each node, ask:
- If this node IS p or q, return this node (we found one of them)
- Recurse left and right
- If BOTH left and right return something, this node is between p and q — it IS the LCA
- If only one side returns something, bubble that result up

### Step-by-Step Dry Run for LCA(4, 6)

```
          1
        /   \
       2     3
      / \   / \
     4   5 6   7

LCA(1): recurse left (looking for 4 and 6)
  LCA(2): recurse left → LCA(4): node.val==4 → return node(4)
          recurse right → LCA(5): returns None
          only left returned something → bubble up node(4)
  LCA(3): recurse left → LCA(6): node.val==6 → return node(6)
          only left returned → bubble up node(6)
LCA(1): left=node(4), right=node(6) → BOTH non-None → return node(1) ← ANSWER
```

```python
def lowest_common_ancestor(node, p, q):
    # Base: if we reach empty or find one of our targets, return it
    if node is None or node.val == p or node.val == q:
        return node

    left  = lowest_common_ancestor(node.left,  p, q)
    right = lowest_common_ancestor(node.right, p, q)

    if left and right:
        return node        # p is in one subtree, q in the other → this is LCA
    return left if left else right  # Both in same subtree — pass result up

lca = lowest_common_ancestor(tree1, 4, 5)
print(lca.val)  # Output: 2   (both 4 and 5 are children of 2)

lca = lowest_common_ancestor(tree1, 4, 6)
print(lca.val)  # Output: 1   (4 is left side, 6 is right side → root is LCA)
```

### Common Mistakes
- Forgetting that a node can be an ancestor of itself (if p == current node and q is in its subtree, this node IS the LCA)

---

## 6. Check if a Tree is Balanced

### What is the Problem?

A **height-balanced** tree is one where for every single node, the height difference between its left and right subtrees is at most 1. This is NOT just checking the root — it must hold at EVERY node.

### Why Does This Matter?

A balanced BST guarantees O(log n) search time. An unbalanced tree degrades to O(n) — as bad as a linked list. Interview favorite!

### The Naive Approach vs Optimal Approach

**Naive (O(n^2))**: Call height() on every node, check if |left_h - right_h| <= 1. But height() itself is O(n), and you call it n times.

**Optimal (O(n))**: Combine the check INTO the height function. Return -1 as a special signal meaning "already unbalanced below here — don't bother checking further."

### Step-by-Step Dry Run (Unbalanced Tree)

```
  1
 /
2
 \
  3

check(3): leaf → return 1
check(2): left=0, right=1 → |0-1|=1 ≤ 1, ok → return max(0,1)+1 = 2
check(1): left=2, right=0 → |2-0|=2 > 1 → return -1 (UNBALANCED)
is_balanced returns: (-1 != -1) is False → NOT balanced
```

```python
def is_balanced(node):
    def check(node):
        if node is None:
            return 0                    # Empty tree has height 0
        left_h  = check(node.left)
        right_h = check(node.right)
        if left_h == -1 or right_h == -1:
            return -1                   # Already unbalanced below — propagate
        if abs(left_h - right_h) > 1:
            return -1                   # Unbalanced at this node
        return max(left_h, right_h) + 1 # Return actual height if balanced

    return check(node) != -1           # -1 means unbalanced was found

print(is_balanced(tree1))              # Output: True

unbalanced = TreeNode(1, TreeNode(2, TreeNode(3)), None)
print(is_balanced(unbalanced))         # Output: False
```

---

## 7. Mirror / Invert a Tree

### What is the Problem?

Swap every left child with its corresponding right child, at every level of the tree. The result is a mirror image.

### Real-World Analogy

Imagine holding a transparent piece of paper with the tree drawn on it, then flipping it horizontally. Every left becomes right, every right becomes left.

### Step-by-Step Dry Run

```
Before:        After:
     1              1
   /   \          /   \
  2     3   →    3     2
 / \   / \      / \   / \
4   5 6   7    7   6 5   4
```

At each node, swap left and right children. Then recursively do the same for the new left and right subtrees. Order matters: use Python's tuple swap to do it atomically.

```python
def mirror_tree(node):
    if node is None:
        return None
    # Swap children (recursively mirror them first, then assign)
    node.left, node.right = mirror_tree(node.right), mirror_tree(node.left)
    return node

mirrored = mirror_tree(TreeNode(1,
                TreeNode(2, TreeNode(4), TreeNode(5)),
                TreeNode(3, TreeNode(6), TreeNode(7))))
print(inorder_traversal(mirrored))  # Output: [7, 3, 6, 1, 5, 2, 4]
```

---

## 8. Path Sum

### What is the Problem?

Given a target value, determine if there is any root-to-leaf path where the sum of all node values equals the target.

### The Key Insight

Instead of building up a sum, SUBTRACT the node's value from the target as you go down. When you hit a leaf, check if the remaining target equals the leaf's value.

This avoids having to carry a running sum AND compare at leaves.

### Step-by-Step Dry Run (target = 22)

```
       5
      / \
     4   8
    /   / \
   11  13   4
  / \        \
 7   2         1

Path 5→4→11→2:
- At 5:  remaining = 22 - 5 = 17
- At 4:  remaining = 17 - 4 = 13
- At 11: remaining = 13 - 11 = 2
- At 2:  leaf, remaining = 2 - 2 = 0 ✓ FOUND!
```

```python
def has_path_sum(node, target):
    if node is None:
        return False
    # Check if it's a leaf AND the value matches remaining target
    if node.left is None and node.right is None:
        return node.val == target
    # Subtract current value and recurse down both sides
    remaining = target - node.val
    return has_path_sum(node.left, remaining) or has_path_sum(node.right, remaining)

sum_tree = TreeNode(5,
               TreeNode(4, TreeNode(11, TreeNode(7), TreeNode(2)), None),
               TreeNode(8, TreeNode(13), TreeNode(4, None, TreeNode(1))))

print(has_path_sum(sum_tree, 22))   # Output: True  (5+4+11+2 = 22)
print(has_path_sum(sum_tree, 26))   # Output: True  (5+8+13 = 26)
print(has_path_sum(sum_tree, 99))   # Output: False
```

### Common Mistakes
- Not checking specifically for LEAF nodes — if you only check `node.val == target` without confirming it's a leaf, you'll get wrong answers when a leaf's parent matches the target

---

## 9. Count Nodes and Leaves

### What is the Problem?

Count all nodes in the tree, or count only the leaf nodes (nodes with no children).

### The Key Insight

Both problems use the same recursive pattern: "My count = 1 + count(left) + count(right)". For leaves, we return 1 only when both children are None; otherwise we recurse.

```python
def count_nodes(node):
    if node is None:
        return 0
    return 1 + count_nodes(node.left) + count_nodes(node.right)

def count_leaves(node):
    if node is None:
        return 0
    if node.left is None and node.right is None:
        return 1          # This is a leaf — count it
    return count_leaves(node.left) + count_leaves(node.right)

print(count_nodes(tree1))   # Output: 7   (nodes: 1,2,3,4,5,6,7)
print(count_leaves(tree1))  # Output: 4   (leaves: 4,5,6,7)
```

---

## 10. Maximum Path Sum

### What is the Problem?

Find the path between any two nodes in the tree that gives the maximum sum of node values. The path does NOT need to pass through the root or even go top-to-bottom — it can go up, cross, and come down.

### The Key Insight

At every node, we have a choice:
- The best contribution I can give my PARENT is: my value + max(0, best from left OR right) — one direction only, since a path to a parent can only come from one side.
- The best PATH through ME (not going to parent): my value + max(0, best from left) + max(0, best from right) — both sides possible since the path ends here.

We update the global maximum with the path through each node, but return only the single-direction gain to the parent.

We use max(..., 0) to ignore any subtree that would decrease the sum (no point going down a negative path).

```python
def max_path_sum(node):
    max_sum = [float('-inf')]

    def helper(node):
        if node is None:
            return 0
        # Gain from going into left/right — ignore if negative
        left_gain  = max(helper(node.left),  0)
        right_gain = max(helper(node.right), 0)

        # Best path through THIS node (can use both sides)
        price_through_node = node.val + left_gain + right_gain
        max_sum[0] = max(max_sum[0], price_through_node)

        # Return to parent: can only go ONE direction
        return node.val + max(left_gain, right_gain)

    helper(node)
    return max_sum[0]

# Tree:   -10
#         /  \
#        9    20
#            /  \
#           15    7
val_tree = TreeNode(-10, TreeNode(9), TreeNode(20, TreeNode(15), TreeNode(7)))
print(max_path_sum(val_tree))  # Output: 42 (path: 15 → 20 → 7 = 42)
```

---

---

## 11. Graph — BFS (Breadth-First Search)

### What is BFS?

BFS explores a graph **level by level** — it visits all nodes that are 1 hop away, then all nodes 2 hops away, then 3 hops, etc.

Real-world analogy: Water spreading from a stone dropped in a pond. The ripple expands outward uniformly in all directions.

### Why Use BFS?

BFS **guarantees the shortest path** in an unweighted graph. When you first reach a node in BFS, you've reached it via the fewest edges possible. This is why BFS is used for things like: "find the minimum number of moves to reach X", social network "6 degrees of separation", GPS navigation on unweighted maps.

### How it Works — The Queue

BFS uses a **queue (FIFO)**. Think of a queue like a line at a grocery store — first in, first out.

1. Put the start node in the queue
2. While queue is not empty: take from the FRONT
3. If not visited: mark visited, add to result, put all its neighbors in the BACK

```
Graph:  0 — 1 — 3
        |   |
        2   4

Start at 0:
Queue: [0]
Process 0 → visited={0}, result=[0], queue=[1,2]
Process 1 → visited={0,1}, result=[0,1], queue=[2,3,4]
Process 2 → visited={0,1,2}, result=[0,1,2], queue=[3,4]
Process 3 → result=[0,1,2,3], queue=[4]
Process 4 → result=[0,1,2,3,4], queue=[]
```

```python
def bfs(graph, start):
    visited   = set()      # Track what we've seen
    queue     = [start]    # Start from the source
    bfs_order = []

    while queue:
        vertex = queue.pop(0)       # Take from FRONT (FIFO)
        if vertex not in visited:
            visited.add(vertex)
            bfs_order.append(vertex)
            queue.extend(graph.get_neighbors(vertex))  # Add neighbors to BACK

    return bfs_order

g = Graph(5)
g.add_edge(0, 1); g.add_edge(0, 2)
g.add_edge(1, 3); g.add_edge(1, 4)
print(bfs(g, 0))  # Output: [0, 1, 2, 3, 4]
```

### BFS vs DFS — When to Use Which?

| Situation                          | Use   |
|------------------------------------|-------|
| Shortest path (unweighted)         | BFS   |
| Connected components               | Either|
| Topological sort                   | DFS   |
| Detect cycle                       | Either|
| Explore ALL paths                  | DFS   |
| Tree level-by-level processing     | BFS   |
| Memory-constrained (wide graphs)   | DFS   |

---

## 12. Graph — DFS (Depth-First Search)

### What is DFS?

DFS dives as deep as possible down one path before backtracking and trying another. Like exploring a maze — you keep going straight until you hit a dead end, then backtrack.

Real-world analogy: Solving a maze by always turning left. You explore one complete path fully before trying alternatives.

### Iterative DFS (uses a Stack)

DFS uses a **stack (LIFO)** — last in, first out. Think of a stack of plates.

```
Graph:  0 — 1 — 3
        |   |
        2   4

Start at 0:
Stack: [0]
Pop 0 → stack=[1,2] (push neighbors in reverse order for correct ordering)
Pop 1 → stack=[2,3,4]
Pop 3 → stack=[2,4]
Pop 4 → stack=[2]
Pop 2 → stack=[]
Result: [0,1,3,4,2]
```

```python
def dfs_iterative(graph, start):
    visited   = set()
    stack     = [start]
    dfs_order = []

    while stack:
        vertex = stack.pop()         # Take from TOP (LIFO)
        if vertex not in visited:
            visited.add(vertex)
            dfs_order.append(vertex)
            for neighbor in reversed(graph.get_neighbors(vertex)):
                stack.append(neighbor)

    return dfs_order

print(dfs_iterative(g, 0))  # Output: [0, 1, 3, 4, 2]
```

### Recursive DFS

The call stack IS the stack. Cleaner to read, but can hit Python's recursion limit on very deep graphs.

```python
def dfs_recursive(graph, start, visited=None):
    if visited is None:
        visited = set()
    visited.add(start)
    result = [start]
    for neighbor in graph.get_neighbors(start):
        if neighbor not in visited:
            result += dfs_recursive(graph, neighbor, visited)
    return result

print(dfs_recursive(g, 0))  # Output: [0, 1, 3, 4, 2]
```

### Common Mistakes
- Not passing `visited` set into the recursive call (creates a new set every call — infinite loop)
- Using `if visited is None: visited = set()` in the signature — critical pattern for Python mutable default arguments

---

## 13. Detect Cycle — Undirected Graph

### What is the Problem?

Does the graph contain a cycle — a path that starts and ends at the same node?

Real-world analogy: If you're walking down roads and you arrive at a city you've already visited WITHOUT taking the same road twice — there's a cycle.

### The Key Insight

In an undirected graph, every edge goes both ways. So when doing DFS from node A to its neighbor B, you CAN go back from B to A — but that's not a cycle, that's just the same edge.

A **real cycle** is when you reach a node that was already visited AND it is NOT the node you just came from (i.e., not your direct parent in the DFS tree).

We pass `parent` to track where we came from, so we can ignore the "return edge".

### Step-by-Step Dry Run (Graph with cycle: 0-1-2-3-0)

```
DFS from 0 (parent=-1):
  Visit 0 → go to 1 (parent=0)
    Visit 1 → go to 2 (parent=1)
      Visit 2 → go to 3 (parent=2)
        Visit 3 → neighbors: [2, 0]
          2 is visited AND 2 == parent → skip (not a cycle)
          0 is visited AND 0 != parent → CYCLE FOUND!
```

```python
def has_cycle_undirected(graph):
    visited = set()

    def dfs(vertex, parent):
        visited.add(vertex)
        for neighbor in graph.get_neighbors(vertex):
            if neighbor not in visited:
                if dfs(neighbor, vertex):  # Recurse with this node as new parent
                    return True
            elif neighbor != parent:       # Visited AND not direct parent = cycle
                return True
        return False

    for v in range(graph.vertices):
        if v not in visited:               # Handle disconnected graphs
            if dfs(v, -1):                 # -1 = no parent for the start node
                return True
    return False

cycle_graph = Graph(4)
cycle_graph.add_edge(0, 1); cycle_graph.add_edge(1, 2)
cycle_graph.add_edge(2, 3); cycle_graph.add_edge(3, 0)
print(has_cycle_undirected(cycle_graph))   # Output: True

tree_graph = Graph(5)
tree_graph.add_edge(0, 1); tree_graph.add_edge(0, 2)
tree_graph.add_edge(1, 3); tree_graph.add_edge(1, 4)
print(has_cycle_undirected(tree_graph))    # Output: False (it's a tree)
```

---

## 14. Detect Cycle — Directed Graph

### What is the Problem?

Same question but for DIRECTED graphs. In directed graphs, you can go A→B but not B→A (unless there's an explicit edge).

### Why is This Different?

In a directed graph, "already visited" is not enough to detect a cycle. Consider:

```
0 → 1 → 2
        ↓
        3 → 4
```

If you're DFS-ing from 0→1→2→3→4, and then separately start DFS from another vertex that also reaches node 2, node 2 is "visited" but there's no cycle — we just reached it via a different path.

The key: a cycle in a DIRECTED graph means there's a path back to a node that is currently on our **active recursion stack** — an ancestor in the current DFS path, not just any visited node.

### Step-by-Step Dry Run (Directed cycle: 1→2→3→1)

```
DFS from 0: 0 → 1 → 2 → 3
  rec_stack = {0,1,2,3}
  From 3, neighbor is 1
  1 is IN rec_stack → CYCLE!
```

```python
class DirectedGraph:
    def __init__(self, vertices):
        self.vertices       = vertices
        self.adjacency_list = {i: [] for i in range(vertices)}

    def add_edge(self, u, v):            # One-way only
        self.adjacency_list[u].append(v)

    def get_neighbors(self, vertex):
        return self.adjacency_list[vertex]

def has_cycle_directed(graph):
    visited   = set()   # All nodes ever visited
    rec_stack = set()   # Nodes on the CURRENT DFS path (ancestors)

    def dfs(vertex):
        visited.add(vertex)
        rec_stack.add(vertex)         # We're exploring from here right now
        for neighbor in graph.get_neighbors(vertex):
            if neighbor not in visited:
                if dfs(neighbor):
                    return True
            elif neighbor in rec_stack:   # Visited AND in current path = cycle
                return True
        rec_stack.discard(vertex)     # Done exploring this path — remove from stack
        return False

    for v in range(graph.vertices):
        if v not in visited:
            if dfs(v):
                return True
    return False

dg = DirectedGraph(4)
dg.add_edge(0, 1); dg.add_edge(1, 2)
dg.add_edge(2, 3); dg.add_edge(3, 1)  # Creates cycle: 1→2→3→1
print(has_cycle_directed(dg))          # Output: True
```

---

## 15. Topological Sort

### What is the Problem?

Given a directed acyclic graph (DAG), produce a linear ordering of all nodes such that for every directed edge A→B, node A comes before node B in the result.

### Real-World Examples

- **Course prerequisites**: If CS201 requires CS101, CS101 must appear before CS201.
- **Build systems**: If module A imports module B, compile B before A.
- **Task scheduling**: Task A must complete before Task B can start.

### How it Works

Run DFS. The key insight: a node should only appear in the output AFTER all nodes reachable from it have been added. So we push a node to the stack AFTER its entire subtree is done. Then reverse the stack.

### Step-by-Step Dry Run

```
DAG:  5 → 2 → 3 → 1
      5 → 0
      4 → 0
      4 → 1

DFS from 5: 5→2→3→1(dead end, push 1)→back to 3(push 3)→back to 2(push 2)
            5→0(dead end, push 0)→back to 5(push 5)
DFS from 4: 4→0(visited)→4→1(visited)→push 4
Stack (reversed): [5, 4, 2, 3, 1, 0]
```

```python
def topological_sort(graph):
    visited = set()
    stack   = []

    def dfs(vertex):
        visited.add(vertex)
        for neighbor in graph.get_neighbors(vertex):
            if neighbor not in visited:
                dfs(neighbor)
        stack.append(vertex)      # Push AFTER all descendants are processed

    for v in range(graph.vertices):
        if v not in visited:
            dfs(v)

    return stack[::-1]            # Reverse gives the correct topological order

dag = DirectedGraph(6)
dag.add_edge(5, 2); dag.add_edge(5, 0)
dag.add_edge(4, 0); dag.add_edge(4, 1)
dag.add_edge(2, 3); dag.add_edge(3, 1)
print(topological_sort(dag))   # Output: [5, 4, 2, 3, 1, 0]
```

### Important Note
Topological sort only works on DAGs. If the graph has a cycle, there is no valid topological ordering (you'd have a course that requires itself as a prerequisite).

---

## 16. Connected Components

### What is the Problem?

Find all groups of nodes in a graph where every node in a group can reach every other node in the same group, but cannot reach nodes in other groups.

Real-world analogy: Islands. Each island is a component — you can walk anywhere on the same island, but you can't walk to a different island.

### How it Works

Run DFS/BFS from every unvisited node. Each new DFS call you make from the outer loop means you've found a brand-new component. Collect all nodes visited in that DFS into one group.

```python
def connected_components(graph):
    visited    = set()
    components = []

    def dfs(vertex, component):
        visited.add(vertex)
        component.append(vertex)
        for neighbor in graph.get_neighbors(vertex):
            if neighbor not in visited:
                dfs(neighbor, component)

    for v in range(graph.vertices):
        if v not in visited:           # New component found!
            component = []
            dfs(v, component)
            components.append(component)

    return components

disconnected = Graph(7)
disconnected.add_edge(0, 1); disconnected.add_edge(1, 2)  # Component 1: 0-1-2
disconnected.add_edge(3, 4)                               # Component 2: 3-4
# 5 and 6 are isolated                                    # Components: [5], [6]
print(connected_components(disconnected))
# Output: [[0, 1, 2], [3, 4], [5], [6]]
```

---

## 17. Bipartite Graph Check

### What is the Problem?

Can you colour every node with one of two colours (say, Red or Blue) so that NO edge connects two nodes of the same colour?

### Real-World Examples

- Can you divide a group of people into two teams such that all friendships are between teams, not within teams?
- Bipartite graphs model many matching problems: students ↔ courses, jobs ↔ workers, etc.

### The Key Insight

A graph is bipartite if and only if it contains **no odd-length cycles**. The BFS 2-colouring approach reveals this naturally: try to colour each node the opposite colour of its neighbour. If you find two adjacent nodes that must be the same colour — the graph is NOT bipartite.

### Step-by-Step Dry Run

```
Square graph: 0-1-2-3-0 (even cycle → bipartite)
Colour 0 = Red
  Neighbour 1 = Blue
    Neighbour 2 = Red
      Neighbour 3 = Blue
        Neighbour 0 = Red ≠ Blue ✓ (no conflict)
Result: Bipartite!

Triangle: 0-1-2-0 (odd cycle → not bipartite)
Colour 0 = Red
  Neighbour 1 = Blue
    Neighbour 2 = Red
      Neighbour 0 = Red == Red ✗ CONFLICT
Result: NOT bipartite
```

```python
def is_bipartite(graph):
    colour = {}   # Maps vertex → 0 or 1

    def bfs(start):
        queue = [start]
        colour[start] = 0
        while queue:
            v = queue.pop(0)
            for neighbor in graph.get_neighbors(v):
                if neighbor not in colour:
                    colour[neighbor] = 1 - colour[v]    # Opposite colour
                    queue.append(neighbor)
                elif colour[neighbor] == colour[v]:     # Same colour = conflict
                    return False
        return True

    for v in range(graph.vertices):
        if v not in colour:
            if not bfs(v):
                return False
    return True

bipartite_g = Graph(4)
bipartite_g.add_edge(0, 1); bipartite_g.add_edge(1, 2)
bipartite_g.add_edge(2, 3); bipartite_g.add_edge(3, 0)
print(is_bipartite(bipartite_g))  # Output: True (even cycle)

odd_cycle = Graph(3)
odd_cycle.add_edge(0, 1); odd_cycle.add_edge(1, 2); odd_cycle.add_edge(2, 0)
print(is_bipartite(odd_cycle))    # Output: False (triangle = odd cycle)
```

---

---

## 18. Dijkstra's Algorithm — Shortest Path

### What is the Problem?

Given a weighted graph, find the shortest distance from a source node to all other nodes. "Shortest" here means least total weight, not fewest hops.

Real-world analogy: Google Maps finding the fastest driving route — roads have different travel times (weights). You want the path with the minimum total travel time, not the one with fewest turns.

### Why Not Just BFS?

BFS assumes all edges have equal weight (cost 1). Dijkstra handles different weights. For example, taking a 10km highway might be faster than two 1km dirt roads combined.

### How Dijkstra Works

Dijkstra is a **greedy algorithm**. It always processes the node with the smallest known distance next.

1. Set distance to source = 0, all others = infinity
2. Use a **min-heap (priority queue)** to always pick the closest unprocessed node
3. For each neighbor of the current node, check if going through the current node gives a shorter path. If yes, update (this is called "relaxing" the edge)
4. Repeat until the heap is empty

### Step-by-Step Dry Run

```
Graph:   0 --(10)--> 1
         0 --(3)-->  2
         2 --(4)-->  1
         1 --(2)-->  3

Start at 0: dist = {0:0, 1:inf, 2:inf, 3:inf}
Heap: [(0, node0)]

Pop (0,0): neighbours: (1,10) (2,3)
  dist[1] = 10, dist[2] = 3
  Heap: [(3,2), (10,1)]

Pop (3,2): neighbours: (1,4)
  Via 2: dist[1] = min(10, 3+4) = 7. UPDATE!
  Heap: [(7,1), (10,1)]

Pop (7,1): neighbours: (3,2)
  dist[3] = 7+2 = 9
  Heap: [(9,3), (10,1 — STALE)]

Pop (9,3): no unvisited improvements
Pop (10,1): 10 > dist[1]=7 → SKIP (stale entry)

Final: {0:0, 1:7, 2:3, 3:9}
```

### Important: Stale Entries in the Heap

A node can be pushed to the heap multiple times when its distance is updated. We check `if current_dist > dist[u]: continue` to skip outdated entries.

```python
import heapq

def dijkstra(graph, source):
    dist = {v: float('inf') for v in range(graph.vertices)}
    dist[source] = 0
    min_heap = [(0, source)]       # (distance, vertex)

    while min_heap:
        current_dist, u = heapq.heappop(min_heap)

        # Skip if we already found a shorter path to u
        if current_dist > dist[u]:
            continue

        for v, weight in graph.get_neighbors(u):
            new_dist = dist[u] + weight
            if new_dist < dist[v]:
                dist[v] = new_dist
                heapq.heappush(min_heap, (new_dist, v))  # May add duplicate

    return dist

wg = WeightedGraph(5)
wg.add_edge(0, 1, 10); wg.add_edge(0, 2, 3)
wg.add_edge(1, 3, 2);  wg.add_edge(2, 1, 4)
wg.add_edge(2, 3, 8);  wg.add_edge(2, 4, 2)
wg.add_edge(3, 4, 5);  wg.add_edge(4, 3, 1)
print(dijkstra(wg, 0))  # Output: {0: 0, 1: 7, 2: 3, 3: 9, 4: 5}
```

### With Path Reconstruction

```python
def dijkstra_with_path(graph, source, target):
    dist     = {v: float('inf') for v in range(graph.vertices)}
    prev     = {v: None         for v in range(graph.vertices)}
    dist[source] = 0
    min_heap = [(0, source)]

    while min_heap:
        current_dist, u = heapq.heappop(min_heap)
        if current_dist > dist[u]:
            continue
        for v, weight in graph.get_neighbors(u):
            new_dist = dist[u] + weight
            if new_dist < dist[v]:
                dist[v] = new_dist
                prev[v] = u             # Remember: we came to v from u
                heapq.heappush(min_heap, (new_dist, v))

    # Walk backwards from target to source using prev[]
    path, node = [], target
    while node is not None:
        path.append(node)
        node = prev[node]
    path.reverse()
    return dist[target], path

cost, path = dijkstra_with_path(wg, 0, 3)
print(f"Cost: {cost}, Path: {path}")    # Output: Cost: 9, Path: [0, 2, 1, 3]
```

### Limitations
- Does NOT work with negative weight edges (use Bellman-Ford instead)
- Only finds one shortest path (there may be ties)

---

## 19. Bellman-Ford — Negative Weights

### What is the Problem?

Find shortest paths from a source, but the graph may have **negative weight edges**.

Real-world analogy: Some roads have a "toll rebate" — driving on them actually reduces your total cost. Dijkstra breaks in this case; Bellman-Ford handles it.

### How It Works

Unlike Dijkstra (greedy), Bellman-Ford uses **dynamic programming**:
1. For a graph with V vertices, the shortest path can have at most V-1 edges
2. Repeat V-1 times: try to improve (relax) EVERY edge
3. On the V-th pass: if any distance still improves, there's a negative cycle (distances would shrink forever)

### Why V-1 Iterations?

In the worst case, the shortest path is a chain of all V vertices with V-1 edges. Each pass guarantees at least one more edge of the true shortest path is "locked in."

```python
def bellman_ford(graph, source):
    dist = {v: float('inf') for v in range(graph.vertices)}
    dist[source] = 0

    # V-1 relaxations
    for _ in range(graph.vertices - 1):
        for u in range(graph.vertices):
            for v, weight in graph.get_neighbors(u):
                if dist[u] != float('inf') and dist[u] + weight < dist[v]:
                    dist[v] = dist[u] + weight

    # V-th relaxation: check for negative cycles
    for u in range(graph.vertices):
        for v, weight in graph.get_neighbors(u):
            if dist[u] != float('inf') and dist[u] + weight < dist[v]:
                return None   # Negative cycle exists!

    return dist

wg2 = WeightedGraph(4)
wg2.add_edge(0, 1, 1); wg2.add_edge(1, 2, 3)
wg2.add_edge(2, 3, 2); wg2.add_edge(0, 2, 10)
print(bellman_ford(wg2, 0))  # Output: {0: 0, 1: 1, 2: 4, 3: 6}
```

### Dijkstra vs Bellman-Ford

| Feature              | Dijkstra         | Bellman-Ford |
|----------------------|------------------|--------------|
| Negative weights     | NO               | YES          |
| Negative cycles      | Cannot detect    | Detects them |
| Time complexity      | O((V+E) log V)   | O(VE)        |
| Works on             | Non-neg weights  | Any weights  |

---

## 20. Floyd-Warshall — All-Pairs Shortest Path

### What is the Problem?

Find the shortest path between **every possible pair** of nodes in the graph. Not just from one source, but from ALL sources to ALL destinations.

Real-world analogy: An airport wants to know: for any two cities in the world, what is the minimum number of connections?

### How It Works

Floyd-Warshall uses dynamic programming with a clever idea:

"Can the path from i to j be improved by going through some intermediate node k?"

We try every possible intermediate node k from 0 to V-1. For each pair (i,j), if `dist[i][k] + dist[k][j] < dist[i][j]`, update it.

### Step-by-Step Example

```
Nodes: 0, 1, 2, 3
Edges: 0-1 (3), 0-3 (7), 1-2 (2), 2-3 (1)

Initial dist matrix:
    0   1   2   3
0 [ 0,  3, inf,  7]
1 [ 3,  0,  2, inf]
2 [inf, 2,  0,  1]
3 [ 7, inf,  1,  0]

After k=0 (route through node 0): no improvements
After k=1: dist[0][2] = min(inf, dist[0][1]+dist[1][2]) = min(inf, 3+2) = 5 ✓
After k=2: dist[0][3] = min(7, dist[0][2]+dist[2][3]) = min(7, 5+1) = 6 ✓
           dist[1][3] = min(inf, 2+1) = 3 ✓
After k=3: some more updates...

Final answer:
    0  1  2  3
0 [ 0, 3, 5, 6]
1 [ 3, 0, 2, 3]
2 [ 5, 2, 0, 1]
3 [ 6, 3, 1, 0]
```

```python
def floyd_warshall(vertices, edges):
    INF  = float('inf')
    dist = [[INF] * vertices for _ in range(vertices)]

    for i in range(vertices):
        dist[i][i] = 0              # Distance to self is 0

    for u, v, w in edges:
        dist[u][v] = w
        dist[v][u] = w              # Remove this line for directed graphs

    for k in range(vertices):       # Try each intermediate node
        for i in range(vertices):
            for j in range(vertices):
                if dist[i][k] + dist[k][j] < dist[i][j]:
                    dist[i][j] = dist[i][k] + dist[k][j]

    return dist

edges = [(0,1,3),(0,3,7),(1,2,2),(2,3,1)]
dist  = floyd_warshall(4, edges)
for row in dist:
    print(row)
# [0, 3, 5, 6]
# [3, 0, 2, 3]
# [5, 2, 0, 1]
# [6, 3, 1, 0]
```

---

## 21. Prim's Algorithm — Minimum Spanning Tree

### What is an MST?

A **Minimum Spanning Tree** connects all vertices using exactly V-1 edges with the minimum total weight, and with no cycles.

Real-world analogy: A city wants to lay electrical cable to connect all neighbourhoods. What is the cheapest way to connect them all, with no redundant connections?

### Prim's Approach — Grow From One Node

Prim's is like growing a tree:
1. Start with any node
2. Always add the cheapest edge that connects a node inside the tree to a node outside it
3. Repeat until all nodes are included

This is similar to Dijkstra. Uses a min-heap to always pick the cheapest available edge.

```python
def prims_mst(graph):
    in_mst    = set()
    min_heap  = [(0, 0, -1)]     # (edge_weight, vertex, parent)
    mst_cost  = 0
    mst_edges = []

    while min_heap and len(in_mst) < graph.vertices:
        weight, u, parent = heapq.heappop(min_heap)

        if u in in_mst:          # Already in MST, skip
            continue

        in_mst.add(u)
        mst_cost += weight
        if parent != -1:
            mst_edges.append((parent, u, weight))

        for v, w in graph.get_neighbors(u):
            if v not in in_mst:
                heapq.heappush(min_heap, (w, v, u))

    return mst_cost, mst_edges

wg3 = WeightedGraph(5)
wg3.add_edge(0, 1, 2); wg3.add_edge(0, 3, 6)
wg3.add_edge(1, 2, 3); wg3.add_edge(1, 3, 8)
wg3.add_edge(1, 4, 5); wg3.add_edge(2, 4, 7)
wg3.add_edge(3, 4, 9)

cost, edges = prims_mst(wg3)
print(f"MST Cost: {cost}")    # Output: MST Cost: 16
print(f"MST Edges: {edges}")  # Output: [(0,1,2),(1,2,3),(1,4,5),(0,3,6)]
```

---

## 22. Kruskal's Algorithm — Minimum Spanning Tree

### How Kruskal's Differs from Prim's

Prim's grows from one starting node. Kruskal's is global — it sorts ALL edges and greedily picks the cheapest edge that does NOT create a cycle.

To check "does adding this edge create a cycle?", Kruskal's uses a data structure called **Union-Find (Disjoint Set Union)**.

### Union-Find — The Key Data Structure

Union-Find tracks which vertices are in the same connected component. Two key operations:
- **find(x)**: returns the root representative of x's component
- **union(x, y)**: merges x's component with y's component; returns False if they're already in the same component (adding this edge would create a cycle)

Path compression (in find) and union by rank make both operations nearly O(1).

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))    # Each node is its own root initially
        self.rank   = [0] * n

    def find(self, x):
        # Path compression: make every node point directly to root
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]

    def union(self, x, y):
        px, py = self.find(x), self.find(y)
        if px == py:
            return False          # Same component — would create a cycle!
        # Union by rank: smaller tree merges under larger tree
        if self.rank[px] < self.rank[py]:
            px, py = py, px
        self.parent[py] = px
        if self.rank[px] == self.rank[py]:
            self.rank[px] += 1
        return True

def kruskals_mst(vertices, edges):
    edges.sort()                   # Sort by weight — greediest first
    uf        = UnionFind(vertices)
    mst_cost  = 0
    mst_edges = []

    for weight, u, v in edges:
        if uf.union(u, v):         # If no cycle, add to MST
            mst_cost += weight
            mst_edges.append((u, v, weight))

    return mst_cost, mst_edges

# edges format: (weight, u, v)
raw_edges = [(2,0,1),(6,0,3),(3,1,2),(8,1,3),(5,1,4),(7,2,4),(9,3,4)]
cost, mst = kruskals_mst(5, raw_edges)
print(f"MST Cost: {cost}")    # Output: MST Cost: 16
```

### Prim's vs Kruskal's

| Feature           | Prim's                    | Kruskal's           |
|-------------------|---------------------------|---------------------|
| Approach          | Grow from one vertex      | Sort all edges      |
| Data structure    | Min-heap                  | Union-Find          |
| Best for          | Dense graphs (many edges) | Sparse graphs       |
| Time complexity   | O((V+E) log V)            | O(E log E)          |

---

## 23. Complexity Cheat Sheet

| Algorithm                | Time             | Space  | Key Condition              |
|--------------------------|------------------|--------|----------------------------|
| Tree Height              | O(n)             | O(h)   | h = height of tree         |
| Tree Diameter            | O(n)             | O(h)   |                            |
| Pre/In/Post-order        | O(n)             | O(h)   |                            |
| Level-order BFS          | O(n)             | O(w)   | w = max width              |
| LCA                      | O(n)             | O(h)   |                            |
| Check Balanced           | O(n)             | O(h)   |                            |
| Path Sum                 | O(n)             | O(h)   |                            |
| Graph BFS                | O(V + E)         | O(V)   | Shortest path unweighted   |
| Graph DFS                | O(V + E)         | O(V)   |                            |
| Cycle Detection          | O(V + E)         | O(V)   |                            |
| Topological Sort         | O(V + E)         | O(V)   | DAG only                   |
| Connected Components     | O(V + E)         | O(V)   |                            |
| Bipartite Check          | O(V + E)         | O(V)   |                            |
| Dijkstra (heap)          | O((V+E) log V)   | O(V)   | No negative weights        |
| Bellman-Ford             | O(VE)            | O(V)   | Handles negative weights   |
| Floyd-Warshall           | O(V^3)           | O(V^2) | All-pairs shortest path    |
| Prim's MST               | O((V+E) log V)   | O(V)   | Dense graphs               |
| Kruskal's MST            | O(E log E)       | O(V)   | Sparse graphs              |

---

*End of Guide — Good luck with interviews!*
