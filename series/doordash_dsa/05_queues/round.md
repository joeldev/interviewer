# Round 05: Queues — "Customer Support Ticket System"

## Metadata
- **Topic:** Queues
- **Difficulty:** 2/5
- **Estimated Time:** 55-60 minutes
- **Key Concepts:** Multi-level queues, priority queue semantics, ticket aging / fairness

## Problem Prompt

> DoorDash's customer support system handles tickets at three priority levels:
>
> | Priority | Meaning |
> |----------|---------|
> | 1 | Urgent (e.g., safety issue, active order problem) |
> | 2 | High (e.g., missing item, refund request) |
> | 3 | Normal (e.g., general inquiry, feedback) |
>
> Design a `TicketQueue` that processes tickets in priority order.  Among tickets
> with the **same** priority, they should be served in **FIFO** (first-in,
> first-out) order.

*(Read the prompt exactly as written. Let the candidate ask about edge cases
before revealing the clarifying answers below.)*

## Clarifying Questions & Hidden Constraints

| # | What the candidate should ask | Answer |
|---|-------------------------------|--------|
| 1 | Can the queue be empty when `dequeue()` is called? | Yes. Return `None` (or raise — either is fine, just be consistent). |
| 2 | Can tickets have priorities other than 1, 2, 3? | No. Only 1, 2, 3 are valid. Raise `ValueError` on anything else. |
| 3 | What does a "ticket" look like? | A ticket is just a string ID like `"T100"`. Priority is supplied separately. |
| 4 | Can the same ticket ID be enqueued twice? | For Stages 1 and 2, assume ticket IDs are unique. |
| 5 | Can a ticket's priority change after enqueuing? | Not in Stage 1. Stage 2 will introduce this. |
| 6 | Is there a maximum queue size? | No hard limit. Assume it fits in memory. |

## Skeleton Code

```python
from collections import deque


class TicketQueue:
    """
    A multi-level priority queue for customer support tickets.
    Priority 1 = urgent, 2 = high, 3 = normal.
    FIFO within the same priority level.
    """

    def __init__(self):
        pass

    def enqueue(self, ticket_id: str, priority: int) -> None:
        """Add a ticket with the given priority (1, 2, or 3)."""
        pass

    def dequeue(self) -> str | None:
        """
        Remove and return the highest-priority (lowest number) ticket.
        Return None if the queue is empty.
        """
        pass

    def peek(self) -> str | None:
        """Return the next ticket without removing it, or None if empty."""
        pass

    def size(self) -> int:
        """Return total number of tickets across all priorities."""
        pass

if __name__ == "__main__":
    q = TicketQueue()
    q.enqueue("T1", 3)
    q.enqueue("T2", 1)
    q.enqueue("T3", 2)
    # Expected: T2 (priority 1 first)
    print(q.dequeue())
    # Expected: T3 (priority 2 next)
    print(q.dequeue())
    # Expected: T1 (priority 3 last)
    print(q.dequeue())
    # Expected: None (empty queue)
    print(q.dequeue())
    # Expected: 0
    print(q.size())
```

---

## Stage 1: Multi-Level Priority Queue (~20 minutes)

### Expected Approach

Use three `collections.deque` instances — one per priority level.

- `enqueue`: validate priority, append to the appropriate deque.
- `dequeue`: check deques in priority order (1, 2, 3); popleft from the first
  non-empty one.
- `peek`: same scan but return without popping.
- `size`: sum of lengths.

This gives O(1) enqueue, O(1) dequeue (constant number of priority levels),
and O(1) size.

### Solution Code

