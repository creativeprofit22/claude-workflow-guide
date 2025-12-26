# Claude Checkpoint Workflow Guide

A systematic pipeline for feature development with Claude Code. Build, validate, refactor — with context clears between phases to prevent bloat.

📦 **Get the commands:** [github.com/creativeprofit22/claude-checkpoint-workflow-guide](https://github.com/creativeprofit22/claude-checkpoint-workflow-guide)

---

## Pipeline Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    PER-FEATURE CYCLE                        │
│                                                             │
│  BUILD          → continuation prompt → clear context       │
│       ↓                                                     │
│  VALIDATE       → continuation prompt → clear context       │
│       ↓                                                     │
│  REFACTOR-HUNT  → continuation prompt → clear context       │
│       ↓                                                     │
│  REFACTORING    → continuation prompt → clear context       │
│       ↓                                                     │
│  NEXT FEATURE   (repeat cycle)                              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

Each phase ends with a context clear. Fresh context = better focus.

---

## Commands

| Command | Purpose |
|---------|---------|
| `/build-checkpoint` | Implement feature → hand off to validation |
| `/validate-checkpoint` | Tests, API, UI, wiring, bottlenecks, bugs → fix loop |
| `/refactor-hunt-checkpoint` | Find refactoring opportunities → produce report |
| `/refactor-checkpoint` | Execute refactors → hand off to next feature |
| `/checkpoint` | Universal hand-off (works in any phase) |
| `/handoff` | Git commit, push, generate HANDOFF.md |

---

## Installation

Copy commands to your Claude Code config:

```bash
# Create commands directory if it doesn't exist
mkdir -p ~/.claude/commands

# Copy commands
cp commands/*.md ~/.claude/commands/
```

---

## Usage

### 1. Setup CLAUDE.md

Ensure your project has a CLAUDE.md with features and scope:

```markdown
## Current Focus
Section: Authentication
Files: src/auth/login.ts, src/auth/session.ts

## Next Steps
1. Add login form validation
2. Add session persistence
3. Add logout functionality
```

### 2. Run the Cycle

```bash
# Implement feature 1
/build-checkpoint
# → Builds the feature
# → Outputs continuation prompt
# → Clear context, paste prompt

# Validate the implementation
/validate-checkpoint
# → Runs tests, checks API, verifies UI
# → Traces wiring, looks for bottlenecks/bugs
# → Fixes issues found
# → Outputs continuation prompt
# → Clear context, paste prompt

# Hunt for refactors
/refactor-hunt-checkpoint
# → Analyzes code quality
# → Produces refactor report (or skips if clean)
# → Outputs continuation prompt
# → Clear context, paste prompt

# Execute refactors
/refactor-checkpoint
# → Applies refactors from report
# → Outputs continuation prompt for next feature
# → Clear context, paste prompt

# Repeat for feature 2, 3, etc.
```

---

## Validation Checks

`/validate-checkpoint` performs comprehensive validation:

| Check | What It Does |
|-------|--------------|
| Tests | Run test suite (npm test, pytest, etc.) |
| API | Verify endpoints work (curl/fetch) |
| UI | Check components render correctly |
| Wiring | Trace data flow (UI → Logic → API) |
| Bottlenecks | Look for performance issues |
| Bugs | Find logic errors, edge cases, race conditions |

Issues are fixed in-place. Loop until all checks pass.

---

## Refactor Categories

`/refactor-hunt-checkpoint` looks for:

| Priority | What to Find |
|----------|--------------|
| **High** | DRY violations, high complexity, tech debt |
| **Medium** | Code clarity, maintainability, inconsistent patterns |
| **Low** | Style consistency, minor improvements |

Each refactor includes:
- File:line location
- Issue description
- Suggested fix
- Effort estimate (S/M/L)

---

## Reports

The pipeline generates reports in your project:

```
[project]/reports/
├── validation-[feature].md   # Validation results
└── refactors-[feature].md    # Refactoring opportunities
```

---

## State Tracking

Pipeline state is tracked in CLAUDE.md:

```markdown
## Pipeline State
Phase: build | validate | refactor-hunt | refactoring
Feature: [current feature]
Files: [scoped files]
Reports:
  - validation: reports/validation-[feature].md
  - refactors: reports/refactors-[feature].md
```

---

## Quick Reference

| Action | Command |
|--------|---------|
| Start building | `/build-checkpoint` |
| Validate implementation | `/validate-checkpoint` |
| Find refactors | `/refactor-hunt-checkpoint` |
| Execute refactors | `/refactor-checkpoint` |
| Generic save point | `/checkpoint` |
| Git + docs | `/handoff` |

---

## Design Principles

1. **Context management** — Clear between phases to prevent bloat
2. **Scope discipline** — Only work on listed files, no exploration
3. **Fix in place** — Validation fixes issues, doesn't just report
4. **Progressive refinement** — Build → Validate → Refactor → Next
5. **Short prompts** — Continuation prompts under 15 lines

---

## Credits

Massive props to Willem for the original approach. This guide fuses that with systematic validation and refactoring workflows.

---

<div align="center">

**Happy Building!**

*Clear context → Paste prompt → Execute → Checkpoint → Repeat*

</div>
