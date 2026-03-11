# Round 13: DFS — "Delivery Dependency Resolution"

## Metadata
- **Topic:** Depth-First Search, Topological Sort, Cycle Detection
- **Difficulty:** 3/5
- **Estimated Time:** 55-60 minutes
- **Key Concepts:** DFS traversal, topological ordering, back-edge cycle detection, backtracking enumeration

## Problem Prompt

> DoorDash has a set of delivery tasks that must be completed during a shift. Some
> tasks depend on others — for example, "pick up from restaurant A" must happen
> before "deliver to customer A." Each task is identified by an integer ID, and
> dependencies are given as pairs `(a, b)` meaning task `a` must be completed
> before task `b`.
>
> Given the number of tasks and their dependencies, determine a valid order in
> which a dasher can execute all tasks.

*(Read the prompt exactly as written. Let the candidate ask about edge cases
before revealing the clarifying answers below.)*

## Clarifying Questions & Hidden Constraints

| # | What the candidate should ask | Answer |
|---|-------------------------------|--------|
| 1 | Are the task IDs always `0` through `n-1`? | Yes. There are `n` tasks labeled `0` to `n-1`. |
| 2 | Can the dependency graph have cycles? | Great question. For Stage 1, assume no cycles. We will address that in a follow-up. |
| 3 | Can there be multiple valid orderings? | Yes. Any valid topological order is acceptable for Stage 1. |
| 4 | Can there be tasks with no dependencies at all? | Yes. They can appear anywhere in the order as long as all their dependents come after them. |
| 5 | Can the dependency list be empty? | Yes. If there are no dependencies, any permutation of tasks is valid. |
| 6 | Can there be duplicate edges in the dependency list? | No, assume all dependency pairs are unique. |

## Skeleton Code

```python
from typing import List, Tuple, Optional

def find_task_order(num_tasks: int, dependencies: List[Tuple[int, int]]) -> List[int]:
    """
    Stage 1: Return a valid execution order for all tasks respecting
    dependencies. dependencies[i] = (a, b) means task a must come before task b.
    Assume no cycles exist.
    """
    pass

if __name__ == "__main__":
    # Expected: [0, 1, 2, 3] (simple chain)
    print(find_task_order(4, [(0, 1), (1, 2), (2, 3)]))
    # Expected: any valid topological order with 0 first and 3 last
    print(find_task_order(4, [(0, 1), (0, 2), (1, 3), (2, 3)]))
    # Expected: [0] (single task)
    print(find_task_order(1, []))
```

---

## Stage 1: Topological Sort via DFS (~20 minutes)

### Expected Approach

1. Build an adjacency list from the dependency pairs.
2. Perform DFS from each unvisited node.
3. After fully exploring all neighbors of a node (post-order), append it to a result list.
4. Reverse the result list at the end to get the topological order.

The key insight is that in a DFS-based topological sort, a node is added to the result *after* all of its descendants have been added, so reversing the post-order gives a valid topological ordering.

### Solution Code

```python
def find_task_order(num_tasks: int, dependencies: List[Tuple[int, int]]) -> List[int]:
    # Build adjacency list
    graph = [[] for _ in range(num_tasks)]
    for a, b in dependencies:
        graph[a].append(b)

    visited = [False] * num_tasks
    order = []

    def dfs(node: int) -> None:
        visited[node] = True
        for neighbor in graph[node]:
            if not visited[neighbor]:
                dfs(neighbor)
        order.append(node)  # post-order

    for task in range(num_tasks):
        if not visited[task]:
            dfs(task)

    order.reverse()
    return order
```

### Complexity

| | |
|---|---|
| **Time** | O(V + E) — each node and edge visited exactly once. |
| **Space** | O(V + E) — adjacency list plus recursion stack (up to O(V) deep). |

### Test Cases

```python
# Test 1: Simple chain
result = find_task_order(4, [(0, 1), (1, 2), (2, 3)])
assert result == [0, 1, 2, 3]

# Test 2: Diamond dependency
result = find_task_order(4, [(0, 1), (0, 2), (1, 3), (2, 3)])
# Valid orderings: [0, 1, 2, 3] or [0, 2, 1, 3]
assert result[0] == 0 and result[-1] == 3
assert result.index(0) < result.index(1)
assert result.index(0) < result.index(2)
assert result.index(1) < result.index(3)
assert result.index(2) < result.index(3)

# Test 3: No dependencies
result = find_task_order(3, [])
assert sorted(result) == [0, 1, 2]

# Test 4: Single task
result = find_task_order(1, [])
assert result == [0]

# Test 5: Two independent chains
result = find_task_order(4, [(0, 1), (2, 3)])
assert result.index(0) < result.index(1)
assert result.index(2) < result.index(3)
```

