# Round 02: Hash Maps — "Restaurant Order Frequency"

## Metadata
- **Topic:** Hash Maps, Nested Dictionaries, Sliding Windows
- **Difficulty:** 1/5
- **Estimated Time:** 55-60 minutes
- **Key Concepts:** Dictionary construction, frequency counting, time-based sliding windows, top-K retrieval with heaps

## Problem Prompt

> DoorDash wants to help restaurants understand ordering trends. You're given a stream of order records, where each record contains a restaurant ID, an item name, and a timestamp. Build a system that can report the most popular item at each restaurant.

*(Read exactly this. Do not clarify tie-breaking, whether the stream fits in memory, or what "most popular" means precisely. Let the candidate ask.)*

## Clarifying Questions & Hidden Constraints

| # | Question the Candidate Should Ask | Answer to Give |
|---|---|---|
| 1 | What format are the order records? | Each record is a tuple `(restaurant_id: str, item_name: str, timestamp: int)`. Timestamps are Unix-style integers (minutes since epoch for simplicity). |
| 2 | How should ties be broken if two items have the same order count? | Break ties alphabetically by item name (lexicographically smallest wins). |
| 3 | Can we assume the data fits in memory? | For Stages 1 and 2, yes. But design with the awareness that real streams can be large. |
| 4 | Are restaurant IDs and item names case-sensitive? | Yes. `"Pizza"` and `"pizza"` are different items. `"rest_1"` and `"Rest_1"` are different restaurants. |
| 5 | Can the order list be empty? | Yes. Return an empty dictionary in that case. |
| 6 | Are timestamps guaranteed to be in sorted order? | Good question. For Stage 1, order doesn't matter. For Stage 2, yes, assume records arrive in non-decreasing timestamp order. |
| 7 | What does "stream" mean — do we process one record at a time? | For Stage 1, you can process the full list at once. We'll make it more streaming-oriented in later stages. |

## Skeleton Code

```python
from typing import List, Tuple, Dict
from collections import defaultdict

# An order record: (restaurant_id, item_name, timestamp)
Order = Tuple[str, str, int]


def top_item_per_restaurant(orders: List[Order]) -> Dict[str, str]:
    """
    Stage 1: Return a dict mapping each restaurant_id to its most
    frequently ordered item_name. Break ties alphabetically.
    """
    pass

if __name__ == "__main__":
    # Expected: {"rest_1": "Pizza", "rest_2": "Ramen"}
    print(top_item_per_restaurant([
        ("rest_1", "Pizza", 1), ("rest_1", "Pizza", 2), ("rest_1", "Burger", 3),
        ("rest_2", "Sushi", 1), ("rest_2", "Ramen", 3), ("rest_2", "Ramen", 4), ("rest_2", "Ramen", 5),
    ]))
    # Expected: {"rest_1": "Apple Pie"} (tie broken alphabetically)
    print(top_item_per_restaurant([("rest_1", "Burger", 1), ("rest_1", "Apple Pie", 2)]))
    # Expected: {}
    print(top_item_per_restaurant([]))
```

---

## Stage 1: Frequency Map — Top Item per Restaurant (~20 minutes)

### Expected Approach

1. Build a nested dictionary: `{restaurant_id: {item_name: count}}`.
2. For each order, increment the count for the corresponding restaurant and item.
3. For each restaurant, find the item with the maximum count. Break ties by choosing the lexicographically smallest item name.

### Solution Code

```python
def top_item_per_restaurant(orders: List[Order]) -> Dict[str, str]:
    # Build frequency map: restaurant -> item -> count
    freq: Dict[str, Dict[str, int]] = defaultdict(lambda: defaultdict(int))

    for restaurant_id, item_name, _timestamp in orders:
        freq[restaurant_id][item_name] += 1

    result = {}
    for restaurant_id, item_counts in freq.items():
        # max by count descending, then by name ascending for tie-break
        best_item = min(item_counts, key=lambda item: (-item_counts[item], item))
        result[restaurant_id] = best_item

    return result
```

### Complexity
- **Time:** O(n) to build the frequency map + O(r * m) to find the max per restaurant, where r is the number of restaurants and m is the average number of distinct items per restaurant. Overall O(n) since the total items across all restaurants is at most n.
- **Space:** O(n) for the frequency map in the worst case (all unique restaurant-item pairs).

### Test Cases

