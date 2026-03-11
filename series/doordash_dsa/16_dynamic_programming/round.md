# Round 16: Dynamic Programming — "Maximum Delivery Earnings"

## Metadata
- **Topic:** Dynamic Programming (Weighted Job Scheduling)
- **Difficulty:** 5/5
- **Estimated Time:** 55-60 minutes
- **Key Concepts:** Memoization, bottom-up DP, binary search for predecessor, backtracking through DP table, interval scheduling, knapsack relationship

## Problem Prompt

*Read this to the candidate:*

> You are building a feature for DoorDash's Dasher app. A dasher sees N possible deliveries available for the day. Each delivery has a start time, an end time, and a payout in dollars. A dasher can only perform one delivery at a time — they cannot overlap. Given the list of deliveries, find the maximum total earnings the dasher can make in a day.
>
> For example, if the deliveries are:
>
> | Delivery | Start | End | Payout |
> |----------|-------|-----|--------|
> | A        | 1     | 3   | $50    |
> | B        | 2     | 5   | $80    |
> | C        | 4     | 6   | $60    |
> | D        | 6     | 8   | $70    |
>
> What is the maximum the dasher can earn?

Do **not** volunteer whether the input is sorted, whether there is travel time between deliveries, or how ties are broken. Let the candidate ask.

## Clarifying Questions & Hidden Constraints

| Question the Candidate Should Ask | Answer |
|---|---|
| Are the deliveries sorted in any order? | Not necessarily. You may sort them however you like. |
| Can deliveries overlap in time? | Yes, that's the core constraint — you must choose a non-overlapping subset. |
| Is there travel time between deliveries? | For this problem, assume no travel time. If delivery A ends at time 5, you can start delivery B at time 5. |
| Can a delivery start and end at the same time? | No, every delivery has a positive duration (end > start). |
| Are start/end times integers? | Yes, you can assume integer times. |
| Can payouts be zero or negative? | All payouts are positive. |
| How large can N be? | Up to 10^5. Your solution should be efficient. |
| What should I return? | The maximum total earnings (a single number). We'll extend this later. |

**Hidden constraint (reveal if asked):** When two deliveries share an endpoint (one ends at time T, another starts at time T), they are compatible — the dasher can do both.

## Skeleton Code

```python
from typing import List, Tuple

def max_earnings(deliveries: List[Tuple[int, int, int]]) -> int:
    """
    Given a list of deliveries where each delivery is (start_time, end_time, payout),
    return the maximum total earnings from a non-overlapping subset.

    Two deliveries are compatible if one ends before or at the time the other starts.
    """
    pass

if __name__ == "__main__":
    # Expected: 180 (take A=$50 + C=$60 + D=$70)
    print(max_earnings([(1, 3, 50), (2, 5, 80), (4, 6, 60), (6, 8, 70)]))
    # Expected: 60 (all non-overlapping, take all)
    print(max_earnings([(0, 1, 10), (1, 2, 20), (2, 3, 30)]))
    # Expected: 0 (empty)
    print(max_earnings([]))
```

---

## Stage 1: Recursive Solution with Memoization (~20 minutes)

### Expected Approach

1. **Sort** deliveries by end time. This is the key insight that makes the problem tractable — it creates a natural ordering so that for each delivery, all compatible predecessors appear earlier in the sorted array.
2. For each delivery `i`, use **binary search** to find the latest delivery `j` (where `j < i`) whose end time is `<= start time of i`. This is the "last compatible" delivery.
3. For each delivery `i`, the candidate faces a choice: either **include** delivery `i` (earn its payout + best earnings from deliveries `0..j`) or **skip** it (best earnings from deliveries `0..i-1`).
4. The recurrence is: `dp(i) = max(payout[i] + dp(last_compatible(i)), dp(i-1))`.
5. **Memoize** the recursive calls to avoid exponential recomputation.

The candidate should recognize that without memoization, the recursion has overlapping subproblems and is O(2^n). With memoization, each subproblem is solved once — O(n) subproblems, each doing O(log n) binary search work during preprocessing.

### Solution Code

