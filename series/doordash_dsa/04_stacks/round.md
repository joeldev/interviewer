# Round 04: Stacks — "Balanced Route Expressions"

## Metadata
- **Topic:** Stacks
- **Difficulty:** 2/5
- **Estimated Time:** 55-60 minutes
- **Key Concepts:** Stack-based matching, index tracking on a stack, deficit counting

## Problem Prompt

> At DoorDash, routes between restaurants, dashers, and customers are encoded as
> *route expressions*. A route expression uses three kinds of nested groupings to
> represent route hierarchies:
>
> - `()` — direct route segments
> - `[]` — alternative route options
> - `{}` — grouped waypoint clusters
>
> Other characters (letters, digits, commas, etc.) may appear between brackets to
> label route segments.  For example:
>
> ```
> {main(A,B)[alt1,alt2]}
> ```
>
> Given a route expression as a string, determine whether the brackets are
> **properly balanced**.

*(Read the prompt exactly as written. Let the candidate ask about edge cases
before revealing the clarifying answers below.)*

## Clarifying Questions & Hidden Constraints

| # | What the candidate should ask | Answer |
|---|-------------------------------|--------|
| 1 | What should we return for an empty string `""`? | An empty string is considered balanced — return `True`. |
| 2 | What about strings that contain no brackets at all, like `"abc"`? | Those are balanced — non-bracket characters are simply ignored for validation. |
| 3 | Can the string contain characters other than brackets? | Yes. Letters, digits, commas, spaces, etc. may appear anywhere. Only the three bracket types matter. |
| 4 | Is there a maximum length? | Assume the string can be up to 10^6 characters. O(n) time is expected. |
| 5 | Should we handle Unicode or only ASCII? | ASCII only. |

## Skeleton Code

```python
def is_balanced(expression: str) -> bool:
    """
    Return True if every '(', '[', '{' in `expression` has a correctly
    matched and properly nested closing counterpart.
    Non-bracket characters should be ignored.
    """
    pass

if __name__ == "__main__":
    # Expected: True
    print(is_balanced("{main(A,B)[alt1,alt2]}"))
    # Expected: True
    print(is_balanced("()[]{}"))
    # Expected: False
    print(is_balanced("(]"))
    # Expected: False
    print(is_balanced("{[}]"))
    # Expected: True
    print(is_balanced(""))
```

---

## Stage 1: Validate Balanced Brackets (~20 minutes)

### Expected Approach

1. Initialize an empty stack.
2. Scan left to right. For every opening bracket, push it.
3. For every closing bracket, check that the stack is non-empty and the top
   matches the corresponding opener. If not, return `False`.
4. Skip all non-bracket characters.
5. After the scan, return `True` only if the stack is empty.

### Solution Code

```python
def is_balanced(expression: str) -> bool:
    """
    Return True if every '(', '[', '{' in `expression` has a correctly
    matched and properly nested closing counterpart.
    Non-bracket characters should be ignored.
    """
    stack: list[str] = []
    matching = {')': '(', ']': '[', '}': '{'}
    openers = set(matching.values())
    closers = set(matching.keys())

    for ch in expression:
        if ch in openers:
            stack.append(ch)
        elif ch in closers:
            if not stack or stack[-1] != matching[ch]:
                return False
            stack.pop()
        # Non-bracket characters are simply ignored.

    return len(stack) == 0
```

### Complexity

| | |
|---|---|
| **Time** | O(n) — single pass through the string. |
| **Space** | O(n) — in the worst case every character is an opener. |

### Test Cases

```python
# Basic valid expressions
assert is_balanced("") == True
assert is_balanced("abc") == True
assert is_balanced("{main(A,B)[alt1,alt2]}") == True
assert is_balanced("()[]{}") == True
assert is_balanced("{[()]}") == True

# Invalid expressions
assert is_balanced("(]") == False
assert is_balanced("{[}]") == False
assert is_balanced("((())") == False
assert is_balanced("route(A,B]") == False

# Edge: only closers
assert is_balanced("}") == False
```

