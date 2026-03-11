# Round 17: Binary Search — "Optimal Delivery Radius"

## Metadata
- **Topic:** Binary Search
- **Difficulty:** 2/5
- **Estimated Time:** 55-60 minutes
- **Key Concepts:** Classic binary search, search in rotated sorted array, bisect for boundary conditions, off-by-one reasoning

## Problem Prompt

*Read this to the candidate:*

> DoorDash is expanding into a new city. They have a list of restaurant distances from a central hub (in miles). Given a sorted list of these distances and a target distance, determine if there is a restaurant exactly at that distance.
>
> For example, given distances `[1.0, 2.5, 4.0, 6.5, 8.0, 10.0]` and target `6.5`, return the index of that restaurant. If the target is not found, return where it would be inserted to keep the list sorted.

Do **not** volunteer details about rotated arrays or the K-restaurant radius problem yet — those come in later stages.

## Clarifying Questions & Hidden Constraints

| Question the Candidate Should Ask | Answer |
|---|---|
| Are the distances sorted? | Yes, in ascending order (for Stage 1). |
| Are there duplicate distances? | For Stage 1, assume all distances are unique. We'll revisit this. |
| Are distances floating-point or integer? | They can be floating-point. |
| What should I return if the target isn't found? | Return the index where it would be inserted to maintain sorted order. |
| Is the list guaranteed to be non-empty? | You should handle empty lists gracefully. |
| Are distances always positive? | Yes, all distances are positive. |

## Skeleton Code

```python
from typing import List

def find_restaurant(distances: List[float], target: float) -> int:
    """
    Given a sorted list of restaurant distances, find the index of the target.
    If not found, return the insertion index to maintain sorted order.
    """
    pass

if __name__ == "__main__":
    # Expected: 3 (target found at index 3)
    print(find_restaurant([1.0, 2.5, 4.0, 6.5, 8.0, 10.0], 6.5))
    # Expected: 3 (insert between 4.0 and 6.5)
    print(find_restaurant([1.0, 2.5, 4.0, 6.5, 8.0, 10.0], 5.0))
    # Expected: 0 (empty list)
    print(find_restaurant([], 5.0))
```

---

## Stage 1: Standard Binary Search with Insertion Point (~20 minutes)

### Expected Approach

1. Classic binary search with `lo = 0`, `hi = len(distances) - 1`.
2. While `lo <= hi`, compute `mid = lo + (hi - lo) // 2` (avoids overflow in other languages; good habit).
3. Compare `distances[mid]` with `target`:
   - If equal, return `mid`.
   - If `distances[mid] < target`, search right: `lo = mid + 1`.
   - If `distances[mid] > target`, search left: `hi = mid - 1`.
4. If the loop ends without finding the target, `lo` is the correct insertion index.
5. The candidate should explain *why* `lo` is the insertion point: at loop exit, `lo > hi`, and `lo` is the first index where `distances[lo] > target` (or `lo == len(distances)` if target is larger than all elements).

### Solution Code

```python
from typing import List

def find_restaurant(distances: List[float], target: float) -> int:
    lo, hi = 0, len(distances) - 1

    while lo <= hi:
        mid = lo + (hi - lo) // 2

        if distances[mid] == target:
            return mid
        elif distances[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1

    # target not found; lo is the insertion point
    return lo
```

### Complexity

- **Time:** O(log n) — halves the search space each iteration.
- **Space:** O(1) — only a few variables.

### Test Cases

```python
# Target found
assert find_restaurant([1.0, 2.5, 4.0, 6.5, 8.0, 10.0], 6.5) == 3

# Target not found — insert in the middle
assert find_restaurant([1.0, 2.5, 4.0, 6.5, 8.0, 10.0], 5.0) == 3

# Target smaller than all — insert at beginning
assert find_restaurant([1.0, 2.5, 4.0], 0.5) == 0

# Target larger than all — insert at end
assert find_restaurant([1.0, 2.5, 4.0], 5.0) == 3

# Single element — found
assert find_restaurant([3.0], 3.0) == 0

# Single element — not found (smaller)
assert find_restaurant([3.0], 1.0) == 0

# Single element — not found (larger)
assert find_restaurant([3.0], 5.0) == 1

# Empty list
assert find_restaurant([], 5.0) == 0
```

---

## Stage 2: Search in a Rotated Sorted Array (~20 minutes)

### Prompt Addition