```python
from collections import deque


class TicketQueue:
    """
    A multi-level priority queue for customer support tickets.
    Priority 1 = urgent, 2 = high, 3 = normal.
    FIFO within the same priority level.
    """

    VALID_PRIORITIES = (1, 2, 3)

    def __init__(self):
        self._queues: dict[int, deque[str]] = {
            1: deque(),
            2: deque(),
            3: deque(),
        }

    def enqueue(self, ticket_id: str, priority: int) -> None:
        """Add a ticket with the given priority (1, 2, or 3)."""
        if priority not in self.VALID_PRIORITIES:
            raise ValueError(f"Invalid priority {priority}. Must be 1, 2, or 3.")
        self._queues[priority].append(ticket_id)

    def dequeue(self) -> str | None:
        """
        Remove and return the highest-priority (lowest number) ticket.
        Return None if the queue is empty.
        """
        for pri in self.VALID_PRIORITIES:
            if self._queues[pri]:
                return self._queues[pri].popleft()
        return None

    def peek(self) -> str | None:
        """Return the next ticket without removing it, or None if empty."""
        for pri in self.VALID_PRIORITIES:
            if self._queues[pri]:
                return self._queues[pri][0]
        return None

    def size(self) -> int:
        """Return total number of tickets across all priorities."""
        return sum(len(q) for q in self._queues.values())
```

### Complexity

| | |
|---|---|
| **Time** | O(1) for all operations (constant number of priority levels). |
| **Space** | O(n) where n is the total number of tickets in the queue. |

### Test Cases

```python
q = TicketQueue()

# Basic enqueue / dequeue respects priority
q.enqueue("T1", 3)
q.enqueue("T2", 1)
q.enqueue("T3", 2)
assert q.dequeue() == "T2"  # priority 1 first
assert q.dequeue() == "T3"  # then priority 2
assert q.dequeue() == "T1"  # then priority 3

# FIFO within same priority
q.enqueue("A", 2)
q.enqueue("B", 2)
q.enqueue("C", 2)
assert q.dequeue() == "A"
assert q.dequeue() == "B"
assert q.dequeue() == "C"

# Empty queue
assert q.dequeue() is None
assert q.peek() is None
assert q.size() == 0

# Invalid priority
try:
    q.enqueue("X", 5)
    assert False, "Should have raised ValueError"
except ValueError:
    pass
```

---

## Stage 2: Priority Upgrade (~20 minutes)

### Prompt Addition

> Support agents sometimes escalate a ticket to a higher priority.  Add a
> method:
>
> ```python
> def upgrade(self, ticket_id: str, new_priority: int) -> None:
> ```
>
> **Rules:**
> - The new priority must be strictly higher (lower number) than the ticket's
>   current priority. If not, raise `ValueError`.
> - If the ticket is not in the queue, raise `KeyError`.
> - Among tickets at the *new* priority level, upgraded tickets should appear
>   **after** any tickets already at that level (i.e., upgrading preserves FIFO
>   relative to existing tickets at the destination priority).
> - Multiple upgrades should preserve their relative order — if T5 is upgraded
>   before T8, T5 should be dequeued before T8 at the new level.

### Starter Code

```python
def upgrade(self, ticket_id: str, new_priority: int) -> None:
    """
    Move a ticket to a higher priority (lower number).
    Raises KeyError if ticket not found.
    Raises ValueError if new_priority is not strictly higher.
    """
    pass

# --- Stage 2 Test Calls ---
# q = TicketQueue()
# q.enqueue("T1", 3)
# q.enqueue("T2", 3)
# q.enqueue("T3", 2)
# q.upgrade("T2", 1)
# Expected: T2 (upgraded to priority 1)
# print(q.dequeue())
# Expected: T3 (priority 2)
# print(q.dequeue())
# Expected: T1 (priority 3)
# print(q.dequeue())
```

### Expected Approach

To support O(1)-ish upgrade:

1. Keep a `dict[str, int]` mapping `ticket_id -> current_priority` for fast
   lookup.
2. On `upgrade`: remove the ticket from its current deque (O(n) in that deque —
   acceptable for an interview) and append it to the new priority deque.
3. Update the lookup dict.

Alternatively, a candidate may use a *lazy deletion* approach: mark the old
entry as invalid and re-enqueue at the new level. On `dequeue`, skip invalid
entries. Both approaches are acceptable.

The direct-removal approach is simpler to reason about and preferred for
clarity.

### Solution Code

