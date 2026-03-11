# Round 12: BFS — "Minimum Hops Between Locations"

## Metadata
- **Topic:** BFS (shortest path in unweighted graphs, path reconstruction, all shortest paths)
- **Difficulty:** 3/5
- **Estimated Time:** 55-60 minutes
- **Key Concepts:** BFS for shortest path in unweighted graphs, parent tracking, path reconstruction, multi-parent BFS for all shortest paths

## Problem Prompt

> DoorDash's delivery map can be modeled as a graph where nodes are locations and edges are direct road connections between them. All road segments are roughly the same length, so we treat them as unweighted.
>
> Given two locations, we want to find the most efficient route between them — the one that crosses the fewest road segments.

*Read this prompt verbatim. Do NOT specify what to return if unreachable, whether start can equal end, or how the graph is provided. Let the candidate ask.*

## Clarifying Questions & Hidden Constraints

The candidate **should** ask these questions. Award communication points for each one asked unprompted.

| # | Question | Answer |
|---|----------|--------|
| 1 | How is the graph represented? Edge list? Adjacency list? | You will receive an **adjacency list** as a dictionary mapping each node to its list of neighbors. The graph is **undirected**. |
| 2 | What should we return if the destination is unreachable? | Return **-1** for distance (Stage 1), an **empty list** for path (Stage 2), and an **empty list** for all paths (Stage 3). |
| 3 | What if start equals end? | Return **0** for distance, **[start]** for path, and **[[start]]** for all paths. |
| 4 | Are the edges weighted? | No. All edges are unweighted — every road segment costs 1 hop. |
| 5 | Can the graph have cycles? | Yes. The graph is a general undirected graph — cycles are possible. BFS handles this naturally via the visited set. |
| 6 | How large can the graph be? | Up to 100,000 nodes and 500,000 edges. Solution should be O(V+E). |
| 7 | Are node IDs always integers? | Yes, but not necessarily contiguous. Use the adjacency list keys as the set of valid nodes. |

## Skeleton Code

```python
from collections import deque
from typing import Dict, List, Optional


def min_hops(
    graph: Dict[int, List[int]],
    start: int,
    end: int,
) -> int:
    """
    Find the minimum number of hops (edges) between start and end.

    Args:
        graph: Adjacency list {node: [neighbors]}.
        start: Source node.
        end: Destination node.

    Returns:
        Minimum number of hops, or -1 if unreachable.
    """
    pass

if __name__ == "__main__":
    graph1 = {0: [1, 2], 1: [0, 3], 2: [0, 3], 3: [1, 2, 4], 4: [3]}
    # Expected: 3 (0->1->3->4)
    print(min_hops(graph1, 0, 4))
    # Expected: 0 (start equals end)
    print(min_hops(graph1, 2, 2))
    # Expected: -1 (unreachable)
    graph2 = {0: [1], 1: [0], 2: [3], 3: [2]}
    print(min_hops(graph2, 0, 3))
```

## Stage 1: Minimum Hop Count (~20 minutes)

### Prompt Addition

> "Given the adjacency list, find the minimum number of road segments (hops) between a start location and an end location. Return -1 if the destination is unreachable."

### Expected Approach

**Standard BFS:**
1. Start from the source node. Initialize a queue with `(start, 0)` where 0 is the distance.
2. Maintain a `visited` set to avoid revisiting nodes.
3. For each node dequeued, check if it is the destination. If so, return the distance.
4. Otherwise, enqueue all unvisited neighbors with distance + 1.
5. If the queue empties without finding the destination, return -1.

**Key insight the candidate should articulate:** BFS explores nodes in order of their distance from the source. The first time we reach any node, that is the shortest distance to it. This is why BFS works for unweighted shortest paths — it processes all nodes at distance `d` before any node at distance `d+1`.

### Solution Code

