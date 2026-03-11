# Round 07: Doubly Linked Lists — "Recently Viewed Restaurants (LRU Cache)"

## Metadata
- **Topic:** Doubly Linked Lists, Hash Maps, Cache Design
- **Difficulty:** 3/5
- **Estimated Time:** 55-60 minutes
- **Key Concepts:** Doubly linked list manipulation, hash map for O(1) lookup, LRU eviction policy, pointer surgery, combined data structure design

## Problem Prompt

> At DoorDash, we want to build a "Recently Viewed Restaurants" feature. When a customer browses restaurants, we show them the last N distinct restaurants they viewed, ordered from most-recently viewed to least-recently viewed.
>
> Design and implement a `RecentlyViewed` class that supports:
> - `view(restaurant_id)` — the customer views a restaurant
> - `get_recent()` — returns the list of recently viewed restaurants, most-recent first
>
> The capacity N is provided at construction time.

*Read this prompt exactly. Do not volunteer constraints — let the candidate ask.*

## Clarifying Questions & Hidden Constraints

The candidate **should** ask these questions. If they don't, let them start coding and see if they discover the issues naturally. Only volunteer a hint if they're stuck for more than 2 minutes.

| # | Question Candidate Should Ask | Answer |
|---|-------------------------------|--------|
| 1 | What happens if the customer views the same restaurant again? | It should move to the front — no duplicates in the list. |
| 2 | What if N is 0? | Capacity 0 means nothing is ever stored. `get_recent()` returns `[]`. |
| 3 | What are restaurant_id types? | Strings or integers — assume hashable. Use `str` for examples. |
| 4 | What should `get_recent()` return if nothing has been viewed? | An empty list `[]`. |
| 5 | Should this be O(1) per operation? | Yes — that's the goal. Ask them what data structures could achieve that. |
| 6 | What happens when we exceed capacity? | Evict the least-recently viewed restaurant. |

## Skeleton Code

```python
class RecentlyViewed:
    """
    Recently Viewed Restaurants — LRU Cache.

    Supports:
      - view(restaurant_id): record that a restaurant was viewed
      - get_recent(): return recently viewed restaurants, most-recent first
    """

    def __init__(self, capacity: int):
        pass

    def view(self, restaurant_id: str) -> None:
        pass

    def get_recent(self) -> list:
        pass

if __name__ == "__main__":
    rv = RecentlyViewed(3)
    rv.view("chipotle")
    rv.view("pizza_hut")
    rv.view("sushi_zone")
    # Expected: ['sushi_zone', 'pizza_hut', 'chipotle']
    print(rv.get_recent())
    rv.view("chipotle")
    # Expected: ['chipotle', 'sushi_zone', 'pizza_hut'] (chipotle moved to front)
    print(rv.get_recent())
    rv.view("burger_king")
    # Expected: ['burger_king', 'chipotle', 'sushi_zone'] (pizza_hut evicted)
    print(rv.get_recent())
```

## Stage 1: Core LRU Cache (~25 minutes)

### Expected Approach

The candidate should recognize this as an LRU cache and propose using a **doubly linked list** combined with a **hash map**.

**Key insight:** A doubly linked list allows O(1) removal of any node (given a pointer), and O(1) insertion at head/tail. A hash map provides O(1) lookup from `restaurant_id` to the corresponding node.

**Design:**
- A doubly linked list ordered from most-recent (head) to least-recent (tail).
- A dictionary mapping `restaurant_id -> Node`.
- Sentinel head and tail nodes to simplify edge cases (no null checks on insert/remove).

**`view(restaurant_id)` logic:**
1. If the restaurant is already in the map, remove its node from its current position in the list.
2. Create a new node (or reuse the removed one) and insert it right after the head sentinel.
3. Add/update the map entry.
4. If the map size exceeds capacity, remove the node just before the tail sentinel and delete its map entry.

**`get_recent()` logic:**
- Walk the list from head sentinel's next to tail sentinel, collecting restaurant IDs.

### Solution Code