---

## Stage 2: Cycle Detection (~20 minutes)

### Prompt Addition

> "Nice work. Now, what if someone enters a circular dependency — task A depends
> on B, B depends on C, and C depends back on A? The system should detect this
> and report the cycle. Modify your approach so that if a cycle exists, you return
> the list of task IDs forming the cycle. If no cycle exists, return `None`."

### Starter Code

```python
def find_cycle(num_tasks: int, dependencies: List[Tuple[int, int]]) -> Optional[List[int]]:
    """
    Stage 2: If the dependency graph contains a cycle, return the list of
    task IDs forming that cycle (in order). If no cycle exists, return None.
    """
    pass

# --- Stage 2 Test Calls ---
# Expected: a cycle like [0, 1, 2, 0]
print(find_cycle(3, [(0, 1), (1, 2), (2, 0)]))
# Expected: None (no cycle)
print(find_cycle(4, [(0, 1), (0, 2), (1, 3), (2, 3)]))
# Expected: None (no edges)
print(find_cycle(3, []))
```

### Expected Approach

Use DFS with a three-state coloring scheme:

- **WHITE (0):** Not yet visited.
- **GRAY (1):** Currently on the DFS recursion stack (being explored).
- **BLACK (2):** Fully explored; all descendants processed.

When we encounter a GRAY node during DFS, we have found a back edge, which means a cycle exists. To reconstruct the cycle, maintain the current DFS path and extract the portion from the repeated node back to itself.

### Solution Code

```python
def find_cycle(num_tasks: int, dependencies: List[Tuple[int, int]]) -> Optional[List[int]]:
    WHITE, GRAY, BLACK = 0, 1, 2

    graph = [[] for _ in range(num_tasks)]
    for a, b in dependencies:
        graph[a].append(b)

    color = [WHITE] * num_tasks
    path = []  # current DFS path
    cycle_found = []

    def dfs(node: int) -> bool:
        """Returns True if a cycle is found."""
        color[node] = GRAY
        path.append(node)

        for neighbor in graph[node]:
            if color[neighbor] == GRAY:
                # Found a cycle: extract it from the path
                cycle_start = path.index(neighbor)
                cycle_found.extend(path[cycle_start:])
                cycle_found.append(neighbor)  # close the cycle
                return True
            if color[neighbor] == WHITE:
                if dfs(neighbor):
                    return True

        path.pop()
        color[node] = BLACK
        return False

    for task in range(num_tasks):
        if color[task] == WHITE:
            if dfs(task):
                return cycle_found

    return None
```

### Complexity

| | |
|---|---|
| **Time** | O(V + E) — standard DFS traversal. |
| **Space** | O(V) — color array, path, and recursion stack. |

### Test Cases

```python
# Test 1: Simple cycle
result = find_cycle(3, [(0, 1), (1, 2), (2, 0)])
assert result is not None
assert len(result) == 4  # e.g., [0, 1, 2, 0]
assert result[0] == result[-1]  # cycle closes

# Test 2: No cycle (DAG)
result = find_cycle(4, [(0, 1), (0, 2), (1, 3), (2, 3)])
assert result is None

# Test 3: Self-loop
result = find_cycle(2, [(0, 0)])
assert result is not None
assert result == [0, 0]

# Test 4: Cycle in a larger graph with non-cyclic parts
result = find_cycle(5, [(0, 1), (1, 2), (2, 3), (3, 1), (0, 4)])
# Cycle is 1 -> 2 -> 3 -> 1
assert result is not None
assert result[0] == result[-1]
cycle_nodes = set(result[:-1])
assert cycle_nodes == {1, 2, 3}

# Test 5: No edges
result = find_cycle(3, [])
assert result is None

# Test 6: Two separate cycles — returns the first found
result = find_cycle(4, [(0, 1), (1, 0), (2, 3), (3, 2)])
assert result is not None
assert result[0] == result[-1]
```

---

## Stage 3: Count All Valid Topological Orderings (~15 minutes)

### Prompt Addition

> "The operations team wants to understand how much flexibility exists in
> scheduling. Given the dependency graph (guaranteed no cycles), count the
> **total number of distinct valid orderings**. For example, if tasks 0 and 1 are
> independent and task 2 depends on both, the valid orderings are `[0, 1, 2]`
> and `[1, 0, 2]` — so the answer is 2.
>
> How would you compute this? How does the runtime scale?"