```python
# Test 1: Basic frequency counting
orders1 = [
    ("rest_1", "Pizza", 1),
    ("rest_1", "Pizza", 2),
    ("rest_1", "Burger", 3),
    ("rest_2", "Sushi", 1),
    ("rest_2", "Sushi", 2),
    ("rest_2", "Ramen", 3),
    ("rest_2", "Ramen", 4),
    ("rest_2", "Ramen", 5),
]
assert top_item_per_restaurant(orders1) == {"rest_1": "Pizza", "rest_2": "Ramen"}

# Test 2: Tie-breaking alphabetically
orders2 = [
    ("rest_1", "Burger", 1),
    ("rest_1", "Apple Pie", 2),
]
# Both have count 1; "Apple Pie" < "Burger" alphabetically
assert top_item_per_restaurant(orders2) == {"rest_1": "Apple Pie"}

# Test 3: Empty input
assert top_item_per_restaurant([]) == {}

# Test 4: Single order
orders4 = [("rest_1", "Taco", 100)]
assert top_item_per_restaurant(orders4) == {"rest_1": "Taco"}
```

---

## Stage 2: Sliding Time Window (~20 minutes)

### Prompt Addition

> "Good. Now DoorDash only cares about recent trends. Modify your solution so it only counts orders from the last W minutes. W is measured relative to the latest timestamp in the entire stream."

### Starter Code

```python
def top_item_in_window(
    orders: List[Order],
    window_minutes: int
) -> Dict[str, str]:
    """
    Stage 2: Only consider orders within the last `window_minutes`
    minutes (relative to the latest timestamp in the stream).
    Return a dict mapping each restaurant_id to its most frequently
    ordered item in that window. Break ties alphabetically.
    """
    pass

# --- Stage 2 Test Calls ---
# Expected: {"rest_1": "Burger"} (only timestamps > 6 count)
print(top_item_in_window([
    ("rest_1", "Pizza", 1), ("rest_1", "Pizza", 2), ("rest_1", "Pizza", 3),
    ("rest_1", "Burger", 8), ("rest_1", "Burger", 9), ("rest_1", "Burger", 10), ("rest_1", "Burger", 11),
], 5))
# Expected: {"rest_1": "Pizza"} (large window includes everything)
print(top_item_in_window([("rest_1", "Pizza", 1), ("rest_1", "Pizza", 2), ("rest_1", "Burger", 3)], 1000))
# Expected: {}
print(top_item_in_window([], 10))
```

### Expected Approach

