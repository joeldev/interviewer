# Round 11: Heaps — "Top K Nearest Dashers"

## Metadata
- **Topic:** Heaps (min-heap, max-heap, streaming top-K, multi-key sorting)
- **Difficulty:** 3/5
- **Estimated Time:** 55-60 minutes
- **Key Concepts:** Max-heap via negation with Python `heapq`, streaming/online algorithms, heap size maintenance, multi-criteria sorting

## Problem Prompt

> A customer just placed an order on DoorDash. We need to find the K nearest available Dashers to assign the delivery.
>
> You are given the customer's location and a list of Dasher locations. Find the K closest Dashers to the customer.

*Read this prompt verbatim. Do NOT specify the distance metric, how ties are broken, or the relationship between K and N. Let the candidate ask.*

## Clarifying Questions & Hidden Constraints

The candidate **should** ask these questions. Award communication points for each one asked unprompted.

| # | Question | Answer |
|---|----------|--------|
| 1 | How is distance measured? Euclidean? Manhattan? | Use **Euclidean distance**. Locations are `(x, y)` coordinates on a 2D plane. You may compare squared distances to avoid computing square roots. |
| 2 | What if two Dashers are the same distance from the customer? | For Stage 1 and 2, **either one is acceptable** — tie-breaking does not matter. Stage 3 introduces an explicit tie-breaking rule. |
| 3 | What is the relationship between K and N? Can K > N? | K is always <= N. You may assume valid input. |
| 4 | Can K be 0? | No, K >= 1. |
| 5 | How are Dashers identified? | Each Dasher has a unique integer `dasher_id`, plus `(x, y)` coordinates. In Stage 3, they also have a `rating` (float, 1.0 to 5.0). |
| 6 | How large can the input be? | Up to 1,000,000 Dashers. Sorting all of them is too slow if K is small relative to N. |
| 7 | Can multiple Dashers be at the same location? | Yes. |

## Skeleton Code

```python
import heapq
from typing import List, Tuple


def k_nearest_dashers(
    customer: Tuple[float, float],
    dashers: List[Tuple[int, float, float]],  # (dasher_id, x, y)
    k: int,
) -> List[int]:
    """
    Find the K nearest Dashers to the customer.

    Args:
        customer: (x, y) coordinates of the customer.
        dashers: List of (dasher_id, x, y) tuples.
        k: Number of nearest Dashers to return.

    Returns:
        List of dasher_ids of the K nearest Dashers (any order).
    """
    pass

if __name__ == "__main__":
    customer = (0.0, 0.0)
    dashers = [
        (1, 1.0, 0.0),   # dist = 1
        (2, 0.0, 2.0),   # dist = 2
        (3, 3.0, 0.0),   # dist = 3
        (4, 0.0, 0.5),   # dist = 0.5
        (5, 10.0, 10.0),  # dist ~14.14
    ]
    # Expected: [1, 2, 4] (in any order)
    print(sorted(k_nearest_dashers(customer, dashers, k=3)))
    # Expected: [30]
    print(k_nearest_dashers((5.0, 5.0), [(10, 5.0, 6.0), (20, 0.0, 0.0), (30, 5.0, 5.5)], k=1))
```

## Stage 1: K Nearest Dashers (~20 minutes)

### Prompt Addition

> "Given all Dasher locations at once, return the K nearest Dasher IDs. Think about efficiency — N could be very large, and K could be much smaller than N."

### Expected Approach

**Max-heap of size K:**

1. Compute the distance (or squared distance) from each Dasher to the customer.
2. Maintain a max-heap of size K. Since Python's `heapq` is a min-heap, store **negated distances** so that the largest distance is at the top.
3. For each Dasher:
   - If the heap has fewer than K elements, push.
   - Otherwise, if this Dasher's distance is smaller than the max in the heap (i.e., the negated top), pop the max and push the new Dasher.
4. The remaining K elements in the heap are the answer.

**Why not just sort?** Sorting is O(N log N). The heap approach is O(N log K), which is better when K is much smaller than N.

