# Round 08: Binary Trees & Tree Traversals — "Menu Category Hierarchy"

## Metadata
- **Topic:** Binary Trees, Tree Traversals (Inorder, Level-Order, Preorder), Lowest Common Ancestor, Serialization
- **Difficulty:** 3/5
- **Estimated Time:** 55-60 minutes
- **Key Concepts:** Recursive tree traversal, BFS with queues, LCA via recursion, serialization/deserialization with null markers

## Problem Prompt

> At DoorDash, a restaurant's menu is organized as a tree. Internal nodes represent categories (like "Appetizers", "Entrees", "Drinks"), and leaf nodes represent individual menu items (like "Spring Rolls", "Pad Thai", "Iced Tea"). The tree is binary — each category has at most two subcategories or items.
>
> Given the root of this menu tree, implement operations to explore and manipulate the hierarchy.
>
> Start by writing a function that returns all the menu items (leaves) of the tree.

*Read this prompt exactly. The candidate should ask about the tree structure, what constitutes a leaf, and whether the tree could be empty.*

## Clarifying Questions & Hidden Constraints

| # | Question Candidate Should Ask | Answer |
|---|-------------------------------|--------|
| 1 | What does a node look like? | Each node has a `val` (string — category name or item name), `left`, and `right`. |
| 2 | How do I distinguish a category from an item? | A leaf node (both children are `None`) is a menu item. An internal node is a category. |
| 3 | Can the tree be empty (root is None)? | Yes. Return an empty list in that case. |
| 4 | Can a category have zero items under it (internal node with no leaves in subtree)? | No — every internal node has at least one child. But a subtree could have only one child (the other is None). |
| 5 | Is the tree balanced? | Not necessarily. It could be heavily skewed. |
| 6 | Are item names unique? | Yes — each menu item name appears exactly once in the tree. |

## Skeleton Code

```python
class TreeNode:
    def __init__(self, val: str = "", left=None, right=None):
        self.val = val
        self.left = left
        self.right = right


def get_menu_items(root: TreeNode) -> list:
    """Return all menu items (leaf nodes) via inorder traversal."""
    pass


def items_by_level(root: TreeNode) -> list:
    """Return menu items grouped by their level in the tree (level-order)."""
    pass

if __name__ == "__main__":
    # Build a small tree:  A
    #                     / \
    #                    B   C
    root = TreeNode("A", TreeNode("B"), TreeNode("C"))
    # Expected: ['B', 'C'] (leaves via inorder)
    print(get_menu_items(root))
    # Expected: [['A'], ['B', 'C']] (level-order grouping)
    print(items_by_level(root))
    # Expected: [] (empty tree)
    print(get_menu_items(None))
```

---

## Stage 1: Traversals (~20 minutes)

### Expected Approach

**Part A — Inorder traversal to collect leaves:**

The candidate should implement a standard inorder traversal (left, root, right) but only collect leaf nodes (nodes where both `left` and `right` are `None`).

- Recursive approach: traverse left subtree, check if current node is a leaf and if so append it, traverse right subtree.
- Iterative approach with an explicit stack is also acceptable.

**Part B — Level-order grouping (BFS):**

Use a queue (collections.deque) to perform BFS. For each level, collect all node values into a sublist. This tests whether the candidate can do BFS on a tree and track level boundaries.

Key decision: the prompt says "items grouped by level." A strong candidate will ask whether to include category names or only leaf items. Answer: include all nodes (both categories and items) grouped by level — this tests BFS, not leaf filtering.

### Solution Code

