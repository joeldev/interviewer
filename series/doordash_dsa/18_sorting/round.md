# Round 18: Sorting Algorithms — "Order Fulfillment Prioritization"

## Metadata
- **Topic:** Sorting Algorithms
- **Difficulty:** 2/5
- **Estimated Time:** 55-60 minutes
- **Key Concepts:** Merge sort, stable sorting, multi-key comparisons, nearly-sorted optimization, min-heap

## Problem Prompt

*Read this to the candidate:*

> You're building the order fulfillment engine for a DoorDash kitchen. Pending delivery orders arrive and need to be sorted so the kitchen knows which to prepare first. Each order has an estimated delivery time (minutes from now) and a priority flag (express or standard).
>
> Given a list of orders, sort them by estimated delivery time so that the most urgent orders come first.

Do **not** volunteer the priority flag sorting, stability requirements, or the nearly-sorted optimization. Let the candidate ask and let the stages reveal these naturally.

## Clarifying Questions & Hidden Constraints

| Question the Candidate Should Ask | Answer |
|---|---|
| What does each order look like? | Each order is an object (or tuple) with at least an `order_id`, `delivery_time` (integer, minutes), and `priority` ("express" or "standard"). |
| Can delivery times be equal? | Yes. When times are equal, maintain the original relative order (this hints at stability). |
| Can I use Python's built-in sort? | For this exercise, please implement the sort from scratch. We want to see you write the algorithm. |
| How large can the input be? | Up to 10^5 orders. Your solution should be O(n log n). |
| Are delivery times always positive? | Yes, positive integers. |
| What about the priority flag? | We'll get to that. For now, just sort by delivery time. |

## Skeleton Code

```python
from typing import List, Tuple

# An order is represented as (order_id: str, delivery_time: int, priority: str)
Order = Tuple[str, int, str]

def sort_orders(orders: List[Order]) -> List[Order]:
    """
    Sort orders by delivery_time in ascending order.
    Must implement the sorting algorithm from scratch (no built-in sort).
    The sort must be stable.
    """
    pass

if __name__ == "__main__":
    orders = [
        ("ORD-001", 30, "standard"),
        ("ORD-002", 10, "standard"),
        ("ORD-003", 20, "express"),
    ]
    # Expected: [('ORD-002', 10, 'standard'), ('ORD-003', 20, 'express'), ('ORD-001', 30, 'standard')]
    print(sort_orders(orders))
    # Expected: [] (empty)
    print(sort_orders([]))
    # Stability test — Expected: same order (all equal times)
    print(sort_orders([("A", 15, "standard"), ("B", 15, "express"), ("C", 15, "standard")]))
```

---

## Stage 1: Implement Merge Sort (~20 minutes)

### Expected Approach

1. **Merge sort** is the right choice here because:
   - It's O(n log n) worst case (unlike quicksort which is O(n^2) worst case).
   - It's **stable** — equal elements maintain their original relative order. This will matter in Stage 2.
2. Implement the full divide-and-conquer:
   - **Base case:** array of length 0 or 1 is already sorted.
   - **Split:** divide the array into two halves.
   - **Recurse:** sort each half.
   - **Merge:** merge two sorted halves into one sorted array.
3. The merge step is the core: use two pointers, compare elements from each half, and build the result. For stability, when elements are equal, take from the **left** half first.

### Solution Code

```python
from typing import List, Tuple

Order = Tuple[str, int, str]

def sort_orders(orders: List[Order]) -> List[Order]:
    """
    Merge sort implementation. Sorts orders by delivery_time ascending.
    Stable: equal delivery_time elements maintain original relative order.
    """
    if len(orders) <= 1:
        return orders[:]

    mid = len(orders) // 2
    left = sort_orders(orders[:mid])
    right = sort_orders(orders[mid:])

    return _merge(left, right)


def _merge(left: List[Order], right: List[Order]) -> List[Order]:
    """Merge two sorted lists into one sorted list. Stable."""
    result = []
    i, j = 0, 0

    while i < len(left) and j < len(right):
        # Compare by delivery_time (index 1)
        # Use <= for stability: take from left when equal
        if left[i][1] <= right[j][1]:
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

- **Time:** O(n log n) — the array is split log n times, and each level of recursion does O(n) work in the merge step.
- **Space:** O(n) — merge sort requires O(n) auxiliary space for the merged arrays. The recursion stack is O(log n).

### Test Cases

```python
# Basic sorting by delivery time
orders_1 = [
    ("ORD-001", 30, "standard"),
    ("ORD-002", 10, "standard"),
    ("ORD-003", 20, "express"),
]
result = sort_orders(orders_1)
assert [o[0] for o in result] == ["ORD-002", "ORD-003", "ORD-001"]