---

## Stage 2: Longest Balanced Substring (~20 minutes)

### Prompt Addition

> Not all route expressions stored in the database are valid — some were
> corrupted. Given a potentially unbalanced expression, find the **length of the
> longest contiguous substring** that is itself balanced.  Non-bracket characters
> between brackets count toward the length.
>
> Example: `"(route[A]extra)bad][ok()]"` — the longest balanced substring is
> `"(route[A]extra)"` with length 15.

### Starter Code

```python
def longest_balanced_substring(expression: str) -> int:
    """
    Return the length of the longest contiguous substring of `expression`
    that is itself a balanced bracket expression (non-bracket characters
    count toward the length).
    """
    pass

# --- Stage 2 Test Calls ---
# Expected: 6
print(longest_balanced_substring("{[()]}"))
# Expected: 15
print(longest_balanced_substring("(route[A]extra)bad][ok()]"))
# Expected: 0
print(longest_balanced_substring("(((("))
```

### Expected Approach

Use a **stack of indices** rather than characters:

1. Push `-1` onto the stack as a sentinel (the index just before the start of
   a valid sequence).
2. For each character at index `i`:
   - If it is an opening bracket, push `i`.
   - If it is a closing bracket:
     - If the stack top holds an index of the matching opener, pop it and
       update `max_len = max(max_len, i - stack[-1])`.
     - Otherwise push `i` (this closer becomes a new boundary).
   - If it is a non-bracket character, do nothing — it simply extends
     whatever valid region surrounds it.
3. Return `max_len`.

The key insight is that non-bracket characters naturally contribute to the
length because we measure length as the distance between indices rather than
counting brackets.

### Solution Code

```python
def longest_balanced_substring(expression: str) -> int:
    """
    Return the length of the longest contiguous substring of `expression`
    that is itself a balanced bracket expression (non-bracket characters
    count toward the length).
    """
    stack: list[int] = [-1]  # sentinel
    matching = {')': '(', ']': '[', '}': '{'}
    openers = set(matching.values())
    closers = set(matching.keys())
    max_len = 0

    for i, ch in enumerate(expression):
        if ch in openers:
            stack.append(i)
        elif ch in closers:
            # Check if top of stack is the matching opener.
            if (
                len(stack) > 1
                and stack[-1] != -1
                and expression[stack[-1]] == matching[ch]
            ):
                stack.pop()
                max_len = max(max_len, i - stack[-1])
            else:
                # Unmatched closer becomes a new boundary.
                stack.append(i)
        # Non-bracket characters: do nothing.

    return max_len
```

### Complexity

| | |
|---|---|
| **Time** | O(n) — single pass. |
| **Space** | O(n) — stack may hold up to n indices. |

### Test Cases

```python
# Entirely valid expression
assert longest_balanced_substring("{[()]}") == 6

# Longest valid portion in the middle
assert longest_balanced_substring("}{[()]}{{") == 6

# Non-bracket characters extend the length
assert longest_balanced_substring("(route[A]extra)bad][ok()]") == 15

# No valid portion
assert longest_balanced_substring("((((") == 0
assert longest_balanced_substring("") == 0

# Multiple valid portions — return the longest
assert longest_balanced_substring("()][()()]") == 6

# Only non-bracket characters (no brackets means nothing to match)
assert longest_balanced_substring("abcdef") == 0

# Mixed with non-bracket chars inside valid brackets
assert longest_balanced_substring("x(a[b]c)y") == 8  # "(a[b]c)" length 8
```

---

## Stage 3: Minimum Insertions to Balance (~15 minutes)

### Prompt Addition

