# Round 14: Dijkstra's Algorithm — "Optimal Delivery Route"

## Metadata
- **Topic:** Dijkstra's Algorithm, Shortest Path, Weighted Graphs
- **Difficulty:** 4/5
- **Estimated Time:** 55-60 minutes
- **Key Concepts:** Min-heap (priority queue), shortest path computation, path reconstruction, multi-stop routing

## Problem Prompt

> DoorDash operates in cities where roads form a weighted network. Each
> intersection is a node, and each road segment between intersections has a
> travel time (in minutes). Roads may be one-way.
>
> Given the road network, a starting intersection, and a destination
> intersection, find the shortest travel time from start to destination.

*(Read the prompt exactly as written. Let the candidate ask about edge cases
before revealing the clarifying answers below.)*

## Clarifying Questions & Hidden Constraints

| # | What the candidate should ask | Answer |
|---|-------------------------------|--------|
| 1 | Are all travel times positive? | Yes. All edge weights are strictly positive integers. This is important — it means Dijkstra's algorithm is applicable. |
| 2 | Can there be one-way streets? | Yes. The graph is directed. A road from A to B does not imply a road from B to A. |
| 3 | Can there be multiple roads between the same two intersections? | No. Assume at most one directed edge between any ordered pair of nodes. |
| 4 | What if the destination is unreachable from the source? | Return -1 (or an appropriate sentinel) to indicate no path exists. |
| 5 | How are intersections identified? | Integer IDs from `0` to `n-1`, where `n` is the number of intersections. |
| 6 | Can the source and destination be the same? | Yes. The shortest distance in that case is 0. |
| 7 | What is the expected input size? | Up to 10,000 intersections and 50,000 road segments. An efficient implementation is expected. |

## Skeleton Code

```python
from typing import List, Tuple, Optional
import heapq

# Each edge is (source, destination, weight).
# Example: (0, 1, 5) means a road from intersection 0 to 1 with travel time 5.

def shortest_travel_time(
    num_intersections: int,
    roads: List[Tuple[int, int, int]],
    source: int,
    destination: int,
) -> int:
    """
    Stage 1: Return the shortest travel time from source to destination.
    Return -1 if the destination is unreachable.
    """
    pass

if __name__ == "__main__":
    # Expected: 6 (0->2->1->3 = 1+2+3)
    print(shortest_travel_time(4, [(0, 1, 4), (0, 2, 1), (2, 1, 2), (1, 3, 3)], source=0, destination=3))
    # Expected: -1 (unreachable)
    print(shortest_travel_time(3, [(0, 1, 5)], source=0, destination=2))
    # Expected: 0 (source equals destination)
    print(shortest_travel_time(3, [(0, 1, 5), (1, 2, 3)], source=0, destination=0))
```

---

## Stage 1: Shortest Travel Time (~25 minutes)

### Expected Approach

Implement Dijkstra's algorithm using Python's `heapq` module:

1. Build an adjacency list from the edge list.
2. Initialize a `dist` array with infinity for all nodes, except `dist[source] = 0`.
3. Push `(0, source)` onto a min-heap.
4. While the heap is non-empty:
   a. Pop the node with the smallest tentative distance.
   b. If this node is the destination, return the distance.
   c. If the popped distance is greater than the known best (`dist[node]`), skip it (stale entry).
   d. For each neighbor, compute `new_dist = dist[node] + weight`. If `new_dist < dist[neighbor]`, update and push.
5. If the destination was never reached, return -1.

Key insight: Python's `heapq` is a min-heap. We use lazy deletion (skip stale entries) rather than decrease-key, which is the standard approach in Python.

### Solution Code

```python
def shortest_travel_time(
    num_intersections: int,
    roads: List[Tuple[int, int, int]],
    source: int,
    destination: int,
) -> int:
    # Build adjacency list
    graph = [[] for _ in range(num_intersections)]
    for u, v, w in roads:
        graph[u].append((v, w))

    # Distance array
    dist = [float('inf')] * num_intersections
    dist[source] = 0

    # Min-heap: (distance, node)
    heap = [(0, source)]

    while heap:
        d, u = heapq.heappop(heap)

        # Early termination: destination reached
        if u == destination:
            return d

        # Skip stale entries
        if d > dist[u]:
            continue

        for v, w in graph[u]:
            new_dist = d + w
            if new_dist < dist[v]:
                dist[v] = new_dist
                heapq.heappush(heap, (new_dist, v))

    return -1
```

### Complexity