```python
from typing import List, Tuple
from bisect import bisect_right

def max_earnings(deliveries: List[Tuple[int, int, int]]) -> int:
    if not deliveries:
        return 0

    # Sort by end time
    deliveries.sort(key=lambda d: d[1])
    n = len(deliveries)

    # Precompute last compatible job for each delivery using binary search.
    # For delivery i, find the largest index j < i such that deliveries[j].end <= deliveries[i].start.
    end_times = [d[1] for d in deliveries]

    def last_compatible(i: int) -> int:
        """Return index of last delivery that ends <= start of delivery i, or -1 if none."""
        start_i = deliveries[i][0]
        # bisect_right gives insertion point for start_i in end_times.
        # We want the largest end_time <= start_i, which is at index (bisect_right - 1).
        idx = bisect_right(end_times, start_i, 0, i) - 1
        return idx

    memo = {}

    def dp(i: int) -> int:
        """Maximum earnings considering deliveries 0..i."""
        if i < 0:
            return 0
        if i in memo:
            return memo[i]

        # Option 1: Skip delivery i
        skip = dp(i - 1)

        # Option 2: Take delivery i
        j = last_compatible(i)
        take = deliveries[i][2] + dp(j)

        memo[i] = max(skip, take)
        return memo[i]

    return dp(n - 1)
```

### Complexity

- **Time:** O(n log n) — sorting is O(n log n), precomputing last_compatible for each delivery is O(log n) each via binary search (O(n log n) total), and the memoized recursion solves n subproblems in O(1) each.
- **Space:** O(n) — for the memo table and recursion stack.

### Test Cases

```python
# Example from the prompt
deliveries_1 = [(1, 3, 50), (2, 5, 80), (4, 6, 60), (6, 8, 70)]
assert max_earnings(deliveries_1) == 180  # Take A($50) + C($60) + D($70)

# Single delivery
deliveries_2 = [(0, 10, 100)]
assert max_earnings(deliveries_2) == 100

# All non-overlapping — take them all
deliveries_3 = [(0, 1, 10), (1, 2, 20), (2, 3, 30)]
assert max_earnings(deliveries_3) == 60

# All overlapping — take the best one
deliveries_4 = [(0, 10, 50), (1, 10, 60), (2, 10, 40)]
assert max_earnings(deliveries_4) == 60

# Empty
assert max_earnings([]) == 0
```

---

## Stage 2: Bottom-Up DP Table (~20 minutes)

### Prompt Addition

> Great. Now convert your recursive memoized solution into a bottom-up iterative DP solution. Analyze its time and space complexity. Can you optimize the space usage?

### Starter Code

```python
def max_earnings_bottomup(deliveries: List[Tuple[int, int, int]]) -> int:
    """
    Given a list of deliveries where each delivery is (start_time, end_time, payout),
    return the maximum total earnings from a non-overlapping subset.
    Bottom-up iterative DP approach.
    """
    pass

# --- Stage 2 Test Calls ---
# Expected: 180
print(max_earnings_bottomup([(1, 3, 50), (2, 5, 80), (4, 6, 60), (6, 8, 70)]))
# Expected: 60
print(max_earnings_bottomup([(0, 1, 10), (1, 2, 20), (2, 3, 30)]))
# Expected: 60 (all overlapping — take the best one)
print(max_earnings_bottomup([(0, 10, 50), (1, 10, 60), (2, 10, 40)]))
```

### Expected Approach

1. Create a DP array of size `n + 1` where `dp[i]` represents the maximum earnings considering the first `i` deliveries (1-indexed for cleaner base case).
2. Iterate from `i = 1` to `n`. For each delivery, compute the same recurrence iteratively: `dp[i] = max(dp[i-1], payout[i] + dp[last_compatible(i) + 1])`.
3. The answer is `dp[n]`.
4. **Space optimization discussion:** Unlike some DP problems (e.g., Fibonacci), we cannot trivially reduce to O(1) space because `last_compatible(i)` can reference any earlier index, not just `i-1` or `i-2`. The array must be kept in full. The candidate should articulate *why* space optimization isn't straightforward here.

### Solution Code

```python
from typing import List, Tuple
from bisect import bisect_right

def max_earnings_bottomup(deliveries: List[Tuple[int, int, int]]) -> int:
    if not deliveries:
        return 0

    deliveries.sort(key=lambda d: d[1])
    n = len(deliveries)
    end_times = [d[1] for d in deliveries]

    def last_compatible(i: int) -> int:
        start_i = deliveries[i][0]
        idx = bisect_right(end_times, start_i, 0, i) - 1
        return idx

    # dp[i] = max earnings from deliveries 0..i-1 (first i deliveries)
    # dp[0] = 0 (no deliveries considered)
    dp = [0] * (n + 1)

    for i in range(1, n + 1):
        # Delivery index is i-1 (0-indexed)
        payout = deliveries[i - 1][2]
        j = last_compatible(i - 1)  # last compatible index (0-indexed), or -1

        # Option 1: skip delivery i-1
        skip = dp[i - 1]

        # Option 2: take delivery i-1
        # j is 0-indexed; dp is 1-indexed, so dp[j+1] gives earnings from first (j+1) deliveries
        take = payout + dp[j + 1]

        dp[i] = max(skip, take)

    return dp[n]
```