```python
from collections import deque
from typing import Dict, List


def min_hops(
    graph: Dict[int, List[int]],
    start: int,
    end: int,
) -> int:
    if start == end:
        return 0

    visited = {start}
    queue = deque([(start, 0)])

    while queue:
        node, dist = queue.popleft()
        for neighbor in graph.get(node, []):
            if neighbor == end:
                return dist + 1
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, dist + 1))

    return -1
```

### Complexity

- **Time:** O(V + E) — each node is enqueued at most once and each edge is examined at most once.
- **Space:** O(V) — the visited set and queue can hold at most all V nodes.

### Test Cases

```python
# Test 1: Simple path
graph1 = {
    0: [1, 2],
    1: [0, 3],
    2: [0, 3],
    3: [1, 2, 4],
    4: [3],
}
assert min_hops(graph1, 0, 4) == 2, "0 -> 1 or 2 -> 3 -> 4 is 3 hops? No, 0->1->3->4 is 3. Wait: 0->1 is 1, 1->3 is 2, 3->4 is 3."
# Actually: 0->1->3->4 = 3 hops, or 0->2->3->4 = 3 hops
assert min_hops(graph1, 0, 4) == 3

# Test 2: Start equals end
assert min_hops(graph1, 2, 2) == 0

# Test 3: Unreachable
graph2 = {
    0: [1],
    1: [0],
    2: [3],
    3: [2],
}
assert min_hops(graph2, 0, 3) == -1

# Test 4: Direct neighbor
assert min_hops(graph1, 0, 1) == 1

# Test 5: Larger graph with multiple paths of different lengths
graph3 = {
    0: [1, 2],
    1: [0, 3],
    2: [0, 4],
    3: [1, 5],
    4: [2, 5],
    5: [3, 4],
}
# 0->1->3->5 = 3 hops, 0->2->4->5 = 3 hops
assert min_hops(graph3, 0, 5) == 3

print("All Stage 1 tests passed.")
```

## Stage 2: Shortest Path Reconstruction (~20 minutes)

### Prompt Addition

> "Now instead of just the hop count, return the actual path — the ordered list of nodes from start to end. If there are multiple shortest paths, return any one of them."

### Starter Code

```python
def shortest_path(
    graph: Dict[int, List[int]],
    start: int,
    end: int,
) -> List[int]:
    """
    Find a shortest path from start to end.

    Returns:
        List of nodes from start to end (inclusive), or [] if unreachable.
    """
    pass

# --- Stage 2 Test Calls ---
# graph1 = {0: [1, 2], 1: [0, 3], 2: [0, 3], 3: [1, 2, 4], 4: [3]}
# Expected: a path of length 4 starting at 0 and ending at 4
# print(shortest_path(graph1, 0, 4))
# Expected: [3]
# print(shortest_path(graph1, 3, 3))
# Expected: [] (unreachable)
# graph2 = {0: [1], 1: [0], 2: [3], 3: [2]}
# print(shortest_path(graph2, 0, 2))
```

### Expected Approach

**BFS with parent tracking:**
1. Run BFS as before, but instead of (or in addition to) a visited set, maintain a `parent` dictionary mapping each node to the node that discovered it.
2. When the destination is found, reconstruct the path by following parent pointers from `end` back to `start`.
3. Reverse the reconstructed path.

**Implementation detail:** The `parent` dictionary also serves as the visited set — if a node is in `parent`, it has been visited.

### Solution Code

```python
def shortest_path(
    graph: Dict[int, List[int]],
    start: int,
    end: int,
) -> List[int]:
    if start == end:
        return [start]

    parent = {start: None}
    queue = deque([start])

    while queue:
        node = queue.popleft()
        for neighbor in graph.get(node, []):
            if neighbor not in parent:
                parent[neighbor] = node
                if neighbor == end:
                    # Reconstruct path
                    path = []
                    current = end
                    while current is not None:
                        path.append(current)
                        current = parent[current]
                    return path[::-1]
                queue.append(neighbor)

    return []  # unreachable
```

### Complexity

