# Round 09: Binary Search Trees — "Dynamic Price Ranking"

## Metadata
- **Topic:** Binary Search Trees, BST Operations, Order-Statistic Queries, Node Deletion
- **Difficulty:** 3/5
- **Estimated Time:** 55-60 minutes
- **Key Concepts:** BST insert/search, closest value search, inorder traversal, k-th smallest element, BST deletion (three cases), balancing concepts

## Problem Prompt

> DoorDash is building a price comparison tool. Menu item prices are added dynamically as restaurants update their menus. We need a system that can:
>
> - Add a new price
> - Given a target budget, find the closest available price
>
> Design a data structure to support these operations efficiently.

*Read this prompt exactly. Let the candidate decide on the data structure. If they don't suggest a BST, guide them toward it.*

## Clarifying Questions & Hidden Constraints

| # | Question Candidate Should Ask | Answer |
|---|-------------------------------|--------|
| 1 | Can there be duplicate prices? | Yes — multiple items can have the same price. Your BST should handle duplicates. Convention: duplicates go to the right subtree. |
| 2 | Are prices integers or floats? | Floats. Use `float` type. |
| 3 | What does "closest" mean — can it be higher or lower? | Closest by absolute difference. If two prices are equally close, return either. |
| 4 | What's the expected number of prices? | Thousands to tens of thousands. O(log n) average-case is fine. |
| 5 | Can the tree be empty when we search? | Yes. Return `None` in that case. |
| 6 | What does "efficient" mean here? | O(h) per operation, where h is the tree height. For a balanced tree, that's O(log n). |

## Skeleton Code

```python
class BSTNode:
    def __init__(self, price: float):
        self.price = price
        self.left = None
        self.right = None


class PriceTracker:
    """Tracks menu item prices using a BST."""

    def __init__(self):
        self.root = None

    def insert(self, price: float) -> None:
        """Add a new price to the tracker."""
        pass

    def find_closest(self, target: float) -> float:
        """Find the price closest to the target budget."""
        pass

if __name__ == "__main__":
    pt = PriceTracker()
    for price in [12.99, 8.50, 15.00, 9.75, 7.25]:
        pt.insert(price)
    # Expected: 9.75
    print(pt.find_closest(10.00))
    # Expected: 12.99
    print(pt.find_closest(12.99))
    # Expected: None (empty tracker)
    empty = PriceTracker()
    print(empty.find_closest(10.00))
```

## Stage 1: Insert, Search, and Closest Price (~20 minutes)

### Expected Approach

**Insert:** Standard BST insertion. Compare the new price with the current node:
- If less, go left; if greater or equal, go right.
- When you reach a `None` position, insert the new node there.
- Can be done recursively or iteratively. Both are fine.

**Find closest:** Walk down the tree, tracking the closest value seen so far.
- At each node, update the best candidate if `|node.price - target|` is less than the current best.
- If `target < node.price`, go left (closer values might be smaller); if `target > node.price`, go right; if equal, return immediately.
- This is O(h) — you only traverse one root-to-leaf path.

The candidate should also implement or discuss basic `search(price) -> bool` to check if an exact price exists. This is straightforward BST search.

### Solution Code

```python
class BSTNode:
    def __init__(self, price: float):
        self.price = price
        self.left = None
        self.right = None


class PriceTracker:
    """Tracks menu item prices using a BST."""

    def __init__(self):
        self.root = None

    def insert(self, price: float) -> None:
        """Add a new price to the tracker."""
        if self.root is None:
            self.root = BSTNode(price)
            return
        self._insert_recursive(self.root, price)

    def _insert_recursive(self, node: BSTNode, price: float) -> BSTNode:
        """Recursively insert a price into the subtree rooted at node."""
        if node is None:
            return BSTNode(price)
        if price < node.price:
            node.left = self._insert_recursive(node.left, price)
        else:
            node.right = self._insert_recursive(node.right, price)
        return node

    def insert(self, price: float) -> None:
        """Add a new price to the tracker."""
        self.root = self._insert_recursive(self.root, price)

    def search(self, price: float) -> bool:
        """Check if an exact price exists in the tracker."""
        node = self.root
        while node:
            if price == node.price:
                return True
            elif price < node.price:
                node = node.left
            else:
                node = node.right
        return False

    def find_closest(self, target: float) -> float:
        """
        Find the price closest to the target budget.
        Returns None if the tree is empty.
        """
        if self.root is None:
            return None

        closest = self.root.price
        node = self.root

        while node:
            # Update closest if this node is nearer to target
            if abs(node.price - target) < abs(closest - target):
                closest = node.price

            # Early exit on exact match
            if node.price == target:
                return node.price

            # Navigate toward the target
            if target < node.price:
                node = node.left
            else:
                node = node.right

        return closest
```