### Complexity

- **Time:** O(n log n) — same as Stage 1. Sorting is O(n log n), binary searches are O(n log n) total, and filling the DP table is O(n).
- **Space:** O(n) — for the DP array. Cannot reduce further because `last_compatible(i)` can point to any earlier index.

### Test Cases

```python
# Same tests as Stage 1 — results must match
deliveries_1 = [(1, 3, 50), (2, 5, 80), (4, 6, 60), (6, 8, 70)]
assert max_earnings_bottomup(deliveries_1) == 180

deliveries_2 = [(0, 10, 100)]
assert max_earnings_bottomup(deliveries_2) == 100

deliveries_3 = [(0, 1, 10), (1, 2, 20), (2, 3, 30)]
assert max_earnings_bottomup(deliveries_3) == 60

deliveries_4 = [(0, 10, 50), (1, 10, 60), (2, 10, 40)]
assert max_earnings_bottomup(deliveries_4) == 60

# Larger test: chain of short deliveries vs one long one
deliveries_5 = [(0, 100, 200)] + [(i, i + 1, 5) for i in range(100)]
assert max_earnings_bottomup(deliveries_5) == 500  # 100 short deliveries at $5 each
```

---

## Stage 3: Backtracking — Return the Chosen Deliveries (~15 minutes)

### Prompt Addition

> Now modify your solution so that instead of returning just the maximum earnings, it also returns the actual list of deliveries chosen. Discuss how this problem relates to the knapsack problem and interval scheduling.

### Starter Code

```python
def max_earnings_with_schedule(
    deliveries: List[Tuple[int, int, int]]
) -> Tuple[int, List[Tuple[int, int, int]]]:
    """
    Returns (max_earnings, list_of_chosen_deliveries).
    Each delivery is (start, end, payout).
    """
    pass

# --- Stage 3 Test Calls ---
# Expected: (180, [(1, 3, 50), (4, 6, 60), (6, 8, 70)])
print(max_earnings_with_schedule([(1, 3, 50), (2, 5, 80), (4, 6, 60), (6, 8, 70)]))
# Expected: (60, [(0, 1, 10), (1, 2, 20), (2, 3, 30)])
print(max_earnings_with_schedule([(0, 1, 10), (1, 2, 20), (2, 3, 30)]))
# Expected: (0, [])
print(max_earnings_with_schedule([]))
```

### Expected Approach

1. After building the DP table, **backtrack** from `dp[n]` to reconstruct which deliveries were included.
2. At each step `i` (going backward from `n` to `1`):
   - If `dp[i] == dp[i-1]`, delivery `i-1` was **skipped** — move to `i-1`.
   - Otherwise, delivery `i-1` was **taken** — record it, then jump to `last_compatible(i-1) + 1`.
3. Return the list of chosen deliveries in order.

**Relationship discussion points:**
- **Knapsack:** This is a variant of 0/1 knapsack where the "weight" is time (an interval) and "incompatibility" between items is defined by overlap rather than a single weight capacity. Classic knapsack has a single capacity dimension; here the constraint is pairwise non-overlap.
- **Interval scheduling:** The unweighted version (all payouts equal) is the classic greedy interval scheduling problem — just pick the job with the earliest end time greedily. Once payouts differ, greedy fails and DP is required.
- **Weighted job scheduling** is the canonical name for this problem.

### Solution Code