```python
class _Node:
    """Doubly linked list node."""
    __slots__ = ('key', 'prev', 'next')

    def __init__(self, key: str = ""):
        self.key = key
        self.prev = None
        self.next = None


class RecentlyViewed:
    """
    Recently Viewed Restaurants — LRU Cache.

    Uses a doubly linked list (most-recent at head) + hash map for O(1) ops.
    """

    def __init__(self, capacity: int):
        self.capacity = capacity
        self.map = {}  # restaurant_id -> _Node

        # Sentinel nodes — simplify insert/remove logic
        self.head = _Node()  # dummy head
        self.tail = _Node()  # dummy tail
        self.head.next = self.tail
        self.tail.prev = self.head

    # ---- internal helpers ----

    def _remove(self, node: _Node) -> None:
        """Remove a node from the doubly linked list."""
        node.prev.next = node.next
        node.next.prev = node.prev

    def _insert_after_head(self, node: _Node) -> None:
        """Insert a node right after the head sentinel (most-recent position)."""
        node.next = self.head.next
        node.prev = self.head
        self.head.next.prev = node
        self.head.next = node

    # ---- public API ----

    def view(self, restaurant_id: str) -> None:
        """Record that a restaurant was viewed."""
        if self.capacity <= 0:
            return

        if restaurant_id in self.map:
            # Already seen — move to front
            node = self.map[restaurant_id]
            self._remove(node)
            self._insert_after_head(node)
        else:
            # New restaurant
            node = _Node(restaurant_id)
            self._insert_after_head(node)
            self.map[restaurant_id] = node

            # Evict LRU if over capacity
            if len(self.map) > self.capacity:
                lru = self.tail.prev
                self._remove(lru)
                del self.map[lru.key]

    def get_recent(self) -> list:
        """Return recently viewed restaurants, most-recent first."""
        result = []
        cur = self.head.next
        while cur is not self.tail:
            result.append(cur.key)
            cur = cur.next
        return result
```

### Complexity

| Operation | Time | Space |
|-----------|------|-------|
| `view()` | O(1) | O(1) amortized (one node + one map entry) |
| `get_recent()` | O(N) where N = capacity | O(N) for output list |
| Overall space | — | O(N) for the cache |

### Test Cases

```python
def test_stage_1():
    # Basic usage
    rv = RecentlyViewed(3)
    rv.view("chipotle")
    rv.view("pizza_hut")
    rv.view("sushi_zone")
    assert rv.get_recent() == ["sushi_zone", "pizza_hut", "chipotle"]

    # Duplicate moves to front
    rv.view("chipotle")
    assert rv.get_recent() == ["chipotle", "sushi_zone", "pizza_hut"]

    # Eviction — pizza_hut is LRU
    rv.view("burger_king")
    assert rv.get_recent() == ["burger_king", "chipotle", "sushi_zone"]

    # Capacity 0
    rv0 = RecentlyViewed(0)
    rv0.view("tacos")
    assert rv0.get_recent() == []

    # Capacity 1
    rv1 = RecentlyViewed(1)
    rv1.view("a")
    rv1.view("b")
    assert rv1.get_recent() == ["b"]
    rv1.view("b")
    assert rv1.get_recent() == ["b"]

    # Empty cache
    rv_empty = RecentlyViewed(5)
    assert rv_empty.get_recent() == []

    print("Stage 1: All tests passed.")

test_stage_1()
```

## Stage 2: Expire and Top-K (~20 minutes)

### Prompt Addition

> Great. Now DoorDash needs two more features:
>
> 1. `expire(restaurant_id)` — a restaurant permanently closes, so remove it from the recently viewed list entirely. If it's not in the list, do nothing.
> 2. `get_recent(k)` — instead of returning all recently viewed, return only the top `k` most-recent. If `k` is omitted or `None`, return all. If `k >= len(list)`, return the full list.

### Starter Code

```python
def expire(self, restaurant_id: str) -> None:
    """Remove a restaurant from the cache (e.g., it closed permanently)."""
    pass

def get_recent(self, k: int = None) -> list:
    """Return the top-k most recently viewed restaurants. If k is None, return all."""
    pass

# --- Stage 2 Test Calls ---
# rv = RecentlyViewed(5)
# rv.view("chipotle"); rv.view("pizza_hut"); rv.view("sushi_zone")
# rv.view("burger_king"); rv.view("taco_bell")
# Expected: ['taco_bell', 'burger_king']
# print(rv.get_recent(2))
# rv.expire("sushi_zone")
# Expected: ['taco_bell', 'burger_king', 'pizza_hut', 'chipotle']
# print(rv.get_recent())
# rv.expire("nonexistent")  # no-op
# Expected: ['taco_bell', 'burger_king', 'pizza_hut', 'chipotle']
# print(rv.get_recent())
```