> The engineering team wants to auto-repair corrupted route expressions by
> inserting the fewest possible brackets.  Given an expression, return the
> **minimum number of single-bracket insertions** required to make it balanced.
>
> Example: `"([)]"` needs 2 insertions (e.g., `"([]())"` — insert `]` after the
> first `[` and `(` before the last `)`) — but note that we only count
> insertions, so the answer is 2.
>
> Wait — actually `"([)]"` is interleaved; fixing it by insertion alone requires
> adding a `]` before `)` and a `(` before `)` to isolate each pair. The minimum
> is 2 insertions.

### Starter Code

```python
def min_insertions_to_balance(expression: str) -> int:
    """
    Return the minimum number of single-bracket insertions needed to make
    `expression` balanced.
    """
    pass

# --- Stage 3 Test Calls ---
# Expected: 0
print(min_insertions_to_balance("()[]{}"))
# Expected: 3
print(min_insertions_to_balance("((("))
# Expected: 2
print(min_insertions_to_balance("([)]"))
```

*(This stage is intentionally subtle. The candidate must realize that mismatched
closers each need exactly one insertion of the corresponding opener.)*

### Expected Approach

1. Maintain a stack of *unmatched openers*.
2. Maintain a counter `insertions` for the number of closers that had no
   matching opener (each such closer requires inserting its opener = 1
   insertion).
3. Scan left to right:
   - Opening bracket: push onto stack.
   - Closing bracket: if the stack top is the matching opener, pop.
     Otherwise, increment `insertions` by 1 (we must insert the matching
     opener).
4. After the scan, each opener remaining on the stack needs its closer
   inserted: add `len(stack)` to `insertions`.
5. Return `insertions`.

### Solution Code

```python
def min_insertions_to_balance(expression: str) -> int:
    """
    Return the minimum number of single-bracket insertions needed to make
    `expression` balanced.
    """
    stack: list[str] = []  # unmatched openers
    matching = {')': '(', ']': '[', '}': '{'}
    openers = set(matching.values())
    closers = set(matching.keys())
    insertions = 0

    for ch in expression:
        if ch in openers:
            stack.append(ch)
        elif ch in closers:
            if stack and stack[-1] == matching[ch]:
                stack.pop()  # matched — no insertion needed
            else:
                # Unmatched closer: we must insert its matching opener.
                insertions += 1
        # Non-bracket characters: ignore.

    # Every remaining opener on the stack needs a closer inserted.
    insertions += len(stack)
    return insertions
```

### Complexity

| | |
|---|---|
| **Time** | O(n) — single pass. |
| **Space** | O(n) — stack for unmatched openers. |

### Test Cases

```python
# Already balanced
assert min_insertions_to_balance("()[]{}") == 0
assert min_insertions_to_balance("") == 0
assert min_insertions_to_balance("abc") == 0

# Simple unmatched openers
assert min_insertions_to_balance("(((") == 3

# Simple unmatched closers
assert min_insertions_to_balance(")))") == 3

# Mixed unmatched
assert min_insertions_to_balance("([)]") == 2

# Complex
assert min_insertions_to_balance("{route(A]B)end}[") == 3
# Explanation: ']' has no matching '[' -> +1
#              ')' matches '(' -> ok
#              '[' at end has no ']' -> +1
#              '{' matches '}' -> ok
#              'A' unmatched closer ']' -> already counted
#              Wait — let's trace:
#   {  -> stack: ['{']
#   r,o,u,t,e -> ignore
#   (  -> stack: ['{', '(']
#   A  -> ignore
#   ]  -> top is '(' != '[', insertions=1
#   B  -> ignore
#   )  -> top is '(' == '(', pop -> stack: ['{']
#   e,n,d -> ignore
#   }  -> top is '{' == '{', pop -> stack: []
#   [  -> stack: ['[']
#   End: stack has 1 item -> insertions = 1 + 1 = 2
assert min_insertions_to_balance("{route(A]B)end}[") == 2

# Only closers of different types
assert min_insertions_to_balance("}])") == 3
```

---

## Scoring Rubric

### Fundamentals (1-4)