> Good. Now the scenario changes. The restaurants are positioned along a circular road around the city center. The distance list was originally sorted, but has been "rotated" — some number of elements from the front were moved to the back.
>
> For example, `[6.5, 8.0, 10.0, 1.0, 2.5, 4.0]` is a rotation of the original sorted list. Given this rotated list and a target, find the index of the target. Return -1 if not found.
>
> As a follow-up, can you also find the K-th smallest element in this rotated array?

### Starter Code

```python
def search_rotated(distances: List[float], target: float) -> int:
    """
    Search for target in a rotated sorted array.
    Returns the index if found, -1 otherwise.
    Assumes all elements are unique.
    """
    pass


def kth_smallest_rotated(distances: List[float], k: int) -> float:
    """
    Find the K-th smallest element (1-indexed) in a rotated sorted array.
    """
    pass

# --- Stage 2 Test Calls ---
# Expected: 3 (target 1.0 is at index 3)
print(search_rotated([6.5, 8.0, 10.0, 1.0, 2.5, 4.0], 1.0))
# Expected: -1 (not found)
print(search_rotated([6.5, 8.0, 10.0, 1.0, 2.5, 4.0], 5.0))
# Expected: -1 (empty)
print(search_rotated([], 1.0))
```

### Expected Approach

**Search in rotated sorted array:**

1. The key insight: at any midpoint, **at least one half** of the array is properly sorted.
2. Determine which half is sorted by comparing `distances[lo]` with `distances[mid]`.
3. If the left half is sorted (`distances[lo] <= distances[mid]`):
   - Check if target lies within `[distances[lo], distances[mid]]`. If so, search left. Otherwise, search right.
4. Otherwise, the right half is sorted:
   - Check if target lies within `(distances[mid], distances[hi]]`. If so, search right. Otherwise, search left.
5. Return `mid` if found, `-1` otherwise.

**K-th smallest element:**

1. Find the rotation pivot (index of the minimum element) using binary search.
2. The K-th smallest is at index `(pivot + K - 1) % n` (using 1-indexed K).

### Solution Code

```python
from typing import List

def search_rotated(distances: List[float], target: float) -> int:
    """
    Search for target in a rotated sorted array.
    Returns the index if found, -1 otherwise.
    Assumes all elements are unique.
    """
    if not distances:
        return -1

    lo, hi = 0, len(distances) - 1

    while lo <= hi:
        mid = lo + (hi - lo) // 2

        if distances[mid] == target:
            return mid

        # Left half is sorted
        if distances[lo] <= distances[mid]:
            if distances[lo] <= target < distances[mid]:
                hi = mid - 1
            else:
                lo = mid + 1
        # Right half is sorted
        else:
            if distances[mid] < target <= distances[hi]:
                lo = mid + 1
            else:
                hi = mid - 1

    return -1


def find_pivot(distances: List[float]) -> int:
    """
    Find the index of the minimum element in a rotated sorted array.
    This is the pivot/rotation point.
    """
    if not distances:
        return -1

    lo, hi = 0, len(distances) - 1

    # If not rotated at all
    if distances[lo] <= distances[hi]:
        return 0

    while lo <= hi:
        mid = lo + (hi - lo) // 2

        # Check if mid is the pivot
        if mid > 0 and distances[mid] < distances[mid - 1]:
            return mid

        # If left side is sorted, pivot is in the right side
        if distances[lo] <= distances[mid]:
            lo = mid + 1
        else:
            hi = mid

    return 0


def kth_smallest_rotated(distances: List[float], k: int) -> float:
    """
    Find the K-th smallest element (1-indexed) in a rotated sorted array.
    """
    if not distances or k < 1 or k > len(distances):
        raise ValueError("Invalid input")

    pivot = find_pivot(distances)
    n = len(distances)
    # The k-th smallest element is at this index
    idx = (pivot + k - 1) % n
    return distances[idx]
```

### Complexity

- **Time:** O(log n) for both search and pivot finding.
- **Space:** O(1).

### Test Cases

