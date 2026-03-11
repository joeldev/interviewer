# Interview Prep System

Multi-series mock technical interview system. Supports multiple interview series (DSA, iOS, system design, etc.) with dynamic generation of new series.

## Structure
- `series/` — all interview series
- `series/<slug>/series.json` — series metadata + progress state
- `series/<slug>/INDEX.md` — series index with difficulty progression
- `series/<slug>/NN_topic/round.md` — individual interview rounds
- `series/<slug>/NN_topic/attempt_N.*` — user attempt files
- `python_cheat_sheet.md` — Python syntax reference

## Usage
- `/interview` — list all series, show overall dashboard
- `/interview <series> status` — view series dashboard
- `/interview <series> next` — start next suggested round
- `/interview <series> <number>` — start or redo a specific round
- `/interview generate` — create a new interview series interactively

## Available Series
- `doordash_dsa` — DoorDash-style DSA interviews (18 rounds, Python)

## Conventions
- Interview rounds follow a strict markdown format with embedded code
- State is tracked in series.json per series; do not edit manually
- The skill file at `.claude/commands/interview.md` defines interviewer behavior
- Series support three validation tiers: full (run code), build-only (compile + unit test), review-only (design review)