### Complexity

| Operation | Average Case | Worst Case (skewed tree) |
|-----------|-------------|-------------------------|
| `insert` | O(log n) | O(n) |
| `search` | O(log n) | O(n) |
| `find_closest` | O(log n) | O(n) |
| Space | O(n) for n nodes | O(n) |

### Test Cases

```python
def test_stage_1():
    pt = PriceTracker()

    # Insert prices
    for price in [12.99, 8.50, 15.00, 9.75, 7.25, 13.50, 20.00]:
        pt.insert(price)

    # Search
    assert pt.search(12.99) is True
    assert pt.search(10.00) is False
    assert pt.search(7.25) is True

    # Find closest
    assert pt.find_closest(10.00) == 9.75  # 9.75 is closer than 12.99
    assert pt.find_closest(12.99) == 12.99  # exact match
    assert pt.find_closest(14.00) == 13.50  # 13.50 is closer than 15.00
    assert pt.find_closest(100.00) == 20.00  # beyond max
    assert pt.find_closest(1.00) == 7.25  # below min

    # Empty tracker
    empty = PriceTracker()
    assert empty.find_closest(10.00) is None

    # Single element
    single = PriceTracker()
    single.insert(5.00)
    assert single.find_closest(100.00) == 5.00
    assert single.find_closest(5.00) == 5.00

    # Duplicate prices
    pt.insert(12.99)
    assert pt.search(12.99) is True
    assert pt.find_closest(12.99) == 12.99

    print("Stage 1: All tests passed.")

test_stage_1()
```

## Stage 2: K-th Smallest Price (~20 minutes)

### Prompt Addition

> A customer wants to browse menu items from cheapest to most expensive. Implement `kth_smallest(k)` that returns the k-th smallest price (1-indexed). For example, if prices are [7.25, 8.50, 9.75, 12.99, 13.50, 15.00, 20.00], then `kth_smallest(1)` returns 7.25 and `kth_smallest(3)` returns 9.75.
>
> Also implement `get_sorted_prices()` that returns all prices in sorted order.

### Starter Code

```python
def get_sorted_prices(self) -> list:
    """Return all prices in ascending order via inorder traversal."""
    pass

def kth_smallest(self, k: int) -> float:
    """
    Return the k-th smallest price (1-indexed).
    Returns None if k is out of range.
    """
    pass

# --- Stage 2 Test Calls ---
# pt = PriceTracker()
# for p in [12.99, 8.50, 15.00, 9.75, 7.25, 13.50, 20.00]: pt.insert(p)
# Expected: [7.25, 8.50, 9.75, 12.99, 13.50, 15.00, 20.00]
# print(pt.get_sorted_prices())
# Expected: 7.25
# print(pt.kth_smallest(1))
# Expected: 9.75
# print(pt.kth_smallest(3))
```

### Expected Approach

**`get_sorted_prices()`:** A straightforward inorder traversal of a BST yields values in sorted order. This is the fundamental property of BSTs.

**`kth_smallest(k)`:**

**Approach A — Inorder traversal with early termination:**
Perform an inorder traversal but count nodes visited. Stop as soon as you've visited the k-th node. This is O(k + h) time — you don't need to traverse the entire tree.

**Approach B — Full inorder then index:**
Do a full inorder traversal to get a sorted list, then return the (k-1)-th element. This is O(n) time. Simpler but less efficient.

**Approach C (bonus, for strong candidates) — Augmented BST:**
Store subtree sizes in each node. At each node, if `left_size + 1 == k`, this is the answer. If `k <= left_size`, recurse left. Otherwise, recurse right with `k - left_size - 1`. This is O(h) time but requires maintaining sizes on insert/delete.

