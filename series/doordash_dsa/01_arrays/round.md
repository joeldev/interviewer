# Round 01: Arrays — "Delivery Time Windows"

## Metadata
- **Topic:** Arrays, Sorting, Interval Problems
- **Difficulty:** 1/5
- **Estimated Time:** 55-60 minutes
- **Key Concepts:** Sorting, interval overlap detection, merging intervals, greedy removal

## Problem Prompt

> You're building a scheduling system for DoorDash dashers. Each delivery has a time window represented as a pair of numbers `[start, end]` indicating when the dasher must be actively delivering that order.
>
> Given a list of delivery time windows, determine whether a single dasher can complete all of the deliveries.

*(Read exactly this. Do not volunteer information about sorting, inclusivity of endpoints, or what "complete all deliveries" means precisely. Let the candidate ask.)*

## Clarifying Questions & Hidden Constraints

| # | Question the Candidate Should Ask | Answer to Give |
|---|---|---|
| 1 | Are the time windows sorted in any order? | No, they can come in any order. |
| 2 | Are the start and end times integers or can they be floats? | Assume integers for simplicity, but your solution should work for floats too. |
| 3 | Is the end time inclusive or exclusive? In other words, if one delivery ends at time 5 and another starts at time 5, do they overlap? | Good question. A dasher finishing one delivery at time 5 can start the next at time 5 — so `[1, 5]` and `[5, 9]` do NOT overlap. End is exclusive for overlap purposes. |
| 4 | Can a time window have start == end? | Yes, treat it as an instantaneous delivery. It does not conflict with a window that starts or ends at that same time. |
| 5 | Can the list be empty? | Yes. An empty schedule has no conflicts. |
| 6 | Can start be greater than end? | No, assume all inputs satisfy `start <= end`. |

## Skeleton Code

```python
from typing import List, Tuple

# A delivery window is a tuple (start_time, end_time).
# Example: (2, 5) means the dasher is busy from time 2 up to (but not including) time 5.

def can_complete_all(windows: List[Tuple[int, int]]) -> bool:
    """
    Stage 1: Return True if a single dasher can complete all deliveries
    without any time overlaps, False otherwise.
    """
    pass

if __name__ == "__main__":
    # Expected: True (no overlaps — sequential deliveries)
    print(can_complete_all([(1, 3), (3, 5), (5, 8)]))
    # Expected: False (overlap between (1,4) and (2,6))
    print(can_complete_all([(1, 4), (2, 6), (8, 10)]))
    # Expected: True (empty schedule)
    print(can_complete_all([]))
```

---

## Stage 1: Overlap Detection (~20 minutes)

### Expected Approach

1. Sort the windows by start time (break ties by end time).
2. Iterate through consecutive pairs. If the current window's start is strictly less than the previous window's end, there is an overlap.
3. Return `False` on the first overlap found; `True` if the entire scan completes cleanly.

Key insight: sorting brings potentially overlapping windows adjacent to each other, reducing an O(n^2) brute-force check to O(n log n).

### Solution Code

```python
def can_complete_all(windows: List[Tuple[int, int]]) -> bool:
    if len(windows) <= 1:
        return True

    sorted_windows = sorted(windows)  # sorts by start, then end

    for i in range(1, len(sorted_windows)):
        prev_end = sorted_windows[i - 1][1]
        curr_start = sorted_windows[i][0]
        if curr_start < prev_end:  # strict < because touching endpoints are OK
            return False

    return True
```

### Complexity
- **Time:** O(n log n) for sorting + O(n) scan = O(n log n)
- **Space:** O(n) for the sorted copy (or O(1) extra if sorting in place)

### Test Cases

```python
# Test 1: No overlap — sequential deliveries
assert can_complete_all([(1, 3), (3, 5), (5, 8)]) == True

# Test 2: Clear overlap
assert can_complete_all([(1, 4), (2, 6), (8, 10)]) == False

# Test 3: Empty list
assert can_complete_all([]) == True

# Test 4: Single delivery
assert can_complete_all([(5, 10)]) == True

# Test 5: Touching endpoints (not overlapping per our definition)
assert can_complete_all([(1, 5), (5, 10)]) == True

# Test 6: Unsorted input with overlap
assert can_complete_all([(5, 10), (1, 6)]) == False
```