- **Time:** O(V + E) for BFS + O(V) for path reconstruction = O(V + E).
- **Space:** O(V) — parent dictionary, queue, and reconstructed path.

### Test Cases

```python
# Test 1: Path exists
graph1 = {
    0: [1, 2],
    1: [0, 3],
    2: [0, 3],
    3: [1, 2, 4],
    4: [3],
}
path1 = shortest_path(graph1, 0, 4)
assert len(path1) == 4, f"Expected path of length 4, got {path1}"
assert path1[0] == 0 and path1[-1] == 4, f"Path must start at 0 and end at 4, got {path1}"
# Verify each consecutive pair is actually an edge
for i in range(len(path1) - 1):
    assert path1[i + 1] in graph1[path1[i]], f"No edge {path1[i]}->{path1[i+1]}"

# Test 2: Start equals end
path2 = shortest_path(graph1, 3, 3)
assert path2 == [3], f"Expected [3], got {path2}"

# Test 3: Unreachable
graph2 = {
    0: [1],
    1: [0],
    2: [3],
    3: [2],
}
path3 = shortest_path(graph2, 0, 2)
assert path3 == [], f"Expected [], got {path3}"

# Test 4: Direct neighbor
path4 = shortest_path(graph1, 0, 1)
assert path4 == [0, 1], f"Expected [0, 1], got {path4}"

# Test 5: Linear chain
graph_linear = {i: [i + 1] if i < 4 else [] for i in range(5)}
for i in range(1, 5):
    graph_linear[i] = graph_linear.get(i, []) + [i - 1]  # undirected
graph_linear = {
    0: [1],
    1: [0, 2],
    2: [1, 3],
    3: [2, 4],
    4: [3],
}
path5 = shortest_path(graph_linear, 0, 4)
assert path5 == [0, 1, 2, 3, 4], f"Expected [0,1,2,3,4], got {path5}"

print("All Stage 2 tests passed.")
```

## Stage 3: All Shortest Paths (~15 minutes)

### Prompt Addition

> "Final challenge. There may be multiple shortest paths of the same minimum length. Find and return ALL of them."

### Starter Code

```python
def all_shortest_paths(
    graph: Dict[int, List[int]],
    start: int,
    end: int,
) -> List[List[int]]:
    """
    Find ALL shortest paths from start to end.

    Returns:
        List of all shortest paths (each a list of nodes), or [] if unreachable.
    """
    pass

# --- Stage 3 Test Calls ---
# graph1 = {0: [1, 2], 1: [0, 3], 2: [0, 3], 3: [1, 2, 4], 4: [3]}
# Expected: [[0,1,3,4], [0,2,3,4]] (two shortest paths)
# print(sorted(all_shortest_paths(graph1, 0, 4)))
# Expected: [[3]]
# print(all_shortest_paths(graph1, 3, 3))
# Expected: [] (unreachable)
# graph2 = {0: [1], 1: [0], 2: [3], 3: [2]}
# print(all_shortest_paths(graph2, 0, 2))
```

### Expected Approach

**Modified BFS with multi-parent tracking:**

The key insight is that during BFS, a node at distance `d` can be reached by multiple nodes at distance `d-1`. Instead of storing a single parent, store **all parents** — every node at the previous level that has an edge to this node.

**Algorithm:**
1. Run BFS, but use a `distance` dictionary instead of a simple visited set. Record each node's distance from start.
2. For each node, maintain a list of parents: `parents[node]` = all neighbors that are at distance `distance[node] - 1`.
3. A neighbor at distance `d` is a valid parent if `distance[neighbor] == distance[node] - 1`.
4. After BFS completes, reconstruct all paths using DFS/backtracking from `end` to `start` through the parent lists.

**Crucial detail:** When processing a neighbor in BFS, we add it as a parent in two cases:
- The neighbor has not been visited yet (standard BFS — set its distance and add to queue).
- The neighbor was already visited **at the same distance** we would assign now (it is at `dist[current] + 1` and we already recorded it at that distance). In this case, we just add the current node as an additional parent.