The candidate should implement at least Approach A. Discussing Approach C shows depth.

### Solution Code

```python
class PriceTracker:
    """Extended with kth_smallest and get_sorted_prices."""

    def __init__(self):
        self.root = None

    def _insert_recursive(self, node, price):
        if node is None:
            return BSTNode(price)
        if price < node.price:
            node.left = self._insert_recursive(node.left, price)
        else:
            node.right = self._insert_recursive(node.right, price)
        return node

    def insert(self, price: float) -> None:
        self.root = self._insert_recursive(self.root, price)

    def search(self, price: float) -> bool:
        node = self.root
        while node:
            if price == node.price:
                return True
            elif price < node.price:
                node = node.left
            else:
                node = node.right
        return False

    def find_closest(self, target: float) -> float:
        if self.root is None:
            return None
        closest = self.root.price
        node = self.root
        while node:
            if abs(node.price - target) < abs(closest - target):
                closest = node.price
            if node.price == target:
                return node.price
            if target < node.price:
                node = node.left
            else:
                node = node.right
        return closest

    def get_sorted_prices(self) -> list:
        """Return all prices in ascending order via inorder traversal."""
        result = []

        def inorder(node):
            if node is None:
                return
            inorder(node.left)
            result.append(node.price)
            inorder(node.right)

        inorder(self.root)
        return result

    def kth_smallest(self, k: int) -> float:
        """
        Return the k-th smallest price (1-indexed).
        Returns None if k is out of range.
        Uses inorder traversal with early termination.
        """
        # We use a list to hold mutable state across the nested function
        count = [0]
        result = [None]

        def inorder(node):
            if node is None or result[0] is not None:
                return

            inorder(node.left)

            count[0] += 1
            if count[0] == k:
                result[0] = node.price
                return

            inorder(node.right)

        inorder(self.root)
        return result[0]
```

### Complexity

| Operation | Time | Space |
|-----------|------|-------|
| `get_sorted_prices` | O(n) | O(n) output + O(h) stack |
| `kth_smallest` | O(k + h) with early termination | O(h) recursion stack |
| `kth_smallest` (augmented BST) | O(h) — not implemented but worth discussing | O(1) extra per node |

### Test Cases

```python
def test_stage_2():
    pt = PriceTracker()
    prices = [12.99, 8.50, 15.00, 9.75, 7.25, 13.50, 20.00]
    for p in prices:
        pt.insert(p)

    # Sorted prices
    sorted_prices = pt.get_sorted_prices()
    assert sorted_prices == [7.25, 8.50, 9.75, 12.99, 13.50, 15.00, 20.00]

    # kth smallest
    assert pt.kth_smallest(1) == 7.25
    assert pt.kth_smallest(3) == 9.75
    assert pt.kth_smallest(7) == 20.00
    assert pt.kth_smallest(4) == 12.99

    # Out of range
    assert pt.kth_smallest(0) is None
    assert pt.kth_smallest(8) is None
    assert pt.kth_smallest(-1) is None

    # With duplicates
    pt.insert(9.75)
    sorted_with_dup = pt.get_sorted_prices()
    assert sorted_with_dup == [7.25, 8.50, 9.75, 9.75, 12.99, 13.50, 15.00, 20.00]
    assert pt.kth_smallest(3) == 9.75
    assert pt.kth_smallest(4) == 9.75  # duplicate
    assert pt.kth_smallest(5) == 12.99

    # Empty tracker
    empty = PriceTracker()
    assert empty.get_sorted_prices() == []
    assert empty.kth_smallest(1) is None

    print("Stage 2: All tests passed.")

test_stage_2()
```

## Stage 3: Deletion and Balancing Discussion (~15 minutes)

### Prompt Addition

> Restaurants sometimes remove menu items. Implement `delete(price)` to remove one occurrence of a price from the BST. Then let's discuss: what happens to the BST's performance if prices are inserted in sorted order? How would you prevent that?

### Starter Code