```python
from collections import deque


class TicketQueue:
    """
    Multi-level priority queue with upgrade support.
    """

    VALID_PRIORITIES = (1, 2, 3)

    def __init__(self):
        self._queues: dict[int, deque[str]] = {
            1: deque(),
            2: deque(),
            3: deque(),
        }
        self._ticket_priority: dict[str, int] = {}

    def enqueue(self, ticket_id: str, priority: int) -> None:
        if priority not in self.VALID_PRIORITIES:
            raise ValueError(f"Invalid priority {priority}. Must be 1, 2, or 3.")
        self._queues[priority].append(ticket_id)
        self._ticket_priority[ticket_id] = priority

    def dequeue(self) -> str | None:
        for pri in self.VALID_PRIORITIES:
            if self._queues[pri]:
                ticket = self._queues[pri].popleft()
                del self._ticket_priority[ticket]
                return ticket
        return None

    def peek(self) -> str | None:
        for pri in self.VALID_PRIORITIES:
            if self._queues[pri]:
                return self._queues[pri][0]
        return None

    def size(self) -> int:
        return sum(len(q) for q in self._queues.values())

    def upgrade(self, ticket_id: str, new_priority: int) -> None:
        """
        Move a ticket to a higher priority (lower number).
        Raises KeyError if ticket not found.
        Raises ValueError if new_priority is not strictly higher.
        """
        if ticket_id not in self._ticket_priority:
            raise KeyError(f"Ticket '{ticket_id}' not found in queue.")
        if new_priority not in self.VALID_PRIORITIES:
            raise ValueError(f"Invalid priority {new_priority}. Must be 1, 2, or 3.")

        current = self._ticket_priority[ticket_id]
        if new_priority >= current:
            raise ValueError(
                f"New priority {new_priority} must be strictly higher "
                f"(lower number) than current priority {current}."
            )

        # Remove from current queue.
        self._queues[current].remove(ticket_id)

        # Add to new queue (at the back, preserving FIFO for upgrades).
        self._queues[new_priority].append(ticket_id)
        self._ticket_priority[ticket_id] = new_priority
```

### Complexity

| | |
|---|---|
| **Time** | `enqueue`: O(1). `dequeue`: O(1). `upgrade`: O(n) in the worst case due to `deque.remove()`. |
| **Space** | O(n) for the queues plus O(n) for the lookup dict. |

### Test Cases

```python
q = TicketQueue()

# Upgrade moves ticket to higher priority
q.enqueue("T1", 3)
q.enqueue("T2", 3)
q.enqueue("T3", 2)
q.upgrade("T2", 1)
assert q.dequeue() == "T2"  # upgraded to priority 1
assert q.dequeue() == "T3"  # priority 2
assert q.dequeue() == "T1"  # priority 3

# Upgraded tickets appear after existing tickets at destination level
q.enqueue("A", 1)
q.enqueue("B", 2)
q.enqueue("C", 3)
q.upgrade("C", 1)
assert q.dequeue() == "A"  # was already at priority 1
assert q.dequeue() == "C"  # upgraded to priority 1, but after A

# Multiple upgrades preserve relative order
q.enqueue("X", 3)
q.enqueue("Y", 3)
q.enqueue("Z", 3)
q.upgrade("X", 2)
q.upgrade("Z", 2)
assert q.dequeue() == "X"  # upgraded first
assert q.dequeue() == "Z"  # upgraded second
assert q.dequeue() == "Y"  # still at priority 3

# Error cases
try:
    q.enqueue("E1", 1)
    q.upgrade("E1", 2)  # downgrade not allowed
    assert False
except ValueError:
    q.dequeue()  # clean up

try:
    q.upgrade("NONEXISTENT", 1)
    assert False
except KeyError:
    pass
```

---

## Stage 3: Fairness Constraint — Aging (~15 minutes)

### Prompt Addition

> Product has reported that during peak hours, priority-3 tickets sometimes wait
> indefinitely because urgent and high-priority tickets keep arriving.
>
> Add a **fairness constraint**: no priority-3 ticket may wait through more than
> `K` dequeue operations without being served.  Specifically, if any priority-3
> ticket has been in the queue for at least `K` dequeues (counting from when it
> was enqueued), the *next* `dequeue()` call **must** serve the oldest priority-3
> ticket — even if higher-priority tickets are waiting.
>
> The constructor now accepts a parameter `fairness_k: int` (default `0` meaning
> no fairness constraint).

### Starter Code

