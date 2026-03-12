# Interviewer

A mock technical interview system powered by [Claude Code](https://docs.anthropic.com/en/docs/claude-code). It conducts realistic, multi-stage coding interviews with scoring, hints, and detailed feedback — like having a patient, rigorous interviewer available any time.

## How It Works

Each interview series contains rounds that follow a consistent format: you're given a problem, work through 2-3 progressively harder stages, and get scored on Fundamentals, Coding, and Communication (each 1-4, pass = 8+/12 with no 1s). Claude acts as the interviewer — it manages the conversation, provides hints when you're stuck, runs your code, and scores you at the end.

Your progress is tracked per-series so you can pick up where you left off across sessions.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI

## Quick Start

```
# Clone and open in Claude Code
git clone git@github.com:joeldev/interviewer.git
cd interviewer

# See available series
/interview

# Start the next round in a series
/interview doordash_dsa next

# Check your progress
/interview doordash_dsa status

# Jump to a specific round
/interview doordash_dsa 5

# Generate a brand new series on any topic
/interview generate
```

## Included Series

### DoorDash DSA (18 rounds, Python)

Data structures and algorithms interview prep based on the DoorDash technical interview format. Covers arrays, hash maps, trees, graphs, dynamic programming, and more — ordered from easiest to hardest.

| Difficulty | Rounds |
|------------|--------|
| 1/5 | Arrays, Hash Maps, Sets |
| 2/5 | Binary Search, Stacks, Queues, Sorting, Singly Linked Lists |
| 3/5 | Doubly Linked Lists, Binary Trees, BSTs, Heaps, BFS, DFS |
| 4/5 | Recursion/D&C, Graphs, Dijkstra's |
| 5/5 | Dynamic Programming |

## Generating New Series

Run `/interview generate` and Claude will walk you through creating a custom interview series for any domain — iOS, system design, frontend, backend, etc. It asks about your target topics, experience level, and language, then generates all the round files.

The system supports three validation tiers depending on the domain:

- **Full validation** (Python, JS) — Claude runs your code and checks output
- **Build-only** (Swift, Kotlin, TS frontend) — Claude compiles and runs unit tests, asks you to verify visual output
- **Review-only** (system design, architecture) — Claude reviews your designs and diagrams, asks probing questions

## Scoring

Each round is scored across three categories:

| Category | What It Measures |
|----------|-----------------|
| **Fundamentals** | Data structure/algorithm knowledge, complexity analysis, optimal approach |
| **Coding** | Code quality, abstractions, clean syntax, self-debugging |
| **Communication** | Clarifying questions, explaining your approach, receiving feedback |

**Pass**: total >= 8/12 with no individual category at 1.

After scoring, you get detailed feedback: what went well, what to improve, comparison to the optimal solution, and study recommendations.

## Project Structure

```
series/
  <series_slug>/
    series.template.json   # Series metadata + clean state (committed)
    series.json            # Your personal progress (gitignored)
    INDEX.md               # Round listing and overview
    01_topic/
      round.md             # Problem, solutions, hints, rubric
      attempt_1.py         # Your work (gitignored)
.claude/
  commands/
    interview.md           # The interview skill that drives everything
```