### Starter Code

```python
def count_valid_orderings(num_tasks: int, dependencies: List[Tuple[int, int]]) -> int:
    """
    Stage 3: Return the total number of distinct valid topological orderings.
    """
    pass

# --- Stage 3 Test Calls ---
# Expected: 6 (3! = 6, all independent)
print(count_valid_orderings(3, []))
# Expected: 1 (total order chain)
print(count_valid_orderings(4, [(0, 1), (1, 2), (2, 3)]))
# Expected: 2 (diamond shape)
print(count_valid_orderings(4, [(0, 1), (0, 2), (1, 3), (2, 3)]))
```

### Expected Approach

Use DFS with backtracking. Maintain in-degree counts. At each step, any node with in-degree 0 (and not yet placed) is a candidate for the next position. Try each candidate, decrement in-degrees of its neighbors, recurse, then undo (backtrack).

This is essentially generating all topological sorts. The time complexity is O(V! / (product of denominators)) in the best case, but can be O(V!) in the worst case (when all tasks are independent). The candidate should recognize and discuss this exponential nature.

### Solution Code

```python
def count_valid_orderings(num_tasks: int, dependencies: List[Tuple[int, int]]) -> int:
    graph = [[] for _ in range(num_tasks)]
    in_degree = [0] * num_tasks

    for a, b in dependencies:
        graph[a].append(b)
        in_degree[b] += 1

    visited = [False] * num_tasks
    count = 0

    def backtrack(placed: int) -> None:
        nonlocal count
        if placed == num_tasks:
            count += 1
            return

        for task in range(num_tasks):
            if not visited[task] and in_degree[task] == 0:
                # Choose this task next
                visited[task] = True
                for neighbor in graph[task]:
                    in_degree[neighbor] -= 1

                backtrack(placed + 1)

                # Undo the choice
                visited[task] = False
                for neighbor in graph[task]:
                    in_degree[neighbor] += 1

    backtrack(0)
    return count
```

### Complexity

| | |
|---|---|
| **Time** | O(V! * V) in the worst case (no edges: every permutation is valid, and at each level we scan all V nodes). Highly dependent on graph structure. |
| **Space** | O(V + E) — adjacency list, in-degree array, and recursion stack of depth V. |

### Test Cases

```python
# Test 1: Fully independent tasks — n! orderings
assert count_valid_orderings(3, []) == 6  # 3! = 6

# Test 2: Total order (chain) — only 1 ordering
assert count_valid_orderings(4, [(0, 1), (1, 2), (2, 3)]) == 1

# Test 3: Diamond shape
assert count_valid_orderings(4, [(0, 1), (0, 2), (1, 3), (2, 3)]) == 2
# Valid: [0, 1, 2, 3] and [0, 2, 1, 3]

# Test 4: Two independent chains
assert count_valid_orderings(4, [(0, 1), (2, 3)]) == 6
# Must have 0 before 1, and 2 before 3. The interleavings:
# [0, 1, 2, 3], [0, 2, 1, 3], [0, 2, 3, 1],
# [2, 0, 1, 3], [2, 0, 3, 1], [2, 3, 0, 1]

# Test 5: Single task
assert count_valid_orderings(1, []) == 1

# Test 6: Fan-out from one root
assert count_valid_orderings(4, [(0, 1), (0, 2), (0, 3)]) == 6
# 0 must be first, then 1, 2, 3 in any order -> 3! = 6
```

---

## Scoring Rubric

### Fundamentals (1-4)