```python
# Update the constructor signature to:
def __init__(self, fairness_k: int = 0):
    pass
# The dequeue() method should now check the fairness constraint
# before following the normal priority order.

# --- Stage 3 Test Calls ---
# q = TicketQueue(fairness_k=3)
# q.enqueue("LOW", 3)
# q.enqueue("URG1", 1)
# q.enqueue("URG2", 1)
# q.enqueue("URG3", 1)
# Expected: URG1 (dequeue_count=1, LOW age=1 < 3)
# print(q.dequeue())
# Expected: URG2 (dequeue_count=2, LOW age=2 < 3)
# print(q.dequeue())
# Expected: LOW (dequeue_count=3, LOW age=3 >= 3 -> forced)
# print(q.dequeue())
```

### Expected Approach

1. Maintain a global `dequeue_count` that increments on every `dequeue()` call.
2. When a priority-3 ticket is enqueued, record the current `dequeue_count` as
   its "enqueue time".
3. On `dequeue()`, before following the normal priority order, check the oldest
   priority-3 ticket.  If `current_dequeue_count - enqueue_time >= K` (and
   `K > 0`), force-serve that ticket.
4. Otherwise, follow the normal priority scan.

### Solution Code

```python
from collections import deque


class TicketQueue:
    """
    Multi-level priority queue with upgrade and fairness support.
    """

    VALID_PRIORITIES = (1, 2, 3)

    def __init__(self, fairness_k: int = 0):
        self._queues: dict[int, deque[str]] = {
            1: deque(),
            2: deque(),
            3: deque(),
        }
        self._ticket_priority: dict[str, int] = {}

        # Fairness tracking
        self._fairness_k = fairness_k
        self._dequeue_count = 0
        # Tracks enqueue-time (in dequeue_count units) for priority-3 tickets.
        self._enqueue_time: dict[str, int] = {}

    def enqueue(self, ticket_id: str, priority: int) -> None:
        if priority not in self.VALID_PRIORITIES:
            raise ValueError(f"Invalid priority {priority}. Must be 1, 2, or 3.")
        self._queues[priority].append(ticket_id)
        self._ticket_priority[ticket_id] = priority
        if priority == 3:
            self._enqueue_time[ticket_id] = self._dequeue_count

    def dequeue(self) -> str | None:
        if self.size() == 0:
            return None

        self._dequeue_count += 1

        # Fairness check: if K > 0, see if the oldest priority-3 ticket has
        # waited >= K dequeues.
        if (
            self._fairness_k > 0
            and self._queues[3]
        ):
            oldest_p3 = self._queues[3][0]
            age = self._dequeue_count - self._enqueue_time[oldest_p3]
            if age >= self._fairness_k:
                ticket = self._queues[3].popleft()
                del self._ticket_priority[ticket]
                del self._enqueue_time[ticket]
                return ticket

        # Normal priority-order dequeue.
        for pri in self.VALID_PRIORITIES:
            if self._queues[pri]:
                ticket = self._queues[pri].popleft()
                del self._ticket_priority[ticket]
                if ticket in self._enqueue_time:
                    del self._enqueue_time[ticket]
                return ticket

        return None  # unreachable given the size check, but defensive

    def peek(self) -> str | None:
        # Peek must also reflect fairness — show what dequeue() would return.
        if self.size() == 0:
            return None

        if self._fairness_k > 0 and self._queues[3]:
            oldest_p3 = self._queues[3][0]
            age = (self._dequeue_count + 1) - self._enqueue_time[oldest_p3]
            if age >= self._fairness_k:
                return oldest_p3

        for pri in self.VALID_PRIORITIES:
            if self._queues[pri]:
                return self._queues[pri][0]
        return None

    def size(self) -> int:
        return sum(len(q) for q in self._queues.values())

    def upgrade(self, ticket_id: str, new_priority: int) -> None:
        if ticket_id not in self._ticket_priority:
            raise KeyError(f"Ticket '{ticket_id}' not found in queue.")
        if new_priority not in self.VALID_PRIORITIES:
            raise ValueError(f"Invalid priority {new_priority}.")

        current = self._ticket_priority[ticket_id]
        if new_priority >= current:
            raise ValueError(
                f"New priority {new_priority} must be strictly higher "
                f"than current {current}."
            )

        self._queues[current].remove(ticket_id)
        self._queues[new_priority].append(ticket_id)
        self._ticket_priority[ticket_id] = new_priority

        # If ticket was priority 3 and is now upgraded, remove its aging record.
        if ticket_id in self._enqueue_time:
            del self._enqueue_time[ticket_id]
```