```python
# Search in rotated array — target exists
assert search_rotated([6.5, 8.0, 10.0, 1.0, 2.5, 4.0], 1.0) == 3
assert search_rotated([6.5, 8.0, 10.0, 1.0, 2.5, 4.0], 8.0) == 1
assert search_rotated([6.5, 8.0, 10.0, 1.0, 2.5, 4.0], 6.5) == 0
assert search_rotated([6.5, 8.0, 10.0, 1.0, 2.5, 4.0], 4.0) == 5

# Target not found
assert search_rotated([6.5, 8.0, 10.0, 1.0, 2.5, 4.0], 5.0) == -1

# Not rotated (rotation by 0)
assert search_rotated([1.0, 2.5, 4.0, 6.5], 4.0) == 2

# Single element
assert search_rotated([3.0], 3.0) == 0
assert search_rotated([3.0], 5.0) == -1

# Empty
assert search_rotated([], 1.0) == -1

# Find pivot
assert find_pivot([6.5, 8.0, 10.0, 1.0, 2.5, 4.0]) == 3
assert find_pivot([1.0, 2.5, 4.0, 6.5]) == 0  # Not rotated

# K-th smallest in rotated array
assert kth_smallest_rotated([6.5, 8.0, 10.0, 1.0, 2.5, 4.0], 1) == 1.0
assert kth_smallest_rotated([6.5, 8.0, 10.0, 1.0, 2.5, 4.0], 3) == 4.0
assert kth_smallest_rotated([6.5, 8.0, 10.0, 1.0, 2.5, 4.0], 6) == 10.0
```

---

## Stage 3: Minimum Radius for K Restaurants (~15 minutes)

### Prompt Addition

> Final scenario. You have a sorted (non-rotated) list of restaurant distances from the hub. DoorDash wants to find the **minimum delivery radius R** such that **at least K restaurants** are within distance R (i.e., distance <= R).
>
> Given the sorted list and K, return the minimum such R. If K is 0, return 0. If K exceeds the number of restaurants, return -1 (impossible).
>
> Be careful with boundary conditions.

### Starter Code

```python
def min_radius_for_k(distances: List[float], k: int) -> float:
    """
    Given a sorted list of restaurant distances, find the minimum radius R
    such that at least K restaurants have distance <= R.

    Returns R if possible, -1 if K > len(distances), 0 if K <= 0.
    """
    pass

# --- Stage 3 Test Calls ---
# Expected: 4.0
print(min_radius_for_k([1.0, 2.5, 4.0, 6.5, 8.0, 10.0], 3))
# Expected: 1.0
print(min_radius_for_k([1.0, 2.5, 4.0], 1))
# Expected: -1.0 (impossible)
print(min_radius_for_k([1.0, 2.5], 5))
```

### Expected Approach

1. Since the array is sorted, the K-th smallest distance is at index `K - 1` (0-indexed). The minimum radius to cover at least K restaurants is exactly `distances[K - 1]`.
2. This is an O(1) lookup in a sorted array — but the interviewer should push the candidate to **prove correctness using `bisect_right`**: the number of restaurants within radius R is `bisect_right(distances, R)`.
3. For the full solution, use `bisect_left` and `bisect_right` to handle duplicates:
   - Multiple restaurants at the same distance mean the candidate must be precise about `<=` vs `<`.
   - `bisect_right(distances, R)` gives the count of elements `<= R`.
   - `bisect_left(distances, R)` gives the count of elements `< R`.
4. Discuss off-by-one errors: the difference between "at least K" vs "exactly K" and how `bisect_left`/`bisect_right` distinguish these.

### Solution Code

```python
from typing import List
from bisect import bisect_left, bisect_right

def min_radius_for_k(distances: List[float], k: int) -> float:
    """
    Given a sorted list of restaurant distances, find the minimum radius R
    such that at least K restaurants have distance <= R.

    Returns R if possible, -1 if K > len(distances), 0 if K <= 0.
    """
    if k <= 0:
        return 0.0
    if k > len(distances):
        return -1.0

    # The K-th smallest distance (1-indexed) is at index k-1
    # Setting R = distances[k-1] ensures exactly bisect_right(distances, R)
    # restaurants are within radius R, which is >= k.
    return distances[k - 1]


def count_within_radius(distances: List[float], radius: float) -> int:
    """
    Count how many restaurants are within the given radius (distance <= radius).
    Uses bisect_right for correct boundary handling.
    """
    return bisect_right(distances, radius)


def count_strictly_within_radius(distances: List[float], radius: float) -> int:
    """
    Count how many restaurants are strictly within the radius (distance < radius).
    Uses bisect_left for correct boundary handling.
    """
    return bisect_left(distances, radius)


def verify_radius(distances: List[float], k: int) -> bool:
    """
    Demonstrates the full verification: find R, then verify using bisect.
    """
    r = min_radius_for_k(distances, k)
    if r == -1.0:
        return False  # impossible
    count = count_within_radius(distances, r)
    return count >= k


def min_radius_binary_search(distances: List[float], k: int) -> float:
    """
    Alternative approach using binary search on the answer.
    This is useful when the candidate doesn't immediately see the O(1) indexing solution,
    or when the problem is generalized (e.g., distances not stored but computed on the fly).

    Binary search on the radius value, checking how many restaurants fall within it.
    """
    if k <= 0:
        return 0.0
    if not distances or k > len(distances):
        return -1.0

    lo, hi = 0, len(distances) - 1

    while lo < hi:
        mid = lo + (hi - lo) // 2
        # If we set radius = distances[mid], how many restaurants are covered?
        # Since array is sorted, distances[0..mid] are all <= distances[mid].
        # But there might be duplicates after mid with the same value.
        count = bisect_right(distances, distances[mid])

        if count >= k:
            hi = mid  # Try a smaller radius
        else:
            lo = mid + 1  # Need a larger radius

    return distances[lo]
```