```python
def delete(self, price: float) -> None:
    """Remove one occurrence of a price from the tracker."""
    pass

# --- Stage 3 Test Calls ---
# pt = PriceTracker()
# for p in [12.99, 8.50, 15.00, 9.75, 7.25, 13.50, 20.00]: pt.insert(p)
# pt.delete(7.25)
# Expected: [8.50, 9.75, 12.99, 13.50, 15.00, 20.00]
# print(pt.get_sorted_prices())
# pt.delete(15.00)
# Expected: [8.50, 9.75, 12.99, 13.50, 20.00]
# print(pt.get_sorted_prices())
```

### Expected Approach

**BST deletion has three cases:**

1. **Leaf node (no children):** Simply remove it — set the parent's pointer to `None`.
2. **One child:** Replace the node with its only child.
3. **Two children:** Find the inorder successor (smallest node in the right subtree), copy its value to the node being deleted, then recursively delete the inorder successor from the right subtree. (Alternatively, use the inorder predecessor from the left subtree.)

**Implementation:** Recursive approach is cleanest. The function returns the (possibly modified) subtree root, so the parent's pointer updates naturally.

**Balancing discussion (conceptual, no code required):**

- If prices arrive in sorted order (e.g., 1, 2, 3, 4, ...), the BST degenerates into a linked list. Height becomes O(n), so all operations degrade to O(n).
- **Self-balancing BSTs** fix this: AVL trees (strict balancing, height difference <= 1) and Red-Black trees (relaxed balancing, used in most standard libraries).
- **AVL trees** maintain balance via rotations on insert/delete. More strictly balanced, so lookups are faster, but inserts/deletes are slightly slower due to more rotations.
- **Red-Black trees** guarantee O(log n) height with fewer rotations on average. Used by Java's TreeMap, C++'s std::map.
- In Python, there's no built-in balanced BST. The `sortedcontainers` library provides `SortedList` (B-tree based). For interviews, acknowledging this trade-off is sufficient.

### Solution Code

```python
class PriceTracker:
    """Full implementation with delete."""

    def __init__(self):
        self.root = None

    def _insert_recursive(self, node, price):
        if node is None:
            return BSTNode(price)
        if price < node.price:
            node.left = self._insert_recursive(node.left, price)
        else:
            node.right = self._insert_recursive(node.right, price)
        return node

    def insert(self, price: float) -> None:
        self.root = self._insert_recursive(self.root, price)

    def search(self, price: float) -> bool:
        node = self.root
        while node:
            if price == node.price:
                return True
            elif price < node.price:
                node = node.left
            else:
                node = node.right
        return False

    def find_closest(self, target: float) -> float:
        if self.root is None:
            return None
        closest = self.root.price
        node = self.root
        while node:
            if abs(node.price - target) < abs(closest - target):
                closest = node.price
            if node.price == target:
                return node.price
            if target < node.price:
                node = node.left
            else:
                node = node.right
        return closest

    def get_sorted_prices(self) -> list:
        result = []
        def inorder(node):
            if node is None:
                return
            inorder(node.left)
            result.append(node.price)
            inorder(node.right)
        inorder(self.root)
        return result

    def kth_smallest(self, k: int) -> float:
        count = [0]
        result = [None]
        def inorder(node):
            if node is None or result[0] is not None:
                return
            inorder(node.left)
            count[0] += 1
            if count[0] == k:
                result[0] = node.price
                return
            inorder(node.right)
        inorder(self.root)
        return result[0]

    def _find_min(self, node: BSTNode) -> BSTNode:
        """Find the node with the minimum value in the subtree."""
        while node.left:
            node = node.left
        return node

    def _delete_recursive(self, node: BSTNode, price: float) -> BSTNode:
        """
        Delete one occurrence of price from the subtree rooted at node.
        Returns the (possibly new) root of the subtree.
        """
        if node is None:
            return None  # price not found

        if price < node.price:
            node.left = self._delete_recursive(node.left, price)
        elif price > node.price:
            node.right = self._delete_recursive(node.right, price)
        else:
            # Found the node to delete — handle three cases

            # Case 1: Leaf node (no children)
            if node.left is None and node.right is None:
                return None

            # Case 2: One child
            if node.left is None:
                return node.right
            if node.right is None:
                return node.left

            # Case 3: Two children
            # Find inorder successor (smallest in right subtree)
            successor = self._find_min(node.right)
            node.price = successor.price
            # Delete the successor from the right subtree
            node.right = self._delete_recursive(node.right, successor.price)

        return node

    def delete(self, price: float) -> None:
        """Remove one occurrence of a price from the tracker."""
        self.root = self._delete_recursive(self.root, price)
```

