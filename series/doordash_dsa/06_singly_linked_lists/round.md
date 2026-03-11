# Round 06: Singly Linked Lists — "Dasher Route Waypoints"

## Metadata
- **Topic:** Singly Linked Lists
- **Difficulty:** 2/5
- **Estimated Time:** 55-60 minutes
- **Key Concepts:** Floyd's cycle detection (tortoise and hare), cycle start identification, cycle removal, in-place list reversal

## Problem Prompt

> A dasher's delivery route is represented as a singly linked list of waypoints.
> Each node contains a waypoint name and a `next` pointer to the following
> waypoint.
>
> Sometimes, due to a bug in the routing system, a waypoint's `next` pointer
> gets corrupted and points back to an earlier waypoint in the route, creating a
> **cycle**.  A dasher stuck in a cycle would drive in circles forever.
>
> Given the head of a dasher's route, determine whether the route contains a
> cycle.

*(Read the prompt exactly as written. Let the candidate ask about edge cases
before revealing the clarifying answers below.)*

## Clarifying Questions & Hidden Constraints

| # | What the candidate should ask | Answer |
|---|-------------------------------|--------|
| 1 | Can the list be empty (`head is None`)? | Yes. An empty list has no cycle — return `False`. |
| 2 | Can the list have just one node? | Yes. It has a cycle only if `node.next == node`. |
| 3 | Can there be duplicate waypoint names? | Yes. Nodes are identified by object identity, not by name. |
| 4 | What space complexity is expected? | O(1) extra space is the target. Using a hash set of visited nodes is O(n) and acceptable as a brute-force starting point, but the interviewer wants the O(1) approach. |
| 5 | Can the cycle start at the head node? | Yes. |
| 6 | If I need to build test linked lists, should I implement the node class? | A `ListNode` class is provided. |

## Skeleton Code

```python
class ListNode:
    """A waypoint in the dasher's route."""

    def __init__(self, name: str, next_node: "ListNode | None" = None):
        self.name = name
        self.next = next_node

    def __repr__(self) -> str:
        return f"ListNode({self.name!r})"


def has_cycle(head: ListNode | None) -> bool:
    """Return True if the linked list starting at `head` contains a cycle."""
    pass


# ── Helper to build test lists ──────────────────────────────────────

def build_list(names: list[str], cycle_to_index: int = -1) -> ListNode | None:
    """
    Build a singly linked list from `names`.
    If `cycle_to_index` >= 0, the last node's `next` points to the node
    at that index (0-based), creating a cycle.
    """
    if not names:
        return None
    nodes = [ListNode(n) for n in names]
    for i in range(len(nodes) - 1):
        nodes[i].next = nodes[i + 1]
    if 0 <= cycle_to_index < len(nodes):
        nodes[-1].next = nodes[cycle_to_index]
    return nodes[0]


def list_to_names(head: ListNode | None) -> list[str]:
    """Convert a non-cyclic linked list to a list of waypoint names."""
    result = []
    current = head
    while current:
        result.append(current.name)
        current = current.next
    return result

if __name__ == "__main__":
    # Expected: False (no cycle)
    route = build_list(["Restaurant", "PickUp", "DropOff"])
    print(has_cycle(route))
    # Expected: True (cycle back to head)
    route = build_list(["A", "B", "C", "D"], cycle_to_index=0)
    print(has_cycle(route))
    # Expected: False (empty list)
    print(has_cycle(None))
```

---

## Stage 1: Cycle Detection (~20 minutes)

### Expected Approach

**Floyd's Tortoise and Hare algorithm:**

1. Initialize two pointers, `slow` and `fast`, both at `head`.
2. Move `slow` one step and `fast` two steps per iteration.
3. If `fast` or `fast.next` is `None`, there is no cycle.
4. If `slow == fast`, there is a cycle.

This is O(n) time and O(1) space.

If the candidate first proposes a hash-set approach (store visited nodes, check
for revisits), acknowledge it as correct O(n) time / O(n) space, then ask for
the O(1) space version.