# Stability test: equal delivery times preserve original order
orders_2 = [
    ("ORD-A", 15, "standard"),
    ("ORD-B", 15, "express"),
    ("ORD-C", 15, "standard"),
]
result = sort_orders(orders_2)
assert [o[0] for o in result] == ["ORD-A", "ORD-B", "ORD-C"]

# Already sorted
orders_3 = [
    ("ORD-X", 5, "express"),
    ("ORD-Y", 10, "standard"),
    ("ORD-Z", 15, "express"),
]
result = sort_orders(orders_3)
assert [o[0] for o in result] == ["ORD-X", "ORD-Y", "ORD-Z"]

# Reverse sorted
orders_4 = [
    ("ORD-1", 30, "standard"),
    ("ORD-2", 20, "standard"),
    ("ORD-3", 10, "express"),
]
result = sort_orders(orders_4)
assert [o[0] for o in result] == ["ORD-3", "ORD-2", "ORD-1"]

# Empty and single element
assert sort_orders([]) == []
assert sort_orders([("ORD-ONLY", 5, "express")]) == [("ORD-ONLY", 5, "express")]
```

---

## Stage 2: Multi-Key Sorting with Priority (~20 minutes)

### Prompt Addition

> Now orders have two sort criteria. Express orders should come before standard orders. Within the same priority group, sort by delivery time ascending.
>
> So the sort order is:
> 1. Express orders first, then standard orders.
> 2. Within each group, earlier delivery times first.
>
> Modify your merge sort to handle this. Can you think of more than one approach?

### Starter Code

```python
def sort_orders_priority(orders: List[Order]) -> List[Order]:
    """
    Sort orders: express before standard, then by delivery_time ascending.
    Must be stable.
    """
    pass

# --- Stage 2 Test Calls ---
# Expected: [ORD-002(express,10), ORD-004(express,15), ORD-005(express,25), ORD-003(standard,20), ORD-001(standard,30)]
print(sort_orders_priority([
    ("ORD-001", 30, "standard"), ("ORD-002", 10, "express"),
    ("ORD-003", 20, "standard"), ("ORD-004", 15, "express"), ("ORD-005", 25, "express"),
]))
# Stability test — Expected: [A(express,10), B(express,10), C(standard,10), D(standard,10)]
print(sort_orders_priority([("A", 10, "express"), ("B", 10, "express"), ("C", 10, "standard"), ("D", 10, "standard")]))
```

### Expected Approach

**Approach 1 — Modify the comparator in merge:**

Change the comparison in the merge step to use a composite key: `(priority_rank, delivery_time)` where express = 0 and standard = 1. This way the existing merge sort naturally handles multi-key sorting.

**Approach 2 — Exploit stability with two passes:**

Since merge sort is stable, sort by delivery time first (secondary key), then sort by priority (primary key). The second stable sort preserves the delivery time ordering within each priority group. This demonstrates deep understanding of stable sort properties.

Both approaches are correct. Approach 1 is more practical; Approach 2 demonstrates conceptual depth. A strong candidate discusses both.

### Solution Code

```python
from typing import List, Tuple

Order = Tuple[str, int, str]

# Approach 1: Composite key in merge
def sort_orders_priority(orders: List[Order]) -> List[Order]:
    """
    Sort orders: express before standard, then by delivery_time ascending.
    Uses a composite sort key in the merge step.
    """
    if len(orders) <= 1:
        return orders[:]

    mid = len(orders) // 2
    left = sort_orders_priority(orders[:mid])
    right = sort_orders_priority(orders[mid:])

    return _merge_priority(left, right)


def _sort_key(order: Order) -> Tuple[int, int]:
    """Composite key: (priority_rank, delivery_time)."""
    priority_rank = 0 if order[2] == "express" else 1
    return (priority_rank, order[1])


def _merge_priority(left: List[Order], right: List[Order]) -> List[Order]:
    """Merge two sorted lists using composite key. Stable."""
    result = []
    i, j = 0, 0

    while i < len(left) and j < len(right):
        # Use composite key for comparison; <= for stability
        if _sort_key(left[i]) <= _sort_key(right[j]):
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1

    result.extend(left[i:])
    result.extend(right[j:])

    return result