**Alternative: `heapq.nsmallest`** — Python provides this built-in, which internally uses a similar approach. Accept this if the candidate explains how it works under the hood.

### Solution Code

```python
import heapq
from typing import List, Tuple


def k_nearest_dashers(
    customer: Tuple[float, float],
    dashers: List[Tuple[int, float, float]],
    k: int,
) -> List[int]:
    cx, cy = customer
    # Max-heap of size K using negated distances.
    # Heap elements: (-squared_distance, dasher_id)
    max_heap = []

    for dasher_id, dx, dy in dashers:
        sq_dist = (dx - cx) ** 2 + (dy - cy) ** 2
        if len(max_heap) < k:
            heapq.heappush(max_heap, (-sq_dist, dasher_id))
        elif sq_dist < -max_heap[0][0]:
            # Current dasher is closer than the farthest in our top-K
            heapq.heapreplace(max_heap, (-sq_dist, dasher_id))

    return [dasher_id for _, dasher_id in max_heap]
```

### Complexity

- **Time:** O(N log K) — each of N Dashers may trigger a heap push/pop, each costing O(log K).
- **Space:** O(K) — the heap stores at most K elements.

### Test Cases

```python
# Test 1: Basic case
customer = (0.0, 0.0)
dashers = [
    (1, 1.0, 0.0),   # dist = 1
    (2, 0.0, 2.0),   # dist = 2
    (3, 3.0, 0.0),   # dist = 3
    (4, 0.0, 0.5),   # dist = 0.5
    (5, 10.0, 10.0),  # dist = ~14.14
]
result = k_nearest_dashers(customer, dashers, k=3)
assert sorted(result) == [1, 2, 4], f"Expected [1, 2, 4], got {sorted(result)}"

# Test 2: K equals N (return all)
customer2 = (0.0, 0.0)
dashers2 = [(1, 1.0, 0.0), (2, 2.0, 0.0)]
result2 = k_nearest_dashers(customer2, dashers2, k=2)
assert sorted(result2) == [1, 2], f"Expected [1, 2], got {sorted(result2)}"

# Test 3: K = 1 (single nearest)
customer3 = (5.0, 5.0)
dashers3 = [
    (10, 5.0, 6.0),   # dist = 1
    (20, 0.0, 0.0),   # dist = ~7.07
    (30, 5.0, 5.5),   # dist = 0.5  <-- closest
]
result3 = k_nearest_dashers(customer3, dashers3, k=1)
assert result3 == [30], f"Expected [30], got {result3}"

# Test 4: All Dashers equidistant
customer4 = (0.0, 0.0)
dashers4 = [(i, 1.0, 0.0) for i in range(5)]
result4 = k_nearest_dashers(customer4, dashers4, k=3)
assert len(result4) == 3, f"Expected 3 Dashers, got {len(result4)}"

print("All Stage 1 tests passed.")
```

## Stage 2: Streaming Dashers (~20 minutes)

### Prompt Addition

> "Now imagine Dashers don't all appear at once. They come online one at a time — a streaming scenario. Design a class that maintains the K nearest Dashers at all times. You need:
> - `add_dasher(dasher_id, x, y)` — called when a new Dasher comes online.
> - `get_nearest()` — returns the current K nearest Dasher IDs."

### Starter Code

```python
class DasherStream:
    """
    Maintains the K nearest Dashers as they arrive one at a time.
    """

    def __init__(self, customer: Tuple[float, float], k: int):
        pass

    def add_dasher(self, dasher_id: int, x: float, y: float) -> None:
        """Process a new Dasher arriving."""
        pass

    def get_nearest(self) -> List[int]:
        """Return the current K nearest Dasher IDs (any order)."""
        pass

# --- Stage 2 Test Calls ---
# stream = DasherStream(customer=(0.0, 0.0), k=2)
# stream.add_dasher(1, 10.0, 10.0)
# Expected: [1]
# print(sorted(stream.get_nearest()))
# stream.add_dasher(2, 1.0, 1.0)
# Expected: [1, 2]
# print(sorted(stream.get_nearest()))
# stream.add_dasher(3, 0.5, 0.5)
# Expected: [2, 3] (dasher 1 replaced)
# print(sorted(stream.get_nearest()))
```