### Expected Approach

**`expire(restaurant_id)`:**
- Look up the node in the hash map.
- If found, remove it from the doubly linked list and delete it from the map.
- O(1) — this is the power of the DLL + map combo.

**`get_recent(k)`:**
- Walk the list from head but stop after collecting `k` items.
- If `k` is `None`, collect everything (same as before).

The candidate should handle:
- Expiring a restaurant that was already evicted (not in the map) — no-op.
- `k = 0` returns `[]`.
- `k` larger than current list size returns the full list.

### Solution Code

```python
class _Node:
    """Doubly linked list node."""
    __slots__ = ('key', 'prev', 'next')

    def __init__(self, key: str = ""):
        self.key = key
        self.prev = None
        self.next = None


class RecentlyViewed:
    """
    Recently Viewed Restaurants — LRU Cache with expire and top-k support.
    """

    def __init__(self, capacity: int):
        self.capacity = capacity
        self.map = {}

        self.head = _Node()
        self.tail = _Node()
        self.head.next = self.tail
        self.tail.prev = self.head

    def _remove(self, node: _Node) -> None:
        node.prev.next = node.next
        node.next.prev = node.prev

    def _insert_after_head(self, node: _Node) -> None:
        node.next = self.head.next
        node.prev = self.head
        self.head.next.prev = node
        self.head.next = node

    def view(self, restaurant_id: str) -> None:
        if self.capacity <= 0:
            return

        if restaurant_id in self.map:
            node = self.map[restaurant_id]
            self._remove(node)
            self._insert_after_head(node)
        else:
            node = _Node(restaurant_id)
            self._insert_after_head(node)
            self.map[restaurant_id] = node

            if len(self.map) > self.capacity:
                lru = self.tail.prev
                self._remove(lru)
                del self.map[lru.key]

    def expire(self, restaurant_id: str) -> None:
        """Remove a restaurant from the cache (e.g., it closed permanently)."""
        if restaurant_id in self.map:
            node = self.map[restaurant_id]
            self._remove(node)
            del self.map[restaurant_id]

    def get_recent(self, k: int = None) -> list:
        """Return the top-k most recently viewed restaurants. If k is None, return all."""
        result = []
        cur = self.head.next
        while cur is not self.tail:
            if k is not None and len(result) >= k:
                break
            result.append(cur.key)
            cur = cur.next
        return result
```

### Complexity

| Operation | Time | Space |
|-----------|------|-------|
| `expire()` | O(1) | O(1) |
| `get_recent(k)` | O(k) or O(N) if k is None | O(k) or O(N) for output |

### Test Cases

```python
def test_stage_2():
    rv = RecentlyViewed(5)
    rv.view("chipotle")
    rv.view("pizza_hut")
    rv.view("sushi_zone")
    rv.view("burger_king")
    rv.view("taco_bell")

    # Top-k
    assert rv.get_recent(2) == ["taco_bell", "burger_king"]
    assert rv.get_recent(0) == []
    assert rv.get_recent(10) == ["taco_bell", "burger_king", "sushi_zone", "pizza_hut", "chipotle"]

    # Expire middle element
    rv.expire("sushi_zone")
    assert rv.get_recent() == ["taco_bell", "burger_king", "pizza_hut", "chipotle"]

    # Expire head element
    rv.expire("taco_bell")
    assert rv.get_recent() == ["burger_king", "pizza_hut", "chipotle"]

    # Expire element not in cache — no-op
    rv.expire("nonexistent")
    assert rv.get_recent() == ["burger_king", "pizza_hut", "chipotle"]

    # After expiry, capacity is freed — new views don't evict
    rv.view("wendy_s")
    rv.view("five_guys")
    assert rv.get_recent() == ["five_guys", "wendy_s", "burger_king", "pizza_hut", "chipotle"]

    # Expire tail element
    rv.expire("chipotle")
    assert rv.get_recent() == ["five_guys", "wendy_s", "burger_king", "pizza_hut"]

    print("Stage 2: All tests passed.")

test_stage_2()
```