# Approach 2: Two stable sorts (secondary key first, then primary key)
def sort_orders_two_pass(orders: List[Order]) -> List[Order]:
    """
    Demonstrates stable sort property.
    First sort by delivery_time (secondary key), then by priority (primary key).
    Because the sort is stable, the delivery_time order is preserved within
    each priority group after the second sort.
    """
    # Pass 1: sort by delivery_time (secondary key)
    by_time = _merge_sort_by(orders, key_index=1)

    # Pass 2: sort by priority (primary key)
    # We need to map priority to a numeric value for comparison
    by_priority = _merge_sort_by_priority(by_time)

    return by_priority


def _merge_sort_by(orders: List[Order], key_index: int) -> List[Order]:
    """Merge sort by a specific tuple index."""
    if len(orders) <= 1:
        return orders[:]

    mid = len(orders) // 2
    left = _merge_sort_by(orders[:mid], key_index)
    right = _merge_sort_by(orders[mid:], key_index)

    result = []
    i, j = 0, 0
    while i < len(left) and j < len(right):
        if left[i][key_index] <= right[j][key_index]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    result.extend(left[i:])
    result.extend(right[j:])
    return result


def _merge_sort_by_priority(orders: List[Order]) -> List[Order]:
    """Merge sort by priority: express (0) before standard (1). Stable."""
    def prio(order):
        return 0 if order[2] == "express" else 1

    if len(orders) <= 1:
        return orders[:]

    mid = len(orders) // 2
    left = _merge_sort_by_priority(orders[:mid])
    right = _merge_sort_by_priority(orders[mid:])

    result = []
    i, j = 0, 0
    while i < len(left) and j < len(right):
        if prio(left[i]) <= prio(right[j]):
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    result.extend(left[i:])
    result.extend(right[j:])
    return result
```

### Complexity

- **Approach 1:** O(n log n) time, O(n) space — same as standard merge sort.
- **Approach 2:** O(n log n) time, O(n) space — two passes but each is O(n log n), so total is O(n log n).

### Test Cases

```python
# Express before standard, then by delivery time
orders_1 = [
    ("ORD-001", 30, "standard"),
    ("ORD-002", 10, "express"),
    ("ORD-003", 20, "standard"),
    ("ORD-004", 15, "express"),
    ("ORD-005", 25, "express"),
]
result = sort_orders_priority(orders_1)
assert [o[0] for o in result] == [
    "ORD-002",  # express, time=10
    "ORD-004",  # express, time=15
    "ORD-005",  # express, time=25
    "ORD-003",  # standard, time=20
    "ORD-001",  # standard, time=30
]

# Verify both approaches produce the same result
result_two_pass = sort_orders_two_pass(orders_1)
assert result == result_two_pass

# Stability within same priority and time
orders_2 = [
    ("ORD-A", 10, "express"),
    ("ORD-B", 10, "express"),
    ("ORD-C", 10, "standard"),
    ("ORD-D", 10, "standard"),
]
result = sort_orders_priority(orders_2)
assert [o[0] for o in result] == ["ORD-A", "ORD-B", "ORD-C", "ORD-D"]

# All express
orders_3 = [
    ("ORD-1", 30, "express"),
    ("ORD-2", 10, "express"),
    ("ORD-3", 20, "express"),
]
result = sort_orders_priority(orders_3)
assert [o[0] for o in result] == ["ORD-2", "ORD-3", "ORD-1"]

# All standard
orders_4 = [
    ("ORD-1", 5, "standard"),
    ("ORD-2", 3, "standard"),
]
result = sort_orders_priority(orders_4)
assert [o[0] for o in result] == ["ORD-2", "ORD-1"]
```

---

## Stage 3: Nearly Sorted Orders — Efficient Sort (~15 minutes)

### Prompt Addition

> New scenario. In practice, orders arrive roughly in delivery-time order but not perfectly — each order is at most K positions away from its final sorted position. For example, if K = 3, an order that belongs at index 5 could currently be at index 2, 3, 4, 5, 6, 7, or 8.
>
> Given this "nearly sorted" guarantee and the value K, implement an efficient sort that beats O(n log n). What approaches are possible, and what are their tradeoffs?

### Starter Code

```python
def sort_nearly_sorted_insertion(orders: List[Order], k: int) -> List[Order]:
    """
    Sort a nearly-sorted list where each element is at most k positions
    from its final sorted position. Uses insertion sort.
    Time: O(nk), Space: O(1) (in-place).
    """
    pass