| | |
|---|---|
| **Time** | O((V + E) log V) — each edge may cause one heap push, each push/pop is O(log V). |
| **Space** | O(V + E) — adjacency list and heap (heap can have up to E entries with lazy deletion). |

### Test Cases

```python
# Test 1: Simple graph
#   0 --(4)--> 1 --(3)--> 3
#   0 --(1)--> 2 --(2)--> 1
assert shortest_travel_time(4, [
    (0, 1, 4), (0, 2, 1), (2, 1, 2), (1, 3, 3)
], source=0, destination=3) == 6  # 0->2->1->3 = 1+2+3

# Test 2: Direct path is not shortest
assert shortest_travel_time(3, [
    (0, 1, 10), (0, 2, 3), (2, 1, 2)
], source=0, destination=1) == 5  # 0->2->1 = 3+2

# Test 3: Destination unreachable
assert shortest_travel_time(3, [
    (0, 1, 5)
], source=0, destination=2) == -1

# Test 4: Source equals destination
assert shortest_travel_time(3, [
    (0, 1, 5), (1, 2, 3)
], source=0, destination=0) == 0

# Test 5: One-way streets matter
assert shortest_travel_time(2, [
    (0, 1, 5)
], source=1, destination=0) == -1  # no road from 1 to 0

# Test 6: Larger graph with multiple paths
assert shortest_travel_time(5, [
    (0, 1, 10), (0, 2, 3), (2, 1, 1), (1, 3, 2),
    (2, 3, 8), (3, 4, 1), (2, 4, 15)
], source=0, destination=4) == 7  # 0->2->1->3->4 = 3+1+2+1
```

---

## Stage 2: Shortest Path Reconstruction (~20 minutes)

### Prompt Addition

> "Great. Now the dasher needs turn-by-turn directions, not just the travel
> time. Return the actual sequence of intersections along the shortest path. If
> the destination is unreachable, return `None`."

### Starter Code

```python
def shortest_path(
    num_intersections: int,
    roads: List[Tuple[int, int, int]],
    source: int,
    destination: int,
) -> Optional[List[int]]:
    """
    Stage 2: Return the actual shortest path as a list of intersection IDs
    from source to destination (inclusive). Return None if unreachable.
    """
    pass

# --- Stage 2 Test Calls ---
# Expected: [0, 2, 1, 3]
# print(shortest_path(4, [(0, 1, 4), (0, 2, 1), (2, 1, 2), (1, 3, 3)], source=0, destination=3))
# Expected: None (unreachable)
# print(shortest_path(3, [(0, 1, 5)], source=0, destination=2))
# Expected: [0]
# print(shortest_path(2, [(0, 1, 5)], source=0, destination=0))
```

### Expected Approach

Extend Dijkstra's with a predecessor (parent) array:

1. Maintain a `prev` array initialized to `-1` for all nodes.
2. Whenever `dist[v]` is updated via node `u`, set `prev[v] = u`.
3. After Dijkstra's completes, reconstruct the path by walking backward from the destination through the `prev` pointers.
4. Reverse the path to get source-to-destination order.

### Solution Code

```python
def shortest_path(
    num_intersections: int,
    roads: List[Tuple[int, int, int]],
    source: int,
    destination: int,
) -> Optional[List[int]]:
    graph = [[] for _ in range(num_intersections)]
    for u, v, w in roads:
        graph[u].append((v, w))

    dist = [float('inf')] * num_intersections
    dist[source] = 0
    prev = [-1] * num_intersections

    heap = [(0, source)]

    while heap:
        d, u = heapq.heappop(heap)

        if u == destination:
            break

        if d > dist[u]:
            continue

        for v, w in graph[u]:
            new_dist = d + w
            if new_dist < dist[v]:
                dist[v] = new_dist
                prev[v] = u
                heapq.heappush(heap, (new_dist, v))

    # Check if destination was reached
    if dist[destination] == float('inf'):
        return None

    # Reconstruct path by walking predecessors
    path = []
    node = destination
    while node != -1:
        path.append(node)
        node = prev[node]

    path.reverse()
    return path
```

### Complexity

| | |
|---|---|
| **Time** | O((V + E) log V) — same as Stage 1, plus O(V) for path reconstruction. |
| **Space** | O(V + E) — adjacency list, dist/prev arrays, heap, and output path. |

### Test Cases