### Complexity

- **Direct indexing approach:** O(1) time, O(1) space (given the array is already sorted).
- **Binary search on answer approach:** O(log n) time, O(1) space.
- **Verification with bisect:** O(log n) time.

### Test Cases

```python
# Basic case
assert min_radius_for_k([1.0, 2.5, 4.0, 6.5, 8.0, 10.0], 3) == 4.0
assert count_within_radius([1.0, 2.5, 4.0, 6.5, 8.0, 10.0], 4.0) == 3

# K = 1 — smallest distance
assert min_radius_for_k([1.0, 2.5, 4.0], 1) == 1.0

# K = all restaurants
assert min_radius_for_k([1.0, 2.5, 4.0], 3) == 4.0

# K = 0
assert min_radius_for_k([1.0, 2.5, 4.0], 0) == 0.0

# K > N — impossible
assert min_radius_for_k([1.0, 2.5], 5) == -1.0

# Duplicates: distances = [1.0, 2.0, 2.0, 2.0, 5.0]
# K = 3 → R = 2.0 covers indices 0,1,2 → at least 3 restaurants
assert min_radius_for_k([1.0, 2.0, 2.0, 2.0, 5.0], 3) == 2.0
# But count_within_radius at R=2.0 is 4 (includes all 2.0 duplicates)
assert count_within_radius([1.0, 2.0, 2.0, 2.0, 5.0], 2.0) == 4

# bisect_left vs bisect_right distinction
assert count_strictly_within_radius([1.0, 2.0, 2.0, 2.0, 5.0], 2.0) == 1
assert count_within_radius([1.0, 2.0, 2.0, 2.0, 5.0], 2.0) == 4

# Binary search approach matches direct approach
assert min_radius_binary_search([1.0, 2.5, 4.0, 6.5, 8.0, 10.0], 3) == 4.0
assert min_radius_binary_search([1.0, 2.0, 2.0, 2.0, 5.0], 3) == 2.0

# Verify correctness
assert verify_radius([1.0, 2.5, 4.0, 6.5, 8.0, 10.0], 3) == True
assert verify_radius([1.0, 2.5], 5) == False
```

---

## Scoring Rubric

### Fundamentals (1-4)

| Score | Criteria |
|-------|----------|
| **4 — Strong** | Implements standard binary search correctly on the first attempt. Handles rotated array confidently — identifies the "one half is always sorted" invariant without hints. Immediately sees that the Stage 3 radius problem is an index lookup. Explains `bisect_left` vs `bisect_right` semantics precisely. |
| **3 — Solid** | Implements standard binary search correctly. Handles rotated array with minor hesitation but gets the logic right. Arrives at the Stage 3 solution, possibly via binary search on the answer rather than direct indexing. Understands bisect usage. |
| **2 — Developing** | Standard binary search has off-by-one errors (e.g., `lo < hi` vs `lo <= hi`) but is corrected with testing. Struggles significantly with the rotated array — may need strong hints. Stage 3 is partially solved. |
| **1 — Insufficient** | Cannot implement binary search correctly. Doesn't understand the loop invariant. Cannot reason about rotated arrays. |

### Coding (1-4)

| Score | Criteria |
|-------|----------|
| **4 — Strong** | Clean, bug-free code for all three stages. Proper use of `lo + (hi - lo) // 2`. Edge cases handled (empty array, single element, not found). Uses Python's `bisect` module where appropriate. |
| **3 — Solid** | Code works for all stages with at most one bug fixed during testing. Handles most edge cases. |
| **2 — Developing** | Persistent off-by-one errors. May cause infinite loops (e.g., forgetting `lo = mid + 1`). Code works for simple cases but fails edge cases. |
| **1 — Insufficient** | Code does not terminate or produces wrong answers for basic cases. |