### Solution Code

```python
def has_cycle(head: ListNode | None) -> bool:
    """Return True if the linked list starting at `head` contains a cycle."""
    slow = head
    fast = head

    while fast is not None and fast.next is not None:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:
            return True

    return False
```

### Complexity

| | |
|---|---|
| **Time** | O(n). In the worst case, the fast pointer traverses at most 2n nodes before meeting the slow pointer. |
| **Space** | O(1). Only two extra pointers regardless of list size. |

### Test Cases

```python
# No cycle
route = build_list(["Restaurant", "PickUp", "DropOff"])
assert has_cycle(route) == False

# Cycle back to head
route = build_list(["A", "B", "C", "D"], cycle_to_index=0)
assert has_cycle(route) == True

# Cycle to middle node
route = build_list(["A", "B", "C", "D", "E"], cycle_to_index=2)
assert has_cycle(route) == True

# Single node, no cycle
route = build_list(["Solo"])
assert has_cycle(route) == False

# Single node, self-loop
solo = ListNode("Solo")
solo.next = solo
assert has_cycle(solo) == True

# Empty list
assert has_cycle(None) == False
```

---

## Stage 2: Find the Cycle Start Node (~20 minutes)

### Prompt Addition

> Good — now we know a cycle exists. But to fix the routing bug, we need to know
> **which waypoint** the cycle loops back to.  Given the head of the list, return
> the **node where the cycle begins**.  If there is no cycle, return `None`.

### Starter Code

```python
def find_cycle_start(head: ListNode | None) -> ListNode | None:
    """
    If the list has a cycle, return the node where the cycle begins.
    If there is no cycle, return None.
    """
    pass

# --- Stage 2 Test Calls ---
# Expected: node with name "A" (cycle starts at head)
# nodes = [ListNode(n) for n in ["A", "B", "C", "D"]]
# for i in range(len(nodes) - 1): nodes[i].next = nodes[i + 1]
# nodes[-1].next = nodes[0]
# print(find_cycle_start(nodes[0]))
# Expected: None (no cycle)
# route = build_list(["X", "Y", "Z"])
# print(find_cycle_start(route))
# Expected: None (empty list)
# print(find_cycle_start(None))
```

### Expected Approach

This is the second phase of Floyd's algorithm:

1. Detect the cycle using the tortoise-and-hare method (Stage 1).
2. If no cycle, return `None`.
3. Once `slow` and `fast` meet inside the cycle, reset one pointer (say `slow`)
   back to `head`.
4. Advance both `slow` and `fast` **one step at a time**.
5. The node where they meet again is the start of the cycle.

**Why this works (the candidate should be able to explain at a high level):**

Let `L` = distance from head to cycle start, `C` = cycle length, and `K` =
distance from cycle start to the meeting point. When they first meet,
`slow` has traveled `L + K` steps and `fast` has traveled `2(L + K)` steps.
The difference `L + K` is a multiple of `C`, so `L + K = mC` for some integer
`m`. Therefore `L = mC - K`, meaning if we start one pointer at the head and
one at the meeting point and advance both by 1, they will meet at the cycle
start after `L` steps.

### Solution Code

```python
def find_cycle_start(head: ListNode | None) -> ListNode | None:
    """
    If the list has a cycle, return the node where the cycle begins.
    If there is no cycle, return None.
    """
    slow = head
    fast = head

    # Phase 1: Detect cycle (find meeting point).
    while fast is not None and fast.next is not None:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:
            break
    else:
        # fast or fast.next is None -> no cycle.
        return None

    # Phase 2: Find cycle start.
    # Reset slow to head; advance both one step at a time.
    slow = head
    while slow is not fast:
        slow = slow.next
        fast = fast.next

    return slow
```

### Complexity

| | |
|---|---|
| **Time** | O(n). Phase 1 is O(n), Phase 2 is O(n) in the worst case. |
| **Space** | O(1). Only two pointers. |