```python
from collections import deque


class TreeNode:
    def __init__(self, val: str = "", left=None, right=None):
        self.val = val
        self.left = left
        self.right = right


def get_menu_items(root: TreeNode) -> list:
    """Return all menu items (leaf nodes) via inorder traversal."""
    result = []

    def inorder(node):
        if node is None:
            return
        inorder(node.left)
        if node.left is None and node.right is None:
            result.append(node.val)
        inorder(node.right)

    inorder(root)
    return result


def items_by_level(root: TreeNode) -> list:
    """
    Return all node values grouped by level (BFS).
    Each sublist contains the values at that depth.
    """
    if root is None:
        return []

    result = []
    queue = deque([root])

    while queue:
        level_size = len(queue)
        level_values = []
        for _ in range(level_size):
            node = queue.popleft()
            level_values.append(node.val)
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
        result.append(level_values)

    return result
```

### Complexity

| Function | Time | Space |
|----------|------|-------|
| `get_menu_items` | O(n) — visits every node | O(h) recursion stack, where h = tree height |
| `items_by_level` | O(n) — visits every node | O(w) queue width, where w = max level width; O(n) for output |

### Test Cases

```python
def build_sample_tree():
    """
    Build this tree:
              Menu
             /    \
        Food       Drinks
        /  \        /   \
    Apps  Entrees  Hot  Cold
    / \     |       |    |
    SR  ED  PT     Tea  Soda

    SR = Spring Rolls, ED = Edamame, PT = Pad Thai
    """
    sr = TreeNode("Spring Rolls")
    ed = TreeNode("Edamame")
    pt = TreeNode("Pad Thai")
    tea = TreeNode("Tea")
    soda = TreeNode("Soda")

    apps = TreeNode("Apps", sr, ed)
    entrees = TreeNode("Entrees", pt, None)
    hot = TreeNode("Hot", tea, None)
    cold = TreeNode("Cold", soda, None)

    food = TreeNode("Food", apps, entrees)
    drinks = TreeNode("Drinks", hot, cold)

    root = TreeNode("Menu", food, drinks)
    return root


def test_stage_1():
    root = build_sample_tree()

    # Inorder leaf collection
    items = get_menu_items(root)
    assert items == ["Spring Rolls", "Edamame", "Pad Thai", "Tea", "Soda"], f"Got: {items}"

    # Level-order grouping
    levels = items_by_level(root)
    assert levels == [
        ["Menu"],
        ["Food", "Drinks"],
        ["Apps", "Entrees", "Hot", "Cold"],
        ["Spring Rolls", "Edamame", "Pad Thai", "Tea", "Soda"],
    ], f"Got: {levels}"

    # Empty tree
    assert get_menu_items(None) == []
    assert items_by_level(None) == []

    # Single node (leaf only — it's both the root and a menu item)
    single = TreeNode("Solo Item")
    assert get_menu_items(single) == ["Solo Item"]
    assert items_by_level(single) == [["Solo Item"]]

    print("Stage 1: All tests passed.")

test_stage_1()
```

## Stage 2: Lowest Common Ancestor (~20 minutes)

### Prompt Addition

> Now suppose a customer selects two menu items and we want to find the most specific category that contains both of them. This is the "lowest common ancestor" (LCA) of the two item nodes.
>
> Implement `find_lca(root, item1, item2)` that returns the LCA node's value. You can assume both items exist in the tree.

### Starter Code

```python
def find_lca(root: TreeNode, item1: str, item2: str) -> str:
    """
    Find the lowest common ancestor of two menu items.
    Returns the LCA node's value. Both items are guaranteed to exist.
    """
    pass

# --- Stage 2 Test Calls ---
# Using the sample tree from Stage 1's test section:
# root = build_sample_tree()
# Expected: Apps
# print(find_lca(root, "Spring Rolls", "Edamame"))
# Expected: Menu
# print(find_lca(root, "Spring Rolls", "Tea"))
# Expected: Food
# print(find_lca(root, "Spring Rolls", "Pad Thai"))
```

### Expected Approach

Classic binary tree LCA algorithm:

1. Base case: if the current node is `None`, return `None`. If the current node's value matches either item, return this node.
2. Recurse on left and right subtrees.
3. If both subtrees return non-None, the current node is the LCA.
4. If only one subtree returns non-None, propagate that result upward.