### Communication (1-4)

| Score | Criteria |
|-------|----------|
| **4 — Strong** | Asks about sorted order, duplicates, and edge cases. Explains the loop invariant clearly. For rotated array, draws or talks through which half is sorted. Discusses `bisect_left` vs `bisect_right` proactively. Identifies off-by-one risks and addresses them explicitly. |
| **3 — Solid** | Explains their approach before coding. Can trace through an example. Discusses boundary conditions when prompted. |
| **2 — Developing** | Starts coding immediately. Gives vague explanations. Cannot clearly articulate why `lo` is the insertion point. |
| **1 — Insufficient** | Cannot explain binary search verbally. Does not discuss their approach. |

---

## Interviewer Guide

### Hints (per stage, progressive: mild, medium, strong)

**Stage 1 — Standard Binary Search**

| Level | Hint |
|-------|------|
| **Mild** | "What happens to `lo` and `hi` when the loop exits without finding the target?" |
| **Medium** | "At loop exit, `lo > hi`. Think about what `lo` represents — it's the first position where all elements to the left are smaller than target." |
| **Strong** | "When target is not found, return `lo`. This is the insertion point. It's the same value Python's `bisect_left` would return." |

**Stage 2 — Rotated Sorted Array**

| Level | Hint |
|-------|------|
| **Mild** | "At any midpoint, what can you say about the two halves of the array?" |
| **Medium** | "One of the two halves is always fully sorted. You can check which one by comparing `distances[lo]` with `distances[mid]`. Once you know which half is sorted, you can determine if the target could be in that half." |
| **Strong** | "If `distances[lo] <= distances[mid]`, the left half is sorted. Check if `distances[lo] <= target < distances[mid]` — if so, go left; otherwise go right. Mirror logic for when the right half is sorted." |

**Stage 3 — Minimum Radius for K**

| Level | Hint |
|-------|------|
| **Mild** | "The array is sorted. If you need at least K restaurants within the radius, what's the smallest possible radius?" |
| **Medium** | "The K-th smallest distance determines the minimum radius. Where is that in a sorted array?" |
| **Strong** | "R = distances[K-1]. Verify with `bisect_right(distances, R)` which counts elements <= R." |

### When to Intervene

- If the candidate writes `while lo < hi` instead of `while lo <= hi` for the standard "find exact target" variant and doesn't catch the bug, ask them to trace through a single-element array.
- If the candidate uses `mid = (lo + hi) // 2` — this is fine in Python (no overflow). Mention that `lo + (hi - lo) // 2` is the safe habit for other languages, but don't penalize.
- If the candidate gets stuck on rotated array for more than 8 minutes, give the medium hint about "one half is always sorted."
- If the candidate over-engineers Stage 3 (e.g., writing a full binary search on a continuous radius value), acknowledge the approach works but guide them to see the simpler O(1) indexing solution.

### Common Mistakes

1. **Off-by-one in loop termination:** Using `lo < hi` when the algorithm requires `lo <= hi` (or vice versa), leading to missing the target when it's the last element checked.
2. **Infinite loops:** Forgetting to advance `lo` or `hi` — particularly `lo = mid + 1` vs `lo = mid` in the rotated array variant.
3. **Rotated array: wrong sorted-half check.** Using `<` instead of `<=` in `distances[lo] <= distances[mid]`, failing when `lo == mid`.
4. **`bisect_left` vs `bisect_right` confusion.** Using the wrong one changes whether boundary elements are counted, which matters with duplicates.
5. **Returning `mid` instead of `lo` for insertion point.** After the loop, `mid` holds a stale value; the correct insertion point is `lo`.
6. **Stage 3: Off-by-one with K.** Returning `distances[K]` instead of `distances[K-1]` for 1-indexed K.

### Edge Cases to Watch For

- **Empty array:** Should return 0 (Stage 1 insertion), -1 (Stage 2 not found), or handle gracefully (Stage 3).
- **Single element array:** All stages should work.
- **Target at boundaries:** First element, last element.
- **Array not actually rotated (rotation = 0):** Stage 2 should still work.
- **All elements identical:** `[3.0, 3.0, 3.0]` — binary search should find it; rotated version is the same array; radius for K=2 is `3.0`.
- **K = 0:** Radius is 0 (no restaurants needed).
- **K = N:** Radius is the maximum distance.
- **K > N:** Return -1 (impossible).