### Test Cases

```python
# Cycle starts at head (index 0)
nodes = [ListNode(n) for n in ["A", "B", "C", "D"]]
for i in range(len(nodes) - 1):
    nodes[i].next = nodes[i + 1]
nodes[-1].next = nodes[0]
assert find_cycle_start(nodes[0]) is nodes[0]

# Cycle starts at middle (index 2)
nodes = [ListNode(n) for n in ["A", "B", "C", "D", "E"]]
for i in range(len(nodes) - 1):
    nodes[i].next = nodes[i + 1]
nodes[-1].next = nodes[2]
assert find_cycle_start(nodes[0]) is nodes[2]

# Cycle starts at last node (self-loop)
nodes = [ListNode(n) for n in ["A", "B", "C"]]
for i in range(len(nodes) - 1):
    nodes[i].next = nodes[i + 1]
nodes[-1].next = nodes[-1]
assert find_cycle_start(nodes[0]) is nodes[-1]

# No cycle
route = build_list(["X", "Y", "Z"])
assert find_cycle_start(route) is None

# Empty list
assert find_cycle_start(None) is None

# Single node self-loop
solo = ListNode("Solo")
solo.next = solo
assert find_cycle_start(solo) is solo
```

---

## Stage 3: Remove Cycle and Reverse (~15 minutes)

### Prompt Addition

> Now fix the route: **remove the cycle** by breaking the back-link, then
> **reverse the entire list** so the dasher can retrace the route back to the
> restaurant.  Return the new head of the reversed list.
>
> If the list has no cycle, just reverse it.  If the list is empty, return
> `None`.

### Starter Code

```python
def remove_cycle_and_reverse(head: ListNode | None) -> ListNode | None:
    """
    If the list has a cycle, remove it.  Then reverse the entire list
    and return the new head.  If the list is empty, return None.
    """
    pass

# --- Stage 3 Test Calls ---
# Expected: ['D', 'C', 'B', 'A'] (no cycle — just reverse)
# route = build_list(["A", "B", "C", "D"])
# new_head = remove_cycle_and_reverse(route)
# print(list_to_names(new_head))
# Expected: None (empty list)
# print(remove_cycle_and_reverse(None))
```

### Expected Approach

**Part A — Remove the cycle:**

1. Find the cycle start using the Floyd method from Stage 2.
2. If there is a cycle, traverse from the cycle start around the cycle until
   you find the node whose `.next` is the cycle start.  Set that node's
   `.next = None`.
   - Special case: if the cycle start is the head and the tail points to head,
     we traverse the cycle to find the node just before the cycle start.

**Part B — Reverse the list:**

Standard iterative reversal:
1. `prev = None`, `curr = head`.
2. While `curr`: save `curr.next`, set `curr.next = prev`, advance `prev` and
   `curr`.
3. Return `prev` as the new head.

### Solution Code

```python
def remove_cycle_and_reverse(head: ListNode | None) -> ListNode | None:
    """
    If the list has a cycle, remove it.  Then reverse the entire list
    and return the new head.  If the list is empty, return None.
    """
    if head is None:
        return None

    # ── Step 1: Detect and remove cycle ──────────────────────────────
    cycle_start = find_cycle_start(head)

    if cycle_start is not None:
        # Walk around the cycle to find the node just before cycle_start.
        runner = cycle_start
        while runner.next is not cycle_start:
            runner = runner.next
        runner.next = None  # break the cycle

    # ── Step 2: Reverse the now-acyclic list ─────────────────────────
    prev = None
    curr = head
    while curr is not None:
        nxt = curr.next
        curr.next = prev
        prev = curr
        curr = nxt

    return prev
```

### Complexity

| | |
|---|---|
| **Time** | O(n). Cycle detection is O(n), cycle removal is O(C) where C is the cycle length (at most n), and reversal is O(n). |
| **Space** | O(1). All operations use a constant number of pointers. |