This works because both items are guaranteed to exist. The LCA is the deepest node that has one target in its left subtree and the other in its right subtree (or is itself one of the targets).

**Important:** This is a general binary tree, NOT a BST. The candidate cannot use value comparisons to decide left/right. They must search both subtrees.

### Solution Code

```python
def find_lca(root: TreeNode, item1: str, item2: str) -> str:
    """
    Find the lowest common ancestor of two menu items.
    Returns the LCA node's value. Both items are guaranteed to exist.
    """

    def helper(node):
        if node is None:
            return None
        if node.val == item1 or node.val == item2:
            return node

        left = helper(node.left)
        right = helper(node.right)

        if left and right:
            # Current node is the LCA — one target in each subtree
            return node
        return left if left else right

    lca_node = helper(root)
    return lca_node.val if lca_node else None
```

### Complexity

| Metric | Value |
|--------|-------|
| Time | O(n) — may visit every node |
| Space | O(h) — recursion stack depth, where h = tree height |

### Test Cases

```python
def test_stage_2():
    root = build_sample_tree()

    # Two items under the same parent category
    assert find_lca(root, "Spring Rolls", "Edamame") == "Apps"

    # Items in different major branches
    assert find_lca(root, "Spring Rolls", "Tea") == "Menu"

    # Items in the same branch but different depths
    assert find_lca(root, "Spring Rolls", "Pad Thai") == "Food"

    # One item is deeper, the other is in a sibling subtree
    assert find_lca(root, "Pad Thai", "Edamame") == "Food"

    # Both items in the Drinks subtree
    assert find_lca(root, "Tea", "Soda") == "Drinks"

    # Item and its immediate parent's sibling's child
    assert find_lca(root, "Edamame", "Soda") == "Menu"

    # Edge case: item1 == item2 (same item)
    assert find_lca(root, "Tea", "Tea") == "Tea"

    print("Stage 2: All tests passed.")

test_stage_2()
```

## Stage 3: Serialize and Deserialize (~15 minutes)

### Prompt Addition

> DoorDash needs to store and transmit menu trees efficiently. Implement two functions:
>
> - `serialize(root)` — converts the tree to a string
> - `deserialize(data)` — reconstructs the tree from that string
>
> The round-trip must be lossless: `deserialize(serialize(tree))` must produce an identical tree.

### Starter Code

```python
def serialize(root: TreeNode) -> str:
    """Serialize a binary tree to a comma-separated string using preorder traversal."""
    pass


def deserialize(data: str) -> TreeNode:
    """Deserialize a comma-separated string back into a binary tree."""
    pass

# --- Stage 3 Test Calls ---
# Expected: "Burger,#,#"
# print(serialize(TreeNode("Burger")))
# Expected: "#"
# print(serialize(None))
# Expected: None
# print(deserialize("#"))
```

### Expected Approach

**Preorder traversal with null markers** is the classic approach:

- **Serialize:** Perform a preorder traversal. For each node, append its value. For each `None` child, append a sentinel (e.g., `"#"`). Use a delimiter (e.g., `","`) between values.
- **Deserialize:** Split the string by the delimiter. Use an iterator/index to consume tokens one by one. For each token, if it's the sentinel, return `None`; otherwise, create a node and recursively build its left and right children.

**Key considerations:**
- The delimiter must not appear in node values. If values can contain commas, use a different delimiter or escape it. For this problem, assume values don't contain commas.
- The null marker must not be a valid node value. `"#"` is a safe choice for menu names.
- Empty tree (root is None): serialize to `"#"`, deserialize back to `None`.

### Solution Code

```python
def serialize(root: TreeNode) -> str:
    """Serialize a binary tree to a comma-separated string using preorder traversal."""
    tokens = []

    def preorder(node):
        if node is None:
            tokens.append("#")
            return
        tokens.append(node.val)
        preorder(node.left)
        preorder(node.right)

    preorder(root)
    return ",".join(tokens)


def deserialize(data: str) -> TreeNode:
    """Deserialize a comma-separated string back into a binary tree."""
    tokens = iter(data.split(","))

    def build():
        val = next(tokens)
        if val == "#":
            return None
        node = TreeNode(val)
        node.left = build()
        node.right = build()
        return node

    return build()
```