## Stage 3: Complexity Analysis & Thread Safety (~10 minutes)

### Prompt Addition

> Let's step back from coding. I have a few conceptual questions:
>
> 1. Walk me through the time and space complexity of every operation.
> 2. If this were deployed on DoorDash's backend serving millions of users concurrently, what would need to change to make it thread-safe?
> 3. What are the trade-offs of different concurrency approaches?

*No code is required. This is a discussion stage.*

### Expected Approach

**Complexity recap (candidate should articulate clearly):**

| Operation | Time | Why |
|-----------|------|-----|
| `view()` | O(1) | Hash map lookup + constant pointer updates |
| `get_recent()` | O(N) worst case, O(k) if k given | List traversal |
| `get_recent(k)` | O(k) | Early termination |
| `expire()` | O(1) | Hash map lookup + constant pointer updates |
| Space | O(N) | N nodes in DLL + N entries in hash map |

**Thread-safety discussion — strong answers will cover:**

1. **Coarse-grained locking:** Wrap every method in a single `threading.Lock`. Simple but limits throughput — all operations are serialized.

2. **Read-write lock (`threading.RWLock` pattern):** Allow concurrent `get_recent()` calls but exclusive access for `view()` / `expire()`. Better for read-heavy workloads. Caveat: `view()` is actually very frequent in a "recently viewed" feature, so the write lock may dominate.

3. **Per-user isolation:** In practice, each user has their own cache instance, so cross-user locking isn't needed. The real concern is concurrent requests *for the same user* (e.g., multiple browser tabs).

4. **Lock-free / CAS approaches:** Mention that lock-free doubly linked lists exist but are extremely complex. Probably not worth it for this use case.

5. **External store (Redis):** In production, DoorDash would likely use Redis with a sorted set (ZADD with timestamp as score) rather than an in-memory Python structure. This gets thread-safety "for free" via Redis's single-threaded command execution.

**Excellent candidates will also mention:**
- The GIL in CPython provides some protection but is not a substitute for proper synchronization.
- Deadlock risks if locking granularity is too fine.
- Testing concurrent code is much harder than testing sequential code.

### Solution Code

```python
# No code required for Stage 3. The following is a reference illustration
# of the coarse-grained locking approach, for interviewer reference only.

import threading


class ThreadSafeRecentlyViewed:
    """Illustrative thread-safe wrapper — interviewer reference only."""

    def __init__(self, capacity: int):
        self._lock = threading.Lock()
        self._inner = RecentlyViewed(capacity)

    def view(self, restaurant_id: str) -> None:
        with self._lock:
            self._inner.view(restaurant_id)

    def expire(self, restaurant_id: str) -> None:
        with self._lock:
            self._inner.expire(restaurant_id)

    def get_recent(self, k: int = None) -> list:
        with self._lock:
            return self._inner.get_recent(k)
```

### Complexity

Same as Stage 2. Thread-safe wrapper adds constant overhead per operation (lock acquire/release).

### Test Cases

```python
# Conceptual stage — no automated tests. Evaluate via discussion.
# Strong candidate should mention:
#   - O(1) for view/expire, O(N) or O(k) for get_recent
#   - Coarse lock as simplest approach
#   - Redis sorted set as production alternative
#   - Per-user isolation reduces contention in practice
```

## Scoring Rubric

### Fundamentals (1-4)

| Score | Description |
|-------|-------------|
| **4** | Immediately recognizes LRU cache pattern. Proposes DLL + hash map unprompted. Clearly explains why arrays/deques alone aren't O(1). Solid complexity analysis. Thread-safety discussion is nuanced (mentions per-user isolation, Redis). |
| **3** | Recognizes the need for DLL + map with minor prompting. Implements correctly. Complexity analysis is correct. Thread-safety discussion covers locking basics. |
| **2** | Needs significant hints to arrive at DLL + map. May initially propose O(n) approaches (e.g., list with linear search). Eventually gets to correct solution. Complexity analysis has gaps. |
| **1** | Cannot articulate why O(1) requires a combined data structure. Struggles with pointer manipulation even with hints. |