### Expected Approach

Same max-heap logic as Stage 1, but encapsulated in a class:

1. The constructor stores the customer location and K.
2. `add_dasher` computes the distance and applies the same heap logic: push if heap is under capacity, else replace the farthest if the new Dasher is closer.
3. `get_nearest` extracts the IDs from the heap.

**Key discussion points:**
- `add_dasher` is O(log K) — constant per Dasher.
- `get_nearest` is O(K) — iterate the heap.
- The heap is maintained as an invariant: it always contains at most K elements, and they are the K closest seen so far.

### Solution Code

```python
class DasherStream:
    def __init__(self, customer: Tuple[float, float], k: int):
        self.cx, self.cy = customer
        self.k = k
        self.max_heap = []  # elements: (-sq_dist, dasher_id)

    def _squared_distance(self, x: float, y: float) -> float:
        return (x - self.cx) ** 2 + (y - self.cy) ** 2

    def add_dasher(self, dasher_id: int, x: float, y: float) -> None:
        sq_dist = self._squared_distance(x, y)
        if len(self.max_heap) < self.k:
            heapq.heappush(self.max_heap, (-sq_dist, dasher_id))
        elif sq_dist < -self.max_heap[0][0]:
            heapq.heapreplace(self.max_heap, (-sq_dist, dasher_id))

    def get_nearest(self) -> List[int]:
        return [dasher_id for _, dasher_id in self.max_heap]
```

### Complexity

- **`add_dasher`:** O(log K) per call.
- **`get_nearest`:** O(K) per call.
- **Space:** O(K) — the heap never exceeds K elements.

### Test Cases

```python
# Test 1: Stream of dashers
stream = DasherStream(customer=(0.0, 0.0), k=2)

stream.add_dasher(1, 10.0, 10.0)  # dist ~14.14
assert sorted(stream.get_nearest()) == [1]

stream.add_dasher(2, 1.0, 1.0)    # dist ~1.41
assert sorted(stream.get_nearest()) == [1, 2]

stream.add_dasher(3, 0.5, 0.5)    # dist ~0.71 — replaces dasher 1
result = sorted(stream.get_nearest())
assert result == [2, 3], f"Expected [2, 3], got {result}"

stream.add_dasher(4, 20.0, 20.0)  # dist ~28.28 — too far, ignored
result2 = sorted(stream.get_nearest())
assert result2 == [2, 3], f"Expected [2, 3], got {result2}"

stream.add_dasher(5, 0.1, 0.1)    # dist ~0.14 — replaces dasher 2
result3 = sorted(stream.get_nearest())
assert result3 == [3, 5], f"Expected [3, 5], got {result3}"

# Test 2: Stream with K=1
stream2 = DasherStream(customer=(5.0, 5.0), k=1)
stream2.add_dasher(10, 0.0, 0.0)
assert stream2.get_nearest() == [10]
stream2.add_dasher(20, 5.0, 5.1)  # much closer
assert stream2.get_nearest() == [20]

# Test 3: Fewer dashers than K
stream3 = DasherStream(customer=(0.0, 0.0), k=5)
stream3.add_dasher(1, 1.0, 0.0)
stream3.add_dasher(2, 2.0, 0.0)
assert len(stream3.get_nearest()) == 2  # only 2 dashers so far

print("All Stage 2 tests passed.")
```

## Stage 3: Multi-Key Ranking (~15 minutes)

### Prompt Addition

> "Final twist. Each Dasher now also has a rating (1.0 to 5.0). After finding the K nearest Dashers, return them sorted by:
> 1. **Rating descending** (higher-rated Dashers first).
> 2. **Distance ascending** as tie-breaker (if ratings are equal, closer Dasher first).
>
> Return the sorted list of Dasher IDs."

### Starter Code

