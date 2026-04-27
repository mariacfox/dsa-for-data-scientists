# Chapter 5 — Trees & Graphs

*Built with [Claude](https://claude.ai) by Anthropic.*

---

## Key Concepts

- **Binary trees** — nodes, depth, height, traversal
- **DFS** — preorder, inorder, postorder (recursive & iterative)
- **BFS** — level-order traversal using a queue
- **Binary Search Trees** — left < node < right property
- **Graphs** — adjacency list/matrix, directed vs undirected
- Graph DFS & BFS, connected components
- Implicit graphs (grids, word ladder)
- **DAGs & Topological Sort** — Kahn's algorithm, cycle detection

---

> **DS/MLE Interview Relevance: HIGH (trees) / MEDIUM (graphs)** — Decision trees make tree traversal unusually intuitive for data scientists: DFS = depth-first recursion through a tree, BFS = processing level by level. Max Depth, Level Order, and Number of Islands are frequently tested. Right Side View and Clone Graph are lower priority — focus your energy on tree DFS/BFS and the islands problem.

> **Coming from DS/ML:** If you've used `sklearn`'s `DecisionTreeClassifier`, you've worked with a tree — the model is literally a tree of decisions. The interview version uses explicit `TreeNode` objects with `.left` and `.right` pointers instead of sklearn's internal arrays, but the traversal logic (go deeper, then come back up) is the same. Graphs are more general: think user-item recommendation graphs, knowledge graphs, or connected components in an image — `scipy.ndimage.label` is a graph algorithm.

---

## Trees — The Mental Model

A tree is a **hierarchical graph with no cycles** and a designated root. In binary trees, each node has at most two children: `left` and `right`.

```
    1          ← root (depth 0)
   / \
  2   3        ← depth 1
 / \
4   5          ← depth 2 (leaves)
```

Key vocabulary: **depth** = distance from root, **height** = longest path to a leaf, **leaf** = node with no children.

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

---

## Tree Traversals

**DFS — 3 orderings** (all O(n) time, O(h) space where h = height):

```python
# Preorder: root → left → right (good for copying/serializing)
def preorder(node):
    if not node: return
    visit(node)
    preorder(node.left)
    preorder(node.right)

# Inorder: left → root → right (gives sorted order for BST!)
def inorder(node):
    if not node: return
    inorder(node.left)
    visit(node)
    inorder(node.right)

# Postorder: left → right → root (good for deletion, bottom-up info)
def postorder(node):
    if not node: return
    postorder(node.left)
    postorder(node.right)
    visit(node)
```

**BFS — level-order** (O(n) time, O(n) space):

```python
from collections import deque
def bfs(root):
    if not root: return
    queue = deque([root])
    while queue:
        node = queue.popleft()
        visit(node)
        if node.left: queue.append(node.left)
        if node.right: queue.append(node.right)
```

**Rule of thumb:** Use DFS when you need info from subtrees (bottom-up). Use BFS when you need level-by-level info or shortest path.

---

## Height of a Binary Tree

**Height** = the number of edges on the longest path from root to a leaf. (Some definitions count nodes instead of edges — LeetCode uses nodes, so `height(single node) = 1`, `height(None) = 0`.)

This is a classic bottom-up DFS: you can't know a node's height until you know its children's heights. The recursion naturally handles it.

```python
def height(node):
    if not node:
        return 0                              # base case: empty tree has height 0
    left_h  = height(node.left)
    right_h = height(node.right)
    return 1 + max(left_h, right_h)          # current node adds 1
```

**Trace on a small tree:**
```
    1
   / \
  2   3
 /
4
```
- `height(4)` → `1 + max(0, 0)` = 1
- `height(2)` → `1 + max(1, 0)` = 2
- `height(3)` → `1 + max(0, 0)` = 1
- `height(1)` → `1 + max(2, 1)` = **3**

**Pattern:** any problem asking you to compute something bottom-up per node uses this same skeleton — compute left, compute right, combine, return. Height is the simplest version; balanced tree check and diameter use the exact same structure.

**DS connection:** `sklearn`'s `DecisionTreeClassifier` exposes `.get_depth()` — that's this function. A deeper tree = more splits = more risk of overfitting, which is why `max_depth` is a key hyperparameter.

---

## Depth of a Node

**Depth** = distance from the root down to a given node. Root has depth 0. The opposite direction from height — you pass depth *down* as a parameter rather than returning it *up*.

```python
def depth(node, target, current_depth=0):
    if not node:
        return -1                                        # not found
    if node.val == target:
        return current_depth
    left  = depth(node.left,  target, current_depth + 1)
    right = depth(node.right, target, current_depth + 1)
    return left if left != -1 else right
```

**Height vs depth — the key distinction:**

| | Direction | Computed | Use when... |
|--|-----------|----------|-------------|
| **Height** | bottom-up | returned from children | "how tall is this subtree?" |
| **Depth** | top-down | passed as parameter | "how far is this node from the root?" |

In practice, most LeetCode tree problems use height (bottom-up). Depth comes up in level-order problems (BFS naturally gives you depth via level count) and path problems where you track how deep you've gone.

**BFS gives depth for free** — the level number in a BFS traversal is the depth of every node on that level:

```python
from collections import deque
def node_depths(root):
    queue = deque([(root, 0)])          # (node, depth)
    while queue:
        node, d = queue.popleft()
        print(f"node {node.val} is at depth {d}")
        if node.left:  queue.append((node.left,  d + 1))
        if node.right: queue.append((node.right, d + 1))
```

### LeetCode Problems — Tree DFS

| # | Problem | Key Insight |
|---|---------|-------------|
| [104](https://leetcode.com/problems/maximum-depth-of-binary-tree/) | Maximum Depth of Binary Tree | `depth(node) = 1 + max(depth(left), depth(right))`. Base case: `None → 0`. Maps to `sklearn.tree.get_depth()` and PyTorch computation graph depth. |
| [112](https://leetcode.com/problems/path-sum/) | Path Sum | DFS tracking a running sum; return `True` if a leaf is reached with exactly 0 remaining. Classic bottom-up information flow. |

### LeetCode Problems — Tree BFS

> **DS/MLE focus:** LC 102 (level order) is very commonly tested and worth mastering. LC 199 is a variant — understand 102 first; 199 follows naturally.

| # | Problem | Key Insight |
|---|---------|-------------|
| [102](https://leetcode.com/problems/binary-tree-level-order-traversal/) | Binary Tree Level Order Traversal | Queue-based BFS; process exactly `len(queue)` nodes per level before moving to the next. Maps to layer-by-layer NN processing. |
| [199](https://leetcode.com/problems/binary-tree-right-side-view/) | Binary Tree Right Side View | Same level-order BFS; take the last node of each level. |

---

## Iterative DFS

Recursive DFS uses the call stack implicitly. Iterative DFS uses an explicit stack — useful when recursion depth is a concern.

```python
# Iterative preorder (root → left → right)
def preorder_iterative(root):
    if not root: return []
    stack, result = [root], []
    while stack:
        node = stack.pop()
        result.append(node.val)
        if node.right: stack.append(node.right)  # right first so left is processed first
        if node.left: stack.append(node.left)
    return result
```

Key insight: push **right before left** so left is on top and gets popped first.

---

## Path Tracking in DFS

```python
def dfs(node, path, result):
    if not node:
        return
    path.append(node.val)
    if not node.left and not node.right:  # leaf
        result.append(list(path))          # snapshot — not a reference!
    dfs(node.left, path, result)
    dfs(node.right, path, result)
    path.pop()                             # backtrack
```

`result.append(path)` appends a reference that gets mutated. Always use `list(path)` or `path[:]` to snapshot.

---

## Binary Search Trees (BST)

The BST property: `left subtree < node.val < right subtree`. **Inorder traversal gives a sorted sequence.** Search/insert/delete are O(log n) on balanced trees.

```python
def search(node, target):
    if not node: return False
    if node.val == target: return True
    if target < node.val: return search(node.left, target)
    return search(node.right, target)
```

---

## Graphs

A graph is a set of **nodes (vertices)** connected by **edges**. Trees are a special case (connected, acyclic).

- **Directed** — edges have direction (A→B ≠ B→A)
- **Undirected** — edges go both ways
- **Weighted** — edges have costs

**Adjacency list** — standard representation:

```python
graph = {
    0: [1, 2],
    1: [0, 3],
    2: [0],
    3: [1]
}
```

**Graph DFS** — use a `visited` set to avoid revisiting:

```python
def dfs(node, graph, visited):
    visited.add(node)
    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs(neighbor, graph, visited)
```

**Graph BFS** — finds shortest path (unweighted):

```python
from collections import deque
def bfs(start, graph):
    visited = {start}
    queue = deque([start])
    while queue:
        node = queue.popleft()
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
```

**Grid as implicit graph** — treat each cell as a node, neighbors are up/down/left/right:

```python
directions = [(0,1),(0,-1),(1,0),(-1,0)]
for dr, dc in directions:
    nr, nc = r + dr, c + dc
    if 0 <= nr < rows and 0 <= nc < cols:
        # valid neighbor
```

---

## Connected Components

To process every node in a disconnected graph, iterate over all nodes and run DFS/BFS from any unvisited one:

```python
def count_components(n, edges):
    graph = defaultdict(list)
    for u, v in edges:
        graph[u].append(v)
        graph[v].append(u)

    visited = set()
    components = 0

    def dfs(node):
        visited.add(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                dfs(neighbor)

    for node in range(n):
        if node not in visited:
            dfs(node)
            components += 1

    return components
```

Pattern: **outer loop over all nodes → inner DFS/BFS → count how many times you start fresh**.

### LeetCode Problems — Graph DFS/BFS

> **DS/MLE focus:** LC 200 (Number of Islands) is very commonly tested and maps directly to image segmentation and connected component labeling — high priority. LC 133 (Clone Graph) is lower priority for DS roles; skip if time is short.

| # | Problem | Key Insight |
|---|---------|-------------|
| [200](https://leetcode.com/problems/number-of-islands/) | Number of Islands | Flood-fill DFS/BFS on a grid. Each unvisited `'1'` starts a new island; mark all connected `'1'`s visited. Maps directly to `scipy.ndimage.label` for connected component labeling. |
| [133](https://leetcode.com/problems/clone-graph/) | Clone Graph | BFS with a `seen` dict mapping `original → clone`. Create clone on first visit; reuse on revisit. Maps to deep-copying a computation graph. |

---

## Implicit Graphs — Word Ladder

Implicit graphs aren't given to you — the graph is defined by a transformation rule. Nodes are states; edges exist between states that differ by one valid transformation.

```python
from collections import deque

def ladderLength(beginWord, endWord, wordList):
    word_set = set(wordList)
    queue = deque([(beginWord, 1)])
    visited = {beginWord}

    while queue:
        word, steps = queue.popleft()
        for i in range(len(word)):
            for c in 'abcdefghijklmnopqrstuvwxyz':
                next_word = word[:i] + c + word[i+1:]
                if next_word == endWord:
                    return steps + 1
                if next_word in word_set and next_word not in visited:
                    visited.add(next_word)
                    queue.append((next_word, steps + 1))
    return 0
```

BFS on implicit graphs finds the **shortest path** (fewest transformations). Grid traversal is also an implicit graph.

---

## Topological Sort (DAGs)

A **DAG** (Directed Acyclic Graph) is a directed graph with no cycles. Topological sort produces a linear ordering of nodes such that for every directed edge A → B, A appears before B in the output.

**DS parallel:** Every time you define tasks in Airflow, dbt, or MLflow, the orchestrator runs a topological sort to determine execution order. When you get a `CycleError` in dbt or a circular dependency warning in Airflow, it means topological sort failed — there's no valid ordering.

### Kahn's Algorithm (BFS-based)

The intuition: **tasks with no remaining dependencies go first.** Process them, reduce the dependency count of their dependents, and repeat.

1. Count **in-degree** (number of incoming edges) for every node
2. Enqueue all nodes with in-degree 0 — these have no prerequisites
3. BFS: pop a node, add it to the result, decrement in-degree of its neighbors; if a neighbor hits 0, enqueue it
4. If result contains all n nodes → valid order. If not → cycle exists.

```python
from collections import deque, defaultdict

def topological_sort(n, edges):
    """
    n: number of nodes (0..n-1)
    edges: list of (u, v) — u must come before v
    Returns: valid ordering, or [] if a cycle is detected
    """
    graph = defaultdict(list)
    in_degree = [0] * n

    for u, v in edges:
        graph[u].append(v)
        in_degree[v] += 1

    queue = deque(i for i in range(n) if in_degree[i] == 0)
    order = []

    while queue:
        node = queue.popleft()
        order.append(node)
        for neighbor in graph[node]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)

    return order if len(order) == n else []  # empty list = cycle detected
```

**Cycle detection is free:** any node stuck in a cycle never has its in-degree reach 0, so it never enters the queue. `len(order) < n` catches it.

**Airflow analogy:**
```
raw_data → clean → featurize → train → evaluate
```
- `raw_data` has in-degree 0 → runs first
- `train` waits until `featurize` completes (in-degree decrements to 0)
- Add a back-edge `evaluate → clean` → cycle → no valid order → pipeline fails to start

### LeetCode Problems

> **DS/MLE focus:** Medium priority — not tested as often as islands or tree DFS, but the DAG-to-pipeline analogy makes it unusually intuitive for DS audiences. LC 207 is the standard version; LC 210 is the same algorithm with the order returned instead of a boolean.

| # | Problem | Key Insight |
|---|---------|-------------|
| [207](https://leetcode.com/problems/course-schedule/) | Course Schedule | Topo sort on a prerequisite graph. Return `True` if no cycle (`len(order) == numCourses`). Direct Airflow/dbt analogy: can all tasks complete given dependencies? |
| [210](https://leetcode.com/problems/course-schedule-ii/) | Course Schedule II | Same Kahn's algorithm — return the ordering instead of just True/False. |

---

## DFS vs BFS Decision Framework

**Rule of thumb: BFS = shortest path. DFS = everything else.**

BFS explores nodes in order of distance from the source — all nodes 1 hop away, then 2 hops, and so on. The first time BFS reaches a target, it has taken the fewest possible steps. DFS will find *a* path, not necessarily the shortest one.

**Signal words to watch for:**

| If the problem says... | Use |
|------------------------|-----|
| "minimum number of steps / moves / operations" | BFS |
| "shortest path" or "fewest transformations" | BFS |
| "level order" or "layer by layer" | BFS |
| "count connected components" or "number of islands" | DFS |
| "flood fill" or "mark all connected cells" | DFS |
| "does a path exist" / "can you reach X" | DFS |
| "all possible paths" or "find every combination" | DFS |
| "max/min depth" or "height of tree" | DFS |
| "validate or compare subtrees" | DFS |
| "detect a cycle" | DFS |
| Backtracking or need info from children first | DFS |

DFS dominates tree problems because most tree questions require information from subtrees — and recursion (DFS) naturally propagates that information back up. BFS shines on graph problems where you need minimum distance.

**Space trade-off:** DFS uses O(h) stack space (O(log n) balanced, O(n) skewed). BFS uses O(w) queue space where w = max width — can be O(n) at the bottom of a full tree. For most interview problems this doesn't matter, but it's the right answer if asked.

---

## Complexity Summary

| Operation | Time | Space |
|-----------|------|-------|
| Tree DFS (recursive) | O(n) | O(h) call stack |
| Tree DFS (iterative) | O(n) | O(h) explicit stack |
| Tree BFS | O(n) | O(w) where w = max width |
| Graph DFS/BFS | O(V + E) | O(V) for visited set |
| BST search/insert | O(log n) balanced, O(n) worst | O(h) |

For a **balanced** binary tree, h = O(log n). For a skewed tree, h = O(n).

---

## Watch Outs

- **Always handle `None` base case first** in recursive tree functions.
- **Graph: mark visited BEFORE enqueuing** in BFS — prevents adding the same node to the queue multiple times.
- **BST vs binary tree** — BST allows O(log n) solutions that plain binary tree can't do. Don't miss the BST property.
- **Snapshot paths** — `result.append(list(path))` not `result.append(path)`.

---

## DS/MLE Connections

- **Decision trees in sklearn** are literally binary trees — splitting left/right based on a feature threshold. Inorder traversal of a BST gives sorted values, just like reading decision tree splits in order.
- **DAG-based pipelines** (Airflow, dbt, MLflow) run a topological sort every time they schedule work. A `CycleError` in dbt is literally a failed topo sort. Kahn's algorithm is the mechanism behind "which tasks are ready to run now?"
- **BFS shortest path** = finding the minimum number of preprocessing steps to transform raw data into a target format.
- **Connected components** maps directly to clustering without labels: nodes in the same component are "similar" by some adjacency rule, just like DBSCAN groups points by density connectivity.
- The `visited` set is the same pattern as `df.drop_duplicates()` — tracking what you've already processed.