### Solution Code

```python
def all_shortest_paths(
    graph: Dict[int, List[int]],
    start: int,
    end: int,
) -> List[List[int]]:
    if start == end:
        return [[start]]

    # BFS to compute distances and collect all parents
    dist = {start: 0}
    parents = {start: []}  # parents[node] = list of nodes at dist-1 that reach node
    queue = deque([start])

    while queue:
        node = queue.popleft()
        current_dist = dist[node]

        for neighbor in graph.get(node, []):
            if neighbor not in dist:
                # First time visiting this neighbor
                dist[neighbor] = current_dist + 1
                parents[neighbor] = [node]
                queue.append(neighbor)
            elif dist[neighbor] == current_dist + 1:
                # Another shortest-path parent found
                parents[neighbor].append(node)

    # If end was never reached, return empty
    if end not in dist:
        return []

    # Backtrack from end to start using parent lists (DFS reconstruction)
    all_paths = []

    def backtrack(node: int, path: List[int]):
        if node == start:
            all_paths.append(path[::-1])
            return
        for p in parents[node]:
            path.append(p)
            backtrack(p, path)
            path.pop()

    backtrack(end, [end])
    return all_paths
```

### Complexity

- **Time:** O(V + E + P) where P is the total length of all shortest paths. The BFS is O(V+E). Path reconstruction visits each path once. In the worst case, the number of shortest paths can be exponential (e.g., a grid graph), but for typical interview graphs this is manageable.
- **Space:** O(V + E + P) — distance/parent dictionaries, plus all reconstructed paths.

### Test Cases

```python
# Test 1: Two shortest paths
graph1 = {
    0: [1, 2],
    1: [0, 3],
    2: [0, 3],
    3: [1, 2, 4],
    4: [3],
}
# Shortest paths from 0 to 4 (length 3):
# 0 -> 1 -> 3 -> 4
# 0 -> 2 -> 3 -> 4
paths1 = all_shortest_paths(graph1, 0, 4)
paths1_sorted = sorted([tuple(p) for p in paths1])
assert paths1_sorted == [(0, 1, 3, 4), (0, 2, 3, 4)], f"Got {paths1_sorted}"

# Test 2: Only one shortest path
graph2 = {
    0: [1],
    1: [0, 2],
    2: [1],
}
paths2 = all_shortest_paths(graph2, 0, 2)
assert paths2 == [[0, 1, 2]], f"Expected [[0,1,2]], got {paths2}"

# Test 3: Start equals end
paths3 = all_shortest_paths(graph1, 3, 3)
assert paths3 == [[3]], f"Expected [[3]], got {paths3}"

# Test 4: Unreachable
graph3 = {
    0: [1],
    1: [0],
    2: [3],
    3: [2],
}
paths4 = all_shortest_paths(graph3, 0, 2)
assert paths4 == [], f"Expected [], got {paths4}"

# Test 5: Diamond with multiple paths at each level
#     0
#    / \
#   1   2
#    \ / \
#     3   4
#      \ /
#       5
graph4 = {
    0: [1, 2],
    1: [0, 3],
    2: [0, 3, 4],
    3: [1, 2, 5],
    4: [2, 5],
    5: [3, 4],
}
# 0->5 shortest paths (length 3):
# 0->1->3->5
# 0->2->3->5
# 0->2->4->5
paths5 = all_shortest_paths(graph4, 0, 5)
paths5_sorted = sorted([tuple(p) for p in paths5])
expected5 = [(0, 1, 3, 5), (0, 2, 3, 5), (0, 2, 4, 5)]
assert paths5_sorted == expected5, f"Expected {expected5}, got {paths5_sorted}"

# Test 6: Direct neighbors
paths6 = all_shortest_paths(graph1, 0, 1)
assert paths6 == [[0, 1]], f"Expected [[0, 1]], got {paths6}"

print("All Stage 3 tests passed.")
```