```python
def k_nearest_by_rating(
    customer: Tuple[float, float],
    dashers: List[Tuple[int, float, float, float]],  # (dasher_id, x, y, rating)
    k: int,
) -> List[int]:
    """
    Find K nearest Dashers, then sort by rating descending.
    Ties in rating are broken by closer distance first.

    Returns:
        List of dasher_ids sorted by rating desc, then distance asc.
    """
    pass

# --- Stage 3 Test Calls ---
# customer = (0.0, 0.0)
# dashers = [
#     (1, 1.0, 0.0, 4.5), (2, 0.0, 2.0, 4.9),
#     (3, 0.5, 0.0, 3.0), (4, 0.0, 0.3, 4.5),
#     (5, 100.0, 100.0, 5.0),
# ]
# Expected: [2, 4, 1, 3]
# print(k_nearest_by_rating(customer, dashers, k=4))
# Expected: [2, 3, 1] (all same distance, sort by rating desc)
# print(k_nearest_by_rating((0.0, 0.0), [
#     (1, 1.0, 0.0, 3.0), (2, 0.0, 1.0, 5.0), (3, -1.0, 0.0, 4.0)
# ], k=3))
```

### Expected Approach

1. **Find K nearest** (reuse Stage 1 logic).
2. **Sort the K results** by `(-rating, distance)`.

This is a two-step process:
- Step 1 is O(N log K) to find the K nearest.
- Step 2 is O(K log K) to sort K elements.

**Key insight:** The sorting key is `(-rating, squared_distance)`. Negating the rating makes descending sort work with Python's default ascending sort. Squared distance preserves the relative ordering (no need for sqrt).

### Solution Code

```python
def k_nearest_by_rating(
    customer: Tuple[float, float],
    dashers: List[Tuple[int, float, float, float]],  # (dasher_id, x, y, rating)
    k: int,
) -> List[int]:
    cx, cy = customer

    # Step 1: Find K nearest using max-heap.
    # Heap elements: (-sq_dist, dasher_id, rating)
    max_heap = []

    for dasher_id, dx, dy, rating in dashers:
        sq_dist = (dx - cx) ** 2 + (dy - cy) ** 2
        if len(max_heap) < k:
            heapq.heappush(max_heap, (-sq_dist, dasher_id, rating))
        elif sq_dist < -max_heap[0][0]:
            heapq.heapreplace(max_heap, (-sq_dist, dasher_id, rating))

    # Step 2: Extract K nearest with their distances and ratings,
    # then sort by (-rating, distance).
    nearest = []
    for neg_sq_dist, dasher_id, rating in max_heap:
        sq_dist = -neg_sq_dist
        nearest.append((dasher_id, sq_dist, rating))

    # Sort: rating descending (negate), then distance ascending
    nearest.sort(key=lambda x: (-x[2], x[1]))

    return [dasher_id for dasher_id, _, _ in nearest]
```

### Complexity

- **Time:** O(N log K + K log K) — find K nearest, then sort K elements. Since K <= N, this simplifies to O(N log K).
- **Space:** O(K) — heap and sorted list.

### Test Cases