def sort_nearly_sorted_heap(orders: List[Order], k: int) -> List[Order]:
    """
    Sort a nearly-sorted list where each element is at most k positions
    from its final sorted position. Uses a min-heap of size k+1.
    Time: O(n log k), Space: O(k).
    """
    pass

# --- Stage 3 Test Calls ---
# orders = [("B", 10, "standard"), ("A", 5, "express"), ("C", 15, "standard"),
#           ("E", 25, "express"), ("D", 20, "standard")]
# Expected delivery times: [5, 10, 15, 20, 25]
# print([o[1] for o in sort_nearly_sorted_insertion(orders, k=2)])
# print([o[1] for o in sort_nearly_sorted_heap(orders, k=2)])
# Expected: [] (empty)
# print(sort_nearly_sorted_insertion([], k=5))
# print(sort_nearly_sorted_heap([], k=5))
```

### Expected Approach

**Approach 1 — Insertion sort: O(nK)**

In a nearly-sorted array where each element is at most K positions from its target, insertion sort is efficient because each element moves at most K positions during insertion. The inner loop runs at most K times for each element.

- Time: O(nK). When K is small (e.g., K = 3), this is effectively O(n).
- Space: O(1) — in-place.
- Simple to implement.

**Approach 2 — Min-heap of size K+1: O(n log K)**

Maintain a min-heap of size K+1. The minimum element in the heap is guaranteed to be the next element in sorted order (because it can be at most K positions away, and we have the next K+1 candidates).

1. Insert the first K+1 elements into the heap.
2. Extract the min and place it in the result. Insert the next element from the array.
3. Repeat until all elements are processed.
4. Drain the remaining elements from the heap.

- Time: O(n log K). Each of the n elements is inserted and extracted from a heap of size at most K+1.
- Space: O(K) for the heap.

The candidate should discuss when each is preferred: insertion sort when K is very small (say, K < 10) due to lower constant factors; heap approach when K is larger but still much less than N.

### Solution Code

```python
from typing import List, Tuple
import heapq

Order = Tuple[str, int, str]

def sort_nearly_sorted_insertion(orders: List[Order], k: int) -> List[Order]:
    """
    Sort a nearly-sorted list where each element is at most k positions
    from its final sorted position. Uses insertion sort.

    Time: O(nk), Space: O(1) (in-place).
    """
    result = orders[:]  # Work on a copy to avoid mutating input
    n = len(result)

    for i in range(1, n):
        key = result[i]
        j = i - 1

        # Only need to look back at most k positions
        while j >= max(0, i - k) and result[j][1] > key[1]:
            result[j + 1] = result[j]
            j -= 1

        result[j + 1] = key

    return result


def sort_nearly_sorted_heap(orders: List[Order], k: int) -> List[Order]:
    """
    Sort a nearly-sorted list where each element is at most k positions
    from its final sorted position. Uses a min-heap of size k+1.

    Time: O(n log k), Space: O(k).
    """
    if not orders:
        return []

    n = len(orders)
    result = []

    # Build a min-heap from the first k+1 elements.
    # Heap elements: (delivery_time, original_index, order) to ensure stable ordering.
    heap = []
    window = min(k + 1, n)

    for i in range(window):
        heapq.heappush(heap, (orders[i][1], i, orders[i]))

    # For each remaining element, extract min and insert new element
    idx = window
    for i in range(n):
        # Extract the smallest from current window
        _, _, order = heapq.heappop(heap)
        result.append(order)

        # Insert next element if available
        if idx < n:
            heapq.heappush(heap, (orders[idx][1], idx, orders[idx]))
            idx += 1

    return result
```

### Complexity

| Approach | Time | Space | Best When |
|----------|------|-------|-----------|
| Insertion sort | O(nK) | O(1) | K is very small (K << log n) |
| Min-heap | O(n log K) | O(K) | K is moderate (log n < K << n) |
| Full merge sort | O(n log n) | O(n) | K is unknown or large |

### Test Cases

```python
# Nearly sorted with k=2
orders_1 = [
    ("ORD-B", 10, "standard"),  # correct position: 0
    ("ORD-A", 5, "express"),    # correct position: 0 → currently at 1 (1 away)
    ("ORD-C", 15, "standard"),  # correct position: 2
    ("ORD-E", 25, "express"),   # correct position: 4 → currently at 3 (1 away)
    ("ORD-D", 20, "standard"),  # correct position: 3 → currently at 4 (1 away)
]
result_ins = sort_nearly_sorted_insertion(orders_1, k=2)
result_heap = sort_nearly_sorted_heap(orders_1, k=2)