```python
from typing import List, Tuple
from bisect import bisect_right

def max_earnings_with_schedule(
    deliveries: List[Tuple[int, int, int]]
) -> Tuple[int, List[Tuple[int, int, int]]]:
    """
    Returns (max_earnings, list_of_chosen_deliveries).
    Each delivery is (start, end, payout).
    """
    if not deliveries:
        return 0, []

    # Keep original indices for output clarity, but sort by end time
    deliveries.sort(key=lambda d: d[1])
    n = len(deliveries)
    end_times = [d[1] for d in deliveries]

    def last_compatible(i: int) -> int:
        start_i = deliveries[i][0]
        idx = bisect_right(end_times, start_i, 0, i) - 1
        return idx

    # Build DP table (same as Stage 2)
    dp = [0] * (n + 1)
    # Store last_compatible results to reuse during backtracking
    lc = [-1] * n

    for i in range(1, n + 1):
        payout = deliveries[i - 1][2]
        j = last_compatible(i - 1)
        lc[i - 1] = j

        skip = dp[i - 1]
        take = payout + dp[j + 1]
        dp[i] = max(skip, take)

    # Backtrack to find chosen deliveries
    chosen = []
    i = n
    while i >= 1:
        payout = deliveries[i - 1][2]
        j = lc[i - 1]
        take = payout + dp[j + 1]

        if take > dp[i - 1]:
            # Delivery i-1 was taken
            chosen.append(deliveries[i - 1])
            i = j + 1  # Jump to after the last compatible delivery
        else:
            # Delivery i-1 was skipped
            i -= 1

    chosen.reverse()  # Restore chronological order
    return dp[n], chosen
```

### Complexity

- **Time:** O(n log n) for sorting and binary searches; O(n) for backtracking. Overall O(n log n).
- **Space:** O(n) for the DP table, last-compatible cache, and output list.

### Test Cases

```python
# Example from the prompt
deliveries_1 = [(1, 3, 50), (2, 5, 80), (4, 6, 60), (6, 8, 70)]
earnings, schedule = max_earnings_with_schedule(deliveries_1)
assert earnings == 180
assert schedule == [(1, 3, 50), (4, 6, 60), (6, 8, 70)]

# All non-overlapping — take them all
deliveries_2 = [(0, 1, 10), (1, 2, 20), (2, 3, 30)]
earnings, schedule = max_earnings_with_schedule(deliveries_2)
assert earnings == 60
assert schedule == [(0, 1, 10), (1, 2, 20), (2, 3, 30)]

# All overlapping — take the best one
deliveries_3 = [(0, 10, 50), (1, 10, 60), (2, 10, 40)]
earnings, schedule = max_earnings_with_schedule(deliveries_3)
assert earnings == 60
assert schedule == [(1, 10, 60)]

# Two equally good options: algorithm picks one deterministically
deliveries_4 = [(0, 5, 100), (0, 5, 100)]
earnings, schedule = max_earnings_with_schedule(deliveries_4)
assert earnings == 100
assert len(schedule) == 1

# Empty input
assert max_earnings_with_schedule([]) == (0, [])
```

---

## Scoring Rubric

### Fundamentals (1-4)

| Score | Criteria |
|-------|----------|
| **4 — Strong** | Immediately recognizes weighted job scheduling. Sorts by end time without prompting. Uses binary search for last compatible job. Correctly identifies overlapping subproblems and applies memoization. Converts to bottom-up DP fluently. Backtracking is clean and correct. Articulates relationship to knapsack/interval scheduling. |
| **3 — Solid** | Recognizes the need for DP after some thought. May need a small hint on sorting by end time or the binary search optimization. Correctly implements memoization. Converts to bottom-up with minor issues. Can backtrack with some guidance. |
| **2 — Developing** | Attempts brute force or greedy first. Needs significant hints to arrive at the DP recurrence. May struggle with the binary search for last compatible job. Can implement one of {memoization, bottom-up} but not both cleanly. Backtracking is incomplete. |
| **1 — Insufficient** | Cannot formulate the recurrence. Gets stuck on sorting or the definition of "compatible." Cannot implement memoization or bottom-up DP. Does not complete Stage 1 within time. |

### Coding (1-4)

| Score | Criteria |
|-------|----------|
| **4 — Strong** | Code is clean, well-structured, and runs correctly on all test cases. Uses `bisect_right` or equivalent correctly. Handles edge cases (empty input, single delivery, all overlapping). Variable names are meaningful. No off-by-one errors. |
| **3 — Solid** | Code is mostly correct with at most one minor bug caught during testing. Reasonable structure. May have a small off-by-one in binary search that is fixed quickly. |
| **2 — Developing** | Code has multiple bugs. May misuse `bisect_left` vs `bisect_right`. Off-by-one errors in DP indexing. Code runs but fails some test cases. |
| **1 — Insufficient** | Code does not run. Fundamental errors in recursion or iteration. Cannot translate the recurrence into working code. |

### Communication (1-4)