### Complexity

| Operation | Average Case | Worst Case |
|-----------|-------------|------------|
| `delete` | O(log n) | O(n) for skewed tree |
| Finding inorder successor | O(h) | O(n) |

**Balancing discussion reference points:**

| Tree Type | Insert | Delete | Search | Height Guarantee |
|-----------|--------|--------|--------|------------------|
| Unbalanced BST | O(n) worst | O(n) worst | O(n) worst | None |
| AVL Tree | O(log n) | O(log n) | O(log n) | h <= 1.44 log n |
| Red-Black Tree | O(log n) | O(log n) | O(log n) | h <= 2 log n |

### Test Cases

```python
def test_stage_3():
    pt = PriceTracker()
    for p in [12.99, 8.50, 15.00, 9.75, 7.25, 13.50, 20.00]:
        pt.insert(p)

    # Delete a leaf node (7.25)
    pt.delete(7.25)
    assert pt.get_sorted_prices() == [8.50, 9.75, 12.99, 13.50, 15.00, 20.00]
    assert pt.search(7.25) is False

    # Delete a node with one child (8.50 has right child 9.75 after 7.25 removed)
    pt.delete(8.50)
    assert pt.get_sorted_prices() == [9.75, 12.99, 13.50, 15.00, 20.00]

    # Delete a node with two children (15.00 has children 13.50 and 20.00)
    pt.delete(15.00)
    assert pt.get_sorted_prices() == [9.75, 12.99, 13.50, 20.00]

    # Delete root (12.99)
    pt.delete(12.99)
    assert pt.get_sorted_prices() == [9.75, 13.50, 20.00]

    # Delete non-existent price — no-op
    pt.delete(999.99)
    assert pt.get_sorted_prices() == [9.75, 13.50, 20.00]

    # Delete from empty tracker — no-op
    empty = PriceTracker()
    empty.delete(5.00)
    assert empty.get_sorted_prices() == []

    # Delete with duplicates — only removes one occurrence
    pt2 = PriceTracker()
    pt2.insert(10.00)
    pt2.insert(10.00)
    pt2.insert(10.00)
    assert pt2.get_sorted_prices() == [10.00, 10.00, 10.00]
    pt2.delete(10.00)
    assert pt2.get_sorted_prices() == [10.00, 10.00]

    # Delete until empty
    pt2.delete(10.00)
    pt2.delete(10.00)
    assert pt2.get_sorted_prices() == []

    # Verify operations still work after deletions
    pt2.insert(5.00)
    assert pt2.find_closest(6.00) == 5.00
    assert pt2.kth_smallest(1) == 5.00

    print("Stage 3: All tests passed.")

test_stage_3()
```

## Scoring Rubric

### Fundamentals (1-4)

| Score | Description |
|-------|-------------|
| **4** | Immediately proposes BST. Implements insert and closest-value search cleanly. Inorder traversal for sorted output is automatic. K-th smallest uses early termination. Deletion handles all three cases correctly. Discusses AVL/Red-Black trees with understanding of rotation concepts and real-world trade-offs. |
| **3** | Proposes BST with minor prompting. Insert and search are correct. Closest-value search works after thinking through the approach. K-th smallest works (even if O(n) approach). Deletion handles all three cases with minor guidance. Knows AVL/Red-Black exist and their purpose. |
| **2** | Needs hints to choose BST. Insert works but closest-value search has bugs. K-th smallest is brute-force (full sort then index). Deletion handles leaf and one-child cases but struggles with two-children case. Vague on balancing. |
| **1** | Cannot implement BST insert correctly. Fundamental confusion about BST property (left < root < right). |

### Coding (1-4)

| Score | Description |
|-------|-------------|
| **4** | All operations are correct, clean, and handle edge cases (empty tree, single node, duplicates). Recursive structure is elegant. No bugs across all three stages. |
| **3** | Stages 1 and 2 are solid. Stage 3 deletion works but may have a minor bug with the two-children case that's caught during testing. |
| **2** | Stage 1 works. Stage 2 kth_smallest has off-by-one errors. Stage 3 deletion is incomplete or buggy. |
| **1** | Insert has structural errors. Cannot produce working BST code. |

