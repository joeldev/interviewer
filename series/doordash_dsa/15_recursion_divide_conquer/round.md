# Round 15: Recursion & Divide and Conquer — "Delivery Zone Partitioning"

## Metadata
- **Topic:** Recursion, Divide and Conquer, Backtracking
- **Difficulty:** 4/5
- **Estimated Time:** 55-60 minutes
- **Key Concepts:** Recursive subset generation (include/exclude), merge sort, closest pair of points with strip optimization

## Problem Prompt

> DoorDash is expanding to new cities and needs to analyze delivery zones.
> Delivery zones are identified by labels, deliveries have associated distances,
> and drop-off points have geographic coordinates.
>
> You will be asked to perform several recursive partitioning operations on this
> delivery data.

*(Read the prompt exactly as written. The problem intentionally starts vague.
Stage-specific prompts will add detail. Let the candidate ask clarifying
questions.)*

## Clarifying Questions & Hidden Constraints

| # | What the candidate should ask | Answer |
|---|-------------------------------|--------|
| 1 | What kind of data are we working with — strings, numbers, coordinates? | It depends on the stage. Stage 1 uses a list of zone labels (strings or integers). Stage 2 uses a list of numeric distances. Stage 3 uses 2D coordinates. |
| 2 | Can there be duplicate values? | Stage 1: assume all zone labels are unique. Stage 2: distances may have duplicates. Stage 3: no two points share the same coordinates. |
| 3 | What is the expected input size? | Stage 1: up to ~20 elements (exponential output). Stage 2: up to 100,000 elements. Stage 3: up to 100,000 points. |
| 4 | Should solutions be iterative or recursive? | Recursive solutions are expected for this round. |
| 5 | For Stage 3, are we working in Euclidean space? | Yes. Standard 2D Euclidean distance. |

## Skeleton Code

```python
from typing import List, Tuple
import math

def power_set(zones: List[str]) -> List[List[str]]:
    """
    Stage 1: Return all subsets of the given delivery zones.
    """
    pass

if __name__ == "__main__":
    # Expected: 8 subsets (2^3), including [] and ["A", "B", "C"]
    result = power_set(["A", "B", "C"])
    print(f"Count: {len(result)}, Subsets: {result}")
    # Expected: [[]] (one subset: the empty set)
    print(power_set([]))
    # Expected: 2 subsets: [] and ["Downtown"]
    print(power_set(["Downtown"]))
```

---

## Stage 1: Power Set of Delivery Zones (~20 minutes)

### Expected Approach

Use the **include/exclude** recursion pattern:

1. Base case: when the index reaches the end of the list, yield the current subset.
2. Recursive case: for the current element, make two recursive calls:
   - **Exclude** the current element (recurse without adding it).
   - **Include** the current element (add it to the current subset, then recurse).
3. Collect all subsets from both branches.

The candidate should recognize that the output has 2^n subsets and discuss why this is inherently exponential.

### Solution Code

```python
def power_set(zones: List[str]) -> List[List[str]]:
    result = []

    def backtrack(index: int, current: List[str]) -> None:
        if index == len(zones):
            result.append(current[:])  # append a copy
            return

        # Exclude zones[index]
        backtrack(index + 1, current)

        # Include zones[index]
        current.append(zones[index])
        backtrack(index + 1, current)
        current.pop()  # undo the choice

    backtrack(0, [])
    return result
```

**Alternative iterative-style recursion** (also acceptable):

```python
def power_set_alt(zones: List[str]) -> List[List[str]]:
    if not zones:
        return [[]]

    rest = power_set_alt(zones[1:])
    # Every subset either includes zones[0] or not
    return rest + [[zones[0]] + subset for subset in rest]
```

### Complexity

| | |
|---|---|
| **Time** | O(n * 2^n) — there are 2^n subsets, and copying each subset takes up to O(n). |
| **Space** | O(n * 2^n) for the output. Recursion stack depth is O(n). |

### Test Cases