### Test Cases

```python
# List with no cycle — just reverse
route = build_list(["A", "B", "C", "D"])
new_head = remove_cycle_and_reverse(route)
assert list_to_names(new_head) == ["D", "C", "B", "A"]

# List with cycle back to head — remove cycle, then reverse
nodes = [ListNode(n) for n in ["A", "B", "C", "D"]]
for i in range(len(nodes) - 1):
    nodes[i].next = nodes[i + 1]
nodes[-1].next = nodes[0]  # cycle to head
new_head = remove_cycle_and_reverse(nodes[0])
assert list_to_names(new_head) == ["D", "C", "B", "A"]

# List with cycle to middle node
nodes = [ListNode(n) for n in ["A", "B", "C", "D", "E"]]
for i in range(len(nodes) - 1):
    nodes[i].next = nodes[i + 1]
nodes[-1].next = nodes[2]  # cycle to index 2 ("C")
new_head = remove_cycle_and_reverse(nodes[0])
assert list_to_names(new_head) == ["E", "D", "C", "B", "A"]

# Single node, no cycle
solo = build_list(["Solo"])
new_head = remove_cycle_and_reverse(solo)
assert list_to_names(new_head) == ["Solo"]

# Single node, self-loop
solo = ListNode("Solo")
solo.next = solo
new_head = remove_cycle_and_reverse(solo)
assert list_to_names(new_head) == ["Solo"]

# Empty list
assert remove_cycle_and_reverse(None) is None

# Two nodes with cycle
a = ListNode("A")
b = ListNode("B")
a.next = b
b.next = a  # cycle
new_head = remove_cycle_and_reverse(a)
assert list_to_names(new_head) == ["B", "A"]
```

---

## Scoring Rubric

### Fundamentals (1-4)

| Score | Description |
|-------|-------------|
| 1 | Cannot explain what a linked list cycle is or why it is a problem. Does not know Floyd's algorithm or any alternative. |
| 2 | Proposes a hash-set solution for cycle detection (O(n) space) but cannot derive the O(1) space approach without heavy hints. |
| 3 | Implements Floyd's algorithm for detection. Understands the Phase 2 logic for finding the cycle start with some guidance. |
| 4 | Implements all three stages fluently. Can explain *why* Phase 2 of Floyd's algorithm works (the `L + K = mC` argument). Discusses O(1) space advantage. |

### Coding (1-4)

| Score | Description |
|-------|-------------|
| 1 | Code does not run. Confused by pointer manipulation (e.g., loses references, infinite loops). |
| 2 | Stage 1 works. Stage 2 or 3 has pointer bugs (e.g., off-by-one in cycle removal, loses head reference in reversal). |
| 3 | Stages 1 and 2 work. Stage 3 is mostly correct with a minor edge-case bug (e.g., single-node self-loop not handled). |
| 4 | All stages correct and clean. Proper `None` checks, no infinite loops, handles all edge cases. Code is well-organized (helper reuse between stages). |

### Communication (1-4)

| Score | Description |
|-------|-------------|
| 1 | Codes silently. Cannot trace through the algorithm when asked. |
| 2 | Explains the slow/fast pointer idea only after prompting. Struggles to articulate Phase 2 logic. |
| 3 | Describes the tortoise-and-hare approach before coding. Walks through a small example. Explains Phase 2 with some hand-waving. |
| 4 | Proactively asks about edge cases (empty list, single node, cycle at head). Draws or traces through the pointer positions. Clearly explains Phase 2 math. Narrates code as they write it. |

---

## Interviewer Guide

### Hints (per stage, progressive: mild -> medium -> strong)

**Stage 1 — Cycle Detection**

- **Mild:** "Is there a way to detect the cycle without storing every node you visit?"
- **Medium:** "What if you had two pointers moving at different speeds? If there is a cycle, what eventually happens?"
- **Strong:** "Use a slow pointer (1 step) and a fast pointer (2 steps). If they ever point to the same node, there is a cycle. If fast reaches `None`, there is no cycle."