| Score | Description |
|-------|-------------|
| 1 | Cannot explain DFS or how it relates to ordering tasks; no understanding of topological sort. |
| 2 | Knows DFS conceptually but cannot connect it to topological ordering. Attempts BFS (Kahn's) instead and struggles, or cannot handle cycle detection. |
| 3 | Implements topological sort correctly. Handles cycle detection with hints. Understands the backtracking approach for Stage 3 at a conceptual level. |
| 4 | Solves all three stages. Explains why DFS post-order gives topological order, why GRAY coloring detects back edges, and why counting orderings is factorial. |

### Coding (1-4)

| Score | Description |
|-------|-------------|
| 1 | Code does not run; major structural issues with recursion or graph representation. |
| 2 | Stage 1 works but cycle detection has bugs (e.g., false positives, does not reconstruct the cycle). |
| 3 | Stages 1 and 2 work correctly. Stage 3 is conceptually right but may have minor backtracking bugs (forgetting to undo in-degree changes). |
| 4 | All three stages produce correct, clean Python. Good use of adjacency lists, appropriate state tracking, and clean backtracking with undo. |

### Communication (1-4)

| Score | Description |
|-------|-------------|
| 1 | Codes silently; cannot explain DFS traversal or why the algorithm works. |
| 2 | Explains after prompting but is unclear on the distinction between pre-order and post-order, or between back edges and cross edges. |
| 3 | Explains approach before coding. Discusses the three-color DFS for cycle detection. Acknowledges the exponential nature of Stage 3. |
| 4 | Proactively asks clarifying questions. Draws out the graph mentally or on paper. Clearly explains trade-offs between DFS and BFS topological sort. Discusses computational complexity of counting orderings without prompting. |

---

## Interviewer Guide

### Hints (per stage, progressive: mild -> medium -> strong)

**Stage 1 — Topological Sort**

- **Mild:** "Think about what DFS does naturally. When does a node have all of its dependencies fully explored?"
- **Medium:** "If you add a node to your result list after you have finished exploring all its neighbors (post-order), what happens to the ordering? What if you then reverse it?"
- **Strong:** "Build an adjacency list. Run DFS, appending each node to a list after all its neighbors are done (post-order). Reverse that list. That is your topological order."

**Stage 2 — Cycle Detection**

- **Mild:** "During DFS, what does it mean if you encounter a node that you are currently in the middle of exploring?"
- **Medium:** "Consider three states for each node: unvisited, currently-being-explored (on the stack), and fully-done. A cycle exists if you ever visit a node that is in the currently-being-explored state."
- **Strong:** "Use WHITE/GRAY/BLACK coloring. WHITE = unvisited, GRAY = on the current DFS path, BLACK = done. If DFS reaches a GRAY node, that is a back edge and you have a cycle. Track the path to extract the cycle nodes."

**Stage 3 — Count Valid Orderings**

- **Mild:** "At each step, which tasks are eligible to be placed next? What property must they have?"
- **Medium:** "At any point, all tasks with in-degree 0 (among unplaced tasks) are candidates. You can try each one and recurse. This is a backtracking approach."
- **Strong:** "Maintain in-degree counts. At each step, try every unplaced task with in-degree 0. Place it, decrement in-degrees of its neighbors, recurse, then undo. Count how many times you place all V tasks. Discuss that this can be O(V!) in the worst case."

### When to Intervene

- If the candidate jumps to Kahn's algorithm (BFS-based topo sort) for Stage 1, that is perfectly fine — it also works. But gently steer them toward DFS since Stage 2 builds on DFS coloring.
- If the candidate is stuck on Stage 1 for more than 12 minutes, give the medium hint and let them code.
- If the candidate confuses "visited" with "on the current path" in Stage 2, draw a small example: nodes 0 -> 1 -> 2 -> 0 and trace the DFS states.
- If Stage 3 stalls, it is acceptable for the candidate to describe the approach verbally and write pseudocode rather than a full implementation, given time constraints.

### Common Mistakes

1. **Stage 1 — Forgetting to reverse the post-order list.** The raw post-order of DFS gives a reverse topological order. Candidates sometimes return it unreversed.
2. **Stage 1 — Not visiting all connected components.** If the graph is disconnected, the outer loop `for task in range(num_tasks)` is essential to visit every node.
3. **Stage 2 — Using only two states (visited/not visited) instead of three.** Two states cannot distinguish between a finished node and a node on the current path, leading to false cycle detection on cross edges.
4. **Stage 2 — Not reconstructing the actual cycle.** Many candidates detect that a cycle exists but cannot extract which nodes form it.
5. **Stage 3 — Forgetting to undo in-degree decrements during backtracking.** This corrupts the state for subsequent branches and produces wrong counts.
6. **Stage 3 — Not recognizing the factorial complexity.** The candidate should explain why this is computationally expensive and not suitable for large inputs.

### Edge Cases to Watch For

- Empty dependency list with `n > 0` — every permutation is valid (Stage 1 returns any order, Stage 3 returns `n!`).
- Single task — trivially valid.
- Self-loop `(0, 0)` — a cycle of length 1 (Stage 2 should detect it).
- Graph with exactly one valid ordering (total chain) — Stage 1 must return that exact order, Stage 3 returns 1.
- Disconnected graph — multiple independent components. Stage 1 must still include all nodes. Stage 3 count is the multinomial coefficient of interleaving the component orderings.
- Large fan-out from a single root — tests whether the candidate's Stage 3 backtracking handles many branches correctly.
