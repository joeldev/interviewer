# DoorDash DSA — Mock Technical Interview Series

18 mock interview rounds based on the DoorDash technical interview preparation guide. Each round tests one primary data structure or algorithm with a realistic, DoorDash-themed problem. All code is Python.

## Quick Start

Run `/interview doordash_dsa` in Claude Code to begin. The system tracks your progress and suggests the next round based on difficulty progression.

## Scoring (from DoorDash Article)

Three categories, each scored 1-4:
- **Fundamentals**: DS knowledge, complexity analysis, optimal solutions
- **Coding**: Abstractions, clean syntax, debugging ability
- **Communication**: Clarifying questions, explaining approach, receiving feedback

**Pass threshold**: Total >= 8/12 with no category at 1.

## All Rounds (Suggested Progression Order)

| Order | # | Topic | Problem Title | Difficulty | Stages |
|-------|---|-------|---------------|------------|--------|
| 1 | 01 | Arrays | Delivery Time Windows | 1/5 | Overlap check → Merge intervals → Min removals |
| 2 | 02 | Hash Maps | Restaurant Order Frequency | 1/5 | Frequency map → Time window → Top-K |
| 3 | 03 | Sets | Unique Delivery Zones | 1/5 | Set ops → N-dasher coverage → Greedy assignment |
| 4 | 17 | Binary Search | Optimal Delivery Radius | 2/5 | Basic search → Rotated array → Bisect boundaries |
| 5 | 04 | Stacks | Balanced Route Expressions | 2/5 | Validation → Longest valid → Min insertions |
| 6 | 05 | Queues | Support Ticket System | 2/5 | Priority queue → Priority upgrade → Fairness |
| 7 | 18 | Sorting | Order Fulfillment Prioritization | 2/5 | Merge sort → Multi-key stable → Nearly-sorted |
| 8 | 06 | Singly Linked Lists | Dasher Route Waypoints | 2/5 | Cycle detect → Cycle start → Remove + reverse |
| 9 | 07 | Doubly Linked Lists | Recently Viewed Restaurants | 3/5 | LRU cache → Expire/top-k → Thread-safety |
| 10 | 08 | Binary Trees | Menu Category Hierarchy | 3/5 | Traversals → LCA → Serialize/deserialize |
| 11 | 09 | BSTs | Dynamic Price Ranking | 3/5 | Insert/closest → K-th smallest → Delete |
| 12 | 11 | Heaps | Top K Nearest Dashers | 3/5 | K-nearest → Streaming → Multi-key sort |
| 13 | 12 | BFS | Minimum Hops Between Locations | 3/5 | Shortest dist → Path reconstruction → All paths |
| 14 | 13 | DFS | Delivery Dependency Resolution | 3/5 | Topo sort → Cycle detect → Count orderings |
| 15 | 15 | Recursion/D&C | Delivery Zone Partitioning | 4/5 | Power set → Merge sort → Closest pair |
| 16 | 10 | Graphs | Delivery Network Connectivity | 4/5 | Reachability → Cycle detect → Bridge finding |
| 17 | 14 | Dijkstra's | Optimal Delivery Route | 4/5 | Shortest dist → Path → Multi-stop |
| 18 | 16 | Dynamic Programming | Maximum Delivery Earnings | 5/5 | Memoized recursion → Bottom-up DP → Backtrack |

## Topic Mapping to DoorDash Article

### Data Structures
| Article Topic | Round |
|---------------|-------|
| Arrays | 01 |
| Hash maps | 02 |
| Sets | 03 |
| Stacks | 04 |
| Queues | 05 |
| Single linked lists | 06 |
| Doubly linked lists | 07 |
| Binary Trees (BT/DAGs) | 08 |
| Binary Search Trees (BST) | 09 |
| Graphs (directed/undirected, cyclic/acyclic) | 10 |
| Heaps | 11 |

### Algorithms
| Article Topic | Round |
|---------------|-------|
| BFS | 12 |
| DFS | 13 |
| Dijkstra's | 14 |
| Recursion & Divide and Conquer | 15 |
| Dynamic Programming | 16 |
| Binary Search | 17 |
| Sorting Algorithms | 18 |