### Complexity

| | |
|---|---|
| **Time** | `dequeue`: O(1). `enqueue`: O(1). `upgrade`: O(n). |
| **Space** | O(n) for queues, lookup dict, and enqueue-time dict. |

### Test Cases

```python
# Without fairness (K=0), priority-3 can starve
q = TicketQueue(fairness_k=0)
q.enqueue("P3", 3)
q.enqueue("P1a", 1)
q.enqueue("P1b", 1)
assert q.dequeue() == "P1a"
assert q.dequeue() == "P1b"
assert q.dequeue() == "P3"

# With fairness K=3, priority-3 must be served after 3 dequeues of waiting
q = TicketQueue(fairness_k=3)
q.enqueue("LOW", 3)       # enqueue_time = 0
q.enqueue("URG1", 1)
q.enqueue("URG2", 1)
q.enqueue("URG3", 1)
q.enqueue("URG4", 1)

assert q.dequeue() == "URG1"  # dequeue_count=1, LOW age=1 < 3
assert q.dequeue() == "URG2"  # dequeue_count=2, LOW age=2 < 3
assert q.dequeue() == "LOW"   # dequeue_count=3, LOW age=3 >= 3 -> forced
assert q.dequeue() == "URG3"  # back to normal priority order
assert q.dequeue() == "URG4"

# Fairness triggers repeatedly
q = TicketQueue(fairness_k=2)
q.enqueue("N1", 3)  # enqueue_time=0
q.enqueue("N2", 3)  # enqueue_time=0
q.enqueue("U1", 1)
q.enqueue("U2", 1)
q.enqueue("U3", 1)
q.enqueue("U4", 1)

assert q.dequeue() == "U1"   # count=1, N1 age=1 < 2
assert q.dequeue() == "N1"   # count=2, N1 age=2 >= 2 -> forced
assert q.dequeue() == "U2"   # count=3, N2 age=3 >= 2 -> wait, N2 enqueue_time=0, age=3 >= 2 -> forced?
# Actually N2 enqueue_time=0, count will be 3, age = 3 >= 2 so N2 is forced
# Let's re-trace:
#   dequeue #1: count=1, N1 age=1<2 -> URG1. Result: U1
#   dequeue #2: count=2, N1 age=2>=2 -> forced. Result: N1
#   dequeue #3: count=3, N2 age=3>=2 -> forced. Result: N2
#   dequeue #4: count=4, no p3 tickets -> U2
#   dequeue #5: count=5 -> U3
#   dequeue #6: count=6 -> U4

q2 = TicketQueue(fairness_k=2)
q2.enqueue("N1", 3)
q2.enqueue("N2", 3)
q2.enqueue("U1", 1)
q2.enqueue("U2", 1)
q2.enqueue("U3", 1)
q2.enqueue("U4", 1)

assert q2.dequeue() == "U1"
assert q2.dequeue() == "N1"
assert q2.dequeue() == "N2"
assert q2.dequeue() == "U2"
assert q2.dequeue() == "U3"
assert q2.dequeue() == "U4"
```

---

## Scoring Rubric

### Fundamentals (1-4)

| Score | Description |
|-------|-------------|
| 1 | Cannot describe FIFO semantics or how a queue differs from a stack. |
| 2 | Understands queues but uses a list with `pop(0)` instead of `deque` (O(n) dequeue). Needs hints for multi-level design. |
| 3 | Implements Stage 1 correctly with `deque`. Makes meaningful progress on Stage 2. |
| 4 | Completes all three stages. Clearly explains why `deque` gives O(1) popleft. Discusses trade-offs of upgrade approaches (lazy deletion vs. direct removal). |

### Coding (1-4)