### Communication (1-4)

| Score | Description |
|-------|-------------|
| **4** | Explains BST property clearly. Draws/describes the tree at each step. Articulates why BST gives O(log n) average case. Deletion explanation covers all three cases with examples. Balancing discussion references real-world systems and trade-offs. |
| **3** | Explains approach before coding. Can trace through deletion cases when asked. Balancing discussion covers the key points. |
| **2** | Codes without much explanation. Answers questions about complexity but doesn't volunteer reasoning. Balancing discussion is surface-level. |
| **1** | Cannot explain BST property or why operations are O(h). Silent during coding. |

## Interviewer Guide

### Hints (per stage, progressive: mild, medium, strong)

**Stage 1 — Insert and Closest:**

| Level | Hint |
|-------|------|
| Mild | "What property of a BST helps you find the closest value without checking every node?" |
| Medium | "As you walk down the tree toward the target, every node you visit is a candidate for closest. Track the best candidate so far." |
| Strong | "At each node: update closest if `abs(node.price - target) < abs(closest - target)`. Then go left if `target < node.price`, else go right." |

**Stage 2 — K-th Smallest:**

| Level | Hint |
|-------|------|
| Mild | "What traversal of a BST gives you elements in sorted order?" |
| Medium | "Inorder traversal visits nodes in ascending order. Can you count as you go and stop early?" |
| Strong | "Do an inorder traversal with a counter. When the counter reaches k, that's your answer. Use early termination to avoid traversing the whole tree." |

**Stage 3 — Deletion:**

| Level | Hint |
|-------|------|
| Mild | "What are the possible cases for the node you're deleting? How many children could it have?" |
| Medium | "Three cases: no children (just remove), one child (replace with child), two children (find successor). Which case is hardest?" |
| Strong | "For two children: find the smallest node in the right subtree (go right once, then all the way left). Copy its value to the current node. Then delete that successor node from the right subtree — it has at most one child, so it's an easier case." |

### When to Intervene

- **If the candidate doesn't propose a BST:** Ask "What data structure maintains sorted order and supports efficient insertion?" If they suggest a sorted array, discuss why insertion is O(n) and guide them to BST.
- **If closest-value search is taking too long (>10 min):** Point out that they're already traversing toward the target during insert — closest search is the same traversal but tracking the best candidate.
- **If k-th smallest is done via full sort + index:** Accept it but ask "Can you do better than O(n)?" and guide toward early termination.
- **If deletion two-children case is taking too long (>8 min):** Walk through an example: "If you delete 15 from [12, 15(13, 20)], what value should replace 15?" This usually unlocks the insight about inorder successor.

### Common Mistakes

1. **Closest-value search: only checking the final leaf.** The closest value might be at any node along the path, not just the leaf where you "fall off" the tree.
2. **K-th smallest: off-by-one error.** Using 0-indexed counting when the problem says 1-indexed, or vice versa.
3. **Deletion — two children: infinite recursion.** If the candidate copies the successor value but forgets to delete the successor, or tries to delete the wrong value, recursion doesn't terminate.
4. **Deletion — not returning the modified subtree root.** The recursive approach must return `node` (or the replacement) so the parent updates its child pointer.
5. **Duplicates on delete:** Deleting a price should remove only one occurrence, but naive implementations may remove the wrong duplicate if they find the first match high in the tree while another exists lower.
6. **Forgetting to handle the empty tree** in `find_closest` (returning `None` rather than crashing).

### Edge Cases to Watch For

- Empty tree for all operations
- Single-node tree: insert, search, delete, find_closest, kth_smallest
- Deleting the root node (all three sub-cases)
- Duplicate prices: insert three copies, delete one, verify two remain
- Sorted insertion order (1, 2, 3, 4, ...) — tree degenerates to a linked list; discuss but don't require code fix
- `kth_smallest(k)` where k = 0, k < 0, or k > number of nodes
- `find_closest` when target is smaller than all prices or larger than all prices
- `find_closest` when target is exactly between two prices (e.g., target=10 with prices 9 and 11)
- Deleting a non-existent price (should be a no-op)
- Alternating inserts and deletes to verify tree integrity across operations