### Complexity

| Metric | Value |
|--------|-------|
| Time (serialize) | O(n) — visits every node once |
| Time (deserialize) | O(n) — processes every token once |
| Space (serialize) | O(n) for the token list + O(h) recursion stack |
| Space (deserialize) | O(n) for the constructed tree + O(h) recursion stack |

### Test Cases

```python
def trees_equal(t1: TreeNode, t2: TreeNode) -> bool:
    """Helper: check if two trees are structurally identical with same values."""
    if t1 is None and t2 is None:
        return True
    if t1 is None or t2 is None:
        return False
    return (t1.val == t2.val
            and trees_equal(t1.left, t2.left)
            and trees_equal(t1.right, t2.right))


def test_stage_3():
    root = build_sample_tree()

    # Round-trip: serialize then deserialize
    data = serialize(root)
    restored = deserialize(data)
    assert trees_equal(root, restored), "Round-trip failed for sample tree"

    # Verify serialized form is a readable string
    # (Just check it's non-empty and contains expected names)
    assert "Menu" in data
    assert "Spring Rolls" in data
    assert "#" in data  # null markers present

    # Empty tree
    assert serialize(None) == "#"
    assert deserialize("#") is None

    # Single node
    single = TreeNode("Burger")
    data_single = serialize(single)
    assert data_single == "Burger,#,#"
    restored_single = deserialize(data_single)
    assert restored_single.val == "Burger"
    assert restored_single.left is None
    assert restored_single.right is None

    # Skewed tree (left-only chain)
    chain = TreeNode("A", TreeNode("B", TreeNode("C")))
    data_chain = serialize(chain)
    restored_chain = deserialize(data_chain)
    assert trees_equal(chain, restored_chain), "Round-trip failed for left-skewed tree"

    # Right-only chain
    right_chain = TreeNode("X", None, TreeNode("Y", None, TreeNode("Z")))
    data_rc = serialize(right_chain)
    restored_rc = deserialize(data_rc)
    assert trees_equal(right_chain, restored_rc), "Round-trip failed for right-skewed tree"

    print("Stage 3: All tests passed.")

test_stage_3()
```

## Scoring Rubric

### Fundamentals (1-4)

