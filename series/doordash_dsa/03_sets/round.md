# Round 03: Sets — "Unique Delivery Zones"

## Metadata
- **Topic:** Sets, Set Operations, Greedy Algorithms
- **Difficulty:** 1/5
- **Estimated Time:** 55-60 minutes
- **Key Concepts:** Set union, intersection, difference, symmetric difference, greedy set cover

## Problem Prompt

> DoorDash operates across many delivery zones. Each dasher is assigned a list of zone names they can serve. Given information about which dashers cover which zones, help DoorDash answer questions about zone coverage.

*(Read exactly this. Do not specify whether zone names are case-sensitive, whether lists can be empty, or whether zone names can repeat within a single dasher's list. Let the candidate ask.)*

## Clarifying Questions & Hidden Constraints

| # | Question the Candidate Should Ask | Answer to Give |
|---|---|---|
| 1 | Are zone names case-sensitive? | Yes. `"Downtown"` and `"downtown"` are different zones. |
| 2 | Can a dasher's zone list be empty? | Yes. A dasher with no assigned zones is valid. |
| 3 | Can a dasher's zone list contain duplicates? | It shouldn't, but be defensive — treat duplicates as a single zone. Convert to a set on input. |
| 4 | How many dashers are we dealing with? | Stage 1 is exactly two dashers. Stages 2 and 3 extend to N dashers. N can be up to a few thousand. |
| 5 | How many zones can a dasher have? | Up to a few thousand. Zone names are short strings. |
| 6 | What should we return if there are no dashers or no zones? | Return empty sets/lists as appropriate. |
| 7 | Are the zone lists sorted? | No, they can be in any order. |

## Skeleton Code

```python
from typing import List, Set, Dict


def zone_comparison(
    zones_a: List[str],
    zones_b: List[str]
) -> Dict[str, Set[str]]:
    """
    Stage 1: Given two dashers' zone lists, return a dict with keys:
      - "intersection": zones served by BOTH dashers
      - "only_a": zones served by dasher A but not B
      - "only_b": zones served by dasher B but not A
      - "union": all zones served by either dasher
    """
    pass

if __name__ == "__main__":
    # Expected: intersection={'Midtown', 'Uptown'}, only_a={'Downtown'}, only_b={'Suburbs'}, union={'Downtown', 'Midtown', 'Uptown', 'Suburbs'}
    print(zone_comparison(["Downtown", "Midtown", "Uptown"], ["Midtown", "Suburbs", "Uptown"]))
    # Expected: intersection=set(), only_a={'Zone_A', 'Zone_B'}, only_b={'Zone_C', 'Zone_D'}, union={'Zone_A', 'Zone_B', 'Zone_C', 'Zone_D'}
    print(zone_comparison(["Zone_A", "Zone_B"], ["Zone_C", "Zone_D"]))
    # Expected: intersection=set(), only_a=set(), only_b=set(), union=set()
    print(zone_comparison([], []))
```

---

## Stage 1: Two-Dasher Zone Comparison (~20 minutes)

### Expected Approach

1. Convert both lists to sets (handles duplicates).
2. Use Python set operations:
   - Intersection: `set_a & set_b`
   - Difference: `set_a - set_b` and `set_b - set_a`
   - Union: `set_a | set_b`
3. Return the results in a dictionary.

This stage tests basic familiarity with Python set operations and defensive input handling.

### Solution Code

```python
def zone_comparison(
    zones_a: List[str],
    zones_b: List[str]
) -> Dict[str, Set[str]]:
    set_a = set(zones_a)
    set_b = set(zones_b)

    return {
        "intersection": set_a & set_b,
        "only_a": set_a - set_b,
        "only_b": set_b - set_a,
        "union": set_a | set_b,
    }
```

### Complexity
- **Time:** O(a + b) where a and b are the lengths of the two zone lists (set construction is O(n), set operations on two sets are O(min(a, b)) for intersection, O(a) or O(b) for difference, O(a + b) for union).
- **Space:** O(a + b) for the sets and output.

### Test Cases

```python
# Test 1: Partial overlap
result1 = zone_comparison(
    ["Downtown", "Midtown", "Uptown"],
    ["Midtown", "Suburbs", "Uptown"]
)
assert result1["intersection"] == {"Midtown", "Uptown"}
assert result1["only_a"] == {"Downtown"}
assert result1["only_b"] == {"Suburbs"}
assert result1["union"] == {"Downtown", "Midtown", "Uptown", "Suburbs"}

# Test 2: No overlap
result2 = zone_comparison(["Zone_A", "Zone_B"], ["Zone_C", "Zone_D"])
assert result2["intersection"] == set()
assert result2["only_a"] == {"Zone_A", "Zone_B"}
assert result2["only_b"] == {"Zone_C", "Zone_D"}
assert result2["union"] == {"Zone_A", "Zone_B", "Zone_C", "Zone_D"}

# Test 3: Identical zone lists
result3 = zone_comparison(["X", "Y"], ["Y", "X"])
assert result3["intersection"] == {"X", "Y"}
assert result3["only_a"] == set()
assert result3["only_b"] == set()
assert result3["union"] == {"X", "Y"}

# Test 4: One empty list
result4 = zone_comparison(["A", "B"], [])
assert result4["intersection"] == set()
assert result4["only_a"] == {"A", "B"}
assert result4["only_b"] == set()
assert result4["union"] == {"A", "B"}

# Test 5: Both empty
result5 = zone_comparison([], [])
assert result5["intersection"] == set()
assert result5["union"] == set()

# Test 6: Duplicates in input
result6 = zone_comparison(["A", "A", "B"], ["B", "B", "C"])
assert result6["intersection"] == {"B"}
assert result6["only_a"] == {"A"}
assert result6["only_b"] == {"C"}
```

---

## Stage 2: N-Dasher Coverage Analysis (~20 minutes)

### Prompt Addition

> "Now scale this up. You have N dashers, each with their own zone list. Find: (1) zones served by ALL dashers, and (2) zones served by EXACTLY one dasher."

### Starter Code

```python
def zone_coverage_analysis(
    all_zones: List[List[str]]
) -> Dict[str, Set[str]]:
    """
    Stage 2: Given a list of zone lists (one per dasher), return:
      - "served_by_all": zones served by EVERY dasher
      - "served_by_exactly_one": zones served by exactly one dasher
    """
    pass

# --- Stage 2 Test Calls ---
# Expected: served_by_all={'C'}, served_by_exactly_one={'A', 'E'}
print(zone_coverage_analysis([["A", "B", "C"], ["B", "C", "D"], ["C", "D", "E"]]))
# Expected: served_by_all={'X', 'Y'}, served_by_exactly_one=set()
print(zone_coverage_analysis([["X", "Y"], ["Y", "X"], ["X", "Y"]]))
# Expected: served_by_all=set(), served_by_exactly_one=set()
print(zone_coverage_analysis([]))
```

### Expected Approach

**Served by all (intersection of all sets):**
1. Convert each dasher's zone list to a set.
2. Start with the first set and intersect iteratively with each subsequent set.
3. Handle edge case: if there are no dashers, "served by all" is an empty set.

**Served by exactly one:**
1. Count how many dashers serve each zone (use a dictionary or `Counter`).
2. Collect zones with count == 1.

Alternative for "served by exactly one": use symmetric difference iteratively. However, this gives zones served by an ODD number of dashers, not exactly one. The counting approach is more reliable and easier to reason about.

### Solution Code

```python
from collections import Counter

def zone_coverage_analysis(
    all_zones: List[List[str]]
) -> Dict[str, Set[str]]:
    if not all_zones:
        return {
            "served_by_all": set(),
            "served_by_exactly_one": set(),
        }

    sets = [set(zones) for zones in all_zones]

    # Intersection of all sets: zones every dasher covers
    served_by_all = sets[0].copy()
    for s in sets[1:]:
        served_by_all &= s

    # Count how many dashers serve each zone
    zone_count: Dict[str, int] = Counter()
    for s in sets:
        for zone in s:
            zone_count[zone] += 1

    served_by_exactly_one = {zone for zone, count in zone_count.items() if count == 1}

    return {
        "served_by_all": served_by_all,
        "served_by_exactly_one": served_by_exactly_one,
    }
```

### Complexity
- **Time:** O(N * Z) where N is the number of dashers and Z is the average number of zones per dasher. The intersection loop is O(N * Z) in the worst case. The counting loop is O(N * Z).
- **Space:** O(N * Z) for storing all the sets, plus O(total_unique_zones) for the counter.

### Test Cases

```python
# Test 1: Three dashers with partial overlaps
result1 = zone_coverage_analysis([
    ["A", "B", "C"],
    ["B", "C", "D"],
    ["C", "D", "E"],
])
assert result1["served_by_all"] == {"C"}
assert result1["served_by_exactly_one"] == {"A", "E"}

# Test 2: All dashers serve the same zones
result2 = zone_coverage_analysis([
    ["X", "Y"],
    ["Y", "X"],
    ["X", "Y"],
])
assert result2["served_by_all"] == {"X", "Y"}
assert result2["served_by_exactly_one"] == set()

# Test 3: No overlap at all
result3 = zone_coverage_analysis([
    ["A"],
    ["B"],
    ["C"],
])
assert result3["served_by_all"] == set()
assert result3["served_by_exactly_one"] == {"A", "B", "C"}

# Test 4: Single dasher
result4 = zone_coverage_analysis([["A", "B", "C"]])
assert result4["served_by_all"] == {"A", "B", "C"}
assert result4["served_by_exactly_one"] == {"A", "B", "C"}

# Test 5: Empty input — no dashers
result5 = zone_coverage_analysis([])
assert result5["served_by_all"] == set()
assert result5["served_by_exactly_one"] == set()

# Test 6: Some dashers have empty zone lists
result6 = zone_coverage_analysis([
    ["A", "B"],
    [],
    ["B", "C"],
])
# served_by_all: intersection with empty set = empty
assert result6["served_by_all"] == set()
assert result6["served_by_exactly_one"] == {"A", "C"}

# Test 7: Zone served by exactly 2 of 3 dashers — not in "exactly one"
result7 = zone_coverage_analysis([
    ["A", "B"],
    ["B", "C"],
    ["C", "D"],
])
# B: 2 dashers, C: 2 dashers → neither is "exactly one"
assert result7["served_by_all"] == set()
assert result7["served_by_exactly_one"] == {"A", "D"}
```

---

## Stage 3: Assign New Dasher to Uncovered Zones (~15 minutes)

### Prompt Addition

> "A new dasher is joining DoorDash. They have a list of candidate zones they're willing to serve. We want to assign them to cover as many currently uncovered zones as possible from their candidate list. A zone is 'uncovered' if no existing dasher serves it. Return the list of zones the new dasher should be assigned, sorted alphabetically."

### Starter Code

```python
def assign_new_dasher(
    all_zones: List[List[str]],
    candidate_zones: List[str]
) -> List[str]:
    """
    Stage 3: Given existing dashers' zones and a new dasher's candidate
    zone list, return the maximum subset of candidate_zones such that
    every zone in the subset is not already covered by any existing dasher.
    In other words, assign the new dasher to cover as many currently
    uncovered zones as possible from their candidate list.
    Return the result sorted alphabetically.
    """
    pass

# --- Stage 3 Test Calls ---
# Expected: ['E', 'F']
print(assign_new_dasher([["A", "B", "C"], ["C", "D"]], ["B", "E", "F", "D"]))
# Expected: []
print(assign_new_dasher([["A", "B"], ["C", "D"]], ["A", "C"]))
# Expected: ['X', 'Y', 'Z']
print(assign_new_dasher([], ["X", "Y", "Z"]))
```

### Expected Approach

1. Compute the set of all currently covered zones (union of all existing dashers' zones).
2. Convert the candidate zones to a set.
3. The answer is the set difference: `candidate_set - covered_set`.
4. Return sorted alphabetically.

This is deliberately simpler than it might first appear — the candidate may overthink it as a set cover problem. The key insight is that we want ALL uncovered zones from the candidate list, not a minimum subset. There is no tradeoff or optimization needed: take every candidate zone that is not already covered.

### Solution Code

```python
def assign_new_dasher(
    all_zones: List[List[str]],
    candidate_zones: List[str]
) -> List[str]:
    # Build the set of all currently covered zones
    covered = set()
    for zones in all_zones:
        covered.update(zones)

    # The new dasher should take every candidate zone not already covered
    candidate_set = set(candidate_zones)
    uncovered_candidates = candidate_set - covered

    return sorted(uncovered_candidates)
```

### Complexity
- **Time:** O(N * Z) to build the covered set + O(C) for the candidate set difference + O(R log R) to sort the result, where C is the candidate list length and R is the result size.
- **Space:** O(total_zones + C)

### Test Cases

```python
# Test 1: Some candidate zones are uncovered
result1 = assign_new_dasher(
    [["A", "B", "C"], ["C", "D"]],
    ["B", "E", "F", "D"]
)
# Covered: {A, B, C, D}. Candidates: {B, E, F, D}. Uncovered candidates: {E, F}
assert result1 == ["E", "F"]

# Test 2: All candidate zones already covered
result2 = assign_new_dasher(
    [["A", "B"], ["C", "D"]],
    ["A", "C"]
)
assert result2 == []

# Test 3: No existing dashers — everything is uncovered
result3 = assign_new_dasher(
    [],
    ["X", "Y", "Z"]
)
assert result3 == ["X", "Y", "Z"]

# Test 4: Empty candidate list
result4 = assign_new_dasher(
    [["A", "B"]],
    []
)
assert result4 == []

# Test 5: Both empty
result5 = assign_new_dasher([], [])
assert result5 == []

# Test 6: Candidate zones with duplicates
result6 = assign_new_dasher(
    [["A"]],
    ["B", "B", "C", "A"]
)
# Covered: {A}. Unique candidates: {A, B, C}. Uncovered: {B, C}
assert result6 == ["B", "C"]

# Test 7: Large-scale check — result is sorted
result7 = assign_new_dasher(
    [["Midtown", "Downtown"]],
    ["Uptown", "Suburbs", "Airport", "Downtown"]
)
# Covered: {Midtown, Downtown}. Uncovered candidates: {Uptown, Suburbs, Airport}
assert result7 == ["Airport", "Suburbs", "Uptown"]
```

---

## Scoring Rubric

### Fundamentals (1-4)

| Score | Description |
|-------|-------------|
| 1 | Does not know Python set operations. Attempts to implement intersection/union manually with nested loops and gets lost. |
| 2 | Knows that sets exist but uses them clumsily — e.g., iterates and checks `if x in set_b` instead of using `&`. Struggles with the counting approach in Stage 2. |
| 3 | Uses set operations fluently (`&`, `|`, `-`). Correctly implements the counting approach for "exactly one." Identifies the set difference insight in Stage 3. |
| 4 | All stages correct and clean. Discusses complexity of set operations. Notes that `Counter` or a dict-based count is better than iterative symmetric difference for "exactly one." Recognizes Stage 3 is simpler than it appears. |

### Coding (1-4)

| Score | Description |
|-------|-------------|
| 1 | Code does not run. Confused by set syntax, `Counter`, or list-to-set conversion. |
| 2 | Code runs but has bugs: doesn't handle empty inputs, forgets to deduplicate, or returns unsorted results when sorted is required. |
| 3 | Code is correct for all test cases. Uses appropriate data structures. Minor style issues (e.g., unnecessary intermediate variables). |
| 4 | Clean, Pythonic code. Uses set comprehensions, `Counter`, `set.update()`, and `sorted()` idiomatically. Handles all edge cases including empty lists and duplicates. |

### Communication (1-4)

| Score | Description |
|-------|-------------|
| 1 | Starts coding immediately without discussing approach. Cannot explain the difference between intersection and symmetric difference. |
| 2 | Gives a vague plan ("I'll use sets") but doesn't explain the specific operations or walk through examples. |
| 3 | Explains the approach clearly before coding. Asks about case sensitivity and empty inputs. Walks through at least one example per stage. |
| 4 | Proactively clarifies all ambiguities. Explains why symmetric difference doesn't work for "exactly one" (it gives odd-count zones). Discusses complexity. Recognizes that Stage 3 is a simple set difference and explains why it's not a set cover problem. |

---

## Interviewer Guide

### Hints

#### Stage 1: Two-Dasher Comparison

| Level | Hint |
|-------|------|
| **Mild** | "Python has a built-in data structure specifically designed for membership testing and set-theoretic operations. What is it?" |
| **Medium** | "Convert both lists to sets. Then Python supports `&` for intersection, `-` for difference, and `|` for union directly on sets." |
| **Strong** | "Write `set_a = set(zones_a)`, `set_b = set(zones_b)`. Then `set_a & set_b` gives you the intersection. `set_a - set_b` gives zones only in A." |

#### Stage 2: N-Dasher Coverage

| Level | Hint |
|-------|------|
| **Mild** | "For 'served by all,' think about what operation you'd chain across all the sets. For 'exactly one,' think about counting." |
| **Medium** | "Intersect all sets iteratively for 'served by all.' For 'exactly one,' count how many sets each zone appears in and filter for count == 1." |
| **Strong** | "Start with `result = sets[0].copy()`, then loop `result &= sets[i]`. For counting, use `Counter()` — loop through each set and update the counter with its elements." |

#### Stage 3: Assign New Dasher

| Level | Hint |
|-------|------|
| **Mild** | "What zones are currently NOT covered by any existing dasher?" |
| **Medium** | "First compute the union of all existing dashers' zones. Then what operation between that union and the candidate list gives you the answer?" |
| **Strong** | "Build `covered = set()`, then `covered.update(zones)` for each dasher. The answer is `sorted(set(candidate_zones) - covered)`." |

### When to Intervene

- If the candidate tries to implement set intersection from scratch with nested loops, let them finish but then ask: "Python has built-in set operations — can you simplify this?" Give them a chance to refactor.
- If the candidate uses symmetric difference (`^`) for "served by exactly one" in Stage 2, ask: "What if three dashers all serve zone X? Is it in the symmetric difference?" This exposes the bug (XOR of three `True`s is `True`, but we want count == 1, not count is odd).
- If the candidate overcomplicates Stage 3 by treating it as a set cover or optimization problem, ask: "Is there any reason NOT to assign every uncovered zone from the candidate list?" This should trigger the simplification.
- If a candidate finishes very early, ask follow-up questions: "What if we want to assign multiple new dashers, each to maximize the marginal uncovered zones they add?" (This is the greedy set cover problem — NP-hard to solve optimally, but a greedy approach gives a log(n)-approximation.)

### Common Mistakes

1. **Not converting lists to sets:** Operating on lists directly leads to O(n*m) operations instead of O(n+m). The candidate might use `if x in list_b` inside a loop instead of set membership.
2. **Using symmetric difference for "exactly one":** `set_a ^ set_b ^ set_c` gives zones in an odd number of sets, not exactly one. This is a subtle and common error.
3. **Forgetting `.copy()` on the initial set for intersection:** Writing `served_by_all = sets[0]` and then `served_by_all &= sets[1]` mutates `sets[0]`. Not necessarily a bug in the final answer, but indicates unclear reasoning about references.
4. **Not handling empty inputs:** `sets[0]` crashes on an empty list of dashers. An empty dasher list should return empty results.
5. **Not sorting the output in Stage 3:** The problem asks for a sorted result. Easy to forget.
6. **Counter misuse in Stage 2:** Using `Counter(zone_list)` counts duplicates within a single dasher. Should count each zone at most once per dasher by converting to a set first.

### Edge Cases to Watch For

- Empty dasher list (no dashers at all)
- A dasher with an empty zone list (valid, contributes nothing to union, collapses intersection to empty)
- Zone lists with duplicates
- Case sensitivity: `"downtown"` vs. `"Downtown"` are different zones
- Single dasher (intersection = their zones, exactly one = their zones)
- All dashers have identical zone lists (intersection = all zones, exactly one = empty)
- Candidate list in Stage 3 that is entirely covered or entirely uncovered
- Very large number of dashers with small zone lists vs. few dashers with large zone lists (same algorithm, but worth discussing performance)