```python
# Test 1: Path through intermediate node
result = shortest_path(4, [
    (0, 1, 4), (0, 2, 1), (2, 1, 2), (1, 3, 3)
], source=0, destination=3)
assert result == [0, 2, 1, 3]  # 1+2+3=6

# Test 2: Direct edge is shortest
result = shortest_path(3, [
    (0, 1, 1), (0, 2, 5), (2, 1, 10)
], source=0, destination=1)
assert result == [0, 1]

# Test 3: Unreachable destination
result = shortest_path(3, [(0, 1, 5)], source=0, destination=2)
assert result is None

# Test 4: Source equals destination
result = shortest_path(2, [(0, 1, 5)], source=0, destination=0)
assert result == [0]

# Test 5: Longer path is faster
result = shortest_path(4, [
    (0, 1, 100), (0, 2, 1), (2, 3, 1), (3, 1, 1)
], source=0, destination=1)
assert result == [0, 2, 3, 1]  # 1+1+1=3 < 100
```

---

## Stage 3: Multi-Stop Route (~10 minutes)

### Prompt Addition

> "In reality, a dasher starts at their current location, drives to a restaurant
> to pick up food, then drives to the customer to deliver it. Given three
> points — `source` (dasher's location), `restaurant`, and `customer` — find the
> shortest total travel time and the complete path from source to restaurant to
> customer.
>
> How would you extend this to multiple stops?"

### Starter Code

```python
def shortest_multi_stop(
    num_intersections: int,
    roads: List[Tuple[int, int, int]],
    source: int,
    restaurant: int,
    customer: int,
) -> Tuple[int, Optional[List[int]]]:
    """
    Stage 3: A dasher must travel from source -> restaurant -> customer.
    Return (total_travel_time, full_path) or (-1, None) if any leg is
    unreachable.
    """
    pass

# --- Stage 3 Test Calls ---
# Expected: (5, [0, 1, 2])
# print(shortest_multi_stop(3, [(0, 1, 2), (1, 2, 3), (0, 2, 10)], source=0, restaurant=1, customer=2))
# Expected: (-1, None) (unreachable restaurant)
# print(shortest_multi_stop(3, [(1, 2, 5)], source=0, restaurant=1, customer=2))
# Expected: (0, [0])
# print(shortest_multi_stop(1, [], source=0, restaurant=0, customer=0))
```

### Expected Approach

Run Dijkstra's twice:
1. `source` -> `restaurant` (get distance and path for leg 1).
2. `restaurant` -> `customer` (get distance and path for leg 2).

Combine the total distance and concatenate the paths (avoiding duplicate restaurant node at the junction).

For the extension discussion: with `k` stops, run Dijkstra's `k` times between consecutive stops. Total time is O(k * (V+E) log V). If a dasher must visit multiple restaurants in any order, the problem becomes a variant of the Traveling Salesman Problem, which is NP-hard. A practical approach would be Dijkstra from each stop to all other stops, then a bitmask DP or greedy heuristic over the small number of stops.

### Solution Code

```python
def shortest_multi_stop(
    num_intersections: int,
    roads: List[Tuple[int, int, int]],
    source: int,
    restaurant: int,
    customer: int,
) -> Tuple[int, Optional[List[int]]]:
    # Leg 1: source -> restaurant
    leg1_path = shortest_path(num_intersections, roads, source, restaurant)
    if leg1_path is None:
        return (-1, None)

    # Leg 2: restaurant -> customer
    leg2_path = shortest_path(num_intersections, roads, restaurant, customer)
    if leg2_path is None:
        return (-1, None)

    # Compute total distance
    leg1_dist = _dijkstra_dist(num_intersections, roads, source, restaurant)
    leg2_dist = _dijkstra_dist(num_intersections, roads, restaurant, customer)
    total_dist = leg1_dist + leg2_dist

    # Combine paths: leg1 includes restaurant, leg2 starts with restaurant
    # Remove duplicate restaurant node
    full_path = leg1_path + leg2_path[1:]

    return (total_dist, full_path)


def _dijkstra_dist(
    num_intersections: int,
    roads: List[Tuple[int, int, int]],
    source: int,
    destination: int,
) -> int:
    """Helper: return shortest distance only (same as Stage 1)."""
    graph = [[] for _ in range(num_intersections)]
    for u, v, w in roads:
        graph[u].append((v, w))

    dist = [float('inf')] * num_intersections
    dist[source] = 0
    heap = [(0, source)]

    while heap:
        d, u = heapq.heappop(heap)
        if u == destination:
            return d
        if d > dist[u]:
            continue
        for v, w in graph[u]:
            new_dist = d + w
            if new_dist < dist[v]:
                dist[v] = new_dist
                heapq.heappush(heap, (new_dist, v))

    return -1
```

**Cleaner alternative** (reuse `shortest_path` and compute distances from the paths):

```python
def shortest_multi_stop(
    num_intersections: int,
    roads: List[Tuple[int, int, int]],
    source: int,
    restaurant: int,
    customer: int,
) -> Tuple[int, Optional[List[int]]]:
    # Build a weight lookup for path distance computation
    weight = {}
    for u, v, w in roads:
        weight[(u, v)] = w

    def path_distance(path: List[int]) -> int:
        return sum(weight[(path[i], path[i + 1])] for i in range(len(path) - 1))

    leg1 = shortest_path(num_intersections, roads, source, restaurant)
    if leg1 is None:
        return (-1, None)

    leg2 = shortest_path(num_intersections, roads, restaurant, customer)
    if leg2 is None:
        return (-1, None)

    full_path = leg1 + leg2[1:]
    total_dist = path_distance(leg1) + path_distance(leg2)

    return (total_dist, full_path)
```

### Complexity

| | |
|---|---|
| **Time** | O((V + E) log V) per Dijkstra call, two calls total = O(2(V + E) log V). |
| **Space** | O(V + E) — same as a single Dijkstra run (we can reuse the data structures). |

### Test Cases

```python
# Test 1: Simple multi-stop
#   0 --(2)--> 1 --(3)--> 2
#   0 --(10)-> 2
dist, path = shortest_multi_stop(3, [
    (0, 1, 2), (1, 2, 3), (0, 2, 10)
], source=0, restaurant=1, customer=2)
assert dist == 5  # 0->1 (2) + 1->2 (3)
assert path == [0, 1, 2]

# Test 2: Restaurant is the source
dist, path = shortest_multi_stop(3, [
    (0, 1, 5), (1, 2, 3)
], source=0, restaurant=0, customer=2)
assert dist == 8  # 0->0 (0) + 0->1->2 (5+3)
assert path == [0, 1, 2]

# Test 3: Unreachable restaurant
dist, path = shortest_multi_stop(3, [
    (1, 2, 5)
], source=0, restaurant=1, customer=2)
assert dist == -1
assert path is None

# Test 4: Restaurant reachable but customer unreachable from restaurant
dist, path = shortest_multi_stop(3, [
    (0, 1, 5)
], source=0, restaurant=1, customer=2)
assert dist == -1
assert path is None

# Test 5: All same node
dist, path = shortest_multi_stop(1, [], source=0, restaurant=0, customer=0)
assert dist == 0
assert path == [0]

# Test 6: Multi-stop with detour
dist, path = shortest_multi_stop(5, [
    (0, 1, 2), (1, 2, 3), (2, 3, 1), (0, 3, 100),
    (3, 4, 2), (1, 4, 50)
], source=0, restaurant=3, customer=4)
assert dist == 8  # 0->1->2->3 (6) + 3->4 (2)
assert path == [0, 1, 2, 3, 4]
```

---

## Scoring Rubric

### Fundamentals (1-4)

| Score | Description |
|-------|-------------|
| 1 | Cannot explain why BFS alone does not work for weighted graphs; no understanding of relaxation or priority queues. |
| 2 | Understands the concept of shortest path but confuses Dijkstra's with BFS or Bellman-Ford. Cannot articulate why negative weights break Dijkstra's. |
| 3 | Implements Dijkstra's correctly with a heap. Handles path reconstruction with minor guidance. Understands multi-stop decomposition. |
| 4 | Clean implementation of all stages. Explains why Dijkstra's works (greedy choice property), why negative weights break it, and the complexity. Discusses extensions to multiple stops and TSP variant unprompted. |

### Coding (1-4)

| Score | Description |
|-------|-------------|
| 1 | Code does not run. Major issues with heap usage, graph representation, or infinite loops. |
| 2 | Stage 1 partially works but has bugs (e.g., missing the stale-entry check, not handling unreachable nodes). |
| 3 | Stages 1 and 2 work correctly. Stage 3 is conceptually right but may have a minor issue with path concatenation or edge cases. |
| 4 | All stages produce correct, efficient, idiomatic Python. Proper use of `heapq`, clean adjacency list construction, and elegant path reconstruction. Handles all edge cases. |

### Communication (1-4)

| Score | Description |
|-------|-------------|
| 1 | Codes silently; cannot explain why the heap is needed or how relaxation works. |
| 2 | Explains after prompting but is vague about complexity or the greedy property. |
| 3 | Walks through the algorithm before coding. Explains relaxation, heap operations, and path reconstruction. Identifies edge cases when prompted. |
| 4 | Proactively asks clarifying questions (positive weights? directed?). Traces through an example before coding. Discusses trade-offs (adjacency list vs. matrix, lazy deletion vs. decrease-key). Unprompted discussion of negative weights and Bellman-Ford. |

---

## Interviewer Guide

### Hints (per stage, progressive: mild -> medium -> strong)

**Stage 1 — Dijkstra's Shortest Distance**

- **Mild:** "For weighted graphs, BFS does not work. What data structure lets you always process the node with the smallest tentative distance first?"
- **Medium:** "Use a min-heap. Start with distance 0 at the source. Pop the smallest, and for each neighbor, check if you can improve its distance. If so, push the new distance."
- **Strong:** "Use `heapq`. Push `(0, source)`. Pop `(d, u)`. For each edge `(u, v, w)`, if `d + w < dist[v]`, update `dist[v]` and push `(d + w, v)`. Skip popped entries where `d > dist[u]` — those are stale."

**Stage 2 — Path Reconstruction**

- **Mild:** "When you find a shorter path to a node, what extra information could you record to trace back later?"
- **Medium:** "Maintain a `prev` array. When you update `dist[v]` through node `u`, set `prev[v] = u`. After Dijkstra's finishes, walk backward from the destination through `prev` to rebuild the path."
- **Strong:** "Add `prev = [-1] * n`. When `new_dist < dist[v]`, also do `prev[v] = u`. After the loop, start at `destination`, follow `prev` to `source`, collect nodes, and reverse."

**Stage 3 — Multi-Stop Route**

- **Mild:** "Can you break the problem into independent subproblems?"
- **Medium:** "The shortest path from A to C via B is the shortest path from A to B plus the shortest path from B to C. Run Dijkstra's twice."
- **Strong:** "Call your Stage 2 function twice: once for `source -> restaurant`, once for `restaurant -> customer`. Add the distances. Concatenate the paths, removing the duplicate restaurant node at the junction."

### When to Intervene

- If the candidate tries to use BFS for a weighted graph, ask: "What if one edge has weight 1 and another has weight 100? Will BFS find the shortest?"
- If the candidate is struggling with `heapq` syntax after 5 minutes, provide the basic pattern: `heapq.heappush(h, (dist, node))` and `heapq.heappop(h)`.
- If the candidate forgets lazy deletion (stale entry check), let them run into a bug with a test case, then ask: "Can the same node be in the heap multiple times? What happens if you pop an outdated entry?"
- If Stage 2 stalls on path reconstruction for more than 8 minutes, give the medium hint so there is time for Stage 3.
- Stage 3 should be primarily a discussion. If the candidate gets the two-call decomposition quickly, push them on the multi-restaurant TSP extension.

### Common Mistakes

1. **Forgetting the stale-entry check.** Without `if d > dist[u]: continue`, the algorithm may process nodes multiple times at suboptimal distances, leading to incorrect results or TLE.
2. **Using a visited set AND a distance array inconsistently.** Some candidates mark nodes as visited on pop but still push better distances later, causing missed updates.
3. **Not building the adjacency list for a directed graph.** Adding edges in both directions when the graph is directed.
4. **Path reconstruction — forgetting the source node.** The `while` loop should terminate when `node == source` (or when `prev[node] == -1` and `node == source`), and `source` must be in the path.
5. **Stage 3 — running Dijkstra's from source to customer directly.** This ignores the restaurant constraint. The dasher must pass through the restaurant.
6. **Stage 3 — double-counting the restaurant node.** When concatenating paths, `leg1` ends at the restaurant and `leg2` starts there. Use `leg1 + leg2[1:]`.

### Edge Cases to Watch For

- Source equals destination — distance is 0, path is `[source]`.
- Disconnected graph — destination unreachable, must return -1/None.
- Single node graph — distance 0, path `[0]`.
- Two nodes with no connecting edge — unreachable.
- Multiple paths with the same shortest distance — any shortest path is acceptable.
- Stage 3: source, restaurant, and customer are all the same node.
- Stage 3: restaurant equals the source — leg 1 has distance 0.
- Stage 3: restaurant equals the customer — leg 2 has distance 0.
- Very large weights — ensure no integer overflow issues (not a concern in Python, but worth mentioning).