```python
# Test 1: Small set
result = power_set(["A", "B", "C"])
assert len(result) == 8  # 2^3
assert [] in result
assert ["A", "B", "C"] in result
assert ["A"] in result
assert ["B", "C"] in result

# Test 2: Empty set
result = power_set([])
assert result == [[]]

# Test 3: Single element
result = power_set(["Downtown"])
assert sorted(result) == sorted([[], ["Downtown"]])

# Test 4: Two elements
result = power_set(["X", "Y"])
assert len(result) == 4
assert [] in result
assert ["X"] in result
assert ["Y"] in result
assert ["X", "Y"] in result
```

---

## Stage 2: Merge Sort on Delivery Distances (~20 minutes)

### Prompt Addition

> "DoorDash needs to sort delivery distances to compute percentiles and detect
> outliers. Implement merge sort on a list of delivery distances. This must be
> a true divide-and-conquer solution — not a call to Python's built-in sort."

### Starter Code

```python
def merge_sort(distances: List[float]) -> List[float]:
    """
    Stage 2: Sort the list of delivery distances using merge sort.
    Return a new sorted list.
    """
    pass

# --- Stage 2 Test Calls ---
# Expected: [0.8, 1.5, 2.1, 3.2, 4.7]
print(merge_sort([3.2, 1.5, 4.7, 0.8, 2.1]))
# Expected: [1, 2, 3, 4, 5]
print(merge_sort([5, 4, 3, 2, 1]))
# Expected: []
print(merge_sort([]))
```

### Expected Approach

Classic merge sort:

1. **Base case:** if the list has 0 or 1 elements, it is already sorted.
2. **Divide:** split the list into two halves at the midpoint.
3. **Conquer:** recursively sort each half.
4. **Merge:** combine two sorted halves into one sorted list by comparing elements from the front of each half.

The candidate should clearly separate the recursive splitting from the merge step. The merge function should use two pointers.

### Solution Code

```python
def merge_sort(distances: List[float]) -> List[float]:
    if len(distances) <= 1:
        return distances[:]

    mid = len(distances) // 2
    left = merge_sort(distances[:mid])
    right = merge_sort(distances[mid:])

    return _merge(left, right)


def _merge(left: List[float], right: List[float]) -> List[float]:
    result = []
    i = j = 0

    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1

    # Append remaining elements
    result.extend(left[i:])
    result.extend(right[j:])

    return result
```

### Complexity

| | |
|---|---|
| **Time** | O(n log n) — the list is split into halves log n times, and each level of recursion does O(n) work during the merge step. |
| **Space** | O(n) — the merge step creates new lists. Recursion stack depth is O(log n). |

### Test Cases

```python
# Test 1: Unsorted distances
assert merge_sort([3.2, 1.5, 4.7, 0.8, 2.1]) == [0.8, 1.5, 2.1, 3.2, 4.7]

# Test 2: Already sorted
assert merge_sort([1, 2, 3, 4, 5]) == [1, 2, 3, 4, 5]

# Test 3: Reverse sorted
assert merge_sort([5, 4, 3, 2, 1]) == [1, 2, 3, 4, 5]

# Test 4: Duplicates
assert merge_sort([3, 1, 3, 2, 1]) == [1, 1, 2, 3, 3]

# Test 5: Empty list
assert merge_sort([]) == []

# Test 6: Single element
assert merge_sort([42.0]) == [42.0]

# Test 7: Two elements out of order
assert merge_sort([7, 3]) == [3, 7]

# Test 8: Large uniform list
assert merge_sort([5] * 100) == [5] * 100
```

---

## Stage 3: Closest Pair of Delivery Points (~15 minutes)

### Prompt Addition

> "DoorDash wants to identify the two closest delivery drop-off points in a
> city — these might indicate duplicate orders or opportunities to batch
> deliveries. Given a list of 2D coordinates `(x, y)`, find the pair of points
> with the smallest Euclidean distance between them.
>
> A brute-force O(n^2) approach is too slow for large cities. Can you do better
> using divide and conquer?"

### Starter Code