assert [o[1] for o in result_ins] == [5, 10, 15, 20, 25]
assert [o[1] for o in result_heap] == [5, 10, 15, 20, 25]

# Already sorted (k=0 still works)
orders_2 = [
    ("ORD-1", 10, "standard"),
    ("ORD-2", 20, "express"),
    ("ORD-3", 30, "standard"),
]
assert [o[1] for o in sort_nearly_sorted_insertion(orders_2, k=0)] == [10, 20, 30]
assert [o[1] for o in sort_nearly_sorted_heap(orders_2, k=0)] == [10, 20, 30]

# k = n-1 (worst case, equivalent to full sort)
orders_3 = [
    ("ORD-C", 30, "standard"),
    ("ORD-B", 20, "express"),
    ("ORD-A", 10, "standard"),
]
assert [o[1] for o in sort_nearly_sorted_insertion(orders_3, k=2)] == [10, 20, 30]
assert [o[1] for o in sort_nearly_sorted_heap(orders_3, k=2)] == [10, 20, 30]

# Single element
orders_4 = [("ORD-ONLY", 42, "express")]
assert sort_nearly_sorted_insertion(orders_4, k=0) == orders_4
assert sort_nearly_sorted_heap(orders_4, k=0) == orders_4

# Empty
assert sort_nearly_sorted_insertion([], k=5) == []
assert sort_nearly_sorted_heap([], k=5) == []

# Larger test: verify both approaches agree
import random
random.seed(42)
base = [(f"ORD-{i}", i * 10, "express" if i % 2 == 0 else "standard") for i in range(100)]
# Shuffle each element at most k=3 positions
nearly = base[:]
for i in range(len(nearly)):
    swap_with = min(len(nearly) - 1, i + random.randint(0, 3))
    nearly[i], nearly[swap_with] = nearly[swap_with], nearly[i]