**Stage 2 — Find Cycle Start**

- **Mild:** "You have the meeting point from Stage 1. Is there a mathematical relationship between the meeting point and the cycle start?"
- **Medium:** "After the two pointers meet, reset one to the head. Now move both one step at a time. Where do they meet?"
- **Strong:** "Let L = distance from head to cycle start, K = distance from cycle start to meeting point, C = cycle length. At the meeting point, slow traveled L+K and fast traveled 2(L+K). The difference L+K is a multiple of C. So if you put one pointer at head and one at the meeting point and step both by 1, they converge at the cycle start after L steps."

**Stage 3 — Remove Cycle and Reverse**

- **Mild:** "Now that you know the cycle start, how do you find the *last* node in the cycle — the one whose `next` pointer creates the back-link?"
- **Medium:** "Walk from the cycle start, following `next`, until you find the node whose `next` is the cycle start. Set its `next` to `None`."
- **Strong (reversal):** "For in-place reversal, use three pointers: `prev`, `curr`, `nxt`. At each step: save `nxt = curr.next`, set `curr.next = prev`, advance `prev = curr` and `curr = nxt`. Return `prev` when `curr` is `None`."

### When to Intervene

- If the candidate jumps to a hash-set approach for Stage 1, let them finish, confirm it works, then ask: "Can you do this in O(1) space?" If stuck for more than 5 minutes after that, give the medium hint.
- If the candidate cannot explain Phase 2 of Floyd's algorithm after 8 minutes on Stage 2, give the strong hint — understanding the proof is nice but not blocking.
- If Stage 3 stalls on cycle removal, remind them they already have `find_cycle_start` and just need to walk the cycle to find the tail.

### Common Mistakes

1. **Stage 1: Checking `slow == fast` before advancing the pointers.** This causes a false positive on the first iteration since both start at `head`. Always advance first, then compare.
2. **Stage 1: Only checking `fast.next is not None` but not `fast is not None`.** If the list has an even number of nodes and no cycle, `fast` can become `None` directly.
3. **Stage 2: Confusing the meeting point with the cycle start.** The meeting point is *inside* the cycle but usually not the start.
4. **Stage 2: Forgetting the `else` clause on the `while` loop.** In Python, `while ... else` executes the `else` block only if the loop completes without `break`. Using `for/while ... else` is idiomatic but many candidates are unfamiliar with it. An alternative is to use a flag variable.
5. **Stage 3: Infinite loop during cycle removal.** If the candidate walks from `head` instead of from `cycle_start`, they may traverse the entire list and not find the back-link efficiently.
6. **Stage 3: Forgetting the self-loop case.** When a node points to itself, the "walk to find predecessor" loop body never executes unless the condition handles `runner.next is cycle_start` where `runner is cycle_start`.
7. **Stage 3: Reversing before removing the cycle.** This creates an infinite loop during reversal. Must remove the cycle first.

### Edge Cases to Watch For

- **Empty list (`None`):** All functions should handle this gracefully.
- **Single node, no cycle:** `has_cycle` returns `False`, reversal returns the same node.
- **Single node, self-loop (`node.next = node`):** `has_cycle` returns `True`, `find_cycle_start` returns the node, removal sets `node.next = None`, reversal returns the same node.
- **Two nodes with cycle (`A -> B -> A`):** Cycle start is `A`, removal breaks `B.next`, reversal gives `B -> A`.
- **Cycle at head (last node points to head):** The entire list is one big cycle. Removal breaks the last-to-first link. All nodes remain in the reversed list.
- **Cycle to the last node (self-loop on the tail):** Only the last node is in the cycle. Removal breaks `tail.next = tail` to `tail.next = None`.
- **Very long tail, short cycle:** e.g., 1000 nodes before the cycle starts, cycle of length 3. Floyd's algorithm still works in O(n).