1. First pass (or track while building): determine the maximum timestamp.
2. Compute the cutoff: `cutoff = max_timestamp - window_minutes`.
3. Filter: only count orders where `timestamp > cutoff` (strictly greater — orders exactly at the cutoff boundary are excluded; if the candidate asks, either choice is fine as long as it's consistent).
4. Apply the same frequency-counting logic from Stage 1 on the filtered orders.

Alternative (streaming-aware) approach for bonus discussion: use a deque per restaurant to maintain a sliding window, evicting expired records. This is more relevant if records arrive one at a time.

### Solution Code

```python
def top_item_in_window(
    orders: List[Order],
    window_minutes: int
) -> Dict[str, str]:
    if not orders:
        return {}

    # Find the latest timestamp to anchor the window
    max_time = max(ts for _, _, ts in orders)
    cutoff = max_time - window_minutes

    # Build frequency map only for orders within the window
    freq: Dict[str, Dict[str, int]] = defaultdict(lambda: defaultdict(int))

    for restaurant_id, item_name, timestamp in orders:
        if timestamp > cutoff:
            freq[restaurant_id][item_name] += 1

    result = {}
    for restaurant_id, item_counts in freq.items():
        best_item = min(item_counts, key=lambda item: (-item_counts[item], item))
        result[restaurant_id] = best_item

    return result
```

*Streaming-aware variant (discuss if time permits):*

```python
from collections import deque

class StreamingWindowTracker:
    """Processes orders one at a time, maintaining a sliding window."""

    def __init__(self, window_minutes: int):
        self.window = window_minutes
        # restaurant -> deque of (timestamp, item_name)
        self.queues: Dict[str, deque] = defaultdict(deque)
        # restaurant -> item -> count (within window)
        self.freq: Dict[str, Dict[str, int]] = defaultdict(lambda: defaultdict(int))

    def add_order(self, restaurant_id: str, item_name: str, timestamp: int):
        # Add new order
        self.queues[restaurant_id].append((timestamp, item_name))
        self.freq[restaurant_id][item_name] += 1

        # Evict expired orders for this restaurant
        cutoff = timestamp - self.window
        q = self.queues[restaurant_id]
        while q and q[0][0] <= cutoff:
            old_ts, old_item = q.popleft()
            self.freq[restaurant_id][old_item] -= 1
            if self.freq[restaurant_id][old_item] == 0:
                del self.freq[restaurant_id][old_item]

    def get_top_item(self, restaurant_id: str) -> str:
        if restaurant_id not in self.freq or not self.freq[restaurant_id]:
            return ""
        item_counts = self.freq[restaurant_id]
        return min(item_counts, key=lambda item: (-item_counts[item], item))
```

### Complexity
- **Batch approach:** O(n) time, O(n) space — same as Stage 1, just with filtering.
- **Streaming approach:** O(1) amortized per `add_order` (each order is enqueued and dequeued at most once). `get_top_item` is O(m) where m is distinct items for that restaurant.

### Test Cases

```python
# Test 1: Window filters out old orders
orders1 = [
    ("rest_1", "Pizza", 1),
    ("rest_1", "Pizza", 2),
    ("rest_1", "Pizza", 3),
    ("rest_1", "Burger", 8),
    ("rest_1", "Burger", 9),
    ("rest_1", "Burger", 10),
    ("rest_1", "Burger", 11),
]
# max_time = 11, window = 5, cutoff = 6
# Only timestamps > 6 count: Burger at 8,9,10,11 (4 times)
# Pizza at nothing within window
assert top_item_in_window(orders1, 5) == {"rest_1": "Burger"}

# Test 2: Large window includes everything (same as Stage 1)
orders2 = [
    ("rest_1", "Pizza", 1),
    ("rest_1", "Pizza", 2),
    ("rest_1", "Burger", 3),
]
assert top_item_in_window(orders2, 1000) == {"rest_1": "Pizza"}

# Test 3: Window of 0 — only the latest timestamp counts
orders3 = [
    ("rest_1", "Pizza", 5),
    ("rest_1", "Burger", 10),
]
# cutoff = 10 - 0 = 10, only timestamp > 10 qualifies — nothing
assert top_item_in_window(orders3, 0) == {}

# Test 4: Multiple restaurants, some fall outside window
orders4 = [
    ("rest_1", "Pizza", 1),
    ("rest_2", "Sushi", 1),
    ("rest_2", "Sushi", 50),
    ("rest_2", "Ramen", 50),
    ("rest_2", "Ramen", 50),
]
# max_time=50, window=10, cutoff=40. rest_1 has nothing > 40.
# rest_2: Sushi=1, Ramen=2 → Ramen wins
assert top_item_in_window(orders4, 10) == {"rest_2": "Ramen"}

# Test 5: Empty orders
assert top_item_in_window([], 10) == {}
```

---

## Stage 3: Top-K Items per Restaurant (~15 minutes)

### Prompt Addition

> "Finally, instead of just the single most popular item, return the top-K most popular items per restaurant, ordered from most to least frequent. Still break ties alphabetically."

### Starter Code

```python
def top_k_items_per_restaurant(
    orders: List[Order],
    k: int
) -> Dict[str, List[str]]:
    """
    Stage 3: Return a dict mapping each restaurant_id to a list of
    its top-K most frequently ordered items, sorted from most to
    least frequent. Break ties alphabetically. If a restaurant has
    fewer than K distinct items, return all of them.
    """
    pass

# --- Stage 3 Test Calls ---
# Expected: {"rest_1": ["Pizza", "Burger"]}
print(top_k_items_per_restaurant([
    ("rest_1", "Pizza", 1), ("rest_1", "Pizza", 2), ("rest_1", "Pizza", 3),
    ("rest_1", "Burger", 4), ("rest_1", "Burger", 5), ("rest_1", "Salad", 6),
], 2))
# Expected: {"rest_1": ["Burger", "Pizza"]} (both count 1, alphabetical)
print(top_k_items_per_restaurant([("rest_1", "Pizza", 1), ("rest_1", "Burger", 2)], 10))
# Expected: {}
print(top_k_items_per_restaurant([], 5))
```

### Expected Approach

1. Build the same frequency map as Stage 1.
2. For each restaurant, sort items by `(-count, item_name)` and take the first K.
3. Alternatively, use a min-heap of size K for efficiency when K is small relative to the number of items.

The straightforward sort approach is O(m log m) per restaurant. A heap-based approach is O(m log K) per restaurant. For an interview, the sort approach is perfectly acceptable.

### Solution Code

```python
def top_k_items_per_restaurant(
    orders: List[Order],
    k: int
) -> Dict[str, List[str]]:
    # Build frequency map
    freq: Dict[str, Dict[str, int]] = defaultdict(lambda: defaultdict(int))

    for restaurant_id, item_name, _timestamp in orders:
        freq[restaurant_id][item_name] += 1

    result = {}
    for restaurant_id, item_counts in freq.items():
        # Sort by count descending, then name ascending
        sorted_items = sorted(
            item_counts.keys(),
            key=lambda item: (-item_counts[item], item)
        )
        result[restaurant_id] = sorted_items[:k]

    return result
```

*Heap-based variant (for discussion):*

```python
import heapq

def top_k_items_per_restaurant_heap(
    orders: List[Order],
    k: int
) -> Dict[str, List[str]]:
    freq: Dict[str, Dict[str, int]] = defaultdict(lambda: defaultdict(int))

    for restaurant_id, item_name, _timestamp in orders:
        freq[restaurant_id][item_name] += 1

    result = {}
    for restaurant_id, item_counts in freq.items():
        # Use nlargest with a key; nlargest keeps the K largest
        # We want largest count, and for ties, smallest name.
        # heapq.nlargest compares tuples: (count, reversed_name_for_tiebreak)
        # Simpler: just use nlargest on (count, item) but we need to
        # invert the name comparison. Easiest approach:
        top_k = heapq.nlargest(
            k,
            item_counts.keys(),
            key=lambda item: (item_counts[item], [-ord(c) for c in item])
        )
        # The above trick inverts alphabetical order so that among equal
        # counts, lexicographically smaller names are "larger" in heap terms.
        # A cleaner alternative using a wrapper:
        result[restaurant_id] = top_k

    return result
```

*(Note: the sort-based approach is cleaner and preferred in an interview setting. Mention the heap approach as an optimization if K << m.)*

### Complexity
- **Sort approach:** O(n) to build map + O(r * m log m) to sort per restaurant = O(n + total_items * log(max_items_per_restaurant))
- **Heap approach:** O(n + total_items * log K)
- **Space:** O(n) for the frequency map

### Test Cases

```python
# Test 1: Basic top-2
orders1 = [
    ("rest_1", "Pizza", 1),
    ("rest_1", "Pizza", 2),
    ("rest_1", "Pizza", 3),
    ("rest_1", "Burger", 4),
    ("rest_1", "Burger", 5),
    ("rest_1", "Salad", 6),
]
assert top_k_items_per_restaurant(orders1, 2) == {"rest_1": ["Pizza", "Burger"]}

# Test 2: K larger than distinct items — return all
orders2 = [
    ("rest_1", "Pizza", 1),
    ("rest_1", "Burger", 2),
]
assert top_k_items_per_restaurant(orders2, 10) == {"rest_1": ["Burger", "Pizza"]}
# Both have count 1, so alphabetical: Burger < Pizza

# Test 3: Multiple restaurants
orders3 = [
    ("rest_1", "Pizza", 1),
    ("rest_1", "Pizza", 2),
    ("rest_1", "Taco", 3),
    ("rest_2", "Sushi", 1),
    ("rest_2", "Ramen", 2),
    ("rest_2", "Ramen", 3),
    ("rest_2", "Ramen", 4),
]
result3 = top_k_items_per_restaurant(orders3, 2)
assert result3 == {"rest_1": ["Pizza", "Taco"], "rest_2": ["Ramen", "Sushi"]}

# Test 4: K = 1 should match Stage 1 behavior
orders4 = [
    ("rest_1", "Alpha", 1),
    ("rest_1", "Beta", 2),
    ("rest_1", "Beta", 3),
]
assert top_k_items_per_restaurant(orders4, 1) == {"rest_1": ["Beta"]}

# Test 5: Empty input
assert top_k_items_per_restaurant([], 5) == {}

# Test 6: Tie-breaking in top-K
orders6 = [
    ("rest_1", "Zebra", 1),
    ("rest_1", "Apple", 2),
    ("rest_1", "Mango", 3),
]
# All count 1. Alphabetical: Apple, Mango, Zebra
assert top_k_items_per_restaurant(orders6, 2) == {"rest_1": ["Apple", "Mango"]}
```

---

## Scoring Rubric

### Fundamentals (1-4)

| Score | Description |
|-------|-------------|
| 1 | Cannot articulate how to use a dictionary for frequency counting. Unfamiliar with Python's `defaultdict` or equivalent patterns. |
| 2 | Builds a flat frequency map but struggles with nesting (restaurant -> item -> count). Gets confused by the sliding window concept in Stage 2. |
| 3 | Correctly builds the nested frequency map. Implements the time window filter. Produces a working top-K solution, possibly with a minor inefficiency. |
| 4 | All stages clean and correct. Discusses time/space complexity. Mentions the streaming variant for Stage 2 unprompted. Knows when to use heaps vs. sorting for top-K. |

### Coding (1-4)

| Score | Description |
|-------|-------------|
| 1 | Code does not run. Confused by Python dict syntax or iteration. |
| 2 | Code runs but has bugs: wrong tie-breaking, off-by-one in window boundary, or crashes on empty input. |
| 3 | Code is correct for all provided test cases. Uses `defaultdict` or manual `setdefault` correctly. Minor style issues. |
| 4 | Clean, idiomatic Python. Uses `defaultdict(lambda: defaultdict(int))`, tuple unpacking in loops, `sorted()` with key functions. Handles all edge cases including empty input and K > distinct items. |

### Communication (1-4)

| Score | Description |
|-------|-------------|
| 1 | Starts coding without explaining. Cannot articulate the data structure choice. |
| 2 | Explains the high-level idea ("I'll use a dictionary") but doesn't walk through the logic or discuss edge cases. |
| 3 | Clearly explains the nested dictionary structure before coding. Walks through an example. Asks about tie-breaking and empty input. |
| 4 | Proactively clarifies all ambiguities, draws out the data structure, explains complexity, discusses streaming vs. batch tradeoffs, and proposes the heap optimization before being asked. |

---

## Interviewer Guide

### Hints

#### Stage 1: Frequency Map

| Level | Hint |
|-------|------|
| **Mild** | "What data structure naturally maps keys to counts?" |
| **Medium** | "You'll need a dictionary of dictionaries — the outer key is the restaurant, the inner key is the item. What does the value represent?" |
| **Strong** | "Use `defaultdict(lambda: defaultdict(int))`. Loop through orders, increment `freq[restaurant][item]`. Then for each restaurant, find the item with the max count." |

#### Stage 2: Sliding Time Window

| Level | Hint |
|-------|------|
| **Mild** | "What defines the boundary of the window? How do you know which orders are 'recent'?" |
| **Medium** | "Find the maximum timestamp first. Then the cutoff is `max_timestamp - W`. Only count orders with timestamp strictly greater than the cutoff." |
| **Strong** | "Add an `if timestamp > cutoff` check inside your loop from Stage 1. Everything else stays the same." |

#### Stage 3: Top-K Items

| Level | Hint |
|-------|------|
| **Mild** | "You already have the frequency map. How would you get the top-K from a dictionary of counts?" |
| **Medium** | "Python's `sorted()` with a `key` function can sort by count descending and name ascending. Then slice the first K." |
| **Strong** | "Use `sorted(item_counts.keys(), key=lambda item: (-item_counts[item], item))[:k]`." |

### When to Intervene

- If the candidate tries to use a single flat dictionary `{(restaurant, item): count}` instead of nesting, let them proceed — it works fine, just discuss the tradeoff of key design.
- If the candidate forgets tie-breaking, let them write the solution first, then ask: "What happens if Pizza and Burger both have 5 orders?" Let them fix it.
- If Stage 2's window logic is wrong (e.g., using `>=` instead of `>`), run through a concrete example to expose the bug before hinting.
- If the candidate is stuck on top-K, suggest they first solve it with `sorted()` and only then discuss heap optimization if time permits.

### Common Mistakes

1. **Flat dictionary instead of nested:** Using `freq[(restaurant, item)]` works but makes the "per restaurant" aggregation in the result step harder. Not wrong, but less clean.
2. **Incorrect tie-breaking:** Forgetting to handle ties, or breaking ties by insertion order (non-deterministic in older Python) instead of alphabetically.
3. **Window boundary off-by-one:** Including or excluding the cutoff timestamp inconsistently. The key is to be explicit and consistent.
4. **Forgetting to handle empty input:** Calling `max()` on an empty sequence raises `ValueError`.
5. **Stage 3 — returning counts instead of names:** Returning `[("Pizza", 5), ("Burger", 3)]` instead of `["Pizza", "Burger"]`.
6. **Not cleaning up zero-count entries:** In the streaming variant, decrementing a count to 0 and leaving it in the dictionary can cause stale items to appear in results.

### Edge Cases to Watch For

- Empty order list
- Single restaurant with one item
- All orders for the same item at the same restaurant (top-K should return a list of length 1)
- K = 0 (should return empty lists for each restaurant)
- Multiple restaurants where one has orders entirely outside the time window (Stage 2 — that restaurant should not appear in the result)
- Items with names that are prefixes of each other: `"Pie"` vs. `"Pied Piper"` (no special handling needed, but confirms string comparison works correctly)
- Very large window that includes all orders (Stage 2 should behave identically to Stage 1)