```python
# Test 1: Different ratings among nearest
customer = (0.0, 0.0)
dashers = [
    (1, 1.0, 0.0, 4.5),   # dist=1.0, rating=4.5
    (2, 0.0, 2.0, 4.9),   # dist=2.0, rating=4.9
    (3, 0.5, 0.0, 3.0),   # dist=0.5, rating=3.0
    (4, 0.0, 0.3, 4.5),   # dist=0.3, rating=4.5
    (5, 100.0, 100.0, 5.0),  # far away, won't be in top K
]
result = k_nearest_by_rating(customer, dashers, k=4)
# K=4 nearest: dashers 1 (d=1.0), 2 (d=2.0), 3 (d=0.5), 4 (d=0.3)
# Sort by rating desc, then dist asc:
#   Dasher 2: rating=4.9, dist=2.0
#   Dasher 4: rating=4.5, dist=0.3  (tied rating with 1, closer)
#   Dasher 1: rating=4.5, dist=1.0
#   Dasher 3: rating=3.0, dist=0.5
assert result == [2, 4, 1, 3], f"Expected [2, 4, 1, 3], got {result}"

# Test 2: All same rating — sort purely by distance
customer2 = (0.0, 0.0)
dashers2 = [
    (1, 3.0, 0.0, 4.0),  # dist=3
    (2, 1.0, 0.0, 4.0),  # dist=1
    (3, 2.0, 0.0, 4.0),  # dist=2
]
result2 = k_nearest_by_rating(customer2, dashers2, k=3)
assert result2 == [2, 3, 1], f"Expected [2, 3, 1], got {result2}"

# Test 3: All same distance — sort purely by rating desc
customer3 = (0.0, 0.0)
dashers3 = [
    (1, 1.0, 0.0, 3.0),  # dist=1, rating=3.0
    (2, 0.0, 1.0, 5.0),  # dist=1, rating=5.0
    (3, -1.0, 0.0, 4.0), # dist=1, rating=4.0
]
result3 = k_nearest_by_rating(customer3, dashers3, k=3)
assert result3 == [2, 3, 1], f"Expected [2, 3, 1], got {result3}"

# Test 4: K=1
customer4 = (0.0, 0.0)
dashers4 = [
    (1, 1.0, 0.0, 3.0),
    (2, 0.5, 0.0, 5.0),
]
result4 = k_nearest_by_rating(customer4, dashers4, k=1)
assert result4 == [2], f"Expected [2], got {result4}"

print("All Stage 3 tests passed.")
```

## Scoring Rubric

### Fundamentals (1-4)

| Score | Criteria |
|-------|----------|
| **4 — Strong** | Immediately identifies that a max-heap of size K gives O(N log K). Explains why Python's `heapq` is a min-heap and uses negation correctly. Understands the streaming variant is the same logic with per-element amortized cost. Correctly identifies that sorting K elements after selection is O(K log K) and negligible. |
| **3 — Satisfactory** | Recognizes the heap-based approach. Correctly uses negation for max-heap behavior. Implements streaming version with minor guidance. Sorts by multiple keys correctly. |
| **2 — Developing** | First instinct is to sort all N elements. Needs a hint to think about heaps. May struggle with negation trick or `heapreplace`. Can do multi-key sort but may get the direction wrong initially. |
| **1 — Insufficient** | Does not know what a heap is or how `heapq` works. Cannot reason about K vs N trade-offs. Cannot implement multi-key sorting. |

### Coding (1-4)

| Score | Criteria |
|-------|----------|
| **4 — Strong** | Clean, correct code on all three stages. Uses `heapq.heapreplace` (or equivalent). Avoids unnecessary square root computations. Class design is clean with proper encapsulation. Multi-key sort is concise using a lambda or tuple key. |
| **3 — Satisfactory** | Working solutions. May use `heappush` + `heappop` instead of `heapreplace` (less efficient but correct). Minor style issues. |
| **2 — Developing** | Stage 1 works but has bugs. Struggles with class implementation in Stage 2. May compute sqrt unnecessarily. Multi-key sort has an error in direction. |
| **1 — Insufficient** | Cannot produce working heap code. Fundamental misunderstanding of `heapq` API. |

### Communication (1-4)

| Score | Criteria |
|-------|----------|
| **4 — Strong** | Asks about distance metric, tie-breaking, and K vs N relationship. Explains the max-heap/negation approach before coding. Discusses why O(N log K) beats O(N log N). Proactively mentions that squared distance avoids floating-point issues with sqrt. |
| **3 — Satisfactory** | Asks 1-2 clarifying questions. Explains approach at a high level. Discusses complexity when prompted. |
| **2 — Developing** | Jumps straight to sorting without considering alternatives. Needs prompting to discuss efficiency. |
| **1 — Insufficient** | Cannot explain the difference between their approach and brute-force sorting. Codes silently. |

## Interviewer Guide

### Hints (per stage, progressive: mild, medium, strong)

**Stage 1: K Nearest Dashers**