| Score | Description |
|-------|-------------|
| 1 | Code does not run; fundamental errors in queue operations. |
| 2 | Stage 1 works but upgrade or fairness logic has bugs the candidate cannot resolve. |
| 3 | Stages 1 and 2 are correct. Stage 3 has minor off-by-one or edge-case issues. |
| 4 | All stages correct, clean, idiomatic. Proper use of `collections.deque`, clear naming, good error handling with `ValueError` / `KeyError`. |

### Communication (1-4)

| Score | Description |
|-------|-------------|
| 1 | Codes silently; cannot explain the multi-queue approach. |
| 2 | Explains approach only when prompted; unclear on trade-offs. |
| 3 | Describes the multi-queue design before coding. Discusses upgrade strategies when asked. |
| 4 | Proactively identifies edge cases (empty queue, invalid priority). Discusses fairness trade-offs. Walks through test scenarios aloud. Asks about requirements before coding each stage. |

---

## Interviewer Guide

### Hints (per stage, progressive: mild -> medium -> strong)

**Stage 1 — Multi-Level Priority Queue**

- **Mild:** "How might you organize tickets so you always know which priority to check first?"
- **Medium:** "What if you had a separate FIFO container for each priority level and checked them in order?"
- **Strong:** "Create three `collections.deque` instances keyed by priority 1, 2, 3. Enqueue appends to the right deque. Dequeue iterates 1, 2, 3 and poplefts from the first non-empty one."

**Stage 2 — Priority Upgrade**

- **Mild:** "How would you find a ticket in the queue quickly, given its ID?"
- **Medium:** "Consider maintaining a dictionary from ticket ID to its current priority. Then you know which deque to search."
- **Strong:** "Use `deque.remove(ticket_id)` to pull it from the old priority's deque, then `append` it to the new priority's deque. Update the dict. Mention that `remove` is O(n) but acceptable here."

**Stage 3 — Fairness / Aging**

- **Mild:** "How could you track how long a priority-3 ticket has been waiting?"
- **Medium:** "What if you recorded a 'timestamp' (in terms of dequeue operations) when each priority-3 ticket enters, and checked the oldest one on each dequeue?"
- **Strong:** "Keep a `dequeue_count`. On enqueue of a priority-3 ticket, store `enqueue_time = dequeue_count`. On each dequeue, increment the counter, then check if the front of the priority-3 queue has `age >= K`. If so, force-serve it."

### When to Intervene

- If the candidate reaches for `heapq` for Stage 1, gently redirect: "Since there are only three priority levels, can you do something simpler than a heap?" A heap works but misses the point of multi-level queues.
- If the candidate uses `list.pop(0)` instead of `deque.popleft()`, ask about the time complexity — this is a teachable moment, not a dealbreaker.
- If Stage 2 stalls for more than 10 minutes, provide the medium hint to keep pace.

### Common Mistakes

1. **Using `list` instead of `deque`.** `list.pop(0)` is O(n). Ask the candidate to discuss complexity to surface this.
2. **Upgrade: not handling the "ticket not found" case.** The candidate should raise `KeyError`.
3. **Upgrade: allowing downgrade or same-priority "upgrade".** Must validate `new_priority < current`.
4. **Fairness: off-by-one in age calculation.** The age should be `dequeue_count - enqueue_time`, not `dequeue_count - enqueue_time - 1`. Clarify: the dequeue that serves the ticket counts as a dequeue.
5. **Fairness: not incrementing `dequeue_count` when the force-served ticket is dequeued.** The counter must increment on *every* dequeue call.
6. **Forgetting to clean up `_enqueue_time` when a priority-3 ticket is upgraded to a higher priority.**

### Edge Cases to Watch For

- Empty queue: `dequeue()` and `peek()` should return `None`, not crash.
- Single ticket in queue: all operations still work.
- All tickets same priority: behaves as a plain FIFO queue.
- Upgrade to priority 1 then dequeue immediately: upgraded ticket should be at the back of priority-1 queue.
- Fairness K=1: every other dequeue serves priority-3 (if any are waiting). This is extreme but correct.
- Fairness with no priority-3 tickets: normal behavior, no forced serves.
- Ticket is upgraded from priority 3 to priority 1: its aging record should be removed so it no longer triggers the fairness constraint.
