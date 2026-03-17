# Interviewer

A mock technical interview system powered by [Claude Code](https://docs.anthropic.com/en/docs/claude-code). It conducts realistic, multi-stage interviews with scoring, hints, and detailed feedback — like having a patient, rigorous interviewer available any time.

## How It Works

You generate interview series tailored to what you're preparing for — data structures & algorithms, system design, iOS, web frontend, backend, or anything else. Each series contains rounds that progressively increase in difficulty. For each round, you work through 2-3 stages, and Claude scores you on Fundamentals, Coding, and Communication.

You can also name specific companies and Claude will search the web to see if there are recent knownor tagged questions for those companies, and fact those into the interview series.

Claude adapts its validation approach to the domain:

- **Coding interviews** (Python, JS, etc.) — Claude runs your code and verifies test cases
- **Client/mobile interviews** (Swift, Kotlin, TypeScript) — Claude compiles your code and runs unit tests; asks you to verify visual/UI behavior yourself (honor system, this is a mock interview after all)
- **System design interviews** — you sketch architectures using Mermaid diagrams; Claude reviews your designs, asks probing questions about trade-offs and failure modes

Your progress, scores, and a cumulative study guide are tracked per-series so you can pick up where you left off.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI

## Quick Start

```bash
git clone git@github.com:joeldev/interviewer.git
cd interviewer

# Generate your first interview series
/interview generate
```

The generate flow will ask you about the domain, experience level, target companies, and language. It searches for real interview questions from your target companies to inform the round topics, then generates everything.

```bash
# List your series
/interview

# Work through a series
/interview my_series next       # start next round
/interview my_series status     # check progress
/interview my_series 5          # jump to round 5
```

## Example Series

Here are some examples of what you can generate:

**Data Structures & Algorithms** — arrays, hash maps, trees, graphs, dynamic programming, sorting. Rounds are Python (or JS/TS), fully validated by running code. Good for general SWE interview prep.

**iOS Development** — SwiftUI, concurrency (actors, structured concurrency), networking, architecture patterns (MVVM, unidirectional data flow), performance optimization. Rounds are Swift, validated by compiling with `swiftc` and running unit tests.

**System Design** — URL shorteners, rate limiters, real-time tracking, payment processing, distributed caches, multi-region architectures. Rounds use Mermaid diagrams for architecture sketches — Claude reviews your designs rather than running code.

**Web Frontend** — React/TypeScript component design, state management, performance, accessibility, real-time features. Validated by compiling with `tsc` and running unit tests.

## Scoring

Each round is scored across three categories (1-4 each):

| Category | What It Measures |
|----------|-----------------|
| **Fundamentals** | Core knowledge, complexity analysis, optimal approach |
| **Coding** | Code quality, abstractions, clean syntax, self-debugging |
| **Communication** | Clarifying questions, explaining approach, receiving feedback |

**Pass**: total >= 8/12 with no individual category at 1.

After each round you get detailed feedback and your series' study guide is updated with concepts to review — building a personalized reference sheet as you go.

## Project Structure

```
.claude/commands/
  interview.md            # The interview skill that drives everything
series/                   # Generated locally, gitignored
  <series_slug>/
    series.template.json  # Series metadata + clean state
    series.json           # Your personal progress
    STUDY_GUIDE.md        # Cumulative study reference
    INDEX.md              # Round listing and overview
    01_topic/
      round.md            # Problem, solutions, hints, rubric
      attempt_1.py        # Your work
```