```python
def closest_pair(points: List[Tuple[float, float]]) -> Tuple[Tuple[float, float], Tuple[float, float], float]:
    """
    Stage 3: Find the closest pair of delivery drop-off points.
    Return (point1, point2, distance).
    """
    pass

# --- Stage 3 Test Calls ---
# Expected: points (0,0) and (1,1) with distance ~1.414
# print(closest_pair([(0, 0), (3, 4), (1, 1), (5, 5)]))
# Expected: points (-1,0) and (1,0) with distance 2.0
# print(closest_pair([(-1, 0), (1, 0), (10, 10), (-10, -10)]))
# Expected: points (0,0) and (3,4) with distance 5.0
# print(closest_pair([(0, 0), (3, 4)]))
```

### Expected Approach

The classic **closest pair of points** algorithm:

1. Sort points by x-coordinate.
2. **Base case:** if 3 or fewer points, solve by brute force.
3. **Divide:** split the sorted points into left and right halves at the median x-coordinate.
4. **Conquer:** recursively find the closest pair in each half. Let `d` = min of the two results.
5. **Combine (strip):** The closest pair might span the two halves. Consider only points within distance `d` of the dividing x-coordinate. Sort these strip points by y-coordinate. For each point in the strip, compare it to at most the next 7 points (a well-known constant bound). Update `d` if a closer pair is found.

The candidate should explain why the strip check is O(n) amortized despite the nested loop — each point compares to at most a constant number of others due to the geometric packing argument.

### Solution Code

```python
def closest_pair(points: List[Tuple[float, float]]) -> Tuple[Tuple[float, float], Tuple[float, float], float]:
    def euclidean_dist(p1: Tuple[float, float], p2: Tuple[float, float]) -> float:
        return math.sqrt((p1[0] - p2[0]) ** 2 + (p1[1] - p2[1]) ** 2)

    def brute_force(pts: List[Tuple[float, float]]) -> Tuple[Tuple[float, float], Tuple[float, float], float]:
        min_dist = float('inf')
        best = (pts[0], pts[1])
        for i in range(len(pts)):
            for j in range(i + 1, len(pts)):
                d = euclidean_dist(pts[i], pts[j])
                if d < min_dist:
                    min_dist = d
                    best = (pts[i], pts[j])
        return (best[0], best[1], min_dist)

    def closest_pair_rec(pts_sorted_x: List[Tuple[float, float]]) -> Tuple[Tuple[float, float], Tuple[float, float], float]:
        n = len(pts_sorted_x)

        # Base case: brute force for small inputs
        if n <= 3:
            return brute_force(pts_sorted_x)

        # Divide
        mid = n // 2
        mid_x = pts_sorted_x[mid][0]
        left_half = pts_sorted_x[:mid]
        right_half = pts_sorted_x[mid:]

        # Conquer
        left_result = closest_pair_rec(left_half)
        right_result = closest_pair_rec(right_half)

        # Best so far
        if left_result[2] < right_result[2]:
            best_p1, best_p2, d = left_result
        else:
            best_p1, best_p2, d = right_result

        # Combine: build the strip
        strip = [p for p in pts_sorted_x if abs(p[0] - mid_x) < d]

        # Sort strip by y-coordinate
        strip.sort(key=lambda p: p[1])

        # Check strip pairs — at most 7 comparisons per point
        for i in range(len(strip)):
            j = i + 1
            while j < len(strip) and (strip[j][1] - strip[i][1]) < d:
                pair_dist = euclidean_dist(strip[i], strip[j])
                if pair_dist < d:
                    d = pair_dist
                    best_p1, best_p2 = strip[i], strip[j]
                j += 1

        return (best_p1, best_p2, d)

    # Sort by x-coordinate initially
    sorted_points = sorted(points, key=lambda p: p[0])
    return closest_pair_rec(sorted_points)
```

### Complexity