| Score | Description |
|-------|-------------|
| 1 | Cannot articulate why a stack is useful; no understanding of LIFO matching. |
| 2 | Understands the concept but struggles to implement it without significant help. |
| 3 | Solves Stage 1 cleanly; makes progress on Stage 2 with minor hints. |
| 4 | Solves all three stages with correct complexity analysis and edge case handling. |

### Coding (1-4)

| Score | Description |
|-------|-------------|
| 1 | Code does not run; major syntax or logic errors. |
| 2 | Stage 1 works but Stage 2/3 have bugs that the candidate cannot debug. |
| 3 | Stage 1 and 2 work; Stage 3 is mostly correct with minor issues. |
| 4 | All three stages produce correct, clean, idiomatic Python. Good variable names, helper structures (e.g., `matching` dict). |

### Communication (1-4)

| Score | Description |
|-------|-------------|
| 1 | Codes silently; cannot explain approach when asked. |
| 2 | Explains only after prompting; explanations are unclear. |
| 3 | Talks through approach before coding; explains trade-offs when asked. |
| 4 | Proactively discusses approach, complexity, and edge cases. Asks clarifying questions before starting. Clearly narrates thought process throughout. |

---

## Interviewer Guide

### Hints (per stage, progressive: mild -> medium -> strong)

**Stage 1 — Balanced Check**

- **Mild:** "What data structure lets you match the most recent unmatched opener with the current closer?"
- **Medium:** "Think LIFO — what if you pushed every opener and popped when you see a closer?"
- **Strong:** "Use a stack. Push openers, pop on a matching closer. If the closer doesn't match the top, it's invalid. At the end, the stack must be empty."

**Stage 2 — Longest Balanced Substring**

- **Mild:** "Instead of pushing characters, what if you pushed *indices* onto the stack?"
- **Medium:** "You need a way to measure the length of the current valid run. If you keep a 'boundary' index on the stack, you can compute `i - stack[-1]` after a successful pop."
- **Strong:** "Start with `-1` on the stack as a sentinel. Push the index of every opener. When a closer matches, pop and compute `i - stack[-1]`. When it doesn't match, push its index as the new boundary."

**Stage 3 — Minimum Insertions**

- **Mild:** "Think about what happens to unmatched closers separately from unmatched openers."
- **Medium:** "Each unmatched closer needs exactly one opener inserted. Each remaining opener on the stack needs exactly one closer inserted."
- **Strong:** "Scan with a stack. For closers that don't match the top, count +1. At the end, add the stack size. That's your answer."

### When to Intervene

- If the candidate is stuck on Stage 1 for more than 10 minutes, give the medium hint and let them code.
- If Stage 2 has stalled for 10 minutes, explicitly suggest the index-on-stack technique so they have time for Stage 3.
- If the candidate finishes Stage 1 in under 10 minutes, move immediately to Stage 2 to leave more time for Stage 3.

### Common Mistakes

1. **Forgetting to ignore non-bracket characters.** Some candidates try to push every character.
2. **Not handling the empty-stack case on a closer.** Popping from an empty stack crashes; always check before popping.
3. **Stage 2: Forgetting the sentinel `-1`.** Without it the first valid match can't compute its length.
4. **Stage 2: Treating non-bracket characters as boundaries.** They should be transparent — only brackets create or resolve boundaries.
5. **Stage 3: Trying to fix interleaving by rearranging instead of inserting.** The problem says *insertions only*.

### Edge Cases to Watch For

- Empty string `""` — balanced, longest = 0, insertions = 0.
- String with no brackets `"abcdef"` — balanced, longest = 0, insertions = 0.
- All openers `"((([[[{{{" ` — not balanced, longest = 0, insertions = 9.
- All closers `")))]]]}}}"` — not balanced, longest = 0, insertions = 9.
- Single character `"("` — not balanced, longest = 0, insertions = 1.
- Alternating matched pairs `"()[]{}()[]{}..."` — balanced, longest = full length, insertions = 0.
- Deeply nested `"{[({[({[(])]})]})]})"` — candidate must trace carefully.
