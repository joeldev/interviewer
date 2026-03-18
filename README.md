# Interviewer

A mock technical interview system powered by [Claude Code](https://docs.anthropic.com/en/docs/claude-code). It conducts realistic, multi-stage interviews with scoring, hints, and detailed feedback — like having a patient, rigorous interviewer available any time.

## How It Works

You generate interview series tailored to what you're preparing for — data structures & algorithms, system design, iOS, web frontend, backend, or anything else. Each series contains rounds that progressively increase in difficulty. For each round, you work through 2-3 stages, and Claude scores you on Fundamentals, Coding, and Communication.

You can also name specific companies and Claude will search the web to see if there are recent known or tagged questions for those companies, and factor those into the interview series.

Claude adapts its validation approach to the domain:

- **Coding interviews** (Python, JS, etc.) — Claude runs your code and verifies test cases
- **Client/mobile interviews** (Swift, Kotlin, TypeScript) — Claude compiles your code and runs unit tests; asks you to verify visual/UI behavior yourself
- **System design interviews** — you sketch architectures using Mermaid diagrams; Claude reviews your designs, asks probing questions about trade-offs and failure modes

Your progress, scores, and a cumulative study guide are tracked per-series so you can pick up where you left off.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI

## Quick Start

```bash
git clone git@github.com:joeldev/interviewer.git
cd interviewer
claude

# Generate your first interview series
/interview generate
```

The generate flow will ask you about the domain, experience level, target companies, and language, then generate everything.

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

**Data Structures & Algorithms** — Arrays, hash maps, trees, graphs, dynamic programming, sorting. Rounds are Python (or JS/TS), fully validated by running code. Good for general SWE interview prep.

**iOS Development** — SwiftUI, concurrency (actors, structured concurrency), networking, architecture patterns, performance optimization. Rounds are Swift — Claude uses `xcodegen` to scaffold Xcode projects and `xcodebuild` to compile and run tests.

**System Design** — URL shorteners, rate limiters, real-time tracking, payment processing, distributed caches, multi-region architectures. You write Mermaid diagrams for architecture sketches and Claude reviews your designs rather than running code.

**Web Frontend** — React/TypeScript component design, state management, performance, accessibility, real-time features. Validated by compiling with `tsc` and running tests with `vitest` or `jest`.

## Generation Details

### Company-Targeted Research

When you name target companies during generation, Claude searches the web for real interview questions — LeetCode tagged problems, Glassdoor reports, engineering blog posts — and uses them to weight the series topics. Questions aren't copied verbatim or themed around a company; they just ensure the series covers patterns that actually come up.

### Toolchain Discovery

During generation, Claude probes your local environment for platform-specific tools — compilers, build systems, project generators, package managers, test runners, linters — and records what's available. This means each round gets the best possible scaffolding for your setup. For example:

- If you have `xcodegen` installed for an iOS series, rounds will get proper Xcode projects with full `Info.plist` keys, build schemes, and simulator targets
- If you have `cargo` for Rust, rounds scaffold with `cargo init`
- If a useful tool is missing, Claude will suggest installing it and explain why

The discovered toolchain is stored in `series.json` so it's used consistently across all rounds.

### Per-Series CLAUDE.md

Each generated series gets its own `CLAUDE.md` with context specific to that domain:

- **Language conventions** — idiomatic patterns (e.g., "prefer value types in Swift", "use hooks over class components in React")
- **Platform patterns** — relevant frameworks, APIs, and architectural expectations
- **Build instructions** — exact commands to compile, test, and run based on discovered toolchain
- **Interview context** — what interviewers in this domain typically care about
- **Common pitfalls** — domain-specific mistakes to watch for

This file is automatically picked up by Claude Code, so interview behavior stays domain-appropriate without the user needing to repeat context.

## Scoring

Each round is scored across three categories (1-4 each):

| Category | What It Measures |
|----------|-----------------|
| **Fundamentals** | Core knowledge, complexity analysis, optimal approach |
| **Coding** | Code quality, abstractions, clean syntax, self-debugging |
| **Communication** | Clarifying questions, explaining approach, receiving feedback |

**Pass**: total >= 8/12 with no individual category at 1.

After each round you get detailed feedback: what went well, what to improve, comparison to the optimal solution, and study recommendations.

## Study Guide

As you complete rounds, a `STUDY_GUIDE.md` accumulates in each series folder. It teaches concepts you struggled with — grouped by topic, not by round — with code examples and explanations. If multiple rounds surface related concepts, they're consolidated under one heading. It's a personalized quick-reference sheet that grows as you practice.

## Project Structure

```
.claude/commands/
  interview.md              # The skill that drives everything
series/                     # Generated locally, gitignored
  <series_slug>/
    CLAUDE.md               # Series-specific conventions for Claude
    series.template.json    # Series metadata + clean starting state
    series.json             # Your personal progress
    STUDY_GUIDE.md          # Cumulative study reference
    INDEX.md                # Round listing and overview
    01_topic/
      round.md              # Problem, solutions, hints, rubric
      attempt_1.swift       # Your work (extension varies by language)
```

All series content is generated locally and gitignored — each user generates their own series via `/interview generate`.