| | |
|---|---|
| **Time** | O(n log^2 n) — the recursion has O(log n) levels, each does O(n) work for the strip construction, but sorting the strip at each level adds an extra O(log n) factor. This can be improved to O(n log n) by pre-sorting by y and maintaining that order through the recursion, but O(n log^2 n) is acceptable for an interview. |
| **Space** | O(n) — for the strip and sorted copies. Recursion stack is O(log n). |

### Test Cases

```python
# Test 1: Simple case
p1, p2, d = closest_pair([(0, 0), (3, 4), (1, 1), (5, 5)])
assert abs(d - math.sqrt(2)) < 1e-9  # (0,0) and (1,1)
assert set([p1, p2]) == {(0, 0), (1, 1)}

# Test 2: Horizontal line
p1, p2, d = closest_pair([(0, 0), (2, 0), (5, 0), (9, 0)])
assert abs(d - 2.0) < 1e-9  # (0,0) and (2,0)

# Test 3: Vertical line
p1, p2, d = closest_pair([(0, 0), (0, 3), (0, 7), (0, 10)])
assert abs(d - 3.0) < 1e-9  # (0,0) and (0,3) or (0,7) and (0,10)

# Test 4: Closest pair spans the divide
p1, p2, d = closest_pair([(-1, 0), (1, 0), (10, 10), (-10, -10)])
assert abs(d - 2.0) < 1e-9  # (-1,0) and (1,0)
assert set([p1, p2]) == {(-1, 0), (1, 0)}

# Test 5: Minimum input (2 points)
p1, p2, d = closest_pair([(0, 0), (3, 4)])
assert abs(d - 5.0) < 1e-9

# Test 6: Clustered points
p1, p2, d = closest_pair([(0, 0), (100, 100), (0.1, 0.1), (50, 50)])
assert abs(d - math.sqrt(0.01 + 0.01)) < 1e-9  # (0,0) and (0.1, 0.1)
```

---

## Scoring Rubric

### Fundamentals (1-4)

| Score | Description |
|-------|-------------|
| 1 | Cannot explain recursion or divide and conquer. Confuses base case and recursive case. Does not understand why recursion terminates. |
| 2 | Understands basic recursion but struggles with the include/exclude pattern. Can describe merge sort at a high level but cannot implement the merge step. Unaware of closest pair algorithm. |
| 3 | Implements power set cleanly. Implements merge sort with correct merge logic. Understands the closest pair approach conceptually and makes solid progress on implementation with minor hints on the strip optimization. |
| 4 | All three stages solved correctly. Explains the include/exclude tree, merge sort recurrence (T(n) = 2T(n/2) + O(n)), and the geometric packing argument for the strip. Discusses time complexity via the Master Theorem. |

### Coding (1-4)

| Score | Description |
|-------|-------------|
| 1 | Code does not run. Base cases missing or recursion does not terminate. |
| 2 | Power set works but merge sort has bugs in the merge step (e.g., dropping elements, off-by-one). |
| 3 | Stages 1 and 2 are correct. Stage 3 has the right structure but may have a bug in strip construction or the y-sorted comparison logic. |
| 4 | All stages produce correct, clean, Pythonic code. Proper use of slicing, helper functions, and clean separation of divide/conquer/combine phases. |

### Communication (1-4)

| Score | Description |
|-------|-------------|
| 1 | Codes silently; cannot explain the recursion tree or why divide and conquer improves time complexity. |
| 2 | Explains after prompting but is vague about the recurrence relation or why the strip has at most O(n) useful comparisons. |
| 3 | Walks through the recursive structure before coding. Draws or describes the recursion tree. Explains the merge step clearly. Discusses why closest pair is better than brute force. |
| 4 | Proactively discusses the recursion tree, recurrence relations, and the Master Theorem. Explains the 2D geometric insight for the strip optimization. Considers practical concerns (stack overflow for deep recursion, in-place vs. out-of-place merge sort). |

---

## Interviewer Guide

### Hints (per stage, progressive: mild -> medium -> strong)

**Stage 1 — Power Set**

