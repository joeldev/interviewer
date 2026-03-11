# Round 10: Graphs — "Delivery Network Connectivity"

## Metadata
- **Topic:** Graphs (adjacency lists, traversal, cycle detection, bridges)
- **Difficulty:** 4/5
- **Estimated Time:** 55-60 minutes
- **Key Concepts:** Adjacency list construction, BFS/DFS traversal, connected components, cycle detection with DFS coloring, bridge finding (simplified Tarjan's)

## Problem Prompt

> DoorDash operates a delivery network that can be modeled as a graph. Nodes represent locations — restaurants, customer addresses, and distribution hubs. Edges represent road segments connecting these locations.
>
> We have data about the road network as a list of connections. We need to analyze this network for operational reliability.
>
> Given the road network data and a central hub location, help us understand the connectivity properties of our delivery network.

*Read this prompt verbatim. Do NOT volunteer whether the graph is directed or undirected, weighted or unweighted, or whether it might be disconnected. Let the candidate ask.*

## Clarifying Questions & Hidden Constraints

The candidate **should** ask these questions. Award communication points for each one asked unprompted.

| # | Question | Answer |
|---|----------|--------|
| 1 | Are the roads one-way or two-way? | For Stage 1 and Stage 3, assume **undirected** (two-way roads). For Stage 2, we will switch to a **directed** version (one-way dependencies). |
| 2 | Are edges weighted (e.g., distance or travel time)? | No. All edges are unweighted for this problem. |
| 3 | Can the graph be disconnected? | Yes — some locations might not be reachable from the hub. That is exactly what we want to detect. |
| 4 | How are locations represented? | Locations are integers from `0` to `n-1`. The hub is given as a specific node ID. |
| 5 | Can there be duplicate edges or self-loops? | No. The input is clean — no duplicates, no self-loops. |
| 6 | How large can the network be? | Up to 100,000 nodes and 500,000 edges. Solution must be O(V+E). |

## Skeleton Code

```python
from collections import defaultdict, deque
from typing import List, Tuple, Set


def build_adjacency_list(n: int, edges: List[Tuple[int, int]]) -> dict:
    """
    Build an adjacency list from an edge list.

    Args:
        n: Number of nodes (0 to n-1).
        edges: List of (u, v) tuples representing roads.

    Returns:
        Adjacency list as a dict mapping node -> list of neighbors.
    """
    pass


def all_reachable(adj: dict, n: int, hub: int) -> Tuple[bool, Set[int]]:
    """
    Determine if all locations are reachable from the hub.

    Args:
        adj: Adjacency list (undirected).
        n: Total number of nodes.
        hub: The hub node ID.

    Returns:
        (all_reachable: bool, unreachable_nodes: set)
    """
    pass

if __name__ == "__main__":
    # Expected: (True, set()) — fully connected
    adj1 = build_adjacency_list(5, [(0, 1), (1, 2), (2, 3), (3, 4)])
    print(all_reachable(adj1, 5, hub=0))
    # Expected: (False, {3, 4}) — disconnected
    adj2 = build_adjacency_list(5, [(0, 1), (1, 2), (3, 4)])
    print(all_reachable(adj2, 5, hub=0))
    # Expected: (True, set()) — single node
    adj3 = build_adjacency_list(1, [])
    print(all_reachable(adj3, 1, hub=0))
```

## Stage 1: Network Reachability (~20 minutes)

### Prompt Addition

> "First, build the graph from the edge list. Then determine whether every location in the network is reachable from our central hub. If not, identify which locations are unreachable."

### Expected Approach

1. **Build adjacency list:** Iterate through the edge list. For each `(u, v)`, add `v` to `adj[u]` and `u` to `adj[v]` (undirected). Use a `defaultdict(list)`.
2. **BFS or DFS from hub:** Start from the hub node. Track all visited nodes.
3. **Check reachability:** Compare the visited set against all nodes `{0, 1, ..., n-1}`. Any node not visited is unreachable.

**Key insight the candidate should articulate:** This is a connected component check. BFS and DFS both work in O(V+E). The candidate should choose one and justify it (BFS is typically more intuitive for "reachability"; DFS is fine too).

### Solution Code

```python
from collections import defaultdict, deque
from typing import List, Tuple, Set


def build_adjacency_list(n: int, edges: List[Tuple[int, int]]) -> dict:
    adj = defaultdict(list)
    # Ensure all nodes exist as keys even if they have no edges
    for node in range(n):
        adj[node]  # access to create default empty list
    for u, v in edges:
        adj[u].append(v)
        adj[v].append(u)
    return adj


def all_reachable(adj: dict, n: int, hub: int) -> Tuple[bool, Set[int]]:
    visited = set()
    queue = deque([hub])
    visited.add(hub)

    while queue:
        node = queue.popleft()
        for neighbor in adj[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)

    all_nodes = set(range(n))
    unreachable = all_nodes - visited
    return (len(unreachable) == 0, unreachable)
```

### Complexity

- **Time:** O(V + E) — each node and edge visited at most once.
- **Space:** O(V + E) — adjacency list storage plus the visited set and queue.

### Test Cases

```python
# Test 1: Fully connected network
edges1 = [(0, 1), (1, 2), (2, 3), (3, 4)]
adj1 = build_adjacency_list(5, edges1)
result1 = all_reachable(adj1, 5, hub=0)
assert result1 == (True, set()), f"Expected all reachable, got {result1}"

# Test 2: Disconnected network
edges2 = [(0, 1), (1, 2), (3, 4)]
adj2 = build_adjacency_list(5, edges2)
result2 = all_reachable(adj2, 5, hub=0)
assert result2 == (False, {3, 4}), f"Expected {{3, 4}} unreachable, got {result2}"

# Test 3: Single node (hub only, no edges)
adj3 = build_adjacency_list(1, [])
result3 = all_reachable(adj3, 1, hub=0)
assert result3 == (True, set()), f"Expected all reachable (single node), got {result3}"

# Test 4: Hub is an isolated node in a larger graph
edges4 = [(1, 2), (2, 3)]
adj4 = build_adjacency_list(4, edges4)
result4 = all_reachable(adj4, 4, hub=0)
assert result4 == (False, {1, 2, 3}), f"Expected {{1, 2, 3}} unreachable, got {result4}"

print("All Stage 1 tests passed.")
```

## Stage 2: Cycle Detection in Directed Dependencies (~20 minutes)

### Prompt Addition

> "Good. Now consider a different scenario: we have a directed graph representing task dependencies in our order pipeline. For example, 'prepare food' must happen before 'package order,' which must happen before 'dispatch dasher.' We need to detect if there are any circular dependencies that would cause deadlocks.
>
> Given a directed edge list where `(u, v)` means task `u` must complete before task `v`, determine if any cycle exists."

### Starter Code

```python
def has_cycle_directed(n: int, edges: List[Tuple[int, int]]) -> bool:
    """
    Detect if a directed graph contains a cycle.

    Args:
        n: Number of nodes.
        edges: List of directed (u, v) edges.

    Returns:
        True if a cycle exists, False otherwise.
    """
    pass

# --- Stage 2 Test Calls ---
# Expected: False (linear chain, no cycle)
print(has_cycle_directed(4, [(0, 1), (1, 2), (2, 3)]))
# Expected: True (simple cycle)
print(has_cycle_directed(3, [(0, 1), (1, 2), (2, 0)]))
# Expected: False (empty graph)
print(has_cycle_directed(3, []))
```

### Expected Approach

**DFS coloring (three-state approach):**
- **WHITE (0):** Unvisited node.
- **GRAY (1):** Currently in the recursion stack (being explored).
- **BLACK (2):** Fully explored — all descendants processed.

If during DFS from a node we encounter a GRAY neighbor, we have found a back edge, which means a cycle exists.

**Why not use the undirected cycle detection approach?** In a directed graph, an edge `(u, v)` and `(v, w)` does not form a cycle. We need a *back edge* specifically. The candidate should explain this distinction.

**Important:** The graph may not be connected (not all nodes reachable from any single source), so we must attempt DFS from every unvisited node.

### Solution Code

```python
def has_cycle_directed(n: int, edges: List[Tuple[int, int]]) -> bool:
    # Build directed adjacency list
    adj = defaultdict(list)
    for u, v in edges:
        adj[u].append(v)

    WHITE, GRAY, BLACK = 0, 1, 2
    color = [WHITE] * n

    def dfs(node: int) -> bool:
        """Returns True if a cycle is found from this node."""
        color[node] = GRAY
        for neighbor in adj[node]:
            if color[neighbor] == GRAY:
                # Back edge found — cycle detected
                return True
            if color[neighbor] == WHITE:
                if dfs(neighbor):
                    return True
        color[node] = BLACK
        return False

    for node in range(n):
        if color[node] == WHITE:
            if dfs(node):
                return True
    return False
```

### Complexity

- **Time:** O(V + E) — each node and edge visited at most once across all DFS calls.
- **Space:** O(V) — color array and recursion stack.

### Test Cases

```python
# Test 1: No cycle (linear dependency chain)
assert has_cycle_directed(4, [(0, 1), (1, 2), (2, 3)]) == False

# Test 2: Simple cycle
assert has_cycle_directed(3, [(0, 1), (1, 2), (2, 0)]) == True

# Test 3: Self-loop
assert has_cycle_directed(2, [(0, 0)]) == True

# Test 4: Diamond shape — no cycle
# 0 -> 1, 0 -> 2, 1 -> 3, 2 -> 3
assert has_cycle_directed(4, [(0, 1), (0, 2), (1, 3), (2, 3)]) == False

# Test 5: Disconnected graph with a cycle in one component
assert has_cycle_directed(5, [(0, 1), (2, 3), (3, 4), (4, 2)]) == True

# Test 6: Empty graph
assert has_cycle_directed(3, []) == False

print("All Stage 2 tests passed.")
```

## Stage 3: Critical Roads (Bridge Finding) (~15 minutes)

### Prompt Addition

> "Finally, back to our undirected road network. Management wants to know which roads are 'critical' — meaning if that single road is closed for construction, some part of the network becomes completely unreachable from the hub. Find all such critical roads (bridges)."

### Starter Code

```python
def find_bridges(adj: dict, n: int) -> List[Tuple[int, int]]:
    """
    Find all bridge edges whose removal disconnects the graph.

    Args:
        adj: Adjacency list (undirected).
        n: Number of nodes.

    Returns:
        List of (u, v) bridge edges.
    """
    pass

# --- Stage 3 Test Calls ---
# Expected: [(0, 1), (1, 2), (2, 3)] — every edge is a bridge
# adj_t1 = build_adjacency_list(4, [(0, 1), (1, 2), (2, 3)])
# print(sorted(find_bridges(adj_t1, 4)))
# Expected: [] — cycle, no bridges
# adj_t2 = build_adjacency_list(3, [(0, 1), (1, 2), (2, 0)])
# print(find_bridges(adj_t2, 3))
```

### Expected Approach

Use a **DFS-based bridge-finding algorithm** (simplified Tarjan's):

1. Maintain a `discovery_time` for each node (the order in which DFS visits it).
2. Maintain a `low` value for each node — the smallest discovery time reachable from the subtree rooted at that node (through back edges).
3. An edge `(u, v)` is a bridge if and only if `low[v] > disc[u]` — meaning there is no back edge from `v`'s subtree that reaches `u` or any ancestor of `u`.

**If the candidate is stuck on Tarjan's:** An acceptable simplified approach is to remove each edge one at a time and re-check connectivity. This is O(E * (V + E)) — less efficient but demonstrates understanding. Award partial credit.

### Solution Code

```python
def find_bridges(adj: dict, n: int) -> List[Tuple[int, int]]:
    disc = [-1] * n
    low = [-1] * n
    bridges = []
    timer = [0]  # mutable container for closure

    def dfs(u: int, parent: int):
        disc[u] = low[u] = timer[0]
        timer[0] += 1

        for v in adj[u]:
            if disc[v] == -1:
                # Tree edge — v not yet visited
                dfs(v, u)
                low[u] = min(low[u], low[v])

                # Bridge condition: no back edge from v's subtree
                # can reach u or above
                if low[v] > disc[u]:
                    bridges.append((min(u, v), max(u, v)))
            elif v != parent:
                # Back edge — update low
                low[u] = min(low[u], disc[v])

    # Handle potentially disconnected graph
    for node in range(n):
        if disc[node] == -1:
            dfs(node, -1)

    return bridges
```

### Complexity

- **Time:** O(V + E) — standard DFS.
- **Space:** O(V) — disc/low arrays plus recursion stack.

### Test Cases

```python
# Test 1: Linear graph — every edge is a bridge
# 0 - 1 - 2 - 3
edges_t1 = [(0, 1), (1, 2), (2, 3)]
adj_t1 = build_adjacency_list(4, edges_t1)
bridges_t1 = find_bridges(adj_t1, 4)
assert sorted(bridges_t1) == [(0, 1), (1, 2), (2, 3)], f"Got {bridges_t1}"

# Test 2: Cycle — no bridges
# 0 - 1 - 2 - 0
edges_t2 = [(0, 1), (1, 2), (2, 0)]
adj_t2 = build_adjacency_list(3, edges_t2)
bridges_t2 = find_bridges(adj_t2, 3)
assert bridges_t2 == [], f"Expected no bridges, got {bridges_t2}"

# Test 3: Mixed — one bridge connecting two cycles
# Cycle A: 0-1-2-0,  bridge: 2-3,  Cycle B: 3-4-5-3
edges_t3 = [(0, 1), (1, 2), (2, 0), (2, 3), (3, 4), (4, 5), (5, 3)]
adj_t3 = build_adjacency_list(6, edges_t3)
bridges_t3 = find_bridges(adj_t3, 6)
assert sorted(bridges_t3) == [(2, 3)], f"Expected [(2, 3)], got {bridges_t3}"

# Test 4: Single node — no bridges
adj_t4 = build_adjacency_list(1, [])
bridges_t4 = find_bridges(adj_t4, 1)
assert bridges_t4 == [], f"Expected no bridges, got {bridges_t4}"

print("All Stage 3 tests passed.")
```

## Scoring Rubric

### Fundamentals (1-4)

| Score | Criteria |
|-------|----------|
| **4 — Strong** | Immediately recognizes graph connectivity, cycle detection, and bridge concepts. Correctly identifies BFS/DFS as O(V+E). Articulates the difference between directed and undirected cycle detection without prompting. Understands Tarjan's bridge algorithm or derives it independently. |
| **3 — Satisfactory** | Builds adjacency list correctly. Implements BFS/DFS for reachability. Implements directed cycle detection with DFS coloring (maybe with a small hint). Understands bridge concept but may need guidance on the algorithm. |
| **2 — Developing** | Can build an adjacency list but struggles with traversal. Confuses directed/undirected cycle detection. Needs significant hints on bridge finding. May use suboptimal brute-force removal approach. |
| **1 — Insufficient** | Cannot build an adjacency list from edges. Does not understand BFS or DFS. Cannot articulate what a cycle or bridge is in graph terms. |

### Coding (1-4)

| Score | Criteria |
|-------|----------|
| **4 — Strong** | Clean, well-structured code. Uses appropriate data structures (defaultdict, deque). No bugs on first pass. Handles edge cases (empty graph, single node, disconnected components). Code is readable with meaningful variable names. |
| **3 — Satisfactory** | Working solutions with minor bugs caught during testing. Uses correct data structures. Code is generally clean. May miss one or two edge cases. |
| **2 — Developing** | Solutions have bugs requiring interviewer hints to fix. May use incorrect data structures (e.g., regular dict without handling missing keys). Code is functional but messy. |
| **1 — Insufficient** | Cannot produce working code for Stage 1. Fundamental syntax errors. Does not know how to use Python collections. |

### Communication (1-4)

| Score | Criteria |
|-------|----------|
| **4 — Strong** | Asks 3+ clarifying questions before coding. Explains approach before writing code. Talks through each design decision. Proactively discusses trade-offs (BFS vs DFS, iterative vs recursive). Identifies edge cases verbally. |
| **3 — Satisfactory** | Asks 1-2 clarifying questions. Explains high-level approach. Communicates while coding. Discusses complexity when asked. |
| **2 — Developing** | Jumps into coding without clarifying. Explains only when prompted. Does not discuss alternatives or trade-offs. |
| **1 — Insufficient** | Codes silently. Cannot explain own approach. Does not respond well to interviewer questions. |

## Interviewer Guide

### Hints (per stage, progressive: mild, medium, strong)

**Stage 1: Network Reachability**

| Level | Hint |
|-------|------|
| **Mild** | "How would you represent this graph in a way that makes traversal efficient?" |
| **Medium** | "Think about starting at the hub and exploring outward. What traversal algorithm visits all reachable nodes?" |
| **Strong** | "Use BFS: start at the hub, maintain a queue and a visited set. Process each node by adding its unvisited neighbors to the queue. At the end, any node not in visited is unreachable." |

**Stage 2: Cycle Detection**

| Level | Hint |
|-------|------|
| **Mild** | "This is now directed. Does your undirected cycle detection approach still work? Why or why not?" |
| **Medium** | "Think about the DFS recursion stack. If you encounter a node that is currently being explored (still in the stack), what does that mean?" |
| **Strong** | "Use three states: WHITE (unvisited), GRAY (in current DFS path), BLACK (fully explored). A back edge to a GRAY node means a cycle. Color each node GRAY when you enter it, BLACK when you finish exploring all its descendants." |

**Stage 3: Bridge Finding**

| Level | Hint |
|-------|------|
| **Mild** | "A bridge is an edge whose removal disconnects the graph. What property of a subtree would tell you it has no 'escape route' back above this edge?" |
| **Medium** | "For each node, track the lowest discovery time reachable from its subtree. If a child's lowest reachable time is strictly greater than the parent's discovery time, what does that imply?" |
| **Strong** | "Maintain two arrays: `disc[u]` for discovery time and `low[u]` for the minimum discovery time reachable from u's subtree. After DFS on child v, set `low[u] = min(low[u], low[v])`. If `low[v] > disc[u]`, then (u, v) is a bridge. For back edges to a non-parent w, set `low[u] = min(low[u], disc[w])`." |

### When to Intervene

- **Stage 1 taking > 25 minutes:** Give the medium hint and move on once they have a working BFS/DFS, even if code is rough.
- **Candidate uses adjacency matrix:** Ask about space complexity for sparse graphs (100K nodes). Guide them to adjacency list.
- **Candidate tries to detect directed cycles with union-find:** Explain that union-find works for undirected cycle detection but not directed. Give them the DFS coloring approach.
- **Stage 3 is clearly beyond them:** It is acceptable if the candidate can only describe the brute-force approach (remove each edge, check connectivity). Award partial credit and spend remaining time discussing the efficient approach conceptually.
- **Recursion limit issues:** If the candidate mentions Python's recursion limit, that shows real-world awareness. Acknowledge it and note that an iterative DFS with an explicit stack would solve it in production. For this interview, the recursive version is fine.

### Common Mistakes

1. **Forgetting to handle disconnected components.** Candidate starts DFS/BFS from only one node for cycle detection. The graph may have multiple components — must iterate over all nodes.
2. **Using `visited` set for directed cycle detection instead of three-color approach.** A simple visited set works for undirected but produces false positives for directed graphs (cross edges vs back edges).
3. **Bridge finding: confusing `low[v] > disc[u]` with `low[v] >= disc[u]`.** The `>=` condition finds articulation points, not bridges. For bridges, the strict `>` is correct.
4. **Not handling the parent check in bridge finding.** When computing `low[u]`, the candidate must skip the parent node (to avoid counting the tree edge as a back edge). The `v != parent` check is essential.
5. **Off-by-one in discovery times.** Starting discovery time at 0 vs 1 does not matter as long as it is consistent. But mixing them up causes bugs.
6. **Mutating the adjacency list while iterating.** Some candidates try to "remove" edges to test bridges by modifying the list. This corrupts the data structure.

### Edge Cases to Watch For

- **Empty graph (n=0 or no edges):** Should not crash. Reachability returns True for a single hub node with no edges.
- **Single node graph:** Trivially reachable, no cycles, no bridges.
- **Self-loops in directed cycle detection:** `(0, 0)` is a cycle. The DFS coloring handles this naturally (node 0 is GRAY when we see the edge back to 0).
- **Parallel paths (diamond shapes):** In directed graphs, `0->1, 0->2, 1->3, 2->3` is NOT a cycle even though node 3 has two parents.
- **Hub is not node 0:** The candidate should not assume the hub is always node 0. It is a parameter.
- **All nodes isolated (no edges):** Every node is its own component. Only the hub's component (just itself) is reachable. All others are unreachable.