### Coding (1-4)

| Score | Description |
|-------|-------------|
| **4** | Clean, bug-free implementation on first try. Uses sentinel nodes. Handles all edge cases (capacity 0, duplicates, empty cache). Helper methods are well-factored. Code is readable. |
| **3** | Working implementation with 1-2 minor bugs caught during testing. Handles most edge cases. Code is reasonably organized. |
| **2** | Implementation has structural issues (e.g., doesn't handle duplicates, off-by-one in eviction). Needs interviewer guidance to fix. |
| **1** | Cannot produce a working DLL implementation even with guidance. Fundamental confusion about pointer manipulation. |

### Communication (1-4)

| Score | Description |
|-------|-------------|
| **4** | Thinks aloud clearly. Asks clarifying questions upfront. Explains design choices before coding. Walks through test cases to verify. Discusses trade-offs proactively in Stage 3. |
| **3** | Communicates approach before coding. Explains most decisions. Responds well to follow-up questions. |
| **2** | Codes silently for long stretches. Needs prompting to explain reasoning. Answers Stage 3 questions superficially. |
| **1** | Cannot articulate their approach. Struggles to explain their own code. |

## Interviewer Guide

### Hints (per stage, progressive: mild, medium, strong)

**Stage 1:**

| Level | Hint |
|-------|------|
| Mild | "What data structures give you O(1) insertion and deletion?" |
| Medium | "A hash map gives O(1) lookup, but how do you maintain ordering? What if you combined it with a linked list?" |
| Strong | "Use a doubly linked list where the head is most-recent. Keep a dictionary mapping restaurant_id to the node in the list. To move a restaurant to the front, remove its node and reinsert it at the head." |

**Stage 2:**

| Level | Hint |
|-------|------|
| Mild | "You already have `_remove` — what do you need to look up the node?" |
| Medium | "For `expire`, it's almost identical to what you already do when moving a node — except you don't reinsert it." |
| Strong | "Look up the node in the map, call `_remove(node)`, then `del self.map[restaurant_id]`." |

**Stage 3:**

| Level | Hint |
|-------|------|
| Mild | "What happens if two threads call `view()` at the same time for the same user?" |
| Medium | "What's the simplest way to prevent concurrent modification of the linked list?" |
| Strong | "A mutex/lock around each method is the simplest approach. In production, you might use Redis instead." |

### When to Intervene

- **If the candidate proposes an array/list-based approach:** Let them describe it, then ask "What's the time complexity of removing an element from the middle of an array?" Guide them toward DLL.
- **If they don't use sentinel nodes and get tangled in null checks:** Suggest sentinels after they've struggled for ~3 minutes. This is a design choice, not a fundamental misunderstanding.
- **If they forget to handle duplicates:** Let them write tests. The failing test should reveal the issue. If they don't write tests, ask "What happens if I view the same restaurant twice?"
- **If Stage 1 takes over 30 minutes:** Provide the DLL helper methods and let them focus on the `view()` logic.

### Common Mistakes

1. **Forgetting to delete from the hash map on eviction.** The DLL node is removed but the map still references it, causing ghost entries.
2. **Not handling the duplicate case.** Viewing a restaurant already in the cache should move it to the front, not add a second copy.
3. **Off-by-one in capacity check.** Checking `len(self.map) >= self.capacity` before insertion vs. `len(self.map) > self.capacity` after insertion.
4. **Pointer bugs in `_remove` or `_insert_after_head`.** Forgetting to update both directions of the doubly linked list. Drawing a diagram helps.
5. **Using `self.tail` as a data node instead of a sentinel.** Mixing sentinel and data roles causes confusing bugs.
6. **Not handling capacity 0.** Results in immediate eviction of every viewed restaurant, which is effectively a no-op but can cause crashes if not guarded.

### Edge Cases to Watch For

- Capacity 0 (should store nothing)
- Capacity 1 (every new view evicts the previous one)
- Viewing the same restaurant many times consecutively
- Expiring a restaurant that was never viewed or was already evicted
- `get_recent(0)` should return `[]`
- `get_recent(k)` where k exceeds the number of items
- Viewing a restaurant, expiring it, then viewing it again (should work normally)
- Interleaving `view`, `expire`, and `get_recent` calls