result_ins = sort_nearly_sorted_insertion(nearly, k=3)
result_heap = sort_nearly_sorted_heap(nearly, k=3)
assert [o[1] for o in result_ins] == [o[1] for o in result_heap]
```

---

## Scoring Rubric

### Fundamentals (1-4)

| Score | Criteria |
|-------|----------|
| **4 — Strong** | Implements merge sort correctly and efficiently from scratch. Understands and can explain stability. Correctly identifies both insertion sort and heap-based approaches for nearly-sorted data and articulates the tradeoffs. Knows when each approach is optimal. |
| **3 — Solid** | Implements merge sort correctly. Understands stability after brief discussion. Gets one of the two nearly-sorted approaches right and can discuss the other when prompted. |
| **2 — Developing** | Merge sort has bugs in the merge step (e.g., loses elements, unstable). May choose quicksort instead and need prompting about stability. Gets insertion sort for Stage 3 but misses the heap approach. |
| **1 — Insufficient** | Cannot implement merge sort. Does not understand stability. Cannot optimize for nearly-sorted input. |

### Coding (1-4)

| Score | Criteria |
|-------|----------|
| **4 — Strong** | Merge sort is clean and handles all edge cases. Multi-key comparison is elegantly implemented (composite key or two-pass). Heap-based nearly-sorted solution uses `heapq` correctly with proper tie-breaking. All test cases pass. |
| **3 — Solid** | Code works with at most one bug caught during testing. Minor issues like forgetting to handle the remaining elements in merge. Correct overall structure. |
| **2 — Developing** | Multiple bugs. Merge step loses elements or produces incorrect order. Off-by-one errors. Heap usage is incorrect (e.g., wrong heap size). |
| **1 — Insufficient** | Code does not run. Merge sort missing key steps. Cannot use `heapq`. |

### Communication (1-4)

| Score | Criteria |
|-------|----------|
| **4 — Strong** | Explains why merge sort is chosen (stable, guaranteed O(n log n)). Proactively discusses stability before Stage 2. Clearly explains the two-pass stable sort trick. Articulates the O(nK) vs O(n log K) tradeoff for nearly-sorted data. Uses concrete examples to illustrate. |
| **3 — Solid** | Explains merge sort approach before coding. Understands stability when prompted. Can discuss tradeoffs for nearly-sorted approaches. |
| **2 — Developing** | Dives into code without explaining. Cannot explain stability. Gives incomplete complexity analysis. |
| **1 — Insufficient** | Cannot explain merge sort. Does not discuss algorithmic choices. Cannot analyze complexity. |

---

## Interviewer Guide

### Hints (per stage, progressive: mild, medium, strong)

**Stage 1 — Merge Sort**

| Level | Hint |
|-------|------|
| **Mild** | "What O(n log n) sorting algorithms do you know? Which ones are stable?" |
| **Medium** | "Merge sort has three steps: split in half, sort each half recursively, then merge. Start with the merge function — given two sorted lists, how do you combine them?" |
| **Strong** | "For the merge: use two pointers, one for each half. Compare current elements, take the smaller one. When elements are equal, take from the left half to maintain stability. Don't forget to append the remaining elements." |

**Stage 2 — Multi-Key Sorting**

| Level | Hint |
|-------|------|
| **Mild** | "How can you encode the two sort criteria into a single comparison?" |
| **Medium** | "Map priority to a number (express=0, standard=1) and compare tuples: (priority_rank, delivery_time). Python tuple comparison works lexicographically." |
| **Strong** | "Alternative approach: since your sort is stable, you can sort by the secondary key first (delivery_time), then sort by the primary key (priority). The first ordering is preserved within groups of equal priority." |

**Stage 3 — Nearly Sorted**

| Level | Hint |
|-------|------|
| **Mild** | "If each element is at most K positions away, how far does insertion sort need to shift each element?" |
| **Medium** | "Insertion sort's inner loop runs at most K times per element, giving O(nK). Can you do better with a data structure that efficiently finds the minimum of K+1 elements?" |
| **Strong** | "Maintain a min-heap of size K+1. The minimum in the heap is guaranteed to be the correct next element. Extract min, insert the next element from the array. Each heap operation is O(log K), and you do n of them: O(n log K)." |

### When to Intervene

- If the candidate tries to write **quicksort** in Stage 1, ask: "Is quicksort stable? Does that matter here?" If they say no or are unsure, explain that stability will matter in Stage 2 and redirect to merge sort.
- If the merge step is **losing elements** (e.g., forgetting `result.extend(left[i:])`), have them trace through a small example where the lists have unequal lengths.
- If the candidate is stuck on the **composite key** for Stage 2, suggest: "What if you turned each order's sort criteria into a tuple and compared tuples?"
- If the candidate jumps to the heap solution in Stage 3 without discussing insertion sort, ask: "What's the simplest approach first? What's its complexity?"
- If the candidate struggles with **heapq** usage, remind them: `heapq.heappush(heap, (priority, tiebreaker, item))` and that Python compares tuples element by element.

### Common Mistakes

1. **Unstable merge:** Using `<` instead of `<=` when comparing equal elements in the merge step, causing the right-half element to be taken first and breaking stability.
2. **Forgetting remaining elements:** After the main merge loop, one list may still have elements. A common bug is only appending from one list.
3. **Not returning a new list:** Modifying the input list in place when the function signature implies returning a new list, or vice versa.
4. **Incorrect composite key for priority:** Mapping "express" to 1 and "standard" to 0 (reversed), causing standard orders to appear before express.
5. **Heap without tie-breaking:** When two orders have the same delivery time, Python's `heapq` will try to compare the next tuple element. If orders aren't comparable, this crashes. Using the original index as a tiebreaker prevents this.
6. **Wrong heap size for nearly-sorted:** Using a heap of size K instead of K+1. The correct window includes the current position plus K positions in either direction, but since we process left to right, K+1 elements suffice.
7. **Insertion sort scanning too far back:** Not limiting the inner loop to K positions, defeating the purpose of the optimization.

### Edge Cases to Watch For

- **Empty list:** Should return empty.
- **Single element:** Should return as-is.
- **All identical delivery times:** Tests stability — output order should match input order.
- **All express or all standard:** Multi-key sort should still work correctly.
- **K = 0 for nearly sorted:** Array is already sorted; both approaches should handle this (insertion sort does nothing, heap of size 1 just passes elements through).
- **K = N - 1:** Fully unsorted. Insertion sort degrades to O(n^2); heap approach is O(n log n). The candidate should note this.
- **Large N with small K:** This is the sweet spot for Stage 3 optimizations. Emphasize that O(n log K) with K << N is much better than O(n log n).
