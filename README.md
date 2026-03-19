# Trees & Graphs — Complete Revision Guide
> All implementations follow the same style as your original code.

---

## Table of Contents
1. [TreeNode & Graph Classes](#1-treenode--graph-classes)
2. [Tree — Height](#2-tree--height)
3. [Tree — Diameter](#3-tree--diameter)
4. [Tree — Traversals](#4-tree--traversals)
5. [Tree — Lowest Common Ancestor](#5-tree--lowest-common-ancestor)
6. [Tree — Check Balanced](#6-tree--check-balanced)
7. [Tree — Mirror / Invert](#7-tree--mirror--invert)
8. [Tree — Path Sum](#8-tree--path-sum)
9. [Tree — Count Nodes & Leaves](#9-tree--count-nodes--leaves)
10. [Tree — Max Path Sum](#10-tree--max-path-sum)
11. [Graph — BFS](#11-graph--bfs)
12. [Graph — DFS](#12-graph--dfs)
13. [Graph — Detect Cycle (Undirected)](#13-graph--detect-cycle-undirected)
14. [Graph — Detect Cycle (Directed)](#14-graph--detect-cycle-directed)
15. [Graph — Topological Sort](#15-graph--topological-sort)
16. [Graph — Connected Components](#16-graph--connected-components)
17. [Graph — Bipartite Check](#17-graph--bipartite-check)
18. [Weighted Graph — Dijkstra Shortest Path](#18-weighted-graph--dijkstra-shortest-path)
19. [Weighted Graph — Bellman-Ford](#19-weighted-graph--bellman-ford)
20. [Weighted Graph — Floyd-Warshall](#20-weighted-graph--floyd-warshall)
21. [Weighted Graph — Prims MST](#21-weighted-graph--prims-mst)
22. [Weighted Graph — Kruskals MST](#22-weighted-graph--kruskals-mst)
23. [Complexity Cheat Sheet](#23-complexity-cheat-sheet)

---

## 1. TreeNode & Graph Classes

```python
class TreeNode:
    def __init__(self, val, left=None, right=None):
        self.val   = val
        self.left  = left
        self.right = right

class Graph:
    def __init__(self, vertices):
        self.vertices       = vertices
        self.adjacency_list = {i: [] for i in range(vertices)}

    def add_edge(self, u, v):
        self.adjacency_list[u].append(v)
        self.adjacency_list[v].append(u)

    def get_neighbors(self, vertex):
        return self.adjacency_list[vertex]

class WeightedGraph:
    def __init__(self, vertices):
        self.vertices       = vertices
        self.adjacency_list = {i: [] for i in range(vertices)}

    def add_edge(self, u, v, weight):
        self.adjacency_list[u].append((v, weight))
        self.adjacency_list[v].append((u, weight))

    def get_neighbors(self, vertex):
        return self.adjacency_list[vertex]

#          1
#        /   \
#       2     3
#      / \   / \
#     4   5 6   7
tree1 = TreeNode(1,
            TreeNode(2, TreeNode(4), TreeNode(5)),
            TreeNode(3, TreeNode(6), TreeNode(7)))
```

---

## 2. Tree — Height

```python
def height_of_tree(node):
    if node is None:
        return 0
    left_height  = height_of_tree(node.left)
    right_height = height_of_tree(node.right)
    return max(left_height, right_height) + 1

print(height_of_tree(tree1))   # Output: 3
```

**How it works:** Recursively find the height of left and right subtrees, return the max + 1.

---

## 3. Tree — Diameter

The diameter is the longest path between any two nodes (may or may not pass through the root).

```python
def diameter_of_tree(node):
    max_diameter = [0]

    def height(node):
        if node is None:
            return 0
        left_h  = height(node.left)
        right_h = height(node.right)
        max_diameter[0] = max(max_diameter[0], left_h + right_h)
        return max(left_h, right_h) + 1

    height(node)
    return max_diameter[0]

print(diameter_of_tree(tree1))  # Output: 4
```

---

## 4. Tree — Traversals

### Pre-order (Root Left Right)
```python
def preorder_traversal(node):
    if node is None:
        return []
    return [node.val] + preorder_traversal(node.left) + preorder_traversal(node.right)

print(preorder_traversal(tree1))  # Output: [1, 2, 4, 5, 3, 6, 7]
```

### In-order (Left Root Right)
```python
def inorder_traversal(node):
    if node is None:
        return []
    return inorder_traversal(node.left) + [node.val] + inorder_traversal(node.right)

print(inorder_traversal(tree1))   # Output: [4, 2, 5, 1, 6, 3, 7]
```

### Post-order (Left Right Root)
```python
def postorder_traversal(node):
    if node is None:
        return []
    return postorder_traversal(node.left) + postorder_traversal(node.right) + [node.val]

print(postorder_traversal(tree1)) # Output: [4, 5, 2, 6, 7, 3, 1]
```

### Level-order BFS Traversal
```python
from collections import deque

def level_order_traversal(node):
    if node is None:
        return []
    result = []
    queue  = deque([node])
    while queue:
        current = queue.popleft()
        result.append(current.val)
        if current.left:
            queue.append(current.left)
        if current.right:
            queue.append(current.right)
    return result

print(level_order_traversal(tree1))  # Output: [1, 2, 3, 4, 5, 6, 7]
```

---

## 5. Tree — Lowest Common Ancestor

```python
def lowest_common_ancestor(node, p, q):
    if node is None or node.val == p or node.val == q:
        return node
    left  = lowest_common_ancestor(node.left,  p, q)
    right = lowest_common_ancestor(node.right, p, q)
    if left and right:
        return node
    return left if left else right

lca = lowest_common_ancestor(tree1, 4, 5)
print(lca.val)  # Output: 2

lca = lowest_common_ancestor(tree1, 4, 6)
print(lca.val)  # Output: 1
```

---

## 6. Tree — Check Balanced

A tree is height-balanced if for every node, the heights of left and right subtrees differ by at most 1.

```python
def is_balanced(node):
    def check(node):
        if node is None:
            return 0
        left_h  = check(node.left)
        right_h = check(node.right)
        if left_h == -1 or right_h == -1:
            return -1
        if abs(left_h - right_h) > 1:
            return -1
        return max(left_h, right_h) + 1

    return check(node) != -1

print(is_balanced(tree1))  # Output: True

unbalanced = TreeNode(1, TreeNode(2, TreeNode(3)), None)
print(is_balanced(unbalanced))  # Output: False
```

---

## 7. Tree — Mirror / Invert

```python
def mirror_tree(node):
    if node is None:
        return None
    node.left, node.right = mirror_tree(node.right), mirror_tree(node.left)
    return node

mirrored = mirror_tree(TreeNode(1,
                TreeNode(2, TreeNode(4), TreeNode(5)),
                TreeNode(3, TreeNode(6), TreeNode(7))))
print(inorder_traversal(mirrored))  # Output: [7, 3, 6, 1, 5, 2, 4]
```

---

## 8. Tree — Path Sum

Check if there is a root-to-leaf path whose values sum to target.

```python
def has_path_sum(node, target):
    if node is None:
        return False
    if node.left is None and node.right is None:
        return node.val == target
    remaining = target - node.val
    return has_path_sum(node.left, remaining) or has_path_sum(node.right, remaining)

sum_tree = TreeNode(5,
               TreeNode(4, TreeNode(11, TreeNode(7), TreeNode(2)), None),
               TreeNode(8, TreeNode(13), TreeNode(4, None, TreeNode(1))))

print(has_path_sum(sum_tree, 22))  # Output: True  (5+4+11+2)
print(has_path_sum(sum_tree, 99))  # Output: False
```

---

## 9. Tree — Count Nodes & Leaves

```python
def count_nodes(node):
    if node is None:
        return 0
    return 1 + count_nodes(node.left) + count_nodes(node.right)

def count_leaves(node):
    if node is None:
        return 0
    if node.left is None and node.right is None:
        return 1
    return count_leaves(node.left) + count_leaves(node.right)

print(count_nodes(tree1))   # Output: 7
print(count_leaves(tree1))  # Output: 4
```

---

## 10. Tree — Max Path Sum

Find the maximum sum path between any two nodes.

```python
def max_path_sum(node):
    max_sum = [float('-inf')]

    def helper(node):
        if node is None:
            return 0
        left_gain  = max(helper(node.left),  0)
        right_gain = max(helper(node.right), 0)
        price = node.val + left_gain + right_gain
        max_sum[0] = max(max_sum[0], price)
        return node.val + max(left_gain, right_gain)

    helper(node)
    return max_sum[0]

val_tree = TreeNode(-10, TreeNode(9), TreeNode(20, TreeNode(15), TreeNode(7)))
print(max_path_sum(val_tree))  # Output: 42
```

---

## 11. Graph — BFS

```python
def bfs(graph, start):
    visited   = set()
    queue     = [start]
    bfs_order = []

    while queue:
        vertex = queue.pop(0)
        if vertex not in visited:
            visited.add(vertex)
            bfs_order.append(vertex)
            queue.extend(graph.get_neighbors(vertex))

    return bfs_order

g = Graph(5)
g.add_edge(0, 1); g.add_edge(0, 2)
g.add_edge(1, 3); g.add_edge(1, 4)
print(bfs(g, 0))  # Output: [0, 1, 2, 3, 4]
```

---

## 12. Graph — DFS

### Iterative
```python
def dfs_iterative(graph, start):
    visited   = set()
    stack     = [start]
    dfs_order = []

    while stack:
        vertex = stack.pop()
        if vertex not in visited:
            visited.add(vertex)
            dfs_order.append(vertex)
            for neighbor in reversed(graph.get_neighbors(vertex)):
                stack.append(neighbor)

    return dfs_order

print(dfs_iterative(g, 0))  # Output: [0, 1, 3, 4, 2]
```

### Recursive
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

---

## 13. Graph — Detect Cycle (Undirected)

```python
def has_cycle_undirected(graph):
    visited = set()

    def dfs(vertex, parent):
        visited.add(vertex)
        for neighbor in graph.get_neighbors(vertex):
            if neighbor not in visited:
                if dfs(neighbor, vertex):
                    return True
            elif neighbor != parent:
                return True
        return False

    for v in range(graph.vertices):
        if v not in visited:
            if dfs(v, -1):
                return True
    return False

cycle_graph = Graph(4)
cycle_graph.add_edge(0, 1); cycle_graph.add_edge(1, 2)
cycle_graph.add_edge(2, 3); cycle_graph.add_edge(3, 0)
print(has_cycle_undirected(cycle_graph))  # Output: True
print(has_cycle_undirected(g))            # Output: False
```

---

## 14. Graph — Detect Cycle (Directed)

```python
class DirectedGraph:
    def __init__(self, vertices):
        self.vertices       = vertices
        self.adjacency_list = {i: [] for i in range(vertices)}

    def add_edge(self, u, v):
        self.adjacency_list[u].append(v)

    def get_neighbors(self, vertex):
        return self.adjacency_list[vertex]

def has_cycle_directed(graph):
    visited   = set()
    rec_stack = set()

    def dfs(vertex):
        visited.add(vertex)
        rec_stack.add(vertex)
        for neighbor in graph.get_neighbors(vertex):
            if neighbor not in visited:
                if dfs(neighbor):
                    return True
            elif neighbor in rec_stack:
                return True
        rec_stack.discard(vertex)
        return False

    for v in range(graph.vertices):
        if v not in visited:
            if dfs(v):
                return True
    return False

dg = DirectedGraph(4)
dg.add_edge(0, 1); dg.add_edge(1, 2)
dg.add_edge(2, 3); dg.add_edge(3, 1)
print(has_cycle_directed(dg))  # Output: True
```

---

## 15. Graph — Topological Sort

Only for Directed Acyclic Graphs (DAGs).

```python
def topological_sort(graph):
    visited = set()
    stack   = []

    def dfs(vertex):
        visited.add(vertex)
        for neighbor in graph.get_neighbors(vertex):
            if neighbor not in visited:
                dfs(neighbor)
        stack.append(vertex)

    for v in range(graph.vertices):
        if v not in visited:
            dfs(v)

    return stack[::-1]

dag = DirectedGraph(6)
dag.add_edge(5, 2); dag.add_edge(5, 0)
dag.add_edge(4, 0); dag.add_edge(4, 1)
dag.add_edge(2, 3); dag.add_edge(3, 1)
print(topological_sort(dag))   # Output: [5, 4, 2, 3, 1, 0]
```

---

## 16. Graph — Connected Components

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
        if v not in visited:
            component = []
            dfs(v, component)
            components.append(component)

    return components

disconnected = Graph(7)
disconnected.add_edge(0, 1); disconnected.add_edge(1, 2)
disconnected.add_edge(3, 4)
print(connected_components(disconnected))
# Output: [[0, 1, 2], [3, 4], [5], [6]]
```

---

## 17. Graph — Bipartite Check

A graph is bipartite if you can colour vertices with 2 colours so no edge connects same-colour vertices.

```python
def is_bipartite(graph):
    colour = {}

    def bfs(start):
        queue = [start]
        colour[start] = 0
        while queue:
            v = queue.pop(0)
            for neighbor in graph.get_neighbors(v):
                if neighbor not in colour:
                    colour[neighbor] = 1 - colour[v]
                    queue.append(neighbor)
                elif colour[neighbor] == colour[v]:
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
print(is_bipartite(bipartite_g))  # Output: True

odd_cycle = Graph(3)
odd_cycle.add_edge(0, 1); odd_cycle.add_edge(1, 2); odd_cycle.add_edge(2, 0)
print(is_bipartite(odd_cycle))    # Output: False
```

---

## 18. Weighted Graph — Dijkstra Shortest Path

Finds shortest path from source to all other vertices. Works only with non-negative weights.

```python
import heapq

def dijkstra(graph, source):
    dist = {v: float('inf') for v in range(graph.vertices)}
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
                heapq.heappush(min_heap, (new_dist, v))

    return dist

wg = WeightedGraph(5)
wg.add_edge(0, 1, 10); wg.add_edge(0, 2, 3)
wg.add_edge(1, 3, 2);  wg.add_edge(2, 1, 4)
wg.add_edge(2, 3, 8);  wg.add_edge(2, 4, 2)
wg.add_edge(3, 4, 5);  wg.add_edge(4, 3, 1)
print(dijkstra(wg, 0))
# Output: {0: 0, 1: 7, 2: 3, 3: 9, 4: 5}
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
                prev[v] = u
                heapq.heappush(min_heap, (new_dist, v))

    path, node = [], target
    while node is not None:
        path.append(node)
        node = prev[node]
    path.reverse()
    return dist[target], path

cost, path = dijkstra_with_path(wg, 0, 3)
print(f"Cost: {cost}, Path: {path}")  # Output: Cost: 9, Path: [0, 2, 1, 3]
```

---

## 19. Weighted Graph — Bellman-Ford

Handles negative weight edges. Detects negative cycles.

```python
def bellman_ford(graph, source):
    dist = {v: float('inf') for v in range(graph.vertices)}
    dist[source] = 0

    for _ in range(graph.vertices - 1):
        for u in range(graph.vertices):
            for v, weight in graph.get_neighbors(u):
                if dist[u] + weight < dist[v]:
                    dist[v] = dist[u] + weight

    for u in range(graph.vertices):
        for v, weight in graph.get_neighbors(u):
            if dist[u] + weight < dist[v]:
                return None  # negative cycle detected

    return dist

wg2 = WeightedGraph(4)
wg2.add_edge(0, 1, 1); wg2.add_edge(1, 2, 3)
wg2.add_edge(2, 3, 2); wg2.add_edge(0, 2, 10)
print(bellman_ford(wg2, 0))  # Output: {0: 0, 1: 1, 2: 4, 3: 6}
```

---

## 20. Weighted Graph — Floyd-Warshall

All-pairs shortest path. O(V cubed).

```python
def floyd_warshall(vertices, edges):
    INF  = float('inf')
    dist = [[INF] * vertices for _ in range(vertices)]
    for i in range(vertices):
        dist[i][i] = 0
    for u, v, w in edges:
        dist[u][v] = w
        dist[v][u] = w

    for k in range(vertices):
        for i in range(vertices):
            for j in range(vertices):
                if dist[i][k] + dist[k][j] < dist[i][j]:
                    dist[i][j] = dist[i][k] + dist[k][j]

    return dist

edges = [(0,1,3),(0,3,7),(1,2,2),(2,3,1)]
dist  = floyd_warshall(4, edges)
for row in dist:
    print(row)
# Output:
# [0, 3, 5, 6]
# [3, 0, 2, 3]
# [5, 2, 0, 1]
# [6, 3, 1, 0]
```

---

## 21. Weighted Graph — Prims MST

Finds the Minimum Spanning Tree by growing from a single vertex.

```python
def prims_mst(graph):
    in_mst    = set()
    min_heap  = [(0, 0, -1)]
    mst_cost  = 0
    mst_edges = []

    while min_heap and len(in_mst) < graph.vertices:
        weight, u, parent = heapq.heappop(min_heap)
        if u in in_mst:
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

## 22. Weighted Graph — Kruskals MST

Finds the Minimum Spanning Tree by sorting edges globally and using Union-Find.

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank   = [0] * n

    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]

    def union(self, x, y):
        px, py = self.find(x), self.find(y)
        if px == py:
            return False
        if self.rank[px] < self.rank[py]:
            px, py = py, px
        self.parent[py] = px
        if self.rank[px] == self.rank[py]:
            self.rank[px] += 1
        return True

def kruskals_mst(vertices, edges):
    edges.sort()
    uf        = UnionFind(vertices)
    mst_cost  = 0
    mst_edges = []

    for weight, u, v in edges:
        if uf.union(u, v):
            mst_cost += weight
            mst_edges.append((u, v, weight))

    return mst_cost, mst_edges

raw_edges = [(2,0,1),(6,0,3),(3,1,2),(8,1,3),(5,1,4),(7,2,4),(9,3,4)]
cost, mst = kruskals_mst(5, raw_edges)
print(f"MST Cost: {cost}")    # Output: MST Cost: 16
print(f"MST Edges: {mst}")    # Output: [(0,1,2),(1,2,3),(1,4,5),(0,3,6)]
```

---

## 23. Complexity Cheat Sheet

| Algorithm                | Time            | Space | Notes                       |
|--------------------------|-----------------|-------|-----------------------------|
| Tree Height              | O(n)            | O(h)  | h = height of tree          |
| Tree Diameter            | O(n)            | O(h)  |                             |
| Pre/In/Post-order        | O(n)            | O(h)  |                             |
| Level-order BFS          | O(n)            | O(w)  | w = max width               |
| LCA                      | O(n)            | O(h)  |                             |
| Check Balanced           | O(n)            | O(h)  |                             |
| Path Sum                 | O(n)            | O(h)  |                             |
| Graph BFS                | O(V + E)        | O(V)  |                             |
| Graph DFS                | O(V + E)        | O(V)  |                             |
| Cycle Detection          | O(V + E)        | O(V)  |                             |
| Topological Sort         | O(V + E)        | O(V)  | DAG only                    |
| Connected Components     | O(V + E)        | O(V)  |                             |
| Bipartite Check          | O(V + E)        | O(V)  |                             |
| Dijkstra (heap)          | O((V+E) log V)  | O(V)  | No negative weights         |
| Bellman-Ford             | O(VE)           | O(V)  | Handles negative weights    |
| Floyd-Warshall           | O(V^3)          | O(V2) | All-pairs shortest path     |
| Prim's MST (heap)        | O((V+E) log V)  | O(V)  | Dense graphs                |
| Kruskal's MST            | O(E log E)      | O(V)  | Sparse graphs               |

---

*End of Guide — Happy Coding!*