| Level | Hint |
|-------|------|
| **Mild** | "If N is a million and K is 5, do we really need to sort all N elements?" |
| **Medium** | "Think about maintaining a collection of the K best candidates so far. What data structure lets you efficiently compare a new element against the worst of your K best?" |
| **Strong** | "Use a max-heap of size K. Python's `heapq` is a min-heap, so negate the distances: push `(-distance, id)`. The smallest negated distance (which is at the heap top) represents the largest actual distance in your K candidates. When a new Dasher is closer than that, replace it with `heapq.heapreplace`." |

**Stage 2: Streaming Dashers**

| Level | Hint |
|-------|------|
| **Mild** | "How is this different from Stage 1? Can you reuse the same logic?" |
| **Medium** | "Think about what state you need to persist between calls. What data structure should be an instance variable?" |
| **Strong** | "Store the heap as `self.max_heap`. In `add_dasher`, apply the same push-or-replace logic. In `get_nearest`, extract the IDs from the heap elements." |

**Stage 3: Multi-Key Ranking**

| Level | Hint |
|-------|------|
| **Mild** | "You already have the K nearest. Now you just need to order them differently. What Python feature makes sorting by multiple criteria easy?" |
| **Medium** | "Python's `sort` is stable and works with tuples. How can you make rating sort descending while distance sorts ascending?" |
| **Strong** | "Sort with key `lambda x: (-x.rating, x.distance)`. Negating the rating makes Python's ascending sort produce descending order for that field." |

### When to Intervene

- **Candidate sorts all N elements in Stage 1:** Let them finish, then ask about time complexity. Ask: "What if N is 10 million and K is 3? Can we do better?" Guide to heap approach.
- **Candidate doesn't know about `heapq`:** Briefly explain the API: `heappush`, `heappop`, `heapreplace`, and that it is a min-heap. This is library knowledge, not algorithmic understanding — don't penalize too heavily.
- **Candidate uses `math.sqrt`:** Let it work, but ask if it is necessary. They should realize that squared distance preserves ordering. This is an optimization insight worth partial credit.
- **Stage 2 taking too long:** If the candidate has Stage 1 working, note that Stage 2 is essentially the same logic in a class wrapper. Move them along.
- **Stage 3 sort key is wrong:** If they have the sort direction inverted, ask them to trace through an example manually. The test case output will make the error obvious.

### Common Mistakes

1. **Using a min-heap instead of a max-heap.** The candidate pushes distances directly and pops the minimum. This gives them the K *farthest* Dashers, not the K nearest. The fix is to negate the distance.
2. **Forgetting `heapreplace` and using `heappop` + `heappush`.** This is functionally correct but does two O(log K) operations instead of one. Point it out as a minor optimization.
3. **Computing Euclidean distance with `sqrt` in the comparison.** Not wrong, but unnecessary and introduces floating-point imprecision. Squared distance comparison is sufficient.
4. **In Stage 2, recomputing from scratch in `get_nearest`.** Some candidates store all Dashers in a list and re-sort every time `get_nearest` is called. This defeats the purpose of the streaming approach.
5. **Multi-key sort direction error.** Sorting by `(rating, distance)` instead of `(-rating, distance)`. This gives ascending rating, which is the opposite of what we want.
6. **Heap size exceeding K.** Forgetting the size check and pushing every element, resulting in a heap of size N.

### Edge Cases to Watch For

- **All Dashers at the same location:** The heap should handle equal distances gracefully. All are equally close.
- **Customer is at the same location as a Dasher:** Distance is 0. This should not cause division by zero or other issues (and it won't since we never divide).
- **Negative coordinates:** Squared distance handles negatives correctly. No special handling needed.
- **K = N:** The heap will be the same size as the input. This is the degenerate case where sorting all would be equally efficient.
- **Very large coordinate values:** Squaring large floats can overflow in some languages, but Python handles arbitrary precision integers and large floats gracefully.
- **Streaming: first few Dashers before heap is full:** The heap should accept all Dashers until size K is reached, then start being selective.