## Scoring Rubric

### Fundamentals (1-4)

| Score | Criteria |
|-------|----------|
| **4 — Strong** | Immediately identifies BFS as the right tool for unweighted shortest path and articulates *why* (level-order exploration guarantees shortest distance). Distinguishes from Dijkstra's (not needed since unweighted). Naturally extends to parent tracking and multi-parent tracking without confusion. Understands that the number of shortest paths can be exponential in worst case. |
| **3 — Satisfactory** | Knows BFS gives shortest path in unweighted graphs. Implements parent tracking for path reconstruction. Can extend to multi-parent tracking with minor guidance. Discusses complexity correctly. |
| **2 — Developing** | Knows BFS but is unsure why it gives shortest paths. May attempt DFS first (which does not guarantee shortest path) and needs correction. Struggles with path reconstruction or multi-parent logic. |
| **1 — Insufficient** | Does not know BFS. Attempts DFS or brute-force enumeration of all paths. Cannot articulate why BFS finds shortest paths in unweighted graphs. |

### Coding (1-4)

| Score | Criteria |
|-------|----------|
| **4 — Strong** | Clean BFS implementation using `deque`. Correctly handles all edge cases (start==end, unreachable, direct neighbors). Path reconstruction is clean using parent pointers. Multi-parent BFS and backtracking are implemented without bugs. Code is readable and well-organized. |
| **3 — Satisfactory** | Working BFS. Path reconstruction works but may have a minor off-by-one or reversal issue caught during testing. All shortest paths implementation works with small hints. |
| **2 — Developing** | BFS works but misses edge cases (e.g., start==end). Path reconstruction has bugs. Multi-parent tracking is incomplete or incorrect. |
| **1 — Insufficient** | Cannot implement BFS correctly. Uses a list instead of `deque` (functionally works but O(N) popleft). Major logical errors. |

### Communication (1-4)

| Score | Criteria |
|-------|----------|
| **4 — Strong** | Asks about unreachable nodes, start==end, graph representation. Explains why BFS (not DFS) is correct for shortest paths before coding. Walks through an example by hand to verify their approach. Discusses the exponential nature of all-shortest-paths and when it might be a concern. |
| **3 — Satisfactory** | Asks 1-2 clarifying questions. Explains BFS approach. Discusses complexity. Walks through at least one test case. |
| **2 — Developing** | Starts coding without much discussion. Explains only when prompted. |
| **1 — Insufficient** | Silent coding. Cannot explain why their approach works. |

## Interviewer Guide

### Hints (per stage, progressive: mild, medium, strong)

**Stage 1: Minimum Hop Count**

| Level | Hint |
|-------|------|
| **Mild** | "Since all edges have the same weight, which graph traversal algorithm naturally finds the shortest path?" |
| **Medium** | "BFS explores nodes level by level — all nodes at distance 1, then distance 2, and so on. The first time you reach the destination, you have found the shortest distance." |
| **Strong** | "Use a queue initialized with `(start, 0)`. For each node you dequeue, check if it is the end — if so, return the distance. Otherwise, add all unvisited neighbors with distance+1 to the queue. Use a visited set to avoid cycles." |

**Stage 2: Path Reconstruction**

| Level | Hint |
|-------|------|
| **Mild** | "You know the distance now. But how would you remember *which* nodes led to the shortest path?" |
| **Medium** | "When you visit a new neighbor for the first time, record who discovered it — its 'parent' in the BFS tree. Once you reach the end, follow parent pointers back to start." |
| **Strong** | "Use a dictionary `parent = {start: None}`. When you enqueue a new neighbor, set `parent[neighbor] = current_node`. This also doubles as your visited check — if a node is in `parent`, it has been visited. To reconstruct: start at `end`, follow `parent[end]`, then `parent[parent[end]]`, etc., until you reach `start`. Reverse the list." |

**Stage 3: All Shortest Paths**