| Score | Description |
|-------|-------------|
| **4** | Fluent with all traversal types. Immediately identifies the correct traversal for each sub-problem. LCA algorithm is recalled or re-derived quickly. Serialization approach (preorder + null markers) is proposed without hints. Complexity analysis is precise. |
| **3** | Solid on inorder and BFS. LCA requires a moment of thought but is correctly implemented. Serialization works with minor guidance. |
| **2** | Can implement inorder but struggles with BFS level grouping (e.g., doesn't track level boundaries). LCA requires significant hints. Serialization approach is unclear. |
| **1** | Cannot implement a basic tree traversal. Confused about recursion or tree structure fundamentals. |

### Coding (1-4)

| Score | Description |
|-------|-------------|
| **4** | All three stages produce clean, correct, runnable code on first attempt. Handles None root, single node, skewed trees. Uses appropriate Python idioms (deque, iterators). |
| **3** | Stage 1 and 2 are correct. Stage 3 has a minor bug (e.g., delimiter issue) but is quickly fixed. |
| **2** | Stage 1 works but Stage 2 has logical errors (e.g., LCA returns wrong node for certain configurations). Stage 3 is incomplete. |
| **1** | Stage 1 has fundamental bugs. Cannot produce working tree code even with guidance. |

### Communication (1-4)

| Score | Description |
|-------|-------------|
| **4** | Draws or describes the tree structure before coding. Clearly explains recursive reasoning ("I check left, check right, if both are non-None then this node is the LCA because..."). Discusses serialization format trade-offs. |
| **3** | Explains approach before coding for each stage. Walks through at least one test case manually. |
| **2** | Jumps into code without explaining. Can explain when asked but doesn't volunteer reasoning. |
| **1** | Cannot explain recursive logic even when asked. Unclear about what their code does. |

## Interviewer Guide

### Hints (per stage, progressive: mild, medium, strong)

**Stage 1 — Inorder:**

| Level | Hint |
|-------|------|
| Mild | "What defines a menu item vs. a category in this tree?" |
| Medium | "For inorder: go left, process current, go right. When do you add to the result?" |
| Strong | "Check `if node.left is None and node.right is None` — that's a leaf. Only append leaves." |

**Stage 1 — BFS:**

| Level | Hint |
|-------|------|
| Mild | "How would you visit nodes level by level? What data structure processes things in FIFO order?" |
| Medium | "Use a queue. At each step, note how many nodes are in the queue — that's the current level's size." |
| Strong | "Before the inner loop: `level_size = len(queue)`. Loop `level_size` times, popping from the front and adding children." |

**Stage 2 — LCA:**

| Level | Hint |
|-------|------|
| Mild | "If you search for both items in the left and right subtrees, what does it mean if you find one in each?" |
| Medium | "Base cases: None returns None. A node matching either target returns itself. Then recurse left and right. If both return non-None, you've found the LCA." |
| Strong | "Write the base case: `if node is None: return None` and `if node.val == item1 or node.val == item2: return node`. Then `left = helper(node.left)`, `right = helper(node.right)`. If both are non-None, return `node`. Otherwise return whichever is non-None." |

**Stage 3 — Serialization:**

| Level | Hint |
|-------|------|
| Mild | "What traversal order naturally supports reconstruction? Think about which traversals uniquely define a tree." |
| Medium | "Preorder traversal with explicit null markers gives you enough information. Use a sentinel like '#' for None nodes." |
| Strong | "Serialize: preorder, appending '#' for None. Deserialize: split by comma, use an iterator. For each token: if '#', return None. Otherwise create a node, recurse for left child, recurse for right child." |

### When to Intervene

- **If the candidate confuses binary tree with BST in Stage 2:** Clarify that nodes are not ordered by any key — you must search both subtrees. This is a critical conceptual point.
- **If BFS level grouping is taking too long (>10 min):** Show the `level_size = len(queue)` pattern and let them fill in the rest.
- **If LCA recursion is correct but they can't convince themselves it works:** Walk through the sample tree with two specific items, tracing the recursion.
- **If Stage 3 is taking too long:** Give the serialized format (`"Menu,Food,Apps,Spring Rolls,#,#,Edamame,#,#,..."`) and ask them to write just the deserializer.

### Common Mistakes

1. **BFS without level tracking.** Putting all nodes in a flat list instead of grouping by level. They process the queue but forget to snapshot the queue size at each level.
2. **LCA: treating it as a BST.** Trying to use value comparisons to decide whether to go left or right. This is not a BST — values are strings with no ordering.
3. **LCA: not handling the case where one item is an ancestor of the other.** The algorithm above handles this correctly (the ancestor returns itself), but candidates who roll their own logic may miss it.
4. **Serialization: forgetting null markers.** Without explicit null markers, the tree structure is ambiguous. E.g., preorder `[A, B, C]` could be multiple tree shapes.
5. **Deserialization: using an index variable incorrectly across recursive calls.** Using a list `[0]` or `iter()` is cleaner than trying to pass and return an integer index.
6. **Not handling the empty tree case** in any of the three stages.

### Edge Cases to Watch For

- Empty tree (`root is None`) for all functions
- Single-node tree (it's both the root and a leaf)
- Heavily left-skewed or right-skewed tree (recursion depth)
- LCA where one target is an ancestor of the other
- LCA where both targets are the same node
- Serialization of a tree where a node has only a left child or only a right child
- Node values that contain spaces (e.g., "Spring Rolls") — delimiter choice matters