---

## Stage 2: Merge Overlapping Windows (~20 minutes)

### Prompt Addition

> "Great. Now imagine the system needs to display the dasher's busy periods on a calendar. Merge all overlapping delivery windows into consolidated blocks and return the merged list."

### Starter Code

```python
def merge_windows(windows: List[Tuple[int, int]]) -> List[Tuple[int, int]]:
    """
    Stage 2: Merge all overlapping delivery windows and return the
    merged list sorted by start time.
    """
    pass

# --- Stage 2 Test Calls ---
# Expected: [(1, 6), (8, 10)]
print(merge_windows([(1, 4), (2, 6), (8, 10)]))
# Expected: [(1, 10)]
print(merge_windows([(1, 10), (2, 5), (3, 7)]))
# Expected: [(1, 2), (3, 4), (5, 6)]
print(merge_windows([(1, 2), (3, 4), (5, 6)]))
```

### Expected Approach

1. Sort windows by start time.
2. Initialize a result list with the first window.
3. For each subsequent window, compare its start with the end of the last window in the result:
   - If `current_start < last_end`, they overlap — extend `last_end` to `max(last_end, current_end)`.
   - Otherwise, append the current window as a new merged block.
4. Return the result list.

Note: because touching endpoints (`[1,5]` and `[5,8]`) mean the dasher is free at that boundary, they should NOT be merged. Only strict overlaps merge. *(If the candidate merges touching windows, discuss the tradeoff — both interpretations are defensible but they should be consistent with Stage 1's definition.)*

### Solution Code

```python
def merge_windows(windows: List[Tuple[int, int]]) -> List[Tuple[int, int]]:
    if not windows:
        return []

    sorted_windows = sorted(windows)
    merged = [list(sorted_windows[0])]

    for start, end in sorted_windows[1:]:
        last = merged[-1]
        if start < last[1]:  # overlapping — strict < to not merge touching
            last[1] = max(last[1], end)
        else:
            merged.append([start, end])

    return [tuple(w) for w in merged]
```

### Complexity
- **Time:** O(n log n)
- **Space:** O(n) for the output list

### Test Cases

```python
# Test 1: Some overlapping, some not
assert merge_windows([(1, 4), (2, 6), (8, 10)]) == [(1, 6), (8, 10)]

# Test 2: All overlapping into one block
assert merge_windows([(1, 10), (2, 5), (3, 7)]) == [(1, 10)]

# Test 3: No overlaps
assert merge_windows([(1, 2), (3, 4), (5, 6)]) == [(1, 2), (3, 4), (5, 6)]

# Test 4: Empty
assert merge_windows([]) == []

# Test 5: Touching endpoints — should NOT merge
assert merge_windows([(1, 5), (5, 10)]) == [(1, 5), (5, 10)]

# Test 6: Nested intervals
assert merge_windows([(1, 10), (3, 5), (4, 8)]) == [(1, 10)]

# Test 7: Unsorted input
assert merge_windows([(5, 8), (1, 3), (2, 6)]) == [(1, 8)]
```

---

## Stage 3: Minimum Removals to Fit a New Window (~15 minutes)

### Prompt Addition

> "A priority order just came in with a specific delivery window. The dasher's existing schedule may conflict with it. What is the minimum number of existing deliveries to cancel so that the new window fits without overlapping any remaining ones?"
>
> Assume the schedule (existing windows) has already been verified to have no overlaps among themselves (i.e., Stage 1 returns True for the existing windows).

### Starter Code

```python
def min_removals_for_new_window(
    windows: List[Tuple[int, int]],
    new_window: Tuple[int, int]
) -> int:
    """
    Stage 3: Return the minimum number of existing windows that must
    be removed so that `new_window` can be added without any overlaps
    among all remaining windows and the new one.
    """
    pass

# --- Stage 3 Test Calls ---
# Expected: 2 (must remove (4,6) and (7,9))
print(min_removals_for_new_window([(1, 3), (4, 6), (7, 9)], (5, 8)))
# Expected: 0 (new window fits perfectly in a gap)
print(min_removals_for_new_window([(1, 3), (5, 7), (9, 11)], (3, 5)))
# Expected: 3 (new window overlaps everything)
print(min_removals_for_new_window([(1, 3), (4, 6), (7, 9)], (0, 100)))
```

### Expected Approach

Since the existing windows are already non-overlapping, they can be sorted and we can find those that conflict with the new window using a straightforward scan (or binary search for efficiency):

1. Sort existing windows by start time.
2. A window `(s, e)` overlaps the new window `(ns, ne)` if and only if `s < ne` and `ns < e`.
3. Count how many existing windows satisfy that condition. That count is the minimum number of removals because no existing windows overlap each other — every conflict is independent.

The insight is that because existing windows are mutually non-overlapping, there is no cascading benefit to removing one window over another. Every window that conflicts with the new one must be removed.

### Solution Code

```python
def min_removals_for_new_window(
    windows: List[Tuple[int, int]],
    new_window: Tuple[int, int]
) -> int:
    ns, ne = new_window
    count = 0

    for s, e in windows:
        # Two intervals overlap iff each starts before the other ends
        if s < ne and ns < e:
            count += 1

    return count
```

*Optional optimization with binary search (mention if candidate finishes early):*

```python
import bisect

def min_removals_for_new_window_optimized(
    windows: List[Tuple[int, int]],
    new_window: Tuple[int, int]
) -> int:
    if not windows:
        return 0

    sorted_windows = sorted(windows)
    ns, ne = new_window
    starts = [w[0] for w in sorted_windows]
    ends = [w[1] for w in sorted_windows]

    # First window whose start < ne: all from index 0..right_idx-1
    right_idx = bisect.bisect_left(starts, ne)
    # Last window whose end > ns: all from index left_idx..len-1
    left_idx = bisect.bisect_right(ends, ns)

    # Overlapping windows are those in the intersection of the two ranges
    overlap_count = max(0, right_idx - left_idx)
    return overlap_count
```

### Complexity
- **Brute-force:** O(n) time, O(1) space
- **Binary search variant:** O(n log n) if we need to sort, O(log n) if pre-sorted

### Test Cases

```python
# Test 1: New window conflicts with middle delivery
assert min_removals_for_new_window(
    [(1, 3), (4, 6), (7, 9)],
    (5, 8)
) == 2  # must remove (4,6) and (7,9)

# Test 2: New window fits perfectly in a gap
assert min_removals_for_new_window(
    [(1, 3), (5, 7), (9, 11)],
    (3, 5)
) == 0

# Test 3: New window overlaps everything
assert min_removals_for_new_window(
    [(1, 3), (4, 6), (7, 9)],
    (0, 100)
) == 3

# Test 4: Empty existing schedule
assert min_removals_for_new_window([], (5, 10)) == 0

# Test 5: Touching endpoints — no overlap
assert min_removals_for_new_window(
    [(1, 5), (10, 15)],
    (5, 10)
) == 0

# Test 6: Single overlapping window
assert min_removals_for_new_window(
    [(1, 5), (6, 10), (12, 15)],
    (4, 7)
) == 2  # (1,5) and (6,10) overlap with (4,7)
```

---

## Scoring Rubric

### Fundamentals (1-4)

| Score | Description |
|-------|-------------|
| 1 | Cannot identify that sorting is helpful; no understanding of interval comparison logic. |
| 2 | Understands the brute-force O(n^2) approach but struggles to implement the sorted scan. Gets confused by endpoint inclusivity. |
| 3 | Implements sorting + linear scan correctly. Handles the merge step with minor bugs. Recognizes the greedy observation in Stage 3. |
| 4 | Clean, correct solutions for all three stages. Discusses time/space tradeoffs unprompted. Mentions binary search optimization in Stage 3. |

### Coding (1-4)

| Score | Description |
|-------|-------------|
| 1 | Code does not run. Significant syntax errors or incomplete functions. |
| 2 | Code runs but fails several edge cases (empty list, touching endpoints, unsorted input). |
| 3 | Code is correct for all standard cases. Minor style issues. Uses reasonable variable names. |
| 4 | Code is clean, well-structured, Pythonic (uses `sorted()`, tuple unpacking, list comprehensions where appropriate). Handles all edge cases. |

### Communication (1-4)

| Score | Description |
|-------|-------------|
| 1 | Dives into code without discussing approach. Cannot explain choices when asked. |
| 2 | Gives a rough outline but skips important details (e.g., how overlap is detected, why sorting helps). |
| 3 | Explains approach before coding, walks through examples, identifies edge cases when prompted. |
| 4 | Proactively asks clarifying questions, states assumptions explicitly, walks through test cases after coding, identifies edge cases without prompting, discusses complexity unprompted. |

---

## Interviewer Guide

### Hints

#### Stage 1: Overlap Detection

| Level | Hint |
|-------|------|
| **Mild** | "What would make it easier to compare windows if they were in a particular order?" |
| **Medium** | "If you sort by start time, what condition between consecutive windows tells you they overlap?" |
| **Strong** | "Sort by start time. Then for each consecutive pair, check if the next start is before the previous end. If so, overlap." |

#### Stage 2: Merge Windows

| Level | Hint |
|-------|------|
| **Mild** | "Can you reuse the sorted order from Stage 1? What changes when you find an overlap now?" |
| **Medium** | "Instead of returning False on overlap, how would you combine two overlapping windows into one?" |
| **Strong** | "Keep a running 'last merged window.' If the next window overlaps it, extend the end to max of both ends. Otherwise start a new merged window." |

#### Stage 3: Minimum Removals

| Level | Hint |
|-------|------|
| **Mild** | "The existing windows are already non-overlapping. Does that simplify anything?" |
| **Medium** | "If no existing windows overlap each other, can removing one window ever 'free up' another that doesn't directly conflict with the new window?" |
| **Strong** | "Since existing windows don't overlap each other, every window that conflicts with the new one is an independent conflict. Just count them." |

### When to Intervene

- If the candidate spends more than 5 minutes on a brute-force O(n^2) overlap check without considering sorting, give the mild hint for Stage 1.
- If the candidate gets the sort right but writes the overlap condition backwards (e.g., `>=` instead of `<`), let them discover it via a test case first, then nudge.
- If Stage 2 merge logic is spiraling into complexity, suggest they walk through the example `[(1,4), (2,6), (8,10)]` by hand on a number line.
- If the candidate finishes all three stages early, ask them to optimize Stage 3 with binary search, or discuss how the solution changes if endpoints are inclusive.

### Common Mistakes

1. **Off-by-one on overlap check:** Using `<=` instead of `<` (or vice versa) for the endpoint comparison. This stems from not clarifying inclusive vs. exclusive boundaries.
2. **Forgetting to sort:** Assuming the input is already sorted.
3. **Mutating the input list:** Sorting in place when the original order matters. Not a bug per se, but worth discussing.
4. **Merge — not taking max of ends:** Writing `last[1] = end` instead of `last[1] = max(last[1], end)`, which breaks for nested intervals like `[(1, 10), (2, 5)]`.
5. **Stage 3 — overcomplicating:** Trying dynamic programming or backtracking when a simple count suffices because existing windows are non-overlapping.

### Edge Cases to Watch For

- Empty input list
- Single-element list
- All windows identical: `[(3, 7), (3, 7), (3, 7)]`
- Windows that touch but don't overlap: `[(1, 5), (5, 10)]`
- A window entirely contained within another: `[(1, 10), (3, 5)]`
- Zero-length windows: `[(5, 5)]`
- New window in Stage 3 that is zero-length: `(5, 5)` — should conflict with nothing since `5 < 5` is false