- **Mild:** "For each element, you have exactly two choices. What are they?"
- **Medium:** "At each recursive step, either include the current element in the subset or exclude it. Each choice leads to a separate branch. When you have made a choice for every element, you have one complete subset."
- **Strong:** "Write a helper `backtrack(index, current)`. Base case: if `index == len(zones)`, append a copy of `current` to the result. Recursive case: call `backtrack(index + 1, current)` for exclude, then append `zones[index]` to `current`, call `backtrack(index + 1, current)`, and pop it off."

**Stage 2 — Merge Sort**

- **Mild:** "The name gives it away: split, sort each half, then merge. What is the tricky part?"
- **Medium:** "The merge step takes two sorted lists and produces one sorted list. Use two pointers, always picking the smaller element. When one list is exhausted, append the rest of the other."
- **Strong:** "Split at `mid = len(arr) // 2`. Recursively sort `arr[:mid]` and `arr[mid:]`. Then merge: initialize two pointers `i = j = 0`, compare `left[i]` vs. `right[j]`, append the smaller, advance that pointer. After the loop, extend with the remaining elements."

**Stage 3 — Closest Pair**

- **Mild:** "Brute force is O(n^2). Can you split the points in half spatially and solve each half?"
- **Medium:** "Sort by x. Split into left/right halves. Recursively find the closest pair in each half, call that distance `d`. The tricky part is: the true closest pair might have one point in each half. These points must both be within distance `d` of the dividing line."
- **Strong:** "Build a strip of points within `d` of the mid x-coordinate. Sort the strip by y. For each point, compare to the next few points in the strip (at most 7, due to a geometric packing argument). This keeps the combine step linear."

### When to Intervene

- If the candidate writes an iterative bit-manipulation power set, acknowledge it works but ask them to convert to a recursive solution since that is the focus of this round.
- If merge sort's merge step is taking more than 10 minutes to debug, walk through a small example on paper: merging `[1, 3]` and `[2, 4]`.
- If the candidate has never seen the closest pair algorithm, give the medium hint for Stage 3 early. The goal is to see if they can reason about the strip optimization, not to test memorization.
- If time is short (less than 10 minutes left when reaching Stage 3), ask the candidate to explain the approach verbally and write pseudocode rather than full code.

### Common Mistakes

1. **Stage 1 — Forgetting to copy the current list.** Appending `current` instead of `current[:]` (or `list(current)`) means all entries in the result share the same mutable list and end up identical.
2. **Stage 1 — Not popping after the include branch.** The backtracking step is essential to undo the include choice before exploring the next sibling branch.
3. **Stage 2 — Off-by-one in the split.** Using `mid = len(arr) // 2` is correct for 0-indexed slicing. Using `mid - 1` or `mid + 1` causes infinite recursion on lists of length 2.
4. **Stage 2 — Not handling the remaining elements after the merge loop.** Forgetting `result.extend(left[i:])` or `result.extend(right[j:])` drops elements.
5. **Stage 2 — Using `<` instead of `<=` in the merge comparison.** Using strict less-than still produces a correct sort, but `<=` preserves stability. Worth discussing.
6. **Stage 3 — Omitting the strip check.** Without checking pairs across the divide, the algorithm misses cases where the closest pair straddles the two halves.
7. **Stage 3 — Not sorting the strip by y-coordinate.** The constant-bound on comparisons (7 neighbors) only holds when the strip is sorted by y. Without this, the strip check degrades to O(n^2).
8. **Stage 3 — Using squared distance inconsistently.** Mixing `sqrt` and non-`sqrt` comparisons can produce wrong results. Either always use squared distance or always use actual distance.

### Edge Cases to Watch For

- **Stage 1:** Empty input returns `[[]]` (one subset: the empty set). Single element returns `[[], [element]]`.
- **Stage 2:** Empty list and single-element list are already sorted. All identical elements should not cause issues. Already-sorted and reverse-sorted inputs should produce correct output.
- **Stage 3:** Only two points (the answer is trivially that pair). Points along a line (all same x or all same y). Closest pair straddling the dividing line (tests the strip logic). Very large coordinate values (test numerical stability).