| Score | Criteria |
|-------|----------|
| **4 — Strong** | Asks excellent clarifying questions (sorted? travel time? endpoint ties?). Explains thought process before coding. Walks through examples. Clearly articulates why greedy fails and DP is needed. Discusses tradeoffs (top-down vs bottom-up). Explains backtracking logic coherently. |
| **3 — Solid** | Asks some clarifying questions. Explains approach at a high level before coding. Can walk through the algorithm on an example when asked. |
| **2 — Developing** | Jumps into coding without clarifying or explaining. Gives partial explanations when prompted. Difficulty articulating the DP recurrence verbally. |
| **1 — Insufficient** | Cannot explain their approach. Does not ask clarifying questions. Cannot walk through their code on a simple example. |

---

## Interviewer Guide

### Hints (per stage, progressive: mild, medium, strong)

**Stage 1 — Recursive + Memoization**

| Level | Hint |
|-------|------|
| **Mild** | "What ordering of the deliveries would make it easier to reason about which ones are compatible?" |
| **Medium** | "If the deliveries are sorted by end time, and you're deciding whether to include delivery i, what do you need to know about the deliveries before it?" |
| **Strong** | "For each delivery i, binary search for the latest delivery j that ends before i starts. Then dp(i) is the max of skipping i or taking i plus the best you can do up through j." |

**Stage 2 — Bottom-Up DP**

| Level | Hint |
|-------|------|
| **Mild** | "Can you eliminate the recursion by filling in values from left to right?" |
| **Medium** | "Create an array dp where dp[i] is the answer for the first i deliveries. What's the recurrence?" |
| **Strong** | "dp[i] = max(dp[i-1], payout[i-1] + dp[last_compatible(i-1) + 1]). Fill from i=1 to n." |

**Stage 3 — Backtracking**

| Level | Hint |
|-------|------|
| **Mild** | "Looking at your filled DP table, how can you tell whether delivery i was included in the optimal solution?" |
| **Medium** | "Compare dp[i] with dp[i-1]. If they're equal, delivery i was skipped. Otherwise it was taken — and you should jump to last_compatible(i) next." |
| **Strong** | "Walk backward from dp[n]. If dp[i] != dp[i-1], add delivery i-1 to your result and set i = last_compatible(i-1) + 1. Otherwise decrement i." |

### When to Intervene

- If the candidate is attempting a **greedy** approach (e.g., always pick the highest payout), let them try briefly, then ask: "Can you construct a case where picking the highest payout delivery doesn't give the global optimum?" (Example: one $80 delivery from time 0-10 vs. two $50 deliveries at 0-5 and 5-10.)
- If the candidate is writing an **O(2^n) brute force** and hasn't mentioned memoization after 5 minutes, nudge: "Are there overlapping subproblems here?"
- If the candidate is stuck on **binary search details** for more than 3 minutes, offer: "Python's `bisect` module has `bisect_right` — how could you use it?"
- If the candidate's **indexing is off** between 0-indexed deliveries and 1-indexed DP, help them pick one convention and stick with it.

### Common Mistakes

1. **Sorting by start time instead of end time.** The recurrence works cleanly only when sorted by end time, because "all deliveries before index i" then form a valid prefix to consider.
2. **Using `bisect_left` instead of `bisect_right`** (or vice versa) for finding the last compatible job, leading to off-by-one errors in compatibility checks.
3. **Forgetting the "skip" option.** The recurrence must consider both taking and skipping each delivery.
4. **Off-by-one in DP table indexing.** Mixing up 0-indexed delivery arrays with 1-indexed DP arrays.
5. **Backtracking errors.** After taking a delivery, jumping to `last_compatible(i)` instead of `last_compatible(i) + 1` (or the equivalent in their indexing scheme).
6. **Not handling the base case.** Forgetting `dp[0] = 0` or `dp(-1) = 0` for the empty prefix.

### Edge Cases to Watch For

- **Empty input:** Should return 0 (and empty list for Stage 3).
- **Single delivery:** Should return its payout.
- **All deliveries overlap:** Should return the maximum single payout.
- **No overlaps at all:** Should return the sum of all payouts.
- **Deliveries with matching endpoints:** `(1, 5, ...)` and `(5, 8, ...)` are compatible (end == start is okay).
- **Deliveries with identical times:** Two deliveries `(1, 5, 50)` and `(1, 5, 80)` — pick the better one.
- **Large N (10^5):** The solution must be O(n log n), not O(n^2). A nested loop checking all pairs will TLE.