| Level | Hint |
|-------|------|
| **Mild** | "In Stage 2, each node had one parent. But if two different nodes at distance `d` both connect to a node at distance `d+1`, could there be multiple 'parents'?" |
| **Medium** | "Instead of `parent[node] = single_node`, use `parents[node] = [list of nodes]`. During BFS, when you encounter a neighbor that is already visited at exactly `current_dist + 1`, add the current node as another parent. After BFS, backtrack from end to start using all parent combinations." |
| **Strong** | "During BFS, track `dist[node]`. When processing node `u` at distance `d`, for each neighbor `v`: if `v` is not in `dist`, set `dist[v] = d+1`, `parents[v] = [u]`, and enqueue. If `v` is already in `dist` and `dist[v] == d+1`, just append `u` to `parents[v]`. After BFS, use recursive backtracking from `end`: at each node, try all parents, building the path. When you reach `start`, save the reversed path." |

### When to Intervene

- **Candidate uses DFS for shortest path:** Ask "Does DFS always find the shortest path?" Let them realize it does not (DFS explores depth-first and may find a longer path first). If they cannot self-correct, explain that BFS is needed for unweighted shortest path.
- **Candidate uses a list instead of `deque`:** Ask about the time complexity of `list.pop(0)`. It is O(N), making the overall BFS O(V*V) instead of O(V+E). `collections.deque` has O(1) popleft.
- **Path reconstruction is backwards:** This is common. If they build the path from end to start but forget to reverse, let them trace through a test case to catch it.
- **Stage 3 takes too long:** If they are stuck on multi-parent tracking after 10 minutes, give the strong hint and focus on getting the backtracking right. The backtracking logic is the harder part.
- **Candidate wants to enumerate all paths and filter by length:** This is exponentially worse than BFS with multi-parent tracking. Point out that you might have millions of paths but only a few shortest ones. BFS prunes non-shortest paths during traversal.

### Common Mistakes

1. **Using DFS instead of BFS.** DFS does not guarantee shortest path in unweighted graphs. This is the most fundamental error. If the candidate insists DFS works, ask them to trace through a graph with a short path and a long path — DFS might find the long one first.
2. **Not marking nodes as visited when enqueuing.** If a node is only marked visited when dequeued (instead of when enqueued), the same node can be added to the queue multiple times, wasting time and potentially returning wrong distances. The correct approach is to mark visited (or set distance) when adding to the queue.
3. **Path reconstruction off-by-one.** Forgetting to include start or end in the path. Or reversing incorrectly.
4. **All-shortest-paths: overwriting parent instead of appending.** In Stage 3, if the candidate writes `parents[v] = u` instead of `parents[v].append(u)`, they lose alternative paths.
5. **All-shortest-paths: adding parents at wrong distances.** The candidate must check that `dist[neighbor] == dist[current] + 1` before adding a new parent. If they skip this check, they add parents from wrong BFS levels, producing incorrect (non-shortest) paths.
6. **Forgetting `graph.get(node, [])` for nodes not in the adjacency list.** If a node has no outgoing edges and is not a key in the dict, accessing `graph[node]` raises a `KeyError`. Using `.get(node, [])` is safer.

### Edge Cases to Watch For

- **Start equals end:** Distance is 0, path is `[start]`, all-paths is `[[start]]`. Many candidates forget this.
- **Start or end not in graph:** Should return -1 / [] / []. The BFS will simply never find the destination.
- **Direct neighbors:** Distance is 1, path is `[start, end]`. Make sure early termination does not skip this.
- **Disconnected graph:** BFS from start will only visit the start's component. Destination in a different component is unreachable.
- **Graph with a single node and no edges:** `{0: []}`. Start and end are both 0 — should return 0 / [0] / [[0]].
- **Large number of shortest paths:** In a grid-like graph, the number of shortest paths between opposite corners is combinatorially large. The candidate should acknowledge this and note that the time to enumerate all paths is proportional to the total output size, which can be exponential. For the interview, small examples are fine.
