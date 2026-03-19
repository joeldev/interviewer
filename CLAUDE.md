# Interview Prep System

Multi-series mock technical interview system. Supports multiple interview series (DSA, iOS, system design, web frontend, etc.) with dynamic generation of new series via `/interview generate`.

## Structure
- `series/` — all interview series (gitignored, generated locally per user)
- `series/<slug>/series.json` — series metadata + progress state
- `series/<slug>/INDEX.md` — series index with difficulty progression
- `series/<slug>/NN_topic/round.md` — individual interview rounds
- `series/<slug>/NN_topic/attempt_N.*` — user attempt files

## Usage
- `/interview` — list all series, show overall dashboard
- `/interview <series> status` — view series dashboard
- `/interview <series> next` — start next suggested round
- `/interview <series> <number>` — start or redo a specific round
- `/interview generate` — create a new interview series interactively

## Conventions
- Interview rounds follow a strict markdown format with embedded code
- State is tracked in series.json per series; do not edit manually
- The skill file at `.claude/commands/interview.md` defines interviewer behavior
- Series support three validation tiers: full (run code), build-only (compile + unit test), review-only (design review)
- All series content is local — each user generates their own series via `/interview generate`
- During generation, the system probes the user's environment for platform-specific toolchain tools (compilers, build systems, project generators, test runners, etc.) and records them in series.json so rounds are set up with the best available scaffolding
- **Round numbering is always by progression order** — the user-facing round number is the 1-based position in `difficulty_order` from series.json, never the folder name prefix (e.g., folder `18_sorting/` may be "Round 7" if it's 7th in difficulty_order)
